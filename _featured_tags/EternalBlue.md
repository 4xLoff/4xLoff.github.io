---
layout: tag-list
type: tag
title: EternalBlue
slug: EternalBlue
category: HTB
sidebar: false
description: >
    Este exploit aprovecha la vulnerabilidad crítica (MS-17-010) presente en sistemas Microsoft. 
    
    El exploit realiza un operación llamada "buffer overflow memove", el cual debido a un error matemático en donde un registro DWORD es sustraido en un registro WORD, el kernel realiza un desbordamiento el cual sobrescribe un buffer del protocolo SMBv1.

    Solución
    Deshabilitar el protocolo SMBv1, ó.

    Se deben actualizar inmediatamente los sistemas afectados a través de la utilidad Windows Update o instalar manualmente la actualización que soluciona esta vulnerabilidad.
---