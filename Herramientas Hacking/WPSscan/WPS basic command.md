
wpscan --url <URL_del_sitio_web>
	(Así Dumpea todo lo posible pero es lento)


wpscan --url <URL_del_sitio_web> --enumerate [**]
	[u] --> usuarios
	[p] --> plugins --> [vp] --> plugins vulnerables
	[t] --> temas--> [vt] -->temas vulnerables
	[m] --> archivos multimedia
	[cb] --> copias de seguridad de la configuracion


wpscan --url <URL_del_sitio_web> --usernames [user1,user2] --passwords /usr/share/wordlist/rockyou.txt

