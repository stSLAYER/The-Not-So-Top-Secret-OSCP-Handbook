# Buffer Overflow in 7 steps

This is a quick 7 step guide on how to go about exploiting a buffer overflow vulnerability, this guide will pretty much serve as a checklist to help you stay on the right path for each step and to get through the exloit a lot faster, prior knowledge on how to exploit this vulnerbility is definitely recommended.

<h1> Lab Setup </h1>
<h4> Remote Desktop into the debugging machine: </h4> 

`xfreerdp /u:admin /p:lab /cert:ignore /v:192.168.xx.xxx` </br>

<h4> configuring !mona on imunity debugger:</h4> 

`!mona config -set workingfolder c:\mona\%p` </br>
(note down the path for later) </br>
`!mona bytearray -b "\x00"`

