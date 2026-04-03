---
title: Evilginx OPSEC 1
description: Trataremos algunas consideraciones sobre OPSEC y evasión en phishing con Evilginx. 
date: 2026-04-03
categories: [Email]
tags: [opsec, Evasion, email, Ofuscación, passwords]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/evilginx1/logo.png

---
## 1. Evilginx y el robo de tokens
Hace ya unos meses motivado por una PoC que estaba desarrollando escribí [**Phishing OPS para Red Teaming**](https://c0nfig17.com/posts/Phishing-OPS/) donde contaba un poco el despliegue de un GoPhish y Evilginx para el robo de credenciales y tokens para accesos iniciales. En este caso quiero profundizar en la parte de despliegue de [**Evilginx**](https://github.com/kgretzky/evilginx2/tree/master/) y captura de tokens. Para esto voy a dar algunas cosas por supuestas que se veían en el post anterior así como no voy a hacer despliegue alguno de Gophish o su integración con Evilginx ya que simplemente no me quiero centrar en ello. Vamos a tratar algunas cuestiones sobre Evilginx y su despliegue, OPSEC, monitorización y como evitar que nuestro phishing sea detectado a los 5 minutos. <br>

![Desktop View](/assets/img/evilginx1/1.jpeg)

Antes de todo, creo que es importante por la actualidad del tema, dejaros este artículo de Kuba Gretzky el creador de Evilginx que creo que es importante. Todo lo que vamos a hacer aquí es usando como base software libre,aunque exista una versión privativa, y creo que no está de más introducir un poco este tipo de cosas dado que al sector de la seguridad creo que nos va a afectar enormemente, echadle un vistazo! [**Open Source is Dying**](https://breakdev.org/open-source-is-dying/) .


## 2. Despliegue rápido
Dicho esto, tras las 8 millones de instalaciones que he tenido que hacer de Evilginx he elaborado un script ya que preveo varios millones de instalaciones más los próximos meses. Si teneís la versión pro de Evilginx teneís otro método de instalación completamente diferente al mio y que os recomiendo usar, yo simplemente he hecho algo sencillo para ubuntu que a mi me funciona. Lo dejo por aquí por si alguien le es de interés, pero no es nada revolucionario, de hecho es una lista de todas las cosas que hacía antes de instalarlo pasado con Claude [**Easy_Evil**](https://github.com/c0nfig-17/easy_evil) . 

![Desktop View](/assets/img/evilginx1/2.jpeg)

## 3. Consideraciones iniciales
Antes de empezar con detalles creo que es importante hacer una lista con algunas cuestiones que es importante tener en cuenta como tal sobre las actividades que vamos a replicar. Vamos a desplegar infra en internet por lo que automáticamente miles de personas van a monitorizarnos como vimos en [**HoneyPots y escáneres masivos**](https://c0nfig17.com/posts/Honeypots-y-esc%C3%A1neres-masivos/) . Es importante no solo desplegar la operación sino aparentar que la misma lo más normal posible de cara a evitar que nuestro phishing se vaya al traste. Estamos haciendo Red Teaming por lo que no vamos a intentar hacer cosas masivas a las que caerá "alguien random". Es por ello que existen cosas que tenemos que tener en cuenta como la reputación de la IP y el dominio sobre el cual desplegamos todo o incluso el sistema operativo que aparenta tener. Cuanto más real parezca o sea todo mejor, de hecho en phishing reales he visto como atacantes usaban servicios comprometidos a terceros en sharepoint por ejemplo para usarlos como redirector al panel de phishing. En resumen, mi idea aquí no es un tutoríal de como desplegar Evilginx, sino enseñar las soluciones que he encontrado a problemas con los que ya me he chocado. En resumen:

![Desktop View](/assets/img/evilginx1/charmander.jpeg)

Según voy escribiendo esto me doy cuenta que probablemente existan puntos que tengamos que ver en un segundo artículo, ya que son bastantes los puntos a tocar. 

## 4. Certificados
Como puedes ver en la propia [**documentación**](https://help.evilginx.com/community/guides/config) existe una implementación con [**certmagic**](https://github.com/caddyserver/certmagic) que permite que obtengas certificados de [**Let´s Encrypt**](https://letsencrypt.org/) de manera sencilla y gratuita. Esta implementación puede ser genial para tu laboratorio pero verás que tiene algunos problemas en la vida real. <br>

![Desktop View](/assets/img/evilginx1/3.png)

Como puedes ver existen algunos limitantes pero básicamente no tienes que hacer gran cosa. <br>

![Desktop View](/assets/img/evilginx1/4.png)

Llega el momento, ponemos nuestro primer Phishlet, generamos el certificado al ponerlo enabled y ..... Mierda! No dejamos de recibir tráfico! <br>

![Desktop View](/assets/img/evilginx1/61.png)

Como podemos ver son basicamente escaneres másivos de xxxxxxxxxxxx <br>
Investigando un poco creo que he encontrado el motivo por el que esto se produce justo cuando generamos nuestro certificado TLS con Let´s Encrypt. El servicio está genial pero existe algo llamado [**Certificate Transparency (CT) Logs de Let´s Encrypt**](https://letsencrypt.org/docs/ct-logs/) la cual hace referencia a [**Certificate Transparency**](https://certificate.transparency.dev/howctworks/) . Es básicamente un mecanismo de "transparencia" que permite monitorizar la creación de certificados en las entidades que se adhieren a esto. Es decir, según creamos nuestro certificado en Let´s Encrypt y automáticamente subimos nuestro phishlet le estamos diciendo a todas las empresas que monitorizan esto "Ey estoy automatizando un proceso con un certificado gratuito, estoy aquí!". Lo cual nos viene francamente mal. <br>

![Desktop View](/assets/img/evilginx1/6.png)

Bueno nos ponemos al lio y levantamos el certificado... Locurita el tráfico que comenzamos a recibir. 

![Desktop View](/assets/img/evilginx1/62.png)
![Desktop View](/assets/img/evilginx1/63.png)
![Desktop View](/assets/img/evilginx1/64.png)

Y muchas muchas más peticiones... Os diría que esto para en algún momento, pero creo que es mejor que seaís conscientes del volumen de tráfico que vaís a recibir y que no va a parar. Por lo menos Evilginx te automatiza parte de la blacklist que verás más abajo. Quiero aclarar que el phishlet todavía ni siquiera funciona, simplemente he conseguido el certificado de Let´s Encrypt. <br>
Vamos a encontrarnos de todo. Por ejemplo yo me he encontrado la ip 91[.]92[.]241[.]197 la cual está asociada a [**hosting de malware**](https://www.socdefenders.ai/threats/eae29087-6e76-43ae-9972-735d0cafda6c) y a la que se pueden asignar actividades maliciosas en [**JoeSandbox**](https://www.joesandbox.com/analysis/1873341/0/html) y [**Any Run**](https://app.any.run/tasks/89dee3e0-c6bf-4afe-b9dc-868fd6a94e27) así que mucho cuidado. <br>

Mi intención era enseñaros algunas IPs de Palo Alto ya que la primera vez que hice este proceso me encontré muchas monitorizando pero no tengo evidencia por lo que os tendreís que fiar de mi! En resumen trata de utilizar certificados legit pero probablemente mejor de pago para llamar menos la atención. 

## 5. Blacklist
Como hemos visto mucha gente va a tratar de echar un vistazo a nuestro phishlet cuando a nosotros nos interesa que solo y exclusivamente lo vea quien queramos. Existen muchas maneras para hacer esto, podemos usar un firewall por ejemplo pero sobre todo podemos usar la [**funcionalidad blacklist**](https://help.evilginx.com/community/guides/blacklist) del propio Evilginx que nos permitirá bloquear peticiones de fisgones. Básicamente la activas con una de las siguientes opciones mediante ```blacklist <mode> ``` . <br>


| Modo   | Descripción |
| :------ | :----------- |
| all    | Bloquea y añade a la blacklist la IP de todas las peticiones (incluso las que apuntan a URLs válidas de lure). Útil si quieres escanear manualmente tus URLs con servicios online y forzar que las IPs de los escáneres queden bloqueadas. |
| unauth | Bloquea y añade a la blacklist la IP de cualquier petición que no apunte a una URL válida de lure de un phishlet habilitado. Es la configuración por defecto. |
| noadd  | Bloquea cualquier petición que no apunte a una URL válida de lure de un phishlet habilitado, pero no añade la IP a la blacklist. |
| off    | Bloquea peticiones no autorizadas, pero ignora el bloqueo de IPs que ya están en la blacklist. |


La blacklist se almacenará en ```/root/.evilginx/blacklist.txt``` con las IPs que detectes. Lo suyo sería tener una lista con los rangos e ips que vayas detectando que están monitorizando comportamientos maliciosos. Creo que es obvio, pero no voy a proporcionar un listado así, si sabes lo que haces sabrás generarlo. <br>

![Desktop View](/assets/img/evilginx1/7.jpeg)

Por si te es de ayuda yo he realizado una blacklilst exponiendo un evilginx real durante un tiempo y extrayendo blacklist de ips de mala reputación, con eso podrás quitarte algo de ruido de manera sencilla, en el script del principio incluyo esa blacklist inicial. <br>


## 6. Easter eggs
Algunos de los IOCs existentes en evilginx están creados a propósito por el creador. Si sabes buscar bien los encontrarás, ya que son necesarios para no enviar información innecesaria. Uno de los repositorios que me han ayudado a localizarlos (pensaba que era solo uno) es [**evilginx2-TTPs**](https://github.com/aalex954/evilginx2-TTPs) . Tendrás que ver como apañartelas para acabar con ello! No te puedo dar todo mascado, aunque si buscas bien ya te he dado herramientas para resolverlo.  <br>

Creo que una vez visto esto hemos avanzado bastante para un solo post. Más adelante publicaré otro con más arreglos a realizar! <br>

![Desktop View](/assets/img/evilginx1/8.jpeg)


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
