---
title: Honeypots y escáneres masivos
description: Vamos a jugar con honeypots en internet y analizar comportamientos maliciosos
date: 2025-10-20
categories: [Defence]
tags: [passwords, Honeypot, subdomain, takeover, opsec]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/honeypots/0.jpg

---


## 1. ¿Que es un Honeypot y para que sirve?
Un Honeypot es básicamente una trampa para detectar o relentizar un comportamiento malicioso. La expresión se refiere a un "tarro de miel" que atrae abejas y vaya simplemente es eso. El tema es que cuando hablamos de honeypots nos podemos referir a muchas cosas dependiendo del uso que le demos. Existen honeypots para casi cada cosa que se nos ocurra y ubicados en un sitio u otro, por lo que sus objetivos son muy diferentes. En mi opinión existen dos grandes grupos:
- Externos: Están expuestos a internet y pueden tener como objetivo monitorizar comportamientos maliciosos en internet en general o relaccionados con tu infra en concreto.
- Internos: Están dentro de tu red y suelen ser mucho más concretos y para herramientas y cuestiones concretas.

<br>
Hay otro termino que se suele confundir con Honeypot, y que no tiene necesariamente nada que ver, que es un Rabbit Hole. Un Rabbit Hole es basicamente una trámpa intencionalmente generada para que el atacante pierda tiempo una vez realizada una explotación o descubierta una gatera. Puede que un Honeypot tenga un Rabbit Hole o que generes uno o que simplemente esté.

## 2. Montamos nuestro propio Honeypot con T-Pot
Hay un proyecto que me gusta mucho para explicar lo que es un Honeypot que es el proyecto T-Pot de Telekom Security. En su [**repositorio oficial**](https://github.com/telekom-security/tpotce) teneís toda la información sobre como usarlo. Lo puedes montar en muchisimos sistemas, hasta en una Raspberry, y es basicamente un repositorio que mezcla una barbaridad de  honeypots de diferentes tipos y te genera una serie de herramientas de monitorización. En mi caso para enseñaroslo he montado un linode cumpliendo las specs que marca el repositorio. <br>

![Desktop View](/assets/img/honeypots/tpot.png)

> Importante! No es la primera vez que me tiran el honeypot por un DDOS provocado por gente tratando de explotarlo. Mucho cuidado con el autoescalado al exponerlo a internet que sino la factura estará chula.
{: .prompt-warning }

<br>
Ya lo he montado un par de veces y en todas he tenido problemas. Esta vez no me han tirado el servicio (tampoco lo he dejado muchos dias funcionando) pero como veís hay picos evidentes con muchísima carga y subiendo la CPU mucho. La caida que aparece es un reinicio creo realizado por mi, pero tened mucho cuidado con esto. Aclarar que es un servidor que cumple de sobra specs son 6 cores, 16GB y 320GBs... No es un tema de recursos.

![Desktop View](/assets/img/honeypots/red.png)

Para instalarlo es super sencillo y aparece muy detallado en el repo. Simplemente sigue las instrucciones del repro

````bash
```
env bash -c "$(curl -sL https://github.com/telekom-security/tpotce/raw/master/install.sh)"
```
````

![Desktop View](/assets/img/honeypots/install.png)

Alguna vez cuando lo instalas necesita un reinicio para levantar todos los servicios. Dale tiempo porque levanta muchas cosas y puede tardar un poco, pero vamos no tarda nada. Pones user y pass y ya tendrás el portal. Enseguida ya verás movimiento en los servicios y tendrás peticiones, te dejo una foro con apenas 5 minutos.

![Desktop View](/assets/img/honeypots/1.png)
![Desktop View](/assets/img/honeypots/2.png)


## 3. ¿Que trae T-Pot?
La respuesta corta es que un poco de todo como veis en la captura.
![Desktop View](/assets/img/honeypots/3.png)
Lo primero pues tenemos enlaces al readme, github y de manera adicional a [**sicherheitstacho**](https://www.sicherheitstacho.eu/#/en/tacho) que es un portal sobre alertas de honeypots en tiempo real donde cuenta un poco el proyecto y puedes ver un poco tráfico en directo. Como lo que hemos montado vaya. <br>

Ahora vamos a nuestro portal y vamos a ir cotilleando que tenemos. <br>
Lo primero de todo el attack map, en el que tenemos un mapa en tiempo real de peticiones por paises categorizando muchos servicios. Es curioso porque lo vereís en otro apartados pero si montas el Honeypot en España, Alemania, EEUU o donde sea y si es en diferentes servicios (un ovh, linode, AWS...) pues el resultado es diferente en algunas IPs. Es verdad que hay escaneres masivos que coinciden pero es curioso investigarlo. Todo lo que verás en este mapa son peticiones en tiempo real, la mayoría de ellas maliciosas y otras de escaneres de inteligencia, iremos más tarde a ello.
![Desktop View](/assets/img/honeypots/4.png)

Luego tendrás una instancia de CyberChef que te podrá ayudar a funcionar. Es el mismo que en el que usas [**online**](https://gchq.github.io/CyberChef/) del [**repositorio**](https://github.com/gchq/CyberChef) . También tendrás una instancia de [**ElasticVue**](https://elasticvue.com/) por si te fuese necesario dadas las configuraciones posibles de t-pot. También tendrás una instancia de [**SpiderFoot**](https://github.com/smicallef/spiderfoot) desplegada en tu servidor para poder realizar consultas OSINT sobre la información que estés recibiendo. <br>

¿24H de Honeypot y 226 detecciones de Elastic no está nada mal no? Este es el landing de Kibana
![Desktop View](/assets/img/honeypots/5.png)
![Desktop View](/assets/img/honeypots/6.png)
![Desktop View](/assets/img/honeypots/7.png)

## 4. ¿Que trae T-Pot?
Ya con Kibana podemos sentarnos a analizar cada uno de los servicios que tenemos expuestos. Tenemos muchisimos Honeypots y logicamente dependiendo de los scaneres y el tiempo tendremos resultados diferentes. Podemos sacar mucha información pero tened en cuenta que dejarle tiempo para que lso escaneres masivos que se lanzan en momentos puntuales toquen y podais sacar esa info puede ser interesante. Por ejemplo si alguien se dedica a escanear servicios SSH todos los martes y pones el honeypot un jueves pues bueno, no lo pillarás. Ya depende un poco del uso que le quieras dar al honeypot y que objetivos tienes. Puede ser útil para bloquear escaneres masivos vaya.
El honeypot de [**SentryPeer**](https://github.com/SentryPeer) es el que más detecciones ha tenido en mi caso.

![Desktop View](/assets/img/honeypots/8.png)
En concreto como veis hay 2 IPs de OVH que están realizando peticiones masivamente.
![Desktop View](/assets/img/honeypots/9.png)

El de [**HoneyTrap**](https://github.com/honeytrap/honeytrap) también ha realizado bastantes detecciones.
![Desktop View](/assets/img/honeypots/10.png)


## 5. Que chulo, ¿y ahora que?
El honeypot te ayuda a obtener datos magro, lo suyo cuando tengas datos suficientes es analizar la información. Te planteo algunas cosas para que puedas hacer tu propia investigación:
- ¿Las IPs están en Virus Total como maliciosas? ¿Las que tienen poco tráfico?
- ¿Las credenciales te dan pistas de a donde quieren llegar? ¿Te pueden ayudar a contribuir a tus diccionarios? Si alguien las prueba masivamente será porque funciona.
- ¿Que barbaridad de proveedores de infra no?
- ¿Y esos CVEs que aparecensiendo explotados?
- ¿Ha salido alguna vuln relaccionada con un servicio que tienes durante esta semana? ¿ha aumentado el tráfico a ese servicio?

Tienes herramientas de sobra para investigar. Como curiosidad cuando presenté esto en una clase dimos con un software de una empresa africana de distribución de petróleo. Una empresa aparentemente legitima que se dedica a enviar masivamente tráfico y explotar algún que otro CVE, por lo que era evidente que les habian comprometido. 


## 6. ¿Usos ofensivos?
La investigación siempre ayuda para luego explotar cosas, no descartes esto por estar enfocado en otro ala. Entiende los datos que se muestran, porque estos actores funcionan así, que buscan, que explotan, de que manera lo hacen.... Muchas veces tratar de reproducir comportamientos maliciosos abstraidos del comportamiento y centrados en simplemente resolver problemas como si de un CTF se tratase quita mucha de la creatividad que aporta el campo. Si estás leyendo esto y no te estás planteando en levantar un honeypot para llevarte el exploit de alguien pa ti y entenderlo... es evidente que puedes hacer muchas cosas con algo así si le echas tiempo y ganas, esta gente no está escaneando masivamente servicios SSH en internet por casualidad. <br>



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

