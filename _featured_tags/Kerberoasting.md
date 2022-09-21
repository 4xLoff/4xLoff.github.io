---
layout: tag-list
type: tag
title: Kerberoasting
slug: Kerberoasting
category: [HTB, THM]
sidebar: false
description: >
    Kerberoasting es un ataque que abusa del protocolo Kerberos para recopilar hash de contraseñas para cuentas de usuario de Active Directory con valores servicePrincipalName (SPN), es decir, cuentas de servicio.
    Un usuario puede solicitar un ticket del servicio de otorgamiento de boletos (TGS) para cualquier SPN, y partes del TGS se pueden cifrar con RC4 utilizando el hash de contraseña de la cuenta de servicio que tiene asignado el SPN solicitado como clave. Por lo tanto, un adversario que pueda robar boletos TGS (ya sea de la memoria o capturándolos rastreando el tráfico de la red) puede extraer el hash de la contraseña de la cuenta de servicio e intentar un ataque de fuerza bruta fuera de línea para obtener la contraseña de texto sin formato.
---