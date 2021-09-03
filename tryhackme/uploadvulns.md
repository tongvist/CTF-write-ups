# TryHackMe Upload Vulnerabilities

## Server info

- IP: 10.10.136.141
- Back-end: Node.js / Express


## Client side filters

- File extension: .jpg or .jpeg
- File size 50 * 8 * 1024
- Magic numbers: First three bytes of jpg-format

By-pass: Intercept /assets/js/upload.js -response and comment out the checks

## Locations

File upload path: http://jewel.uploadvulns.thm/content
Can be enumerated with gobuster using custom wordlist for the room:

<code>gobuster dir -u http://jewel.uploadvulns.thm/content -w files/wordlist.txt -x jpg</code>

## Server side filters

- File type: By-pass by naming payload to shell.jpg. Additionally spoofed magic numbers of the file to 'FF D8 FF DB' (jpg).

## Exploitation

The server renames all files when uploaded, so the uploaded shell needs to be found with gobuster by comparing the names of previous results and finding the new one. File size can also be used to determine which file is the shell: 

<code>gobuster dir -u http://jewel.uploadvulns.thm/content -w files/wordlist.txt -x jpg</code>

Typing something to the input field at /admin and intercepting the request with BurpSuite shows that the key for the input is 'cmd', hinting that the input will perhaps be evaluated as a command on the server. The reminder text in the UI supports this, so the shell can be run with this input. Based on the gobuster results, both /modules and /content are located at the root of the system, so the shell can be run with:

<code>../content/*shellname*.jpg</code>