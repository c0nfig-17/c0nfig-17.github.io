---
title: Evasión de Antivirus: 1.Introducción a los métodos de detección
description: Comenzamos una serie de artículos sobre métodos de detección, analisís y evasión de malware. 
date: 2026-02-11
categories: [evasion]
tags: [evasion, opsec, Formacion, redteam, malware, ofuscación]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/malware1/malware1.png
---

## 0. De que va este post
Realmente no es "este post" sino "esta serie de posts". Mi idea es ir haciendo una introducción, poco a poco en pequeños artículos de como funcionan las herramientas de antimalware y como detectan comportamientos maliciosos. Iré poco a poco y el objetivo tampoco es exponer técnicas super enrevesadas o completas desde que tienes tu propio becon custom a tener una evasión de un EDR sobre todo porque eso no tiene sentido. Tratar de entender (y si estás en el lado ofensivo evadir) detecciones no es trivial o facilmente automatizable además de que es fácil quedarse anticuado antes las mejoras que hay para detectar malware. <br>
Quizás sea un poco pesado pero repito, el objetivo no es dar un paso a paso ni una solución necesaria a todos los problemas que se presentan desde el lado ofensivo, ya que además como veremos muchos de los" problemas" que se plantean a nivel ofensivo pueden quererse solucionar o no a propósito. <br>
Aclarar que tampoco se van a dar "soluciones maestras" a los problemas que se van a ir planteando, ni tampoco definiciones academicas, sino comportamientos reales y el aprendizaje que humildemente puedo tener sobre el campo y que puede (o no) ayudar a compañeros/as del campo (o eso espero). <br>
Una vez realizadas 80 aclaraciones sobre el tema, creo que se ha entendido el punto. 

![Desktop View](/assets/img/malware1/1.jpeg)


## 1. Que es esto del malware y los virus
Podría buscar una definición muy rebuscada, pero vamos a definirlo como "software que va en contra de tus intereses y por ello trata de ocultarse". Es la manera más sencilla de explicarlo porque también es la más real. Cuando hablamos de un malware solemos pensar en cosas loquisimas que hacen que una organización se vaya a pique o pensamos en ese "falso positivo" cuando nos descargamos photoshop pirata o un juego de ElAmigos (Que nos conocemos, insensatos). Realmente de lo que estamos hablando es de algo que no tiene porque ser muy elaborado o complejo pero simplemente nos hace daño y trata de pasar desapercibido. Para entendernos, un malware puede ser un [**ransomware como Lockbit**](https://latam.kaspersky.com/resource-center/threats/lockbit-ransomware) o puede ser un simple anydesk (herramienta legitima para administración de ordenadores) vitaminado con dos o tres cosillas como demostré en este [**post**](https://c0nfig17.com/posts/Dropper-de-AnyDesk-y-m%C3%A9todos-de-persistencia/) donde emulaba métodos de persistencia. También se puede categorizar como malware esa revershell que haces para saltar a una máquina en tu acceso inicial. <br>

En este apartado podría haber realizado una de las 40mil definiciones que existen dependiendo del impacto, pero hacía lo que nos interesa a nosotros el impacto queda un poco a un lado. Además todas esas definiciones son todas correctas (y a la vez algunas veces contradictorias) y no aportan en este punto, ya que el impacto a un activo puede variar mucho y no es el objetivo. <br>

De hecho como vengo de sistemas me permitiré citar una definición poco ortodoxa de lo que es un virus: <br>

_Causa por la que, después de varios desafortunados intentos de hacer algo por las bravas, el ordenador se queja incomprensiblemente. Ej: Mandar 1000 copias de un documento de 700 páginas a una impresora sin papel_ - [**Diccionario Luser Cristiano de Wardog**](https://mundowdg.com/blog/2007/09/02/diccionario-luser-cristiano/).


## 2. Que es es un antimalware/Antivirus y un EDR
Un antivirus es la herramienta que se nos ocurrió a los informáticos para parar un problema que habíamos generado nosotros mismos. Básicamente es un software que es capaz de realizar x tipo de comprobaciones (en las cuales profundizaremos) que permiten discernir que es bueno y que "parece ser" malo. Como hemos dicho en este campo la intención importa y diferenciarlo es dificil. Es por ello que los antivirus han ido volviendose más sofisticados y los atacantes han ido saltandose esas sofisticaciones. Parche tras parche y según todo se hacía más complejo aparecieron herramientas nuevas o complementarias al antivirus que funcionan como un conjunto del mismo para defenderte del malware, todo eso hasta llegar a la máquina definitiva: los Endpoind Detection & Response (EDR). Los EDR los podemos definir como un cacharro de cacharros más potente (y caro) que pasa a entender el malware de un dispositivo a un conjunto de dispositivos. Entraremos con mucho más detalle en adelante e iremos viendo diferentes controles. Es importante entender que no es solo un antivirus de empresa, tiene su módulo básico de antimalware pero tiene chorrecientasmil configuraciones y detalles más los cuales deben de estar operados por alguien, normalmente un SOC en 24/7 de la propia empresa o subcontratado, que se encarga de diferenciar entre el bien, el mal y lo que parece bueno. 

![Desktop View](/assets/img/malware1/2.jpg)

Es importante que entendamos en este punto dos cosas 1) Nos vamos a frustrar tratando de entender como bypasear los controles de un Antivirus y cuando lleguemos la EDR pensaremos que nada a valido la pena y 2) Nada en esta vida es irrompible, siempre tendrás una manera de saltarte un control. <br>

Un EDR pasa a controlar cosas que no controlabas con el antivirus porque ni tenias capacidad de gestión, ni interes, ni eras un activo lo suficientemente importante. Es un cacharro dificil de gestionar que no te puedes poner en casa. Lo comento antes de tiempo no vaya a ser que alguno piense que como es mucho más potente hay que instalarse alguna marca pública y usarlo en el ordenador de casa... Lo cual por poner un simil es como ir a la panadería de la esquina a por el pan en camión. <br>

Al final un EDR nos permite ver mucho más en capas diferentes, tomando acciones e investigaciones diferentes para objetivos variados. Tened en cuenta que en vuestro ordenador personal diferenciar entre el bien y el mal puede (o no) ser fácil, pero en un sitio donde trabajan 4000 personas la cosa se vuelve bastante diferente. 4000 ordenadores y sus correspondientes servidores, con diferentes herramientas, mantenimientos, exposiciones variadas.... y sus correspondientes [**luser**](https://es.wikipedia.org/wiki/Luser). Es decir algo muy dificil que gestionar que necesita de visibilidades diferentes. Creo que la siguiente imagen te puede ayudar a dar una imágen algo más visual al problema. 

![Desktop View](/assets/img/malware1/3.png)

Dicho todo esto. Muy buena la retaila de chistes malos pero ¿y como funciona un antivirus? ¿como me detecta? ¿Porque dice ese chico de youtube que apague mi antivirus antes de descargarme phothoshop de su enlace porque es un falso positivo?. Bueno, vamos a ello. 


## 3. Detección por hash
El método de detección primigenio utilizado para la detección de un virus es su hash. Un hash es una representación matemática inequivoca de un fichero en un momento y estado de la historia. Básicamente si nosotros cogemos cualquier fichero podremos calcular su hash con un algoritmo y obtendremos una cadena de caraceteres. Este algoritmo nos permite generar un cadena de caracteres que no es reversible, es decir que desde esa cadena o hash no podemos obtener el fichero. 

![Desktop View](/assets/img/malware1/4.png)

Esa cadena de caracteres nos permite identificar unicamente ese fichero. Pero es muy importante entender que si cambiamos ese fichero aunque sea un poco este hash cambiará. En este caso he creado un fichero muy sencillo y he calculado su SHA256, como puedes ver añadiendo un espacio y un punto el hash del fichero ha cambiado completamente. 

![Desktop View](/assets/img/malware1/5.png)

Verás que existen multitud de hashes, y en concreto se suelen usar mucho desde el punto de vista del analista SHA256, SHA-1 y MD5. Estos dos últimos se usan menos ya que tienen un mayor número de colisiones (explicado fácil es la posibilidad teórica de que el algoritmo represente dos ficheros diferentes con el mismo hash, lo cual suele ser remoto pero puede pasar sobre todo con algoritmos más antiguos) y están deprecados, aunque como con todo se siguen usando para la variedad de reglas que puedes tener y herramientas que los usan.

¿Parece igual de útil que sencillo de bypasear no? Bueno esto es así pero fue un avance importante y hoy día es una ayuda importante desde el punto de vista del analista. Ten en cuenta que no todo el malware es una cosa custom super compleja, sino que hay cosas muy mundanas y sobre todo repetidas ¿porque vas a tropezar dos veces en la misma piedra? Un hash nos permite determinar cual fue la piedra en la que caimos por última vez, categorizarla como algo malicioso y posteriormente compartirlo con el mundo. Eso evita que los ficheros masivos o que los ficheros ya conocidos puedan seguir difundiendose. El antivirus tiene una base de datos actualizada de hashes de tooodos los ordenadores que lo tienen instalado y todas las capacidades de inteligencia que tienen, por lo que lo comparan, si está en su base de datos pueden saber que esto es malicioso.

![Desktop View](/assets/img/malware1/6.png)

Este es el motivo por el que cuando descargas el phothosop pirata te piden quitar el antivirus. El antivirus ya sabe que tiene bicho porque ya lo analizó en el pasado y no te va a permitir que toque disco algo que sabe que es malo. <br>

Ahora, vale tu phothosp pirata es imposible que sea un virus y menos por ese motivo, por lo que te voy a dar un ejemplo práctico. Nos vamos a ir a [**Malware Bazaar**](https://bazaar.abuse.ch/sample/b70e78971fc44f8413447139afe777f19fd1b0d203f2d40f66f41cd6100a1c95/) y buscaremos un malware que queramos, en mi caso he buscado un ransomware que parece ser Conti el cual ha sido detectado y flagueado y del que te puedes descargar una muestra, la propia web ya te dice que es malicioso.

![Desktop View](/assets/img/malware1/7.png)

Vamos a irnos a Virus total y hacer una busqueda sencilla. Como podemos ver hemos podido consultar si un fichero es malware, aunque sabemos que si pusiesemos una palabra en el programa el hash cambiaría y dejaría de ser detectable con este método. 

![Desktop View](/assets/img/malware1/8.png)

Esta detección ya sabemos que es sencilla y si quieres instalar algo que no sabes lo que lleva allá tú, pero cuando el ordenador no es tuyo no puedes asumir el riesgo por otros. Entonces tanto la detección por firmas nos ayuda a detectar cosas ya conocidas como a simplemente trazar ficheros que tenemos dentro de nuestra infra para nuestras investigaciones y poder diferenciarlos de otros que no son iguales pero pueden parecerse. 

## 4. Firmado de software
Uno de los vectores comunes que se han usado en la historia es tratar de confundir al usuario y claro yo puedo comparar el hash de un fichero antes de descargarlo y después para asegurarme que no hay nadie manipulando que me descargo mediante un hash, pero ¿y si incluimos esa verificación directamente en el software?. Es decir, yo puedo diferenciar software legitimo y del que me fio de ilegitimo permitiendo que un tercero firme estos binarios con su propio certificado digital que solo poseen ellos. Por ejemplo si me descargo del sitio oficial de Spotify el ejecutable para windows puedo consultar facilmente el certificado y ver si es malo o no, o si dice ser spotify pero alguien ha alterado el fichero o no es el que dice ser. Es algo mucho más fácil que los hashes ya que almacenar un número infinito por cada versión y validar que son buenos es complicado.

![Desktop View](/assets/img/malware1/9.png)

Como podemos ver el ejecutable de spotify es marcado como no malicioso. Es un binario conocido, que fue analizado hace poco el cual tiene un certificado conocido.

![Desktop View](/assets/img/malware1/10.png)

Ahora podriamos entender que existen tres posibilidades en este aspecto:

- exe firmado reputado: Como spotify, en principio pasa.
- exe firmado no reputado: Yo puedo firmar ficheros también, si se necesita una firma yo la doy.
- exe sin firma: No tiene porque ser malo pese a que no este firmado.

El primer caso es evidente. Sobre el tercero creo que se puede poner el ejemplo conocido de 7Z el programa de compresión de ficheros mundialmente utilizado hasta hace poco no estab firmado, por lo que discernir que algo es bueno o malo solo por estar firmado tampoco es una garantiza 100% para discernir que algo es bueno o no. Además estamos hablando de ejecutables pero claro, no puedo estar firmando cada documento que pasa por mi ordenador... <br>

Existen varias maneras de jugar con los firmados de cara a mejorar la evasión de un fichero. Es muy conocido las noticias de ataques en las que se han robado firmas y posteriormente distribuido malware firmado como si fuese legitimo, además si buscas un poco puedes encontrar firmas públicas, aunque no vamos a dar indicaciones ya que son acciones ilegales que se estarían empezando a tomar dentro del proceso de evasión (y no necesariamente alineadas tampoco con un OPSEC en el depliegue de un ejercicio). Pero si entra la posibilidad de autofirmarnos. Vamos a hacer la prueba, primero generaremos un payload sencillo con msfvenom en el que basicamente ejecutamos la calculadora. 

![Desktop View](/assets/img/malware1/12.png)

Posteriormente lo analizamos con virustotal y podemos ver que se detecta por todos los motores de manera muy sencilla (sigo pensando que hay muchos que son fake).

![Desktop View](/assets/img/malware1/13.png)

Bueno ahora vamos a firmar nuestro propio binario con nuestro propio certificado para ver si algo cambia. El certificado como somos pobres lo vamos a generar nosotros mismos, aunque podrías comprar uno y que lo firme un tercero. Primero generamos el certificado

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365
openssl pkcs12 -inkey key.pem -in cert.pem -export -out sign.pfx 
```

Posteriormente firmamos nuestro msfvenom, podemos hacerlo con el propio sdk de microsoft que tiene una herramienta para ello o en linux usar ossslsigncode.

```bash
osslsigncode sign -pkcs12 sign.pfx -pass galleta -in calc.exe -out calc_signed.exe
```

Subimos el fichero a virustotal de nuevo y como vemos ha cambiado el hash del fichero. 

![Desktop View](/assets/img/malware1/14.png)

Bueno, hemos descubierto que un certificado autofirmado consigue bajar en 2 los antivirus que detectan que un payload muy conocido es malicioso. Desde luego menos da una piedra, pero ¿y si con solo eso y usando algo custom nos ayuda a reducir el número? Desde luego de la misma manera que no usar un certificado puede ayudar a desconfiar el usarlo pero que no sea de confianza puede levantar sospechas, además si detectan un payload firmado y no es una firma reputada se pueden realizar detecciones a partir de la firma, por lo que es una cuestión de ajustarnos a la necesidad que tenemos en un momento concreto y profundizar en los métodos de detección. 


![Desktop View](/assets/img/malware1/11.jpg)


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
