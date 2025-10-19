---
title: OPSEC Tricks en delivery
description: ¿Que es el OPSEC? ¿Para que lo utilizamos? ¿De que viene? ¿Es una herramienta?
date: 2025-06-15
categories: [opsec]
tags: [opsec, malware]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/tips/opsec.jpg
---


## 1. Purple Dragon: OPSEC y el motivo de este post
La diferencia entre que las cosas funcionen y ser un n00b en este campo no suele devenir de la suerte de conocer una técnica, sino ser capaz de hacerla funcionar. No liarla antes de empezar a liarla suele ser fundamental. <br>
Yo mismo he podido experimentar cuando "esa técnica inflaible" o "eso que había probado mil veces" deja de funcionar donde debería y no se ni como ni porqué. Siendo una sola persona haciendo un ejercicio de Pentesting que esto pase ya es complicado ¿donde he metido la pata? pero en equipo, o una empresa, ¿donde de esta maraña de procesos habremos metido la pata?.... Es una pregunta tremendamente dificil de responder. En muchas ocasiones se puede deber a que no somos conscientes de lo que hacemos, como lo hacemos y de que medida lo tenemos implementado en nuestra operación diaria. Aquí es donde viene el motivo de este post, el OPSEC. El OPSEC es un termino acuñado para referirse a la Seguridad Operacional, lo cual es un termino acuñado por la NSA para hacer referencia, en una explicación un poco vaga, a toda la inteligencia sobre la información crítica que manejamos que puede ser utilizada para atacarnos y hacer que nuestra operación falle. El OPSEC viene a solucionar ese problema tratando de poner foco a la operación y como trabajamos en ella. Pese a que el uso del término sea popular en el campo de la ciberseguridad tiene un hito fundacional recogido por la NSA en su documento [`Purple Dragon: The Origin and Development of the United States OPSEC Program`](https://www.nsa.gov/portals/75/documents/news-features/declassified-documents/cryptologic-histories/purple_dragon.pdf) el cual es público y accesible como documento desclasificado. El documento nos explica cuales fueron los motivos del desarrollo e implantación de OPSEC en las operaciones de los EEUU durante la guerra de Vietnam. <br>
![Desktop View](/assets/img/tips/opsec1.jpg) <br>

Esta metodología se fundamenta en 5 pasos que deberiamos implementar en nuestras operaciones:
1. Identificar la información crítica
2. Analizar las amenazas
3. Analizar la vulnerabilidad
4. Evaluar el riesgo
5. Aplicar Contramedidas

Si bien estos puntos son complejos de diseñar y abordar tienen que estar en el centro de las operaciones de los Red Team en cada uno de los ejercicios que realizamos para conseguir funcionar realmente como un atacante. En ocasiones se usa el OPSEC exclusivamente como referencia a un buen uso operacional de la información, pero el el campo del Red Team es imprescindible entender que esta parte es central. Vamos a poner algunos ejemplos.

| **Paso** | **Transformaciones** |
| ---- | ---- |
| **Identificar la información crítica** | ¿Tienes en tu payload información sobre el destino verdad? ¿Se te puede indentificar? ¿Tienes patrones en tu forma de trabajo que ayuden a un analista? |
| **Analizar las amenazas** | Te vas a enfrentar a un analista que tendrá acceso a mucha información ¿Como ha llegado ese payload? ¿El EDR saltará? ¿Me pillará el IPS? ¿Tendrán una política de ZeroTrust por la que no pasaré? |
| **Analizar la vulnerabilidad** | En los CTFs está muy bien, pero tener el payload en claro y pa´ alante es la diferencia entre poder realizar un ataque o no. O que este sea frustrado en minutos y vuelta a empezar. |
| **Evaluar el riesgo** | Si funciono sin estas medidas... Quizás me cargo un vector de ataque perfecto por no haber sido capaz de adaptar mi payload.  |
| **Aplicar Contramedidas** | Quizás localizar desde donde, como, en que momento y el que voy a desplegar. Haciendo enfasis en como retrocedería yo mis pasos puede ser fundamental.  |

En resumen: La vas a liar si no sabes pasar de un CTF y un entorno controlado a la vida real. Te vas a chocar con controles que no esperas y algunos que todavía no conocias. La enumeración pasa de ser un NMAP a generar una alerta desde un punto del mundo para saber que WAF hay. Tendrás que hacer OSINT para ver si la empresa que auditas tiene un precioso post en linkedin de una marca de EDR o si en la página de una solución de ZeroTrust sale como caso de éxito quien estás auditando. <br>
Esto no es un puto juego. <br>
¿o si? <br>
Tampoco soy nadie para decirte que es un juego, pero creo que si que tengo algunos truquillos para ayudarte que me ayudaron a mi en el delivery de mis herramientas. Esto es solo una parte del OPSEC que vas a implementar, pero he tenido que pasar por la Guerra de Vietnam para explicartelo, por lo que... Go ahead.

![Desktop View](/assets/img/tips/forest.jpg) <br>


## 2. Cadena de suministro e infratructura
Desde luego era un vector en auge, pero desde los ataques de Israél a Hezbolá esto ha pasado a otro plano y se ha vuelto una verdadera preocupación. Me refiero al ataque en el que la inteligencia Israelí consiguió, no se sabe todavía muy bien como, interceptar decenas de walkie-talkies del proveedor de Hezbolá, introdudir explosivos y hacerlos explotar de manera síncrona. Multitud de medios como [`Wired`](https://es.wired.com/articulos/decenas-walkie-talkies-hezbola-libano) el [`NY Times`](https://www.nytimes.com/2024/09/22/opinion/israel-pager-attacks-supply-chain.html) por señalar solo algunos se hicieron eco de este ataque que sin duda pone en el centro el compromiso de las cadenas de suministro de todo el mundo.  <br>
Si bien técnicamente esto es muy común en nuestro campo, existiendo multitud de ataques que de vienen de otros en la cadena de suministros, es probablemente la resiliencia a estos uno de los más dificiles de verificar ya que se parte del compromiso de un tercero al cual se puede no estar auditando. Puede ser interesante a día de hoy valorar, dentro de los vectores que haya que validar con el White Team, la posibilidad de comprometer a terceros los cuales se encuentren en un grado de sincronía con el Target muy fuerte. Por poner un ejemplo, si estoy en pleno ejercicio con CocaCola quizás puedo pedirles permiso para comprometer un vector de Fanta y ampliar el alcance para verificar otros controles... Desde luego si los ejercicios de Red Team suelen traer quebraderos de cabeza el no poder verificar esto puede ser uno de ellos en los que hay que trabajar. 

> Por dar algo de luz sobre este tema, y no ser super negativos, existen avances relevantes en notificación a terceros y sanciones en base al cumplimiento del Esquema Nacional de Seguridad (ENS), así como la pertenencia a la Red Nacional de Soc (RNS) que permiten agilizar el alertad. Si me compromenten, quien forma parte de la red será notificado y ayudará a que no me utilicen para comprometer a terceros.
{: .prompt-tip }

## 3. Lo propio y lo ajeno: Abusa de una confianza ya existente
El punto de partida de este post es que no vamos a usar un dominio que sea jKNKDBi.com para descargarnos un exe y que pretendamos hacerlo pasar por legitimo. Pero... ¿Y si conseguimos algo legitimo? Vamos a profundizar en algunos trucos:

### 3.1 Subdomain Takeover
¿Que tal si distribuimos malvare a c0nfig S.A. desde su propia infra? Si bien esta vulnerabilidad suele dejarse en un reporte informativo puede ser una gran ayuda en el momento que necesitas realizar movimientos dentro de la infrastructura del target. ¿O no estará c0nfig.com excepcionado desde dentro porque sino no funciona nada?.... todos sabemos la respuesta. <br>
El Subdomain Takeover nos permite robar un subdominio del target y publicar lo que queramos abusando de una mala gestión de su infrastructura. Revisar el repo [`can-i-take-over-xyz`](https://github.com/EdOverflow/can-i-take-over-xyz) puede sernos de gran ayuda, aunque existen miles de herramientas que podemos usar para ver si tenemos algún subdominio vulnerable y luego trabajar sobre la posibilidad de hacernos con él.

### 3.2 Dominios internos.... ¿fuera?
Es muy común usar dominios con nuestros propios DNSs de manera interna para poder desplegar software corporativo por ejemplo solo dentro de nuestra red. Estoy pensando en esa wiki, esa intranet... ¿Estarán excepcionados dentro no?. Es verdad que saldrás a una IP pública en internet en vez de a una IP en tu red, pero puede servir para ocultarte en algunos contextos. <br>
La duda es... ¿de donde saco esa información?... aquí algunos tips:
- Una buena enumeración en webs que realizan llamadas internas.
- Leaks y OSINT. ¿Algo filtrado en videos corporativos?
- Leaks de compromisos previos ¿algún grupo de ransom que haya publicado algo que nos ayude? 
- Metadatos... Es verdad que [`FOCA`](https://github.com/ElevenPaths/FOCA) está descontinuado ya, pero el volumen de información que podemos encontrar solo de ficheros nos puede ser de ayuda. Github o repositorios de código también nos pueden dar alguna pista. 

### 3.3 Las excepciones... benditas excepciones...
Todos sabemos que las compañías grandes tienen un nivel de excepcionado que es inversamente proporcional al volumen de revisiones que se hacen de estas excepciones... Alguien necesitó una vez una herramienta y se excepcionó hasta en el software que se usa para pedir la comida en el comedor (muy lógico), pero... ¿alguien controló esa excepción?. <br>
Sabemos que es muy habitual usar servicios reconocidos y respetables como Github o Google Drive para exfiltrar información o introducir información en compañías de manera ilegitima (y esto suele suponer un quebradoero de cabeza). Pero prefiero mostraros una manera muy ocurrente de gestionar las llamadas a un C2 que tuvo un atacante. En este reporte de [`The DFIR Report`](https://thedfirreport.com/2025/03/31/fake-zoom-ends-in-blacksuit-ransomware/) un atacante utiliza varias tecnicas de evasión, pero entre ellas oculta a que IP se deben realizar llamadas introduciendo en el nombre de usuario de Steam las IPs para parsearlas y asegurarse de que no tiene caidas y que en caso de tenerlas puede redirigir a nuevos servicios su malware.
![Desktop View](/assets/img/tips/steam.png) <br>
Tan original como terrorífico para el SOC.

## 4. Dominios confiables
Aveces lo sencillo funciona, y solo queremos que funcione. En ese sentido podemos trabajarnos unos dominios que nos permitan funcionar sin hacer mucho ruido. Creo que es útil seguir una lista de comprobaciones usando difernetes lógicas que nos permitan buscar lo que necesitamos. 

1. **Dominios caducados:** Si realizas una buena enumeración podrás encontrar dominios que se hayan dejado de usar por parte de la entidad en referencias en su código o webs. Si esos dominios están libres y puedes comprarlo probablemente te sean de utilidad.
2. **Dominios similares y Typosquatting:** Siempre puedes comprar un dominio similar. Si yo tengo c0nfig17.com quizás puedas usar cOnfig17.com . También puedes usar simplemente metodologías de Typosquatting para tratar de colar en un dominio con la letra "a" la letra cirilica "а" (Unicode U+0430). Es un truco un poco usado, por lo que tendrás que valorar los controles que te puedes encontrar o usar alguna herramienta (en local siempre) para ver si vas a hacer ruido. Es de sobra conocido que existen controles preventivos que detectan el alta de estos dominios, por lo que tienes que valorar si es peor el remedio que la enfermedad. Algo como [`dnstwist`](https://github.com/elceef/dnstwist) o [`urlcrazy`](https://github.com/urbanadventurer/urlcrazy) quizás te es de ayuda.
3. **Dominios confiables caducados de terceros:** Quizás te encuentras un dominio con buena reputación que ha caducado y no lo han renovado. Puedes buscar un poco por internet enlaces caidos y estudiar con WayBackMachine si alguna vez sufrieron un deface o si parecian legitimos. Las firmas sobre el dominio serán las mismas aunque el contenido haya cambiado, como es lógico, lo cual te puede ayudar para pasar inadvertido.
4. **Caracteristicas de los dominios:** Alomejor estás interesado/a en valorar elphishing como vector en este punto o la correcta implantación de SPF y DKIM para descargar el malware. En tal caso te dejo este otro [`post`](https://c0nfig-17.github.io/posts/ES-Email-Spoofing-DKIM,-SPF,-DMARC-&-Fails/) que quizás te es de ayuda.


> Es algo polemico, ya que no se puede usar para todo, pero ten en cuenta que en muchas ocasiones existirán "dominios defensivos" los cuales se compran para evitar que alguien los use o por proteger una posible marca en desarrollo. Puede ser interesante ir por esta parte.
{: .prompt-tip }



## 5. Una buena enumeración dentro vale por dos.
Es un cliché dentro del sector. Enumera, enumera y enumera. Si no sabes que hacer enumera. Si estás en un callejón "sin salida" enumera. Si no sabes hacer algo probablemente es solo que no has enumerado bien. <br>
Abandonando la broma el estudio de la infrastructura es clave para realizar movimientos. La diferencia entre un ejercicio exitoso y varias semanas de frustración suele estar ahí. Pongo dos causísticas concretas que alomejor pueden ayudar a salir del esquema habitual de un CTF:
- Imagina que te encuentras en un entorno donde descubres que hay muchos procesos c0nfig.ps1 los cuales se ejecutan en algunas máquinas y en otras no. ¿Porque? Estudialo, quizás hay algo de información con la que no has dado. Alomejor no es un proceso que puedas comprometer, pero entre ejecutar "IUBIUnjGSK.exe" en el escritorio de un usuario y ejecutar un fichero malicioso "c0nfig.ps1" que has creado en una ruta similar a donde suele estar instalado puede suponer que no salte alerta por alguna excepción o simplemente que no sea tenido en cuenta porque ese script está desperdigado por toda la infra y el SOC no sabe todavía porque.
- En entornos grandes la monitorización de sistemas puede ser una manera de conocer por parte de un atacante la infra. Además suelen ser procesos que generan mucho ruido. Usa la imaginación.
- Los usuarios de tareas para ejecutar scripcillos y cosas que nadie sabe que existen son una tónica general. Hubo un momento en el que no existía la ciberseguridad y en la que crear usuarios que ejecutan cosas random en equipos (de manera local o en dominio) era la norma. Enumera la estructura, los privilegios, la forma de funcionamiento... te podrán ayudar a pasar desapercibido.


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

