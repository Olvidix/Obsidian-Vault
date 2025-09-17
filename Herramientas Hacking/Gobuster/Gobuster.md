## Enumeración normal
gobuster dir -u http://planning.htb/ -t 50 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,sh,txt,xml,log,js,cgi,py 


## Enumeración de subdominios