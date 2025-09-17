```markup
script /dev/null -q -c bash
```
Luego, pulsamos Cntrl + z para salir de la terminal e introducimos el segundo comando:
```markup
stty raw -echo; fg
```
Y debajo de este comando a la derecha, escribimos reset:
```markup
reset 
```
y cuanto nos pregunte que tipo de terminal:
```markup
xterm
```
Y después:
```markup
export SHELL=bash
export TERM=xterm
```

```
stty rows 50 columns 236 
```


# Otras opciones
```
python -c 'import pty; pty.spawn("/bin/sh")' 
```
