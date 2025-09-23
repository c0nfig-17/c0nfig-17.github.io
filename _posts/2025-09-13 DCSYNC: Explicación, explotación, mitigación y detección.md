---
title: DCSYNC Explicación, explotación, mitigación y detección
date: 2025-09-13
categories: [AD]
tags: [passwords, ad, domain]     # Los tags deben estar siempre en minúsculas.

---

## 1. ¿Que es DCSYNC?
En este post vamos a profundizar en el uso de DCSYNC como técnica que usan los atacantes en entornos de Active Directory. Lo primero que tenemos que hacer es tratar de entender como funcionan los elementos más básicos de un entorno de Active Directory. Sabemos que un Active Directory en su forma más básica tiene un "dominio" el cual en nuestro caso será "galletas.corp" esta infrastructura se despliega bajo los llamados Domain Controllers (DCs) los cuales son servidores de windows. Estos servidores siempre cumplen algunas características básicas, como que tienen que ser varios para que podamos seguir operando con cualquier problema o que suelen estar en la misma versión del sistema para evitar problemas de compatibilidad.

![Desktop View](/assets/img/dcsync/1.png) <br>

Posteriormente tenemos que entender que en este entorno que hay en nuestra compañía existen muchos y diferentes equipos, así como usuarios y otros elementos, los cuales "pertenecen" a nuestro dominio o AD. Es decir, nuestro ordenador de la empresa es de la empresa y por eso está en su dominio y aplica sus políticas, pasa lo mismo con nuestro usuario de la empresa. En ocasiones puede ser complicado de entender por algunos usuarios pero "tu usuario" realmente no es tuyo sino que te lo cede la empresa u organización en la que trabajas para que operes de la misma manera que se hace con un ordenador. 

![Desktop View](/assets/img/dcsync/2.png) <br>

Una vez entendemos esto entendemos que existen los DCs cumplen una función fundamental ya que el entorno de Active Directory tiende a ser crítico con facilidad en cualquier empresa. Si a esto le sumamos que tener este entorno tiene un coste en mantenimiento elevado y que necesitemos que esté al dia siempre es fácil que nos demos cuenta de que cualquier problema de cofiguración puede tener un alto impacto.

Todo esto está muy bien pero ¿que es DCSYNC?. Vamos al grano. Como comprenderás para que la infrastructura de Active Directory funcione correctamente los DCs deben almacenar la misma información en tiempo real. Es decir el usuario "Trabajador01" no puede existir en DC01 pero en DC02 no. Tampoco puede pasarnos que la contraseña de "Trabajador01" sea diferente la que se almacena en DC02 que en DC04 ni nos puede pasar que el ordenador "Workstation01" exista en DC03 pero no en DC01. Es por ello que la información entre todos los DCs debe estár sincronizada y ser exacta en todo momento ya que sino podría dar lugar a incongruencias graves en los sistemas que están bajo tu dominio.

![Desktop View](/assets/img/dcsync/3.png) <br>

La manera en la que se resuelve esto por Microsoft es mediante el protocolo [**MS-DRSR Directort Repliation Service**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47) con el cual sincroniza los datos y los replica en el entorno entre los diferetes DCs.
DCSYNC es una técnica que abusa de como se configura esta replicación abusando de este comportamiento legitimo diseñado por microsoft para replicar la información de un DC a tu máquina. 

![Desktop View](/assets/img/dcsync/4.png) <br>

Tened en cuenta que podeís encontrar información sobre esta técnica bajo la [**Técnica T1003.006 de Mitre**](https://attack.mitre.org/techniques/T1003/006/) que pertenece a la [**Técnica T1003 de Mitre**](https://attack.mitre.org/techniques/T1003) .


## 2. Profundizando en DCSYNC
En muchas ocasiones se explica la posibilidad de usar DCSYNC con un resumen muy mecanicista y simple "si tienes un Domain Admin haces DCSYNC". Esta visión no solo no profundiza en el ataque sino te coarta de utilizar maneras creativas para abusar de DCSYNC teniendo obligatoriamente que realizar una escalada previa o generando un ruido que quizás es innecesario. 
En este caso teneís que entender cuales son los permisos exactos que posibilitan recplicar un DC en un entorno de Active Directory:

- The “DS-Replication-Get-Changes” 
CN: DS-Replication-Get-Changes
GUID: 1131f6aa-9c07-11d1-f79f-00c04fc2dcd2

- The “Replicating Directory Changes All” 
CN: DS-Replication-Get-Changes-All
GUID: 1131f6ad-9c07-11d1-f79f-00c04fc2dcd2

- The “Replicating Directory Changes In Filtered Set” 
CN: DS-Replication-Get-Changes-In-Filtered-Set
GUID: 89e95b76-444d-4c62-991a-0facbeda640c

Después de esto podemos entrar en la conversación de que por defecto los "Domain Admin" al ser usuarios que administran el dominio los tienen por defecto. Pero debemos de conocer desde donde se origina el DCSYNC.

![Desktop View](/assets/img/dcsync/domainadmin.jpg) <br>

Creo que es relevante en este punto recomendaros [**AADInternals.com**](https://aadinternals.com/aadkillchain/) de Dr Nestori Syynima @DrAzureAD que trabaja como Security Researcher de Microsoft y en este recurso puede proporcionaros muchisima información y contexto con el que profundizar. 


## 3. Descubrimiento y targets
Una vez comprendemos "que" hace que podamos abusar de DCSYNC podemos comenzar a profundizar y ampliar bien nuestros objetivos. En primer lugar es evidente, los usuarios Domain Admin serán en la mayoría de ocasiones buenas vías de entrada pero si estás en una infrastructura madura los Domain Admin tendrán algunos permisos o usos segmentados o incluso está monitorizada la creación de algunos de ellos. Por eso podemos usar algunas herramientas para localizar algo de información en profundidad.

Posiblemente en este punto sabrás que usar [**Bloodhound**](https://bloodhound.specterops.io/get-started/introduction) puede ser un buen aliado para entender los problemas de configuración que pueden existir para escalar.

![Desktop View](/assets/img/dcsync/bloodhound.png){: width="500" height="305" }

En mi caso prefiro usa ultimamente [**AD Explorer**](https://learn.microsoft.com/en-us/sysinternals/downloads/adexplorer) de [**Sysinternals**](https://learn.microsoft.com/en-us/sysinternals/). Fue un truco en el que no había profundizado que otro pentester me comentó y la verdad que es una herramienta firmada por Microsoft muy útil en estos casos ya que el nivel de ruido generado es mucho inferior y se comporta de una manera dificil de discernir si es legitimo o no. Una vía para enumerar puede ser el uso de [**PowerView**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) o [**SharpView**](https://github.com/tevora-threat/SharpView)

En este punto el objetivo general es buscar una vía de acceso al DCSYNC. En mi opinión existen varias vías:
1- Las basadas en Manipulaciones de permisos: Podemos encontrarnos un usuario el cual tenga el permiso "Generic Write" que nos permita escalar a los permisos que necesitamos. Será probablemente más sencillo encontrar manipulaciones a los Domain Admins en máquinas genéricas, pero es menos evidente cuando consigues un usuario así si no eres metódico.
2- Las basadas en problemas en la configuración de permisos o gestión de usuarios: Puedes dar con usuarios que estén en grupos privilegiados o sean Domain Admin debido a una mala gestión de los usuarios y de las máquians a las que acceden. Suele ser la vía común que se trata en el abuso de DCSYNC.
3- Las basadas en Manipulaciones de Tickets

## 4. Explotación
Como con todo una vez hemos realizado una escalada podemos usar multitud de herramientas para explotar una vulnerabilidad concreta. De manera habitual existen dos herramientas que se usan [**Impacket**](https://github.com/fortra/impacket) y mimikatz [**Mimikatz**](https://github.com/gentilkiwi/mimikatz). También puedes usar [**DSInternals**](https://github.com/MichaelGrafnetter/DSInternals/tree/master) para realizar los pasos similares como se explica en su [**blog**](https://www.dsinternals.com/en/retrieving-active-directory-passwords-remotely/)

En resumen existen multitud de software para realizar esa extracción como tal esto no es un problema ya que no necesitas estar en una máquina en dominio sino tener alcance al DC contra el que vayas para así poder realizarlo

```text
# Contraseña en texto plano
secretsdump -outputfile 'dcsync' -dc-ip "$DC_IP" "$DOMAIN"/"$USER":"$PASSWORD"@"$DC_HOST"

# Pass-the-Hash (PTH)
secretsdump -outputfile 'dcsync' -hashes :"$NT_HASH" -dc-ip "$DC_IP" "$DOMAIN"/"$USER"@"$DC_HOST"

# Pass-the-Ticket (PTT)
KRB5CCNAME=ticket.ccache secretsdump -k -no-pass -outputfile 'dcsync' -dc-ip "$DC_IP" @"$DC_HOST"
```

Una vez lo ejecutemos obtendremos una salida de los hashes de todo el dominio replicados por lo que movernos será fácil y un verdadero problema para el sitio que se haya comprometido.

![Desktop View](/assets/img/dcsync/mimikatz.png) <br>

> He estado revisando en varios sitios y se menciona que por alguna razón no documentada, secretsdump de Impacket se basa en SMB antes de realizar un DCSync (por lo que requiere un SPN CIFS/DCx cuando se utilizan tickets Kerberos). mientras que Mimikatz se basa en LDAP antes de realizar el DCSync (por lo que requiere un SPN LDAP/DCx cuando se utilizan tickets Kerberos). Esto puede generarte algunso quebraderos de cabeza en si no sabes que herramienta usar correctamente. 
{: .prompt-warning }

## 5. Mitigaciones
Os detallo algunas mitigaciones de DCSYNC que podemos aplicar en nuestros entornos:
- Principalmente tienes que hacer una buena gestión de los Domain Admin y usuarios con privilegios excesivos o en máquinas Críticas.
- Revisar grupos privilegiados y usuarios que no deberían estar ahí.
- Realizar una revisión del principio de minimo privilegio... Hay muchas veces en que "para que funcione es necesario un Domain Admin" porque o no queremos/podemos profundizar en algo concreto o que el proveedor no es nada concreto en los permisos que utiliza y necesita. Esto acaba siendo un verdadero quebradero de cabeza cuando pasan 10 años y nadie se atreve a tocar un usuario. 
- Me consta que existen herramientas que se implementan en tu AD y ayudan a automatizar una revisión crítica de todos los permisos.

## 6. Detección
Os dejo una lista de algunos controles que podemos implementar para controlar la ejecución de técnicas de DCSYNC en nuestra infra:
- Tienes que tener en cuenta que la detección no tiene que nacer de donde se origina la petición sino del tráfico o de donde se realiza. Es decir, detectar un mimikatz en una máquina no resuelve este problema. Si ese era el punto de partida de tu detección tienes que volver a analizarlo todo. 
- Podemos utilizar un IDS para detectar si hay solicitudes DsGetNCChange https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/b63730ac-614c-431c-9501-28d6aca91894 cuyo origen no es una lista de replicaciones de DC permitidas (Que serían nuestros DCs). Esto debería ayudarnos a detectar el momento exacto y origen del intento de ejecución de un DCSYNC.
- Monitorizar eventos 4662 de windows https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4662 con el GUID 1131f6aa-9c07-11d1-f79f-00c04fc2dcd2 los cuales pertenecen a DS-Replication-Get-Changes y DS-Replication-Get-Changes-All y el GUID 89e95b76-444d-4c62-991a-0facbeda640c para DS-Replication-Get-Changes-In-Filtered-Set. Este evento es principal para la replicación mediante DCSYNC y tendrá sentido siempre que sea desde DCs y máquinas autorizadas.
- También puedes monitorizar los eventos 4670 que se refieren a DS-Replication-Get-Changes-All https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4670
- Me consta que existen herramientas en el mercado que te ayudan a monitorizar algunos aspectos de tu AD así como a establecer honeypots que permiten detectar cuando esto se produce.
- Monitorizar usuarios de grupos privilegiados.
- Monitorizar usuarios privilegiados. 
- Monitorizar Privilegios críticos seteados en usuarios. 

![Desktop View](/assets/img/dcsync/budy.png) <br>



Quizás puedan serte de ayuda de cara a la detección los siguientes recursos: <br>
[**Kibana DCSYNC**](https://www.elastic.co/docs/reference/security/prebuilt-rules/rules/windows/credential_access_dcsync_replication_rights) <br>
[**Threath Hunter Playbook**](https://threathunterplaybook.com/library/windows/active_directory_replication.html?highlight=dcsync#directory-replication-services-auditing) <br>
[**Alteredsecurity sobre DCSYNC**](https://www.alteredsecurity.com/post/a-primer-on-dcsync-attack-and-detection) <br>




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
