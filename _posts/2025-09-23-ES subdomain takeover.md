---
title: Subdomain takeover
date: 2025-09-23
categories: [domain]
tags: [domain, subdomain, takeover]     # Los tags deben estar siempre en minúsculas.

---

## 1. Teoría sobre Subdomain Takeover
La idea es bastante sencilla, vamos a aprovecharnos de un problema de gestión y mantenimiento que viene de un mal diseño de muchas herramientas. <br>
Vamos al principio. Cuando contratamos un dominio podemos poner todos los subdominoos que queramos y cada un apunta a lo que queramos. Es muy común en grandes empresas tener no solo muchos dominios sino también muchos subdominios, por lo que las infrastructuras se hacen dificiles de sostener. De manera general la representación sería algo así, cada subdominio apunta a un servicio diferente.

![Desktop View](assets/img/subdomain/subdomain1.png)

El problema por el que se origina la posibilidad de realizar un subdomain takeover es el momento en el que uno de los subdominios que tenemos apunta a un servicio el cual ya no está activo. En ese momento tenemos un subdominio (en este caso el subdominio2) el cual sigue apuntando a galletas.github.io. Esto normalmente se da porque quien gestiona el servicio pidió dar de alta un subdominio pero no se acordó o no planificó el que esa redirección debe retirarse.

![Desktop View](assets/img/subdomain/subdomain2.png)

En este punto es donde surje la vulnerabilidad. El subdominio sigue apuntando a ese CNAME por lo que si consigues colocarte donde esta úbicada la redirección podrás publicar un contenido que use el subdominio que es vulnerable de manera sencilla y sin gran conocimiento técnico. 

![Desktop View](assets/img/subdomain/subdomain3.png)


## 2. ¿Quien ha originado el problema? ¿como lo gestionamos?
La pregunta es la misma que la del huevo o la gallina. Si no tienes un proceso bien asentado para gestionar altas y bajas así como monitorizar estos problemas serán recurrentes. Para el alta de servicios suelen existir procesos más asentados que para retirar servicios y se generan estos problemas. ¿Quien ha generado el problema? Pues probablemente la ausencia de un proceso, la cual ha sido agrabada porque quien retiró el servicio no se preocupó de esto así como el equipo que gestiona los dominios no tiene capacidad de monitorizar estos problemas. <br>

Si tienes una infrastructura grande es muy posible que no tengas 1 subdominio así, sino muchísimos los cuales "impone" eliminar porque "no sabes si alguien los usa". Normalmente lo que "no se sabe de quien es" pertenece a quien quiera darle uso. 


## 3. Problemas e impacto
Muchas veces es dificil visibilizar el impacto que tiene esta vulnerabilidad, por lo que voy a plantear algunas preguntas:
- Si alguien utiliza tu subdominio para ejecutar ataques de phishing ¿que harías? Es tu dominio, es un servicio aparentemente legitimo.
- Si alguien utiliza tu subdominio para exfiltrar información en tu infra o la de un tercero ¿Dificultará o no que tu equipo de seguridad detecte que es malicioso?
- Si alguien utiliza tu subdominio para distribuir malware a terceros o tu propia infra ¿como sabrás diferenciarlo de un uso legitimo?

<br>
El control de activos tiende a ser una de las principales dificultades de los equipos de ciber y esta vuln nace en concreto de una gestión de activos deficiente que si bien no es la puerta de entrada es una llave muy valiosa para un atacante. 


## 4. Descubrimiento de subdominios vulnerables
Para descubrir subdominios que sean vulnerables puedes usar herramientas como [**DNS Reaper**](https://github.com/punk-security/dnsReaper) o [**Subzy**](https://github.com/PentestPad/subzy) los cuales te permiten comprobar discrepancias en los subdominios. <br>

Dentro de esta técnica existen multitud de servicios y algunos son más fáciles de vulnerar y otros requieren o del abandono de una cuenta o de un compromiso a esas cuentas. En ese sentido existe un repositorio el cual es [**Can I takeover XYZ?**](https://github.com/EdOverflow/can-i-take-over-xyz) y te podrá guiar un poco. Es verdad que recomiendo asegurarte mucho de que servicio y si existe información en este repo o por otro lado que no haya sido incluido en el repo, ya que la existir miles de servicios y el propio paso del tiempo es dificil en ocasiones encontrar recursos aunque si has entendido el concepto será sencillo de explotar. 

![Desktop View](assets/img/subdomain/lugia.jpg)



**Te recomiendo algunos recursos para que puedas profundizar:**
[**Punk Security**](https://punksecurity.co.uk/dnsreaper/)
[**Hackerone a guide to subdomain takeovers 2.0**](https://www.hackerone.com/blog/guide-subdomain-takeovers-20)




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
