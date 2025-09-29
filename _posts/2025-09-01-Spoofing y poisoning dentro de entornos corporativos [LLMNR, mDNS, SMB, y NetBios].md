---
title: Spoofing y poisoning dentro de entornos corporativos [LLMNR, mDNS, SMB, y NetBios]
description: Protocolos deprecados, mal implementados o directamente no gestionados... son la puerta de entrada a comportamientos de atacantes.
date: 2025-09-01
categories: [AD]
tags: [spoofing, ad, domain, smb, netbios, mdns, llmnr]     # Los tags deben estar siempre en minúsculas.
image:
  path: assets/img/llmnr/poisoning.jpg
---

## 1. ¿Que es eso del spoofing?
Básicamente el spoofing consiste en suplantar un equipo o identidad legitima para que podamos los atacantes obtener una información con la que poder dar pasos adelante en nuestro ataque, sobre todo centrados en suplantar una identidad que es legitima. Dentro de esto existen multitud de maneras y protocolos sobre los cuales con un poco de maña, y algún problema de configuración, podemos abusar. Hay que tener en cuenta que como enseñaré más adelante no necesariamente son "problemas de configuración" sino configuraciones inseguras las cuales deben ser así por diseño. El que algo sea inseguro por diseño no significa que se deba de mirar a otro lado, sino tratar de resolverlo, mitigarlo, gestionar el riesgo... siempre hay una opción diferente a hacer como si el problema no existiese. 

Hay que ser consciente que no siempre este vector de ataque se explotará porque existirá una vulnerabilidad que permita usarlo, sino también mediante un uso diferente de algo que existe correctamente. En este post trataremos de explicar algunos protocolos, como abusar de ellos y como mitigar que alguien abuse de ellos. 
Por si quereís profundizar hoy vamos a ver técnicas que estan bajo T1557 en la [**Matriz Mitre**](https://attack.mitre.org/techniques/T1557/001/):  en concreto la T1557.001

## 2. Algunos protocolos, resoluciones y mitigaciones

## 2.1 LLMNR Spoofing

### 2.1.1 Explicación y explotación
El protocolo LLMNR (Link-Local Multicast Name Resolution) es un protocolo de [**Microsoft**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-llmnrp/02b1d227-d7a2-4026-9fd6-27ea5651fe85): creado para resolver nombres de equipos de una misma red cuando no hay un servidor DNS disponible. 
Por diseño el protocolo lo que ayuda es a que un equipo, tras consultar al DNS y no encontrar el equipo que encuentra, preguntar a todos los equipos de la red para tratar de encontrarlo. Este protocolo era antes muy utilizado en infrastructuras pequeñas o que ya tienen unos años. Un ordenador legitimo buscará esa información, si decimos que tenemos esa información tratará de autenticarse contra nosotros y ahí podremos obtener el hash de sus credenciales. 

Para tratar de explicar la lógica del ataque he realizado este gráfico (creo que con menos publicidad que los que hay por interenet se entiende bastante mejor). Es básicamente lo que haremos en la mayoría de pasos de este post, por lo que con entenderlo una vez entenderás la mayoría de variantes, lo importante es entender que 1)simplemente confundes al protocolo y 2)el objetivo es que trate de validar contra tí para obtener el hash.
![Desktop View](/assets/img/llmnr/image1.png){: width="972" height="589" }

Para abusar de este protocolo podemos usar la herramienta [**Responder**](https://github.com/lgandx/Responder):
Simplemente tendremos que levantar responder en la máquina
```text
sudo responder -I eth0 -dwP
```

y esperar a que obtengamos la información de una victima

```text
[+] Listening for events...
[SMBv2] NTLMv2-SSP Client   : 10.0.1.1
[SMBv2] NTLMv2-SSP Username : DOMINIO\usuario
[SMBv2] NTLMv2-SSP Hash     : usuario::DOMINIO:61dde887aeb2af2a:76DD8039B9606119586BC9A4EF5F3C1:010100000000000080C0653150DE09D20107929B9D6080F5BB000000000200080053004D004200330001001E00570049004E002D0056005600400140053004D00420033002E006C006F00630061006C0003001E00570049004E002D005600560053004D00420033002E006C006F0063006100070008008
```

Una vez tengamos el hash NTLMv2 habrá que usar Hashcat o tu herramienta de cracking favorita para trabajar el hash.

```bash
hashcat –m 5600 <hashfile.txt> <wordlist.txt>
```

### 2.1.2 Detección e investigación de explotación:
Existen varios enfoques para el SOC, principalmente:
1. Algunos EDRs y AV (me consta que Sentinel de Defender) tienen capacidad de detección sobre comportamientos anómalos. Si un equipo empieza a dar respuesta a todas las consultas DNS fallidas os saltará alguna alerta.
2. Analizar eventos... cuantos eventos de resolución de DNS fallidos hay??? han aumentado?? puedes ver los KO del DNS?? 
3. Encontrar el hostname que hace esa respuesta maliciosa os puede ayudar mucho. Normalmente la gente no es tan cuidadosa como para simular que su equipo está en dominio antes de levantar un responder... eso debería daros pistas de que equipo, donde está ubicado y como se está comportando.... El problema es que tienes que ya sabes que ese es el vector. 
4. Todas aquellas recomendaciones que ya se recomiendan en la sobre todo aquellas creadas a raíz de eventos de windows.

### 2.1.3 Mitigación
Existen varias vias para deshabilitar este protocolo de raiz:

#### 2.1.3.1 Deshabilitar via PowerShell
Puedes ejecutar como administrador lo siguiente

```powershell
New-Item "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT" -Name DNSClient -Force
New-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Name
EnableMultiCast -Value 0 -PropertyType DWORD -Force
```

Ten en cuenta que siempre puedes generar una GPO que lo ejecute o tener una implementación diferente. Por ejemplo puedes crear una GPO que aplique el siguiente script como startup.
```powershell
set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces\tcpip* -Name NetbiosOptions -Value 2
```

#### 2.1.3.2 Deshabilitar via GPO
Puedes realizar uan GPO la cual se aplique en todos los ordenadores que esten bajo tu dominio. Para deshabilitarlo via GPO debes ir a `Computer Configuration > Administrative Templates > Network > DNS Client y buscar "turn off multicast name resolution"` , deberás ponerlo en estado "enabled". Luego como siempre los equipos deberán reiniciarse o deberás usar "gpoupdate /force".

#### 2.1.3.3 Verificar mitigación
Podemos verificar que la implementación es correcta cuando ejecutamos lo siguiente y obtenemos como respuesta un `0`.

```powershell
$(Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows NT\DNSClient" -name EnableMulticast).EnableMulticast
```


## 2.2 NetBios Poisoning and Relay

### 2.2.1 Explicación y explotación
El protocolo NetBios (Network Basic Input/Ouptu Systems) es un software desarrollado para enlazar un SO con un hardare especifico. Es similar en su funcionamkiento y en el impacto que genera un atacatnte, simplemente en vez de usar LLMNR usará NetBios para resolver donde se encuentra un equipo y recibiremos las credenciales. 

### 2.2.2 Detección e investigación de explotación:
Todas aquellas recomendaciones que ya se recomiendan en la [**T1557.001 de ATT&CK Mitre**](https://attack.mitre.org/techniques/T1557/001/) sobre todo aquellas creadas a raíz de eventos de windows.

### 2.2.3 Mitigación
Puedes deshabilitarlo equipo a equipo lo cual obviamente es un dolor, pero puedes hacerlo desde las opciones avanzadas de TCP/IP simplemente pulsando en deshabilitar (puedes hacerlo así quizás en un equipo de maqueta con el que generes la imagen de bastionado, pero no es nada útil. ) Por eso vamos a ver más opciones

##### 2.2.3.1 Mediante una GPO
Puedes utilizar el siguiente script de powershell como script de starutp para deshabilitar en todos los equipos cuando actualicen la GPO el protocolo NetBios al inciar.
```powershell
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey |foreach { Set-ItemProperty -Path "$regkey\$($_.pschildname)"
-Name NetbiosOptions -Value 2 -Verbose}
```

También tienes esta opción con esta consulta WMI por si te es más útil

```powershell
Get-WmiObject -Class Win32_NetworkAdapterConfiguration | % {$_.SetTcpipNetbios(2)}
```

#### 2.2.3.2 Claves de registro
Puedes modificar la siguiente clave de registro

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces
```

el valor debe ser `2`.

## 2.3 mDNS Poisoning and Relay

### 2.3.1 Explicación y explotación
mDNS (Multicast DNS) es muy similar a NetBios y LLMNR por lo que no profundizaremos en su forma de explotación. 

### 2.3.2 Detección e investigación de explotación:
Todas aquellas recomendaciones que ya se recomiendan en la [**T1557.001 de ATT&CK Mitre**](https://attack.mitre.org/techniques/T1557/001/)  sobre todo aquellas creadas a raíz de eventos de windows.

### 2.3.3 Mitigación

#### 2.3.3.1 Mediante una GPO
Puedes generar un script en startup con:

```powershell
set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters\"
-Name EnableMDNS -Value 0 -Type DWord
```

#### 2.3.3.2 Claves de registro

```text
REG ADD "HKLM\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters" /v " EnableMDNS" /t REG_DWORD /d "0" /f
```

#### 2.3.3.3 Regla en el firewall local
Puedes crear una regla en el firewall de la máquina, ya sea defender o el que sea, que bloquee el tráfico mDNS.


## 2.4 SMB Relay

### 2.4.1 Explicación y explotación
Cuando hablamos de Server Message Block (SMB) las cosas se ponen serías. Estamos hablando de un protocolo con una gran implementación y uso con funicones muy muy diversas y que es muy dificil de eliminar o limitar en una compañía mediana y grande. Además nos da muchas facilidades a los atacantes para operar ya que permite, además de compartir información, realizar ataques similares a los previos. Además contamos con hasta 3 [**Versiones Diferentes**](https://learn.microsoft.com/es-es/windows-server/storage/file-server/troubleshoot/detect-enable-and-disable-smbv1-v2-v3?tabs=server) de SMB en Windows con mejoras obviamente desde la primera versión.
En este caso los atacantes podemos aprovechar sobre todo funcionalidades mal implementadas de SMB o de otras herramientas que tienen SMB habilitado para así poder obtener hashes NTLM de equipos los cuales estemos atacando. 

En este caso podemos usar la herramienta ntlmrelayx de la suite de [**impacket de Fortra**](https://github.com/fortra/impacket) para forzar a que se realice una conexión hacia nuestra máquina y obtener esos hashes.

### 2.4.2 Detección e investigación de explotación:
Voy a recomendar en este caso la guia de investigación y analisis que ha publicado [**Kibana**](https://www.elastic.co/guide/en/security/8.19/potential-machine-account-relay-attack-via-smb.html):  la cual ayuda un poco a guiar los pasos de investigaciones, determinar falsos postivos y organziar la respuesta. 

### 2.4.3 Mitigación

Es importante lo primero profundizar en este post de [**Microsoft**](https://learn.microsoft.com/es-es/windows-server/storage/file-server/smb-secure-traffic): el cual se puede reducir en: si no usas SMB deshabilitalo. Es fundamental entender que una mitigación va a partir de entender que el riesgo existirá en muchisimás máquinas, y que por ello los usuarios que sean Administradores de Dominio o Locales deben estar limitados para tareas especificas y no usarse para tareas o máquinas las cuales usen SMB para evitar que sea una forma sencilla de escalada.
La mitigación general a esto será mediante la habilitación del firmado SMB en todos los dispostivios. Para ello debemos ir al editor de políticas y establecer las siguientes en `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options`:

En el lado del cliente:

- `Microsoft network client: Digitally sign communications (always)`
- `Microsoft network client: Digitally sign communications (if server agrees)`

En el lado del servidor:

- `Microsoft network server: Digitally sign communications (always)`
- `Microsoft network server: Digitally sign communications (if client agrees)`

> Hay que entender que esto no consiste exclusivamente en el tráfico SMB que tiene la máquian windows, sino también en el que el software del que haces uso y que puede trasladar trazas que sean interceptadas o quizás haciendo un uso indebido de una herramienta mal desarrollada o configurada. Es importante entender que no solo "la solución al problema" soluciona el problema, sino que en ocasiones no detectamos la causa raiz particular, sino que mitigamos la general exclusivamente. 
{: .prompt-warning }

Podemos confirmar la mitigación mediante el siguiente comando en CMD:

```text
reg query HKLM\System\CurrentControlSet\Services\LanManServer\Parameters | findstr /I securitysignature
```

Si nos devuelve `0x1` estará resuelto. 

Os recomiendo de manera particular este postr de [**WOSHUB**](https://woshub.com/smb-1-0-support-in-windows-server-2012-r2/) en el que se detalla para diferentes versiones de windows como deshabilitar versiones o tomar ciertas accioens relativas a SMB 

Para finalizar, más que una aportación una reflexión.
![Desktop View](/assets/img/llmnr/image2.jpeg){: width="972" height="589" }

Espero que os sea de ayuda!

---
## Apoya el contenido de ciberseguridad en castellano

Si esta publicación te ha sido útil y quieres apoyar mi trabajo para que continúe creando más contenido, aquí te dejo algunas formas de apoyar:

1. **Compartir el contenido**  📲
   Si crees que esta guía puede ser útil para otras personas, compartirla en tus redes sociales es una gran ayuda. 

2. **Donar en Ko-fi**  💖
   Puedes hacer una donación rápida a través de Ko-fi para ayudarme a seguir publicando guías y tutoriales. ¡Cada aportación cuenta y es muy apreciada! 

   <script type='text/javascript' src='https://storage.ko-fi.com/cdn/widget/Widget_2.js'></script><script type='text/javascript'>kofiwidget2.init('Apoya este contenido!', '#455d85', 'A0A41BO608');kofiwidget2.draw();</script> 
   
3. **Usa mi enlace de afiliado de NordVPN y NordPass para mejorar tu seguridad y apoyar la creación de contenido**  🛡️
   Puedes suscribirte a [**NordVPN**](https://go.nordvpn.net/aff_c?offer_id=15&aff_id=132246&url_id=902) con un 75% de descuento y 3 meses gratis o a [**NordPass**](https://nordpass.com/special/?utm_medium=affiliate&utm_term&utm_content&utm_campaign=off488&utm_source=aff132246&aff_free) con un 53% de descuento y 3 meses gratis y mejorar tu seguridad a la vez que apoyas el contenido de ciberseguridad en español. <br>
   
![Desktop View](/assets/img/Nordvpn/logonordvpn.png){: width="250"}
![Desktop View](assets/img/Nordvpn/logonordpass.png){: width="200"}

---

¡Gracias por tu apoyo! 🙏
![Desktop View](assets/img/banner.png) <br>
