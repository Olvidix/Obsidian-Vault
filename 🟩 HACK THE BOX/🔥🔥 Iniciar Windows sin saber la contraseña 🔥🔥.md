1. Iniciamos Windows normalmente y le damos a reiniciar el PC (Manteniendo el Shift Pulsado)

2. Se nos abrirá una pantalla azul con opciones, hacemos click en "solucionar problemas > opciones avanzadas > símbolo del sistema" y escribimos `regedit`

3. Se nos abrirá el editor de registros , desde ahí nos vamos a "Equipo > HKEY_LOCAL_MACHINE" una vez esto seleccionado le damos en la vista de arriba de la ventana en "Archivo > Cargar subárbol" 

4. Se nos abrirá una carpeta de system32 pero no queremos ese exactamente necesitamos llegar a otro para ello seleccionaremos a la izquierda "Equipo" y nos dirigimos a "[NUESTRO DISCO DURO] (Por ejemplo C:) > Windows > System32 > config > ==SYSTEM" HAY QUE HACER DOBLE CLICK EN SYSTEM==

5. Y escribimos lo que queramos por ejemplo "Test" y le damos a aceptar

6. Ahora nos dirigimos a la carpeta que acabamos de crear "Test > Setup > ==C[]()mdLine" DOBLE CLICK EN CmdLine==

7. Y escribimos en "Información del valor" ponemos "cmd.exe"

8. Después en el mismo sitio donde CmdLine hacemos doble click en SetupType

9. Y escribimos un 2

10. Después seleccionamos la carpeta que hemos creado antes (Test) y le damos a arriba en las pestañas a "Archivo > Descargar subárbol"

11. Cerramos el CMD que teníamos usando y le damos a continuar

12. Ahora al iniciar la máquina tendremos un CMD y si hacemos un `whoami` podremos ver que tenemos el usuario administrador "System32"

13. Ahora usamos "net user" para ver todos los usuarios de la máquina

14.  Y usamos el siguiente comando `net user [USUARIO] [NUEVA_CONTRASEÑA]`

15. Y listo, ahora podremos cerrar esa CMD y entrar con ese usuario con la contraseña nueva