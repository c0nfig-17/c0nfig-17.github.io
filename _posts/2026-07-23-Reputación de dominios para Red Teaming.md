---
title: Reputación de dominios para Red Teaming
description: En este post profundizaremos en la categorización y mejora de reputación de dominios para red teaming.
date: 2026-07-23
categories: [domain]
tags: [redteam, evasion, email, opsec, phishing]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/domain/0.jpg

---
## 0. Introducción y motivos
Este post viene de la necesidad de organizar por mi parte algunas técnicas que he utilizado en ocasiones pero creo que necesitan ser puestas en orden. Como puedes ver este es un post exclusivamente hablando de evasión de dominios para C2 y Phishing siempre desde el enfoque de un ejercicio de Red Teaming realista y buscando ayudar a compañeros para desplegar infarestructura. No es el objetivo de este post incentivar o alentar a realizar acciones maliciosas. <br>

En este post no resumo ni mucho menos todas las técnicas existentes ni todas las herramientas que pueden ayudarte, no es el objetivo, sino comprender algunas de ellas y remarcar algunas que me han sido de utilidad o por las cuales he pasado para comprender cuales eran mejores para mi caso de uso. <br>


## 1. Controles, problemas y monitorización: entender el problema
Cuando afrontamos el problema de exponer un activo para realizar un ejercicio de manera real la gente suele tratart de ir al grano "como lo hacemos y hagamoslo". En mi caso me gusta dar rodeos y no estoy acostumbrado (ni quiero estarlo) a plantear respuestas sencillas a problemas complejos. Si buscas una receta única y sencilla para esto no es tu post. <br>

Es importante que tengas en cuenta una cuestión durante todo este post ¿para que quiero mi dominio? Es decir ¿Será un servidor C2? ¿Será un phishlet de Evilginx? ¿Un Clicfix? ¿Un repositorio para exfiltrar?. Hay muchas cosas a tener en cuenta y estas determinarán que estás usando. Por ejemplo me gusta mucho la web [**Living Off Trusted Sites (LOTS)**](https://lots-project.com/) el cual nos puede ayudar a definir servicios que usar para determinadas fases en nuestro ejercicio ¿para que te vas a preocupar por un dominio reputado cuando puedes subirlo todo a un one drive y salir por la puerta?. Creo que este proyecto pone bastante en el foco a lo que me refiero con este punto. Recomiendo profundizar en él. <br>

Puede que al leer este artículo estes pensando en desarrollar una campaña de phishing para un ejercicio, en tal caso te recomiendo revisar mi post anterior [**Phishing OPS**](https://c0nfig17.com/posts/Phishing-OPS/) por si te es de ayuda.

Para explicar un poco el problema vamos a tratar diferentes apartados. <br>

### 1.1 Herramientas contra las que nos enfrentamos
Durante la ejecución de cualquier prueba vas a tocar con diferentes capas de control tanto en un acceso inicial como en un phishing que tiene que afrontar diferentes controles. Por ejemplo imaginemos el caso de que conseguimos que alguien descargue e instale un malware dentro de un intento de acceso inicial en un ejercicio de Red Team <br>
En este punto podemos ver que con una simple ejecución de nuestro malware tendremos que enfrentarnos a que el AV/EDR que se encuentre en el host pueda analizar a donde se realizan las peticiones, de manera adicional puede que nos encontremos con soluciones de Zero Trust y con el propio proxy que da salida al tráfico de la infraestructura. Todo esto en la primera ejecución no debe fallar lo cual es complicado y pone a diferentes adversarios (legitimos) con diferente inteligencia a trabajar en nuetra contra. Por todo ello es importante que trabajemos en como pasar desapercibidos frente a esta capa. <br>

![Desktop View](/assets/img/domain/1.png)

También nos podemos encontrar con situaciones como la llamada a un portal de phishing el cual por ejemplo también puede contrar con los propios controles del navegador por defecto o navegador empresarial por poner un ejemplo. También tendrás que controlar la detección por defecto de la bandeja así como la infra que has montado o sistemas adicionales de control en el correo.. En general, tienes una barbaridad de causísticas y vas a tener que tratar de ser lo más eficiente posible diseñando lo que vas a hacer con un objetivo hacía que lo quieres usar.  <br>

### 1.2 La Capa 8
Tienes que tener inventiva con el [**usuario**](https://mundowdg.com/blog/diccionario-luser-cristiano/) para no acabar pareciendote a él.  <br>

![Desktop View](/assets/img/domain/2.jpeg)


### 1.3 El analista e Inteligencia
Contra targets interesantes te vas a encontrar problemas mayores (como es lógico). Parte de ese problema se llama inteligencia y analistas. Normalmente las propias soluciones de seguridad tienen ya una recopilación de inteligencia que ponen a tu disposición, pero en el caso de empresas más grandes te vas a encontrar a un equipo de analistas y proveedores de inteligencia. En donde reinen las malas prácticas tendrás suerte y un triaje excaso o ineficiente incluso te servirá de impulso para tus actividades. <br>

![Desktop View](/assets/img/domain/4.jpg)

En la mayoría de casos los equipos no solo cuentan con herramientas sino con gente capaz que te va a poner las cosas dificiles. Te encontrarás no solo con situaciones en las que han aprendido de sus errores y tienen limitado tráfico inseguro o dominios de reciente creación, sino con enormes bases de datos de inteligencia e información compartida entre empresas que te harán más dificil trabajar. <br>

Una vez escuche a un compañero referirse a los analistas de inteligencia como las "luces largas" de un coche, ya sabes, eso que te ayuda a ver si hay un jabalí en la carretera y te vas a meter una torta. Contar con inteligencia suele complicar la cosa ya que permite no solo que tus herramientas funcionen mejor y de manera más depurada sino acortar el tiempo en el que las nuevas técnicas, métodos o herramientas son útiles así como preveer cosas como que le pase a una empresa lo mismo que le pasa a otra. Para esto existen las [**MISP**](https://www.misp-project.org/) o programas de compartición de IOCs (y muchas otras cosas) como en el caso de España es la [**Red nacional de SOCs**](https://rns.ccn-cert.cni.es/) .  <br>

![Desktop View](/assets/img/domain/3.jpeg)

El resumen de todo esto es: te va a tocar innovar y ser original. <br>


## 2. Técnicas de generación de nombres y subdominios
Existen multitud de tecnicas relaccionadas con localizar o generar nombres de dominio. Detallamos algunas. 

### 2.1 Typosquatting 
Siempre puedes comprar un dominio similar. Si yo tengo c0nfig17.com quizás puedas usar cOnfig17.com . También puedes usar simplemente metodologías de Typosquatting para tratar de colar en un dominio con la letra “a” la letra cirilica “а” (Unicode U+0430) . En general existen bastantes herramientas para investigar este tipo de cambios, te recomiendo revisar [**urlinsane**](https://github.com/rangertaha/urlinsane) . <br>

Tanto para generar como para analizar typosquatting te recomiendo utilizar [**urlcrazy**](https://github.com/urbanadventurer/urlcrazy) y [**dnstwist**](https://github.com/elceef/dnstwist) . Ten en cuenta que estas herramientas se utilizan para detectar modificaciones de un dominio de manera fácil por lo que aunque te pueden dar una idea quizás no es el mejor enfoque utilizar el output directamente. De hecho una buena opción puede ser asegurarse en local que el dominio que vas a comprar no es una permutación directa de herramientas de este estilo de cara a evitar ser localizados con estos automatismos. 

### 2.2 Nombres de dominios y gestión de servicio 
Esto es un poco de cajón, pero no necesitas tener un super dominio relaccionado con Microsoft. Tener conocimiento de la compañía, como funciona, empresas relaccionadas, cadena de suministro o departamentos es más útil. En ese sentido mi recomendación no siempre es obtener un dominio que por ejemplo simule ser microsoft sino alomejor un portal de soporte. También ten en cuenta que puedes usar subdominios una vez categorizado el dominio, por lo que no necesariamente necesitas usar trucos para incluir contenido en el mismo. 

### 2.3 Diferentes objetivos diferentes recursos
Lo normal debe de ser utilizar diferentes dominios para diferentes funcionalidades. Con esto no me estoy refiriendo solo a diferenciar entre tu C2 y tu phishing, sino de las funcionalidades de exfiltración, el delivery, el dominio del mail... Preparar la infra en función de la funcionalidad que le daremos es fundamental.

### 2.4 Inteligencia 
Como he explicado previamente tienes a equipos analizando internet en búsqueda de recursos que puedan ser maliciosos. Si utilizas técnicas sencillas y comunes puede que existan scaners que proactivamente te localicen. Te pongo un ejemplo sencillo, si tienes que atacar a Micosoft con su dominio "Microsoft.com" muy probablemente tengan contratada inteligencia que busca proactivamente si alguien compra "micr0s0ft.com" con el simple objetivo de banearlo antes incluso de que un atacante despliegue lo que sea. Por eso en muchas ocasiones es más importante analizar bien que vas a hacer que utilizar una técnica para confundir al usuario. <br>

### 2.5 Nombres de dominios internos fuera
En ocasiones existen nombres de dominio DNS internos que externamente no son comprados. Si descubres mediante analisis de metadatos o filtración de información estos dominios y los registras fuera puede ayudarte de cara a comprometer a un usuario en un momento dado. 


## 3. Algunas técnicas relaccionadas con la exposición

### 3.1 Subdomain Takeover y apropiación de dominios anteriores
En el caso del subdomain takeover no me detendré ya que tengo un [**post detallado**](https://c0nfig17.com/posts/Subdomain-takeover/) solo sobre esta técnica y sus usos maliciosos. Utilizar un propoio subdominio de la infraestructura puede agilizar todo el proceso y en más ocasiones de lo que piensas puede ser sencillo. <br>
Por otro lado recomiendo realizar una revisión exahustiva en la fase de descubrimiento de todos aquellos dominios que han sido utilizados por la infraestructura. Si tienes suerte, la propia empresa a auditar dió de baja dominios hace tiempo y los puedes comprar, esto es algo que puedes comprobar cruzando tu descubrimiento con [**wayback machine**](https://web.archive.org/). Puede que te lleves alguna sorpresa que te permita avanzar rápido. También es recomendable evaluar dominios que esten a punto de expirar ya que si bien podemos generar impacto también pueden ser los próximos en ser abandonados y útiles para nuestras funciones. 

### 3.2 Domain Fronting
Una manera de evadir la categorización de un proxy o una herramienta de seguridad es utilizar Domain Fronting para evadir esos controles. La técnica reportada por [**Raphael Mudge**](https://www.youtube.com/watch?v=IKO1ovl7Ky4)  viene bien explicada ya en 2018 en el [**blog de cobalt strike**](https://www.cobaltstrike.com/blog/high-reputation-redirectors-and-domain-fronting). Esta técnica consiste en ocultar el destino real de una conexión HTTPS haciendo que el nombre del dominio visible en la capa TLS sea distinto del dominio real soliciado en la cabecer HTTP. Esto nos permite enmascarar el tráfico. Hay que tener en cuenta que en muchas ocasiones esto es conocido y puede ser detectado por algunas soluciones. 

### 3.2 Uso de redirectores
El objetivo de este post no es profundizar en esto pero si que me parece relevante el señalar que la posibilidad de utilizar redirectores para facilitarte el trabajo. Te dejo un post del [**blog de cobalt strike**](https://www.cobaltstrike.com/blog/simple-dns-redirectors-for-cobalt-strike) por dejar alguna referencia


## 4. Consideraciones respecto el activo a exponer

Estaba escribiendo esto sin profundizar para nada en el activo posteriormente expuesto a posteriori, pero creo que es importante definir algunas consideraciones mínimas para no cargarte todo el trabajo.

- Ubicación del Activo: Ubica el servidor cerca del target. No pongas el servidor en Kazajistan con un target Francés o utilices IPs de paises que estarán baneados. Entender cual es el target es relevante, si tienen ataques de DDOsia constantemente tendrán bloqueaod el tráfico a IPs rusas y tu ataque caerá en una tontería.
- Utiliza ASNs de calidad, si utilizas ASNs los cuales tienen mala reputación no te servirá de nada.
- Evita usar IPs que tengan ya mala reputación y estén en ficheros [**DROP**](https://www.spamhaus.org/blocklists/do-not-route-or-peer/) .
- No debes tener actividades maliciosas prvias en la IP. Busca en ViruS Total, abuseipdb, Barracuda IP, talos intelligence o tu herramienta de preferencia.
- Bloquea el acceso directamente por IP a tu máquina. Para esto hay mil métodos, te dejo el [**DbgMan**](https://0xdbgman.github.io/posts/the-art-of-phishing-part-one/#blocking-direct-ip-access) .
- De cara analizar el MailServer que podrías usar en el caso de un phishing te recomiendo usar [**multirbol.valli**](https://multirbl.valli.org/). Personalmente para este paso prefiero usar SMTPs Relays para tener buena reputación, pero existen muchas causísticas.
- Recuerda configurar bien el DNS, no dejes todo por defecto por que canta. 
- Dependiendo de lo que quieras hacer con el activo y la infra que vas a desplegar necesitarás establecer métodos de resiliencia y resistencia no solo a ataque sino a takedowns. Ten en cuenta que este no es un post relativo a montar una infra de red teaming o para tu C2, no vamos a abordar todo. 


## 5. Calidad y categorización del dominio

### 5.1 Proveedor de nombres de dominio
En este caso me voy a limitar a citar el apartado de DbgMan en [**The art of phishing**](https://0xdbgman.github.io/posts/the-art-of-phishing-part-one/#1-pick-the-registrar-domain-seller) respecto a las compras de proveedores de nombres de dominio. Es importante tanto utilizar las configuraciones de WHOIS en Privacy como el utilizar proveedores conocidos y que nos den confianza. En mi caso tengo experiencia con otros proveedores y dependerá un poco del ruido que hagas.

### 5.2 Delivery y categorización en correo
Te recomiendo revisar el post [**SPF, DKIM, DMARC & Fails**](https://c0nfig17.com/posts/Email-Spoofing-DKIM,-SPF,-DMARC-&-Fails/) y configurar todas aquellas funcionalidades que tendría un email legitimo. Te recomiendo en general implementar todas las funcionalidades que necesites para parecer un correo legitimo, aquí guiate por los profesionales del sector como [**mailchimp**](https://mailchimp.com/es/resources/email-authentication/). Guiarte de empresas que trabajan email marketing puede ser de ayuda para evitar caer en SPAM, por ejemplo aquí una buena explicación sobre que es una [**lista negra**](https://mailchimp.com/es/resources/email-authentication/) y como salir de una, aunque no nos interesa estar nunca. En este punto te recomiendo profundizar en otras herramientas como [**BIMI**](https://sendmarc.com/es/bimi/que-es-bimi/) para dar reputación al dominio. 

### 5.3 Antiguedad del dominio
Puede que estes leyendo esto y tengas que tener la categorización para mañana. Complicado. Un elemento importante es la antiguedad del dominio, sobre todo en el caso del phishing aquí tienes [**un caso real**](https://vitamail.vitanur.com/blogs/domain-age-email-deliverability-guide) .  Como ya puedes intuir cuanto más tiempo mejor, pero volver al pasado imposible. Por lo general no se recomienda usar dominios recien creados o menores a 6 meses. <br>
En ese sentido la mejor opción es comprar dominios genéricos con tiempo y si tienes clientes habituales comprar para estos clientes con mucho tiempo. Así cuando los necesites en un ejercicio no estarás pillado. Lo recomendable en esta situación es proactivamente subir blogs que parezcan legitimos sobre temas poco polémicos o no categorizables como nsfw. Aunque puede ser una faena usar un wordpress por ejemplo con algunas herramientas de SEO puede ayudarte a mejorar la reputación poco a poco. <br>

![Desktop View](/assets/img/domain/5.jpg)

Si vas sin tiempo tienes que ir a por dominios expirados que puedas localizar. Puedes utilizar [**expired domains**](https://www.expireddomains.net/) para encontrar dominios legitimos y analizar antes de comprarlo si tienen buena reputación con diferentes herramientas. En ese sentido existen diferentes herramientas que sirven para optimizar tu busqueda. <br>

####  5.3.1 CatMyPish y AirMaster
Los proyectos  [**CatMyPish**](https://github.com/Mr-Un1k0d3r/CatMyPhish) y [**AirMaster**](https://github.com/t94j0/AIRMASTER) te permiten buscar dominios categorizados en expireddomains.net y cruzarlos con [**Symantec**](https://sitereview.bluecoat.com/#/) . Este momento son proyectos con poco mantenimiento y puede que no te sea de ayuda. Tendrás que valorarlo. 

####  5.3.2 Domain Hunter
Por otro lado tenemos [**DomainHunter**](https://github.com/threatexpress/domainhunter) que nos va a permitir cruzar información de diferentes fuentes y proporcionar información sobre dominios elegibles en base a su categorización. 


### 5.4 Categorización del dominio
En general tienes que saber que existen categorizaciones diferentes usadas para analizar el contenido de una web. Tu web puede estar categorizado como Technology, Health, nsfw, Finance, News.... Logicamente nos interesa estar categorizado bajo las mejores categorias como son financiero o salud. Por ejemplo estar categorizado como una web de busqueda de empleo puede ser negativo ya que algunas empresas pueden bloquear estas webs. <br>

En la categorización del dominio también entrará el SEO y posicionamiento de las webs. Pongo un ejemplo claro, tratar de categorizar un dominio como legitimo del sector financiero pero tener una web con anuncio random o contenido de fútbol hará que no sea coherente, pero también que no este posicionado bien en los buscadores. No se trata de aparecer los primeros pero si de parecer legitimos ¿ Una gestoría que envia un mail con un dominio bien categorizado pero que no tiene web? ¿su web está hecha entera con IA? ¿Es tu web un copia y pega de otra?  Son elementos facilmente analizables y que pueden llevar a categorizar tu servicio como malicioso. 

####  5.4.1 DomCat
La gente de [**Black Hills infosec**](https://www.blackhillsinfosec.com/domcat-a-domain-categorization-tool/) (los cuales recomiendo muchisimo) publicaron sobre una herramienta reciente llamada DomCat [**DomCat**](https://github.com/IcyLance/DomCat). La pongo la primera tanto por que es relativamente nueva como por que tiene soporte para win y linux. Para usarla necesitarás API de [**NameSilo**](https://www.namesilo.com/) la cual la herramienta basicamente usa para utilizar la base de datos de [**expired domains**](https://www.expireddomains.net/), de manera adicional necesitaras una API de CLoudflare Intel. 

####  5.4.2 Chameleon
Otra manera de realizar una buena enumeración y completa de categorizaciones es mediante la herramienta [**Chameleon**](https://github.com/mdsecactivebreach/Chameleon) desarrollada por [**MDsec**](https://www.mdsec.co.uk/2017/07/categorisation-is-not-a-security-boundary/) y basada en herramientas que he comentado anteriormente como DomainHunter y CatMyPhish. Chameleon nos permite automatizar parte del trabajo de categorizar recursos. 

####  5.4.3 Reclamar mejoras de categorización
Puedes abrir tickets en los siguientes servicios para reclamar una categorización. Esta categorización puede tardar días en ejecutarse y propagarse por internet

[**Trellix**](https://www.trustedsource.org/) <br>
[**McAfee**](https://sitelookup.mcafee.com/) <br>
[**Symantec**](https://sitereview.bluecoat.com/#/) <br>
[**Office MS**](https://sender.office.com/) <br>
[**zvelo**](https://tools.zvelo.com/) <br>


### 6. Evaluación y mejora de reputación
En general vas a necesitar revisar y tratar de mejorar la reputación poco a poco. Para ello te dejo una lista de servicios con los que realizar estas comprobaciones aunque muchos los hemos visto durante el propio post. 
[**Sender Score**](https://senderscore.org/) <br>
[**Barrracuda**](https://www.barracudacentral.org/lookups) <br>
[**MXtoolbox**](https://mxtoolbox.com/domain) <br>
[**FortiGuard**](https://www.fortiguard.com/webfilter) <br>
[**PaloAlto**](https://urlfiltering.paloaltonetworks.com/) <br>
[**trend Micro**](https://global.sitesafety.trendmicro.com/) <br>
[**VirusTotal**](www.virustotal.com) <br>

Recuerda que pueden existir cambios en tu dominio que no se hayan propagado y en consecuencia no hayan llegado a nuevos servicios. Te recomiendo revisar  [**dnschecker**](https://dnschecker.org/) .  <br>

Dicho esto, mucho ánimo con la busqueda. Si te tiran un dominio no pasa nada.... 


![Desktop View](/assets/img/domain/6.jpeg)




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

