---
title: AdaptixC2
description: Uso y consideraciones sobre AdaptixC2
date: 2026-06-26
categories: [c2]
tags: [redteam, bypass, evasion, pentesting, opsec,malware, ofuscación ]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/adaptix/0.jpg

---
## 1. Adaptix C2 
Hoy vamos a ver un C2 que está haciendose común y recomendando mucho últimamente que es [**AdaptixC2**](https://github.com/Adaptix-Framework/AdaptixC2) . Voy a aprovechar que tengo que realizar algunas tareas con este C2 para comentaros algunas ventajas de este C2 de código libre. <br>

![Desktop View](/assets/img/adaptix/1.png)

Adaptix se ha hecho famoso tanto por su parecido con CobaltStrike como el que están saliendo una serie de agentes custom que permiten mejorar sus capacidades de evasión. Aún así hay que aclarar que como siempre las capacidades de evasión no vienen por defecto y que existe mucho trabajo por realizar de manera manual. Un buen ejemplo de los indicadores que despliega Adaptix es el siguiente post de [**Hunt.io sobre AdaptixC2**](https://hunt.io/blog/adaptixc2-uncovered-capabilities-tactics-hunting) el cual profundiza en mucha de las configuraciones por defecto e infraestruturas facilmente señalables. También puedes ver este [**artículo de Censys**](https://censys.com/blog/adaptixc2-open-source-c2-framework/) que profundiza en el firgerprinting de Adaptix y es bastante reciente. 

![Desktop View](/assets/img/adaptix/2.png)

Para el despliegue es relavante hacer uso de la [**Wiki oficial de AdaptixC2**](https://adaptix-framework.gitbook.io/adaptix-framework) que explica con bastante detalle como realizar la instalación. Una cuestión relevante de cara a la instalación en máquinas Kali es que existe un [**limitado número de versiones estables**](https://adaptix-framework.gitbook.io/adaptix-framework/adaptix-c2/getting-starting) . Aclaro esto porque por ejemplo en Kali en las últimas versiones al instalarlo desde el repositorio a mi me ha generado muchos problemas, entre ellos la instalación correcta pero que la máquina no sea capaz de reiniciarse. Aún así ten en cuenta que existe un [**Paquete oficial de Kali**](https://www.kali.org/tools/adaptixc2/) para la versión 1.1 de Adaptix. <br>

Además ten en cuenta que existe un [**Extension Kit**](https://github.com/Adaptix-Framework/Extension-Kit) oficial para Adaptix que te pone a tu disposición una gran cantidad de BOFs que te facilitarán la vida.  <br>


## 2. Setup, Agentes y listeners
En mi caso voy a instalar un agente bastante famoso de Adaptix llamado [**kharon**](https://github.com/entropy-z/Kharon) el cual es un agente con capacidades de evasión mejoraras respecto a los agentes que existen por defecto en el C2. La instalación resulta bastante sencilla. <br>

```text
/setup_kharon.sh --ax ../AdaptixC2/
[+] Action: Full installation (all)
{...}
[+] Built listener
[+] Copied agent distribution files to dist
[+] Copied listener distribution files to dist
[+] Installation completed successfully
================================================================
Action: all
Agent: agent_kharon
Listener: listener_kharon_http
Location: /opt/adaptixc2/AdaptixC2
================================================================
```

Además voy a preparar todos los bofs del [**Extension Kit**](https://github.com/Adaptix-Framework/Extension-Kit) <br>


```text
┌──(root㉿kali)-[/opt/adaptixc2/Extension-Kit]
└─# make
=====>>  AD-BOF
# 64-bit builds
{...}
 32-bit builds
[+] smartscan (x32)
[+] nbtscan (x32)
[+] quser (x32)
```

En mi caso ya que voy a utilizar Kharon tengo que configurar en el profile el extender de kharon, lo cual es básicamente añadir estas lineas. <br>

```text
    - "extenders/agent_kharon/config.yaml" 
    - "extenders/listener_kharon_http/config.yaml"
```
En los axscripts hay que añadir <br>

```text
    - "PostEx-Arsenal/kh_modules.axs"
```

Con esto simplemente inicio mi servidor <br>

```text
./adaptixserver --profile profile.yaml 

[===== Adaptix Framework v1.2 =====]

[+] Starting server -> https://0.0.0.0:4321/endpoint [20/06 12:19:16]
[*] Restore data from Database... [20/06 12:19:16]
   [+] Restored 0 agents [20/06 12:19:16]
   [+] Restored 0 pivots [20/06 12:19:16]
   [+] Restored 0 listeners [20/06 12:19:16]
   [+] Restored 0 screenshots [20/06 12:19:16]
   [+] Restored 0 downloads [20/06 12:19:16]
   [+] Restored 0 credentials [20/06 12:19:16]
   [+] Restored 0 targets [20/06 12:19:16]
[+] The AdaptixC2 server is ready [20/06 12:19:16]
```

Inicias el cliente <br>

![Desktop View](/assets/img/adaptix/3.png){: width="486" height="294" }

Inicias sesión y te conectas al C2 <br>

![Desktop View](/assets/img/adaptix/4.png)

Una cosa que me gusta de Adaptix es que puedes tener también clientes en Windows, lo cual a mi personalmente me facilita la vida para temas de evasión. Aunque tienes que tener en cuenta que muchas configuraciones no se replicarán ya que dependen del perfil del cliente y no del teamserver. Por lo que puede ser una facilidad pero en Cobalt Strike mi impresión es que se trabaja de manera un poco más ágil. <br>

![Desktop View](/assets/img/adaptix/5.png){: width="486" height="294" }

En este punto cargamos el AXS del extension Kit y ya tendriamos todos los BOFs disponibles en nuestra sesión actual. <br>

![Desktop View](/assets/img/adaptix/6.png)

El proceso de uso es bastante similar al de CobaltStrike. Primero debes de generar un listener y después tendrás que generar un beacon para el uso concreto que quieras hacer. <br> 

![Desktop View](/assets/img/adaptix/7.png){: width="486" height="294" }


> Personalmente he perdido más tiempo del que me gustaría reconocer por no leer [**la documentación oficial sobre el profile de Kharon**](https://github.com/entropy-z/Kharon/blob/main/doc/5.%20HttpProfile.md) , la cual funciona como un Malleable. He tenido bastantes errores simplemente porque el propio Adaptix me pide un perfil en json que estaba al parecer en la ruta /opt/adaptixc2/Kharon/listener_kharon_http/profiles . 
{: .prompt-warning }

Una vez configurado el listener tienes que generar tu primer agente. Ya aquí a gusto del consumidor respecto de su configuración. <br>

![Desktop View](/assets/img/adaptix/8.png){: width="486" height="294" }

> Recuerda revisar bien el setup que has hecho de [**la documentación oficial sobre el profile de Kharon**](https://github.com/entropy-z/Kharon/blob/main/doc/2.%20Setup.md) si no quieres tener errores.
{: .prompt-warning }

Simplemente te llevas el agente a tu máquina windows y ahí lo tienes! <br>

![Desktop View](/assets/img/adaptix/9.png)

Una cosa chula de Adaptix es que existe la posible generar beacons para máquinas Linux. En mi caso voy a usar el agente Gopher para desplegar un beacon de text en mi propia Kali. <br>

![Desktop View](/assets/img/adaptix/10.png)


## 3. Evasion

Por supuesto el payload que hemos generado está cumpletamente firmado como podemos ver utilizando [**ThreatCheck**](https://github.com/rasta-mouse/threatcheck). Este post no tiene como objetivo trabajar la metodología de evasión que ya explica Rasta Mouse en el artículo [**Bypassing Defender with ThreatCheck & Ghidra**](https://offensivedefence.co.uk/posts/threatcheck-ghidra/) o con mucho más detalle en la [**CRTO**](https://c0nfig17.com/posts/Review-Certificaci%C3%B3n-CRTO/) . Simplemente, pues somos detectados con facilidad. 

```text
C:\Users\windows\Desktop\Tools>ThreatCheck.exe -f Kharon.x64.exe
[+] Target file size: 77312 bytes
[+] Analyzing...
[*] Testing 38656 bytes
```

![Desktop View](/assets/img/adaptix/11.png)

Como es la versión por defecto podemos hasta ilustrarlo con VirusTotal ya que realmente no importa perder reputación en este binario.

![Desktop View](/assets/img/adaptix/12.png)

En mi caso no me voy a liar mucho con la evasión ya que la tengo trabajada de otras ocasiones. Usaré la función de IAT hidding que tengo disponible y [**beatrice**](https://github.com/raskolnikov90/Beatrice.py) de cara a parchear la detección de mi binario.

![Desktop View](/assets/img/adaptix/13.png)

En este punto simplemente lo analizo y.....

![Desktop View](/assets/img/adaptix/14.png)

> Ojo no uso la función -s por casualidad. Beatrice parchea mal algunas partes de mi binario con la solución normal.
{: .prompt-tip }


En este punto comprobamos el funcionamiento correcto y que recibo la conectividad. Y listo! Adaptixc2 desplegado y con evasión estática de defender realizada!
![Desktop View](/assets/img/adaptix/16.png)


![Desktop View](/assets/img/adaptix/15.jpeg){: width="486" height="294" }


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


---

¡Gracias por tu apoyo! 🙏
![Desktop View](/assets/img/banner.png) <br>

