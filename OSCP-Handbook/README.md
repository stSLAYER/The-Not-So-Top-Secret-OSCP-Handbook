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

`!mona findmsp -distance 3000`








 
