---
title: ES DCSHADOW: Explicación, explotación, mitigación y detección
date: 2025-09-13
categories: [AD]
tags: [passwords, ad, domain]     # Los tags deben estar siempre en minúsculas.

---

## 1. ¿Que es DCSHADOW?
En este post vamos a profundizar en la técnica DCSHADOW que se enmarca bajo [**Técnica T1207 de Mitre Rogue Domain Controller**](https://attack.mitre.org/techniques/T1207/). La técnica se puede resumir de manera muy sencilla en el meme que su propio creador hizo
![Desktop View](/assets/img/dcshadow/dcshadow.jpg) <br>
Esta técnica se presentó en la Blue Hat de 2018 por Vicente Le Toux y Benjamin Delphy, os dejo la charla <br>
{% include embed/youtube.html id='KILnU4FhQbc' %}
Para cualquier referencia sobre el descubrimiento, y por supuesto para dejar crédito a quienes lo descubrieron os dejo la web [**dcshadow.com**](https://www.dcshadow.com/) donde teneís mucho detalle sobre la técnica y cuando se presentó y el detalle. 

La idea principal de esta técnica de persistencia es utilizar una máquina controlada por el atacante para que funcione como un DC real y forzar así a que el resto de DCs replique sus contenidos.

![Desktop View](/assets/img/dcshadow/1sh.png) <br>


## 2. Profundizando en DCSHADOW
Explicar DCSHADOW aveces es complejo ya que hay que explicar que no es una vulnerabilidad sino un uso de protocolos los cuales están documentados 
- [**MS-ADTS**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/d2435927-0999-4c62-8c6d-13ba31a52e1a?redirectedfrom=MSDN)
- [**MS-DRSR**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47?redirectedfrom=MSDN) 
y para los cuales simplemente debes detener los privilegios adecuados, que demanera habitual son Domain Admin o Enterprise Admin. 

Es importante que tengas en cuenta lo siguiente:
![Desktop View](/assets/img/dcshadow/feature.jpg) <br>

## 3. Explotación
Existen varios requerimientos previos:
1- Registrar el DC en el dominio
Para ello tenemos que Modificar el SPN del ordenador a `GC/$HOSTNAME.$DOMAIN/$DOMAIN` y añadir una entrada como `CN=$HOSTNAME,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=$DOMAIN` con los siguientes atributos:
- `objectClass: server`
- `dNSHostName: $HOSTNAME.$DOMAIN`
- `serverReference: CN=$HOSTNAME,CN=Computers,DC=$DOMAIN`
2- Tienes que tender capacidad de enviar y recibir tráfico RPC de las llamadas:
- DRSBind
- DRSUnbind
- DRSCrackNames
- DRSAddEntry
- DRSReplicaAdd
- DRSReplicaDel
- DRSGetNCCnages

Realmente una vez obtenidos los privilegios como Domain Admin el proceso con [**Mimikatz**](https://github.com/gentilkiwi/mimikatz) es sencillo consiguiendo añadir el usuario al grupo de Domain Admins

```cmd
lsadump::dcshadow /object:usuariocomprometido /attribute:primaryGroupID /value:512
```

Posteriormente tienes que abrir otra terminal y forzar un push en entono del Active Directory para que el dominio actualice el dato.

```cmd
lsadump::dcshadow /push
```
Con esto generar persistencia es relativamente sencillo simulando el comportamiendo de un DC. 


## 4. Mitigaciones
Os detallo algunas de las mitigaciones que puedes realizar:
- Controlar los usuarios con privilegios elevados.
- Políticas de control mediante firewall de conexiones RDP.


## 6. Detección
Algunos controles que podeís tener son:
- Monitorización de los eventos 4928 y 4929 de los entornos de Active Directory, os dejo el detalle de los eventos que proporciona [**Microsoft**](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd941628(v=ws.10))
- Las llamadas a las APIs DrsAddEntry, DrsReplicaAdd y GetNCChanges pueden ser consideradas como sospechosas. Ten en cuenta que las replicaciones suelen tardar 15 minutos agrupando diferentes cambios. 
- Si usas Splunk hay un [**script**](https://gist.github.com/gentilkiwi/dcc132457408cf11ad2061340dcb53c2) para facilitar la detección junto con DCSYNC.

Quizás puedan serte de ayuda de cara a la detección los siguientes recursos:
[**DCShadow.com**](https://www.dcshadow.com/) <br>
[**Hacking Articles**](https://www.hackingarticles.in/domain-persistence-dc-shadow-attack/) <br>
[**Hacker Recipes**](https://www.thehacker.recipes/ad/persistence/dcshadow/) <br>



---
## Apoya el contenido de ciberseguridad en castellano

Si esta publicación te ha sido útil y quieres apoyar mi trabajo para que continúe creando más contenido, aquí te dejo algunas formas de apoyar:

1. **Compartir el contenido**  📲
   Si crees que esta guía puede ser útil para otras personas, compartirla en tus redes sociales es una gran ayuda. 

2. **Donar en Ko-fi**  💖
   Puedes hacer una donación rápida a través de Ko-fi para ayudarme a seguir publicando guías y tutoriales. ¡Cada aportación cuenta y es muy apreciada! 

   <script type='text/javascript' src='https://storage.ko-fi.com/cdn/widget/Widget_2.js'></script><script type='text/javascript'>kofiwidget2.init('Apoya este contenido!', '#455d85', 'A0A41BO608');kofiwidget2.draw();</script> 

---

¡Gracias por tu apoyo! 🙏
![Desktop View](assets/img/banner.png) <br>
