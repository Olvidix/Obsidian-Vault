## Arreglar autocompletar
`sudo activate-global-python-argcomplete`

## Arreglar ratón que desaparece:

```
sudo mkdir -p /etc/X11/xorg.conf.d  
sudo nano /etc/X11/xorg.conf.d/20-cursor.conf
```  

Y dentro del archivo, añade lo siguiente:  
  
```
Section "Device"  
Identifier "Device0"  
Driver "modesetting"  
Option "SWCursor" "true"  
EndSection  
```  
💡 Guarda, reinicia… ¡y tu ratón volverá a la vida! 🖱️

## Escritorio no carga correctamente

```
pkill xfdesktop
rm -rf ~/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-desktop.xml
xfdesktop & disown
```

Y para que no vuelva a pasar al iniciar otra vez:
1. Nos dirigimos a opciones de kali y buscamos "Session and Startup"
2. Una vez dentro le damos en "Application Autostart"
3. Le damos a "+Add" y ponemos:
	Name: Desktop
	Description: (LO DEJAMOS EN BLANCO)
	Command: xfdesktop
4. Y le damos a OK y listo

### Fix rápido (Prueba este primero por si las moscas)
Puede que se caiga el escritorio a mitad de la sesión o algo (Sin botones de minimizar y el fondo negro)
```
xfwm4 --replace & disown
xfdesktop & disown
```