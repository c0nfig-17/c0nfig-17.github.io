---
title: Phishing OPS
description: POC de un ejercicio de phishing
date: 2025-10-28
categories: [Email]
tags: [passwords, email, subdomain, takeover, opsec, evasion]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/phishingops/1.jpg

---


## 1. Explicación del proyecto
En este post voy a tratar de explicar en detalle una demo que he estado preparando. La idea es poder realizar operaciones de phishing emulando la killchain completa de un infostealer o un Initial Acces Broker. Un Initial Access Broker (IAB) es un tipo de actor que se especializa en comprometer activos los cuales son la entrada de otros atacantes. Por poner un ejemplo un IAB tiene contacto de varios grupos, y se encarga de comprometer puntos de la superficie de ataque que son de interés para actores maliciosos (grupos cibercriminales, APTs...) los cuales pagarán un dinero, dependiendo del tipo de activo que se ha comprometido y el impacto que tiene en la consecución de acciones. Un usuario no vale lo mismo que el usuario de un administrador o que el compromiso de una VPN o el establecimiento ya de un kit en una máquina dentro de la red... Cada cosa tiene un precio... <br>
Para que se entienda mejor os dejo una captura realizada por [**Dark Web Informer**](https://darkwebinformer.com/alleged-sale-of-citrix-access-to-fujitsu-japan/) de la venta de un acceso por parte de un usuario de un usuario de Citrix de Fujitsu valorado en 20K$ . Los equipos de Threath Intelligence suelen estar revisando si estos activos pueden encajar en el perfil de tu organización. Aqui es claro, es algo de Fujitsu, pero normalmente suelen ser cripticos y decir la revenue el sector y el tipo de acceso para que no sepas si por tu en concreto estás entre los impactados. 

![Desktop View](/assets/img/phishingops/2.png) 

Es importante entender que estos foros son solo una parte de la venta de accesos. Los grupos de Ransomware por ejemplo tiene gente a la que compran ese tipo de accesos, así como los APTs o las organizaciones que venden Ransom As A Service (RAAS) logicamente tienen diferentes proveedores. Lo relevante aqui es entender que estamos tratando de simular. Vamos a probar la resiliencia frente aun ataque de un IAB para obtener credenciales de uno o varios usuarios.


## 2. Definición de objetivos
Vamos a tratar de marcar algunos objetivos con los que trabajaremos durante esta PoC. Tenemos que tener en cuenta que nuestro objetivo es trabajar como un IAB que busca obtener un acceso inicial que vender, en ese sentido hemos definido varios objetivos:
- Bypasear controles de phishing
- Obtener credenciales
- Obtener 2FA
- Inciar sesión en una cuenta
<br>
¿Como lo vamos a hacer? Con un phishing dirigido

![Desktop View](/assets/img/phishingops/3.png)

## 3. Software que utilizaremos
En mi caso me voy a valer de dos proyectos para poder realizar la operación. El primero el [**gophish**](https://getgophish.com/) el cual tiene su [**repositorio oficial en github**](https://github.com/gophish/gophish) aunque yo haré uso del [**repositorio de kgretxky**](https://github.com/kgretzky/gophish) el cual es un fork oficial que añade una serie de implementaciones. El proyecto de gophish pese a no tener todas las actualizaciones que nos gustaría, muchos issues parados.... en general no tener un gran mantenimiento (pese a ser muy buen proyecto) nos va a ayudar. Hay que tener en cuenta que es conocido, es por ello que si estuviesemos haciendo una operación con un OPSEC un poco más trabajado quizás evitar configuraciones y algunas cosas por defecto puede ser de ayuda. <br>

![Desktop View](/assets/img/phishingops/4.svg)
Por si os es de ayuda he encontrado este repositorio de bigb0ss llamad [**gogophish**](https://github.com/bigb0sss/gogophish) que os ayuda a desplegar más rápido gophish.

Por otro lado usaremos otro proyecto llamado Evilginx en concreto en su versión gratuita, aunque os dejo su [**web oficial**](https://evilginx.com/) por si os interesa para vuestras operaciones. Nosotros nos valdremos del [**repositorio**](https://github.com/kgretzky/evilginx2) con la versión gratuita creada por [**Kuba Gretzky**](https://x.com/mrgretzky) y os dejo su [**blog**](https://breakdev.org/) por si os interesa. El motivo por el que utilizaba la versión de gophish del repositorio de krebretzky en vez del oficial es que nos ayudará luego a la implementación con evilgynx. <br>

![Desktop View](/assets/img/phishingops/5.png)

Todo son versiones gratuitas, accesibles, con código abierto... realmente son recursos útiles y sencillos de utilizar. 

## 4. Infraestructura desplegada
Si bien no tiene sentido contar el paso a paso de como realizamos la instalación ya que en los propios repositorios hay documentación (bastante, actualizada, recursos...) creo que si es la primera vez que haces algo así puede ser interesante contarte que es lo que si que he tenido que montar como infra. En mi caso son dos cosas:
- Dominio: He comprado un dominio que me ayudaría a realizar un poco las tareas. No he comprado cualquier cosa sin sentido y aunque no lo puedo reflejar sigue la siguiente estructura que les he robado a unos scamers. El dominio consiste en comprar un ".xyz" o ".com" o lo que quieras peeeeero haciendolo comenzar por ".com". Vale no se ha entenido nada. En vez de usar un dominio chulo con un typosquatting que saltase rapidamente aun equipo de threath intelligence al registrarlo he comprado un dominio "com7g-alleta.xyz". Dirás "esto canta un montón". Vale ¿y si pongo "agenciatributaria.com7g-alleta.xyz" ? ¿la cosa cambia? es una tonteria pero empezar el dominio por com7 da la impresión de ser agenciatributaria.com/g-alleta.xyz y claro ahi viene el engaño. Así puedo tener un dominio dando por saco más tiempo y solo dependo de los subdominios. Es verdad que no hemos trabajado en este punto el [**OPSEC**](https://c0nfig17.com/posts/OPSEC-Tricks-en-delivery/) mucho pero para un phishing sencilo un truco así nos soluciona bastante (y pican bastante).

![Desktop View](/assets/img/phishingops/6.png)

- Phishing server: En mi caso yo me manejo bien con [**Linode**](https://cloud.linode.com/linodes) de Akamai más que con AWS, Azure o cualquiera de estas... Me es muy sencillo, por lo que viendo un poco las specs me he montado un linode muy simple con poca capacidad y para arriba. No vamos a realizar trabajos muy grandes por lo que he estimado que 1 core y 2GB era más que suficiente. En mi caso he montado una Ubuntu 24.04, no es necesario montar una kali o rizar el rizo ya que realmente el servidor servirá para tareas contadas.

![Desktop View](/assets/img/phishingops/7.png)

> Linode tiene una advertendia de que bloquean puertos SMTP en algunos de sus nodos. En mi caso fue así y explicando el uso que iba a dar me dejaron seguir, pero tenerlo en cuenta de cara a usar este u otro servicio si os es más fácil, sobre todo si pensaís montar el servicio SMTP en el mismo nodo.
{: .prompt-warning }

- Servicio SMTP: Ten en cuenta que vas a enviar correos, por lo que necesitarás un servidio SMTP el cual se encargue de enviar los correos. Tienes varias opciones, usar algo como [**Postman**](https://www.postman.com/) si quieres montarte tu propio servicio en tu servidor (lo cual os comento luego pero os dará problemas en algunos contextos), usar un servicio externo de SMTP como [**Sendgrid**](https://www.mailersend.com/) el cual vincules a tu Gophish o usar una contraseña para aplicaciones con un correo que creeis como un gmail (os permitirá enviar correos en nombre de ese correo, aunque puede ser más fácil de detectar). Aquí ya para gustos colores.

![Desktop View](/assets/img/phishingops/postman.png)
![Desktop View](/assets/img/phishingops/sendgrid.png)

Puede que os sea de ayuda para el servicio SMTP usar [**Mailhog**](https://github.com/mailhog/MailHog) para realizar pruebas y ver si lo que habeís instalado funciona. 

## 5. Perfilado y diseño
Bueno tenemos que ser conscientes que hay dos fases, la primera es el diseño de un phishing y la siguiente es la elaboración del portal a partir del cual extraeremos las credenciales de las personas a las cuales vamos a tratar de engañar en este ejercicio. Creo que es importante que antes de esto (teniendo en cuenta que parte de esto se trabaja para ejercicios de red team y como apoyo a pentesters) es importante tener en cuenta ¿que narices queremos obtener?. Está genial colgarse la medallita de que has conseguido un usuario y pass... pero ¿has perfilado los objetivos a los cuales quieres comprometer? Es decir, enviar un phishing masivo y chustero lo hace cualquiera.... enviar uno que de verdad sea creible y definiendo los objetivos.... eso ya es otra cosa. <br>

![Desktop View](/assets/img/phishingops/8.jpeg)

Si le has dicho a tu jefe que mañana vas a hacer un phising porque se os está resistiendo encontrar una gatera... si son las 4 de la mañana y no sabes porque ese exploit no funciona cuando lo has tocado ya 5 veces y chatgpt ya te está vacilando.... respira... tu puedes.... duerme y haz el perfilado para el phising. Pero no dispares a todo el mundo que la lias. Te dejo algunas preguntas para que te hagas de cara al perfilado: 

- ¿Has conseguido correos legitimos por internet? (Un poquitillo de OSINT no le hace daño a nadie. Que no puedes mandarlo a correos infinitos)
- ¿Has tratado de hacerte con un organigrama? En cuanto envies 500 correos vas a tener al SOC encima... Estudia la infra... ¿quien guarda las llaves de esa casa?. Está chulisimo decir que el CEO se ha comido un phishing del nuevo Iphone 35 pero ¿en serio? ¿Que vas a hacer con ese acceso? ¿Te va a ayudar en el ejercicio? Si cotilleas un poco por linkedin y haces un poco de OSINT puedes conseguir mil usuarios que tendrán más privilegios que el CEO en esa infra y serán más útiles.
- ¿Tienes la estructura de correos? ¿Un apellido? ¿Dos apellidos? ¿Que pasa cuando hay dos nombres? ¿Trabajadores internos y externos? ¿correos temporales? Muchas veces te haces con un correo electrónico y simplemente copiando la estructura sabes el resto. Si Fulanito Perez Martinez tiene un correo que es fulanitop.martinez@galletas.com sabes de sobra que el correo del administrador de sistemas llamado Paco Perez Lopez es pacop.lopez@galletas.com. No es dificil, solo es perfilar. 
- Hay gente que tiene verdaderos desastres en sus buzones compartidos. Empresas en los que 50 personas contestan un buzón.... a patadas....  revisa porque te puede interesar o no. Son 50 peces para pescar pero también 100 ojos mirando lo mismo... <br>

Dicho esto creo que se entiende el concepto. Perfila la victima si quieres que sirva para algo todo lo que vas a hacer. El objetivo es que la única manera de que se escapen sea cumpliendo este meme:

![Desktop View](/assets/img/phishingops/read.jpg)

### 5.1 Diseño del correo phishing con Gophish
Una vez hecho el perfilado diseñar el correo es mucho más sencillo. Os recomiendo el repositorio [**gophish training templates**](https://github.com/HailBytes/gophish-training-templates) que quizás os puede dar ideas aunque la mejor opción siempre es revisar tu bandeja de SPAM donde vas a tener multitud de ejemplos que podrás usar. También usar [**Phishing pot**](https://github.com/rf-peixoto/phishing_pot) donde tendreís multitud de ejemplos de phishings que quizás os pueden ayudar. 

Algunos consejos en el diseño que os ayudarán a mejorar el OPSEC:
- Usad nombres de departamentos reales. Haced un poco de OSINT y podreís conseguir este tipo de info, Linkedin puede que te ayude. Usar el nombre correcto de un departamento puede ser clave entre que cuele o no. 
- Obtener una firma legitima de correo. Buscando leaks de correos en internet podeís conseguir las tipicas firmas o el mensaje de legal que aparece abajo en los correos lo cual le dará cierta credibilidad a tu correo (ya que quien lo vea estará acostumbrado a que aparezca).
- Las compañías tienen lenguaje propio... cada uno el suyo... Tonterías como correos muy formales o todo lo contrario dependiendo de la compañía pueden ser clave. Piensa en como recibirá el objetivo el correo.
- Tened en cuenta el idioma... Es super claro cuando encuentras correos con trazas en inglés cuando los equipos están todos basados en España o cosas así. En el propio cuerpo de las plantillas se suelen dejar rastros.
- En algunas plantillas se usan fuentes que no son las tipicas, sobre todo de los correos de notificación... Tened en cuenta que si usaís algo que el usuario es la primera vez que ve probablemente no se lo crea.

![Desktop View](/assets/img/phishingops/9.png)

Realmente es crear el Sending Profile en Gophish con el servicio SMTP que desees, elaborar la plantilla y configurar la campaña para que se envie desde un correo que sea útil. Si envias un correo desde dbkjNKJ@sitiocomprometido.com7wdhwoid.com claramente irá a la papelera, pero si le pones el nombre de un departamento y haces que sea creible puede que cuele. Existe mucha documentación e incluso consejos en la [**documentación oficial**](https://docs.getgophish.com/user-guide/).

![Desktop View](/assets/img/phishingops/nocampaing.png)

### 5.2 Diseño del panel phishing con Evilginx
En este punto tenemos que elaborar el panel de phishing con [**Evilginx**](https://evilginx.com/) y guiandonos un poco por la [**documentación oficial**](https://help.evilginx.com/community). 

> Tienes que tener en cuenta que los requerimientos si es un servicio local para hacer pruebas, o si es algo que estás haciendo en debug los caminos son diferentes en la instalación. 
{: .prompt-warning }

Puede ser un poco desesperante la primera vez pero cuando realizas 3 o 4 pruebas tendrás enseguida Evilginx desplegado. Y luego es bastante sencillo de usar. <br>
Vereís que una de las cosas que es más importante dentro de Evilginx (dentro de las facilidades de configuración que da) es tener buenos Phishlets (Que vienen a ser los portales de phishing.) Encontratreís muchos por internet (cuidado con lo que montaís no la vayaís a estar liando, revisar siempre)  pero os recomiendo un repo que a mi me ha ayudado que es [**Evilginx2-Phishlets**](https://github.com/An0nUD4Y/Evilginx2-Phishlets) de An0nUD4Y. <br>

En mi caso usé una de las phishlets de o365 que vienen en el repositorio y quedó algo bastante chulo para el objetivo que tenía. 
![Desktop View](/assets/img/phishingops/10.png)


## 6. Evasión y mejoras de OPSEC

### 6.1 Cuestiones generales
- Aprovechate de la existencia de algún [**Subdomain Takeover**](https://c0nfig17.com/posts/Subdomain-takeover/) para poder publicar contenidos dentro del propio subdominio del target.
- Revisa si puedes abusar de algún problema de configuración [**SPF, DKIM o DMARK**](https://c0nfig17.com/posts/Email-Spoofing-DKIM,-SPF,-DMARC-&-Fails/) de cara a spoofear algún correo real.
- Trata de utilizar IPs estáticas de cara a evitar detecciones por  [**Dynamic User List (DUL)**](https://docs.trendmicro.com/en-us/documentation/article/email-reputation-services-service-central-glossary-dial-up-use). Puedes tratar de registrar el servicio para que no salte este control (sobre todo si quieres usar Postman directamente y ahorrarte gastar más). Mi manera de saltarme este control ha sido usar un servicio de SMTP con reputación alta como [**Sendgrid**](https://www.mailersend.com/), por eso lo recomendé al principio. En mi caso enviar con Postman correos a un gmail personal no ha sido problema, pero al usarlo contra un correo corporativo enseguida ha saltado el control de DUL.
- Evita reutilizar elementos completamente. Reutilizar phishlets o plantillas de phishing es super común y tiende a ser fácil de localizar.
- Compra dominios de buena reputación.
- Es obvio pero si simplemente vas a usar esto para ver cuanta gente cae y mejorar la formación a algunos perfiles deberias pensar en no complicarte y permitir los correos entrantes de una determinada dirección sin mucho control y así ir más rápido. Si estás en medio de un ejercicio de Red Team toca saltarselo todo!

### 6.2 Evadiendo la monitorización sobre Gophish
Hay un repositorio bastante recomendado (y con razón) para tratar de ocultar un poco tu servidor de gophish llamado [**Sneaky gophish**](https://github.com/puzzlepeaches/sneaky_gophish). Como verás desde el primer momento recibirás mucho tráfico entrante e intentos de monitorización de tu servidor los cuales (dependiendo de la operación que estés llevando a cabo) pueden ser un fastidio ya que pueden levantar alarmas o complicarte todo más de la cuenta. Te recomiendo mucho este artículo de [**sprocketsecurity**](https://www.sprocketsecurity.com/blog/never-had-a-bad-day-phishing-how-to-set-up-gophish-to-evade-security-controls) donde te dará consejos y mejoras bastante útiles para el despliegue de Gophish.

### 6.3 Evasiones y mejoras con Evilginx
Lo primero os recomiendo leer el repositorio de [**Evilginx2-Phishlets**](https://github.com/An0nUD4Y/Evilginx2-Phishlets) de An0nUD4Y profundamente ya que incluye multitud de consejos muy útiles y trucos que son necesarios desde un punto de partida. He encontrado este otro repositorio que quizás puede seros de ayuda también [**Evilginx-Phishing-Infra-Setup**](https://github.com/An0nUD4Y/Evilginx-Phishing-Infra-Setup?tab=readme-ov-file#evilginx-installation-scripts). Vamos con otras recomendaciones:

- El analista Keanu Nys reportó en [**insights.spotit.be**](https://insights.spotit.be/2024/03/01/flaw-in-evilginx-allows-instant-detection-by-scanners-despite-blacklisting/) un método de detección de Evilginx el cual el propio Breakdev reportó en su [**blog**](https://breakdev.org/evilginx-3-3-go-phish/) y ayudaba a escaneres masivos a detectar portales de phishing.
- Verás a lo que me refiero según levantes tu primer phishlet.... vas a recibir muchas peticiones de enumeración según obtengas el certificado TLS. Lo verás super claro porque no darán al Path y la propia herramienta lo reportará. Los escanerés hacen su trabajo, haz el tuyo y evita que masivamente se detecte. Si despliegas los mismos certificados y de la misma manera que todos será sencillo detectarte.
- Crea tus propios [**Phishlets**](https://help.evilginx.com/community/phishlet-format) para evitar ser detectado. Así también podrás hacerlos más creibles y actualizados (si usas phishlets que hay por ahí ya sabes que es fácil que te detecten.)
- Ten en cuenta que puedes crear tu propia [**blacklist**](https://help.evilginx.com/community/guides/blacklist) y no esperar a que esta vaya surgiendo según te lanzan peticiones masivas. Puedes buscar IPs de escaneres masivos en algún repo. Se me ocurre el desplegar un par de phishlets de diferentes maneras durante un par de semanas como hoenypot, obtener los rangos de IPs y meterlos en la blacklist de tu servicio real para evitar la detección. 
- Crea un [**path custom**](https://help.evilginx.com/community/guides/lures) en tu lure con el objetivo de que sea más creible. Por ejemplo agenciatributaria.com7g-alleta.xyz/superconfiable/7/recursos/muyseguro/documento.pdf . Recuerda que algunas extensiones pueden ser bloqueadas bajo algunas configuraciones de navegadores.
- Usa bien la [**documentación**](https://help.evilginx.com/community/guides/lures) y ten en cuenta que puedes filtrar por user-agent si vas contra un objetivo en especifico. Apaga y esconde los phishlets o pausalos cuando no las uses. Exponerlo 24/7 no tiene sentido y solo va a dar lugar a mejorar la capacidad de respuesta del SOC o de un equipo de Threath Hunting.


## 7. Operación
Ya "solo" queda ponerse en marcha. Tienes una lista de usuarios perfilados, un correo bien diseñado y un phishlet desplegado esperando. Tienes todo para funcionar en este momento cumpliendo el siguiente proceso.

![Desktop View](/assets/img/phishingops/11.png)

Nada, en este punto solo toca enviarlo y esperar 

![Desktop View](/assets/img/phishingops/12.png)

Puedes ir recibiendo las notificaciones de Gophish en su campaña y de Evilginx con el comando sessions. Además puedes echar un vistazo al repositorio [**AddTelegramBotToGophish**](https://github.com/glennzw/AddTelegramBotToGophish) para crear un bot de telegram que te notifique de cada credencial que vayas capturando.


## 8. Recursos adicionales
El creador de Evilginx tiene un curso llamado [**Evilginx Mastery**](https://academy.breakdev.org/evilginx-mastery) en el que creo que podreís profundizar en muchos más elementos. He podido echar un vistazo al curso de [**Practical Phishng Campaigns de TCM Security**](https://academy.tcm-sec.com/p/practical-phishing-campaigns) y aunque no le he podido dedicar mucho rato parece bastante completo. Y ya que me pongo echadle un vistazo al curso de phishing de [**Maldev Academy**](https://maldevacademy.com/phishing-course) que la verdad es el que más profundiza (Almenos aparentemente por los contenidos). <br>
Por si te ayuda tengo una [**lista**](https://github.com/stars/c0nfig-17/lists/phishing) en mis estrellas de Github donde voy metiendo recursos de phishing que voy encontrando en github, puede que haya algún recurso que te sea de ayuda. Hay varios articulos en este blog que tratan temas relaccionados con el Phishing o que quizás te puedan servir, espero que sean de ayuda!. <br>


Dicho todo esto, ve preparando recursos de formación para los usuarios si no quieres acabar así.... 
![Desktop View](/assets/img/phishingops/questions.jpeg)

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

