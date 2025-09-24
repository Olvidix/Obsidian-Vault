Con una Wordlist de posibles nombres generada por ejemplo con [[Username Anarchy]] podremos usarlo:

```
./kerbrute userenum --dc [IP_DC] --domain [DOMINIO.MAQUINA] names.txt
```

Con esto nos dará usuarios validos para ese entorno de AD

Con esto y la contraseña que podremos sacar con [[Netexec]] podremos conectarnos mediante [[WinRM (5985,5986)]]