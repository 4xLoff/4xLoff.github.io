---
layout:   post
title:    "Expresiones Regulares Cheet-Sheet"
subtitle: "Chuleta basica de Regex"
category: Blog
tags:      Cheet-Sheet Curiosidades REGEX  
---                

***
![list](/assets/img/regex/regext.png){:.lead width="800" height="100" loading="lazy"}

***
<!--more-->

1. this ordered seed list will be replaced by the toc
{:toc}

***

## Coincidencias Basicas:

| Caracter.       |Descripcion.                                                   |
|-----------------|:--------------------------------------------------------------|
|\d               |- Digitos (0-9).                                               |
|\D               |- No digitos (0-9).                                            |
|\w               |- Caracter de palabra (a-z, A-Z, 0-9, _).                      |
|\W               |- No caracter de palabra.                                      |
|\s               |- Espacio en blanco (espacio, tab, nueva linea).               |
|\S               |- No espacio en blanco (espacio, tab, nueva linea).            |
|.                |- Cualquier caracter excepto nueva linea (codicioso - greedy). |
|\                |- Cancela caracteres especiales.                               |


***

## Limites:

| Caracter.       |Descripcion.                                                   |
|-----------------|:--------------------------------------------------------------|
|\b               |- Limite de Palabra.                                           |
|\B               |- No es un Limite de Palabra.                                  |
|^                |- Inicio de una cadena de texto.                               |
|$                |- Final de una cadena de texto.                                |


***

## Cuantificadores:

| Caracter.       |Descripcion.                                                   |
|-----------------|:--------------------------------------------------------------|
|*                | - 0 o más (codicioso - greedy).                               |
|+                | - 1 o más (codicioso - greedy).                               |
|?                | - 0 or 1 (perezoso - lazy).                                   |
|{3}              | - Numero exacto.                                              |
|{n,}             | - Numero n+                                                   |
|{3,4}            | - Rango de números (Minimo, Maximo).                          |


***

## Conjuntos de Caracteres:

| Caracter.       |Descripcion.                                                   |
|-----------------|:--------------------------------------------------------------|
|( )              | - Grupos.                                                     |
|[]               | - Encuentra caracteres en corchetes.                          |
|[^ ]             | - Encuentra caracteres que no están dentro de corchetes.      |
| pleka  \|       | - Condicional O.                                              |

## Remplazar:

| Caracter.       |Descripcion.                                                   |
|-----------------|:--------------------------------------------------------------|
|$0               | - Obtiene toda la expresión.                                  |
|$1               | - Obtiene el contenido del grupo 1.                           |
|\1               | - Referencias.                                                |


```shell
🎉 Felicitaciones ya sabes lo basico de expresiones regulares! 🎉
```
{:.centered}

***