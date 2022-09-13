---
layout:   post
title:    "Crear un blog en GitHub con el tema Hydejack. "
subtitle: "Intalacion y personalizacíon"
category:  Blog
tags:      Github Curiosidades Portafolio
---
![list](/assets/img/ferris/post1-01.png){:.lead width="800" height="100" loading="lazy"}

***

En la actualidad con el auge de la tecnología es impórtate crear contenido y una de esas maneras

es crear por ejemplo una página web o un blog para dar a conocer tus ideas o portafolio etc.

Usaremos un tema super profesional de [Hydejack] a mi parecer es uno de los mejores que encontre

en la pagina de [jekyll] super personalizable.

Suerte!

[Hydejack]: https://hydejack.com
[jekyll]: http://jekyllthemes.org/

***
<!--more-->

1. this ordered seed list will be replaced by the toc
{:toc}

***

## Instalar git

Primero debemos ir a la página oficial de git y dirigirnos a [Downloads] en este caso como vamos a 

instalar en windows 10, nos descargamos el de 64bit , si quieren hacer el proceso en linux git ya 

viene instalado por defecto, asi que nos saltaremos la instalcíon de git.

[Downloads]: https://git-scm.com/download/win

![list](/assets/img/ferris/post1-02.png){:.lead width="800" height="100" loading="lazy"}

Descargamos el git.
{:.figcaption}

***

![list](/assets/img/ferris/post1-03.png){:.lead width="800" height="100" loading="lazy"}

Descargamos según el sistema operativo que vamnos a usar.
{:.figcaption}

![list](/assets/img/ferris/post1-08.png){:.lead width="800" height="100" loading="lazy"}

Instalamos el ejecutable escogemos lo siguiente.
{:.figcaption}

![list](/assets/img/ferris/post1-08.png){:.lead width="800" height="100" loading="lazy"}

Eleguimos el primero lo que significa que solo se habra en la terminal de git o si quieren que 

tambien se ejecute los comandos git en la teminal de windows eligen la otra.

![list](/assets/img/ferris/post1-09.png){:.lead width="800" height="100" loading="lazy"}

Eleguimos si queremos la apiencia del terminal sea  windows o linux en mi caso yo elegire linux 

usted elija que mas le guste.

Aparecera un acceso directo en el escritorio o simplemente con clik dercho del raton y parecera

`git bash here` darle click y se habre el terminal git segun donde estes situado de la siguiente 

forma.

![list](/assets/img/ferris/post1-04.png){:.lead width="800" height="100" loading="lazy"}

La terminal "git bash" se abrira donde este situado.
{:.note}

{:.note title="Attention"}
Cabe recalcar que la terminal "git bash" es una terminal linux por lo que seria importante tener 

conocimientos minimos sobre dicho sistema para poder moverse con fluides a la hora de ejecutar 

comando y moverse entre carpetas.


***

## Instalar Ruby

Para que vamos instalar Ruby se preguntan, esto se debe que para usar el tema de forma local y 

editar el tema en si o para hacer nuestras publicaciones debemos usar el comando 

`blunde install` para arrancar las dependecias y el comando `bundle exec jekyll serve` para 

arrancar el thema Hydejack en [http://localhost:4000].

 [http://localhost:4000]: http://localhost:4000

Primero nos vamos a la pagina oficial de [Ruby para windows], nos decargamos el `Devkit de 64bit`, 

em cambio si queremos en linux nos descargamos de [Rubygem.org].

[Ruby para windows]: https://rubyinstaller.org/downloads/
[Rubygem.org]: https://rubygems.org/pages/download

![list](/assets/img/ferris/post1-05.png){:.centered}{:.lead width="800" height="100" loading="lazy"}

Descargamos de la pagina oficial.
{:.figcaption}

![list](/assets/img/ferris/post1-06.png){:.centered}{:.lead width="800" height="100" loading="lazy"}

Escogemos el de 64bits.
{:.figcaption}

***

Una ves decargado lo instalamos; en caso de windows  por defecto de guarda en la ruta `C:\`.


![list](/assets/img/ferris/post1-11.png){:.lead width="800" height="100" loading="lazy"}


Copiamos la ruta del archivo `C:\Ruby31-x64` en mi caso y lo vamos a pergar en el `path` por 

cosiguiete nos vamos a propiedades de equipo y buscamos las configuraciones avanzadas el 

sistema como se muestra en la siguente imagen,luego opciones "variables de entorno" , bucamos 

variables de usuario su "usuario" y en el `path` pegamos la ruta antes copiada.


![list](/assets/img/ferris/post1-12.png){:.lead width="800" height="100" loading="lazy"}

Copiamos la ruta.
{:.figcaption}


![list](/assets/img/ferris/post1-13.png){:.lead width="800" height="100" loading="lazy"}

Integramos la ruta a la varieble de entorno del usuario.
{:.figcaption}

***

Una ves finalizado la instalación aparecera la "CMD" en donde nos perdira que ingresemos un 

numero del uno al tres para instalar el kit y tambien `Enter` por defecto hasta que se 

termine el proceso para vereficar si todo esta corecto escribimos.


![list](/assets/img/ferris/post1-14.png){:.lead width="800" height="100" loading="lazy"}


Con el comando:


```default
ruby -v
```

![list](/assets/img/ferris/post1-15.png){:.lead width="800" height="100" loading="lazy"}
El ouput de comando.
{:.figcaption}


***

## Instalacion y del tema hydejack

Una ves completamos los anteriores pasos procedemos a la instalación de [Hydejack]

Para instalarlo hay multiplies maneras como lo son:

***

### Clonar el repositorio

Para clonara el [repositorio] oficial de hydejack de Github simplemete copiamos la url o en el 

recuadro verde que dice Code y en el terminal pegamos el comando 

`git clone https://github.com/hydecorp/hydejack` y esperamos la decarga.

[repositorio]: https://github.com/hydecorp/hydejack


![list](/assets/img/ferris/post1-16.png){:.lead width="800" height="100" loading="lazy"}


Una ves podremos montar la pagina en github desde la "branch master" o "rama master" 

sino que se hace desde la v9 según la [documentación].

[documentación]: https://hydejack.com/docs/

En mi caso la v9 no me funcíono yo pude desplegar la v8. 
{:.note}


***

{:.note title="Attention"}
Para movernos de rama en git debmos usar el comando `git branch <rama>` pero para eviarnos 

el andar movienedonos de rama y aparte para evitar descargarnos el repositorio entero con 

todas las versiones lo que podemos hacer es decargarnos solo la rama que nos funciona en mi 

caso la v8 y lo hacemos de la siguiente manera en "git bash" con el comando 

`https://github.com/hydecorp/hydejack -d v8` la babndera `-d` es para escoger la rama.


Ahora bien esto para la parte del local es decir tu ordenador para subirlo a Github debemos 

crear un nuevo repositorio conel nombre de tu Owner es decir Owner.github.io , si ponemos una 

descripcion rapida y creamos el repositorio ahi mismo nos explica los siguientes pasos para que 

se suba el repositorio que tenemos enn local al remoto y se sincronicen.

```bash
//or create a new repository on the command line

echo "el nombre de tu repositorio" >> README.md
git init                                                             //para inicializar la coneccion del repo \local y el remoto
git add .                                                            //para comtemplar todos los archivos para subir al repositorio
git commit -m "first commit"                                         //para registrar el primer cambio
git branch -M master                                                 //master o main el nombre que usted quiera
git remote add origin https:\//el nombre de tu repositorio.git
git push -u origin master
```
***

```bash
//or push an existing repository from the command line

git remote add origin https:\//el nombre de tu repositorio.git
git branch -M main
git push -u origin main
```
***

### Fork de un repositorio.

Este caso es mas simple en la parte superio derecha hay un boton que dice "fork" le damos ahi 

ponesmos el nombre del repositorio como ya vimos anteriormente y segurmos los mismos pasos.

Cuando hacemos un "fork" se copia todo el repositorio por lo que tendremos que movernos a la rama v8.
{:.note}

***

### Descargar del repositorio.

La descarga de hacer desde el [repositorio] de Hydejack y dirigirno a la parte de dowloads.

No recomiendo esta opción porque aparte que debemos escoger la branch que sirva en mi caso la 

V8 el tema no se despliega el slider y no esta editado es una version mas pura que las anteriores.
{:.note}


***

## Edicion del tema y sincronicaion del local con el remoto.

{:.note}
Apartir de este punto ya es cuestion de editar lo que vermos mas adelantea su gusto; en el 

siguiente [link] hay una personalizacion super bonita y profecional y superfacil de enteder.

[link]: https://lazyren.github.io/devlog/how-i-customized-hydejack-theme.html

Si no quieres hacer por tu cuenta toda la personaliozacion porque te parece tedioso o no se te da 

lo que puedes hacer es clonar directamente su repositorio o hacerle un "fork" hacer lo mismo que 

hicimos antes.


Para personalizar el tema o bien redactar un blog y verlo en local hasta que decidamos subirlo al 

remoto para que se publique debemos hacer los siguentes comandos:

```bash
bundle install
```
inicia las dependecias,debe asegurarse estar en el directorio donde _config.yml se encuentra. 

Antes de ejecutar por primera vez.
{:.note}

***

Ahora puede ejecutar Jekyll en su máquina local:


```bash
bundle exec jekyll serve
```
Inicia el el [http://localhost:4000] la previsualizacion para que obsserver los cambios en tiempo real.
{:.note}

***

Para [escribir] una publicacion revisamos la documentacion de la pagina oficial.

[escribir]: https://hydejack.com/docs/writing/

***

```shell
🎉 Felicitaciones ya tienes tu propio blog! 🎉
```
{:.centered}

***
