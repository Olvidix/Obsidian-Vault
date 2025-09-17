### Primero puerto -D y luego puerto -L

La -D
es el por el que se envian los comandos de la kali por proxychains a las otrsa pasando por el runel

La -L
Es el tunel en si


SSH ubuntu ubuntu

ya estariamos en la VM1 
Esta tiene unas credenciales en el puerto 80 web:
La pass de la VM2 es: Pivoting2341 

``` NO_SESIESTABIEN
Una vez con esto hacemos el primer tunel:
ssh ubuntu@IP -p X -D 9050

```
despues el otro tunel
ssh ubuntu@IP -L Pueokali(ejemplo:2222):ipkali:22

tenemos la nueva ip
Una ves dentro nmap con proxychains (En las maquinas solo tienen 22,80)

nmap -sT -Pn -n -p 22 IP_PIVOTING2
sudo proxychains nmap -sT -Pn -n -p 22,80 IP_PIVOTING2


------------------------------------------------------------------

# Túnel a la maquina 2

ssh ubuntu@localhost -p 2222 -D 9050 
La contrasseña Pivoting2341



![[Pasted image 20250804190345.png]]
![[Pasted image 20250804190412.png]]

--------------------------------------------------------------------------
# Túnel a la 3

   ssh ubuntu(de_vm2)@IP_Localhost -p 2222  -L localhost(Esto se puede quitar):8081:IP_ipv3:80


ssh ubuntu@ipkali -p 2222 -L:8080:ipv3:80


para la siguiente

ssh ubuntu@localhost -p 2222-L  2223:ipvm4:22

ssh -p 2223 ip




# -R
Es un tunel reverso, por ejemplo si nos montamos un server en python para pasar cosas de kali a VN3

Aqui el comando es al reves obviamente

ssh kali@IP_kali -p -R IP_vm3:PUERTO_vm3:IP_kali:4444


ssh ubuntu3@localhost -p 2223 -R IP_vm3:PUERTO_VM3(Ejemplo:4443):IP_kali:80(Lo elegimos nosotros tambien)