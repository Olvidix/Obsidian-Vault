Una herramienta de reconocimiento basada en Python que ofrece diversos módulos para diferentes tareas, como la verificación de certificados SSL, la recopilación de información Whois, el análisis de encabezados y el rastreo. Su estructura modular permite una fácil personalización según las necesidades específicas.


Instalación:
```
git clone https://github.com/thewhiteh4t/FinalRecon.git

cd FinalRecon

pip3 install -r requirements.txt

chmod +x ./finalrecon.py
```


Uso:
```
finalrecon.py --url [TARGET]
```


|Option|Argument|Description|
|---|---|---|
|`-h`, `--help`||Show the help message and exit.|
|`--url`|URL|Specify the target URL.|
|`--headers`||Retrieve header information for the target URL.|
|`--sslinfo`||Get SSL certificate information for the target URL.|
|`--whois`||Perform a Whois lookup for the target domain.|
|`--crawl`||Crawl the target website.|
|`--dns`||Perform DNS enumeration on the target domain.|
|`--sub`||Enumerate subdomains for the target domain.|
|`--dir`||Search for directories on the target website.|
|`--wayback`||Retrieve Wayback URLs for the target.|
|`--ps`||Perform a fast port scan on the target.|
|`--full`||Perform a full reconnaissance scan on the target.|