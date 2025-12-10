---
title: Review Certificación CRTO
description: Experiencia sobre la certificación CRTO
date: 2025-12-10
categories: [Formacion]
tags: [redteam, bypass, evasion, pentesting, opsec,]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/crto/0.jpeg

---
## 1. ¿Que es la CRTO?
La Certificación Red Team Operator de [**Zero Point Security**](https://www.zeropointsecurity.co.uk/) es una certificación centrada en los conocimientos necesarios para realizar ejercicios de Red Team. Aquí es importante diferenciar algo que comunmente se confunde y que el la certificación hace hincapié en el principio, nos centramos en red teaming, es decir en simulación y emulación de amenazas. La certificación está recomendada por la propia [**Fortra**](https://www.cobaltstrike.com/support/training?_gl=1*ar09np*_up*MQ..*_gs*MQ..&gclid=CjwKCAiAxc_JBhA2EiwAFVs7XFdfZHgldRjySaFCO-9_G-v7DYKPardAY-SiJ7IImWAbEKcFqrq7WBoC64EQAvD_BwE&gbraid=0AAAAAofeg-j1ZZzrzZdtMF2erOvVpfFM2) como partner formativo para profundizar en Cobalt Strike y durante toda la certificación harás un uso extensivo de Cobalt Strike constantemente. <br>
![Desktop View](/assets/img/crto/logo.png)

Es importante aclarar por mi parte que yo he realizado el curso y examen previo de 2025 antes de la gran actualización que ha realizado Zero Point Security, aunque estoy viendo la nueva versión del curso y tengo acceso a la CRTOII por lo que conozco los cambios. 

## 2. Red Teaming y profundización
Durante todo el curso vas a profundizar en tecnicas avanzadas pero sobre todo en el OPSEC y la forma de ejecución de las mismas. Te vas a forzar a superar la lógica del CTF y el pentesting acotado por lo que tienes que pasar a entender muchos más factores y la detección. Es muy importante entender que puedes conocer algunas técnicas pero que por mucho que ponga que el curso dura 20 horas.... ni en broma.... tienes que dominarlas, si no lo dominas no superarás el examen. Son cosas lógicas las que vas a ir aprendiendo y en muchos casos son cosas que si trabajas en esto ya sabes, pero si no sabes que es [**MotW**](https://c0nfig17.com/posts/Funcionamiento-y-evasi%C3%B3n-de-MotW/) palmarás al segundo uno por ejemplo por que es un indicador demasiado sencillo. <br>
![Desktop View](/assets/img/crto/cobalt.png)
En general durante el curso darás sentido a muchas técnicas y la correlacción entre ellas. Vas a dejar de hacer cosas aisladas si vienes del mundo del CTF para centrarte a un entorno de AD con las complicaciones que tiene la detección como tal. <br>
He de decir que el curso si te motiva no es dificil, de hecho es el curso (de los muchos que he hecho) que con diferencia creo que he aprendido más y se me ha hecho más ameno y práctico todo, pero hay que echarle ganas. Las cosas fallan y no hay una respuesta sencilla sino que probablemente sean muchos factores la causa de ese fallo si no has realizado las cosas bien. Desde la nueva versión los laboratorios son mucho más parciales y eso está bien y el contenido vale cada euro, pero eso hablarémos más adelante. <br>
Durante el curso tendrás el servidor de Discord donde puedes hacer preguntas y el propio creador es bastante accesible. En general todo el mundo es amable y te echa un cable, además muy probablemente si sabes buscar alguien ha tenido ya el mismo problema que tú y lo ha resuelto por lo que es bastante útil. 


## 3. Examen
Rasta Mouse (el creador) es explicito, claro y meridiano: Si haces el laboratorio haces el examen y puedo dar fé de ello. En mi caso fue sobre el examen antiguo y he de decir que la experiencia fue genial. Yo no tuve problemas con el laboratorio durante el examen de 2025 pero de igual manera tienes bastante margen de maniobra y lo gente de Zero Point es bastante accesible. Por explicar como ayudan yo me presenté a la CRTO y fue el apagón de 2025 durante el examen.... obviamente fue un agobio grande y fueron muy comprensivos y tuve un código para volverme a presentar. <br>
El propio curso te explica todo con bastante detalle y no hay nada de contenido fuera del examen que entre de manera colateral, lo que estudias es lo que hay. Ahora, tienes que dominar el curso. Está muy bien hecho para saber que dominas el contenido, de manera que antes de presentarte al examen el propio curso te recomienda hacer toda la parte práctica con las evasiones pertinentes ya que al examen vas con MS Defender activo. <br>
Como consejo para el examen, fuera de los generales de cualquier examen de este tipo, es prepararte para funcionar de manera rápida con Cobalt Strike. Debes de tener preparadas algunas evasiones para ir más rápido. Además te recomiendo tener dominadas las persistencias ya que no solo agilizaran tu examen sino que te permitirán poder descansar tranquilo (Cosa que en otros examenes no pasa) ya que cuando vuelvas del descanso tus Beacons seguiran activos. En mi caso yo perfeccioné una pequeña estrategia con todo lo que iba a necesitar, por ejemplo las persistencias las elaboré siempre de la misma manera y las dejé preparadas en un cheetsheet con el que pudiese funcionar ágilmente según entrase a los activos. Otro truco por mi lado fue que me preparé recursos en C# con los que pudiese realizar acciones directamente en memoria mediante Cobalt Strike sin tener que pasar por Powershell ya que personalmente los bypasses de Amsi me daban problemas en los laboratorios e iba más seguro. Es por eso que muchas de las herramientas que recomendaba Rasta las preparaba en c# y cargaba para usar lo primero. Pese a que el examen no tenga reporte es importante tomar nota de todo para seguir el ejercicio, sino será un kaos, ve mentalizado de tomar notas (yo en mi caso suelo hacer esquemas de la infra en papel para hacerme un mapa mental de los activos y asentarlo mejor. ) Una cosa de lógica en estos examenes no es solo que no te van a preguntar por lo que no te han enseñado, sino que tampoco te van a preguntar por lo que no puedes hacer.... el examen te da el entorno y las herramientas para resolverlo, si faltan tus skills no lo resolverás pero si además faltan herramientas debería indicar algo....  

## 4. Precio y conclusión
Solo diré que esta parte, y sobre todo con la update que han hecho, es genial. Calidad precio es espectacular y supera en ambas a otras certificaciones y cursos del mercado de lejos. El curso y examen cuesta 399 libras peeeeeeeeero tiene una política de precios adaptados por paises lo cual hace que tengas un 30% de descuento si por ejemplo vives en España.<br>
![Desktop View](/assets/img/crto/1.png) <br>
Ahora el problema es que con el Iva vuelve a subir un poco el precio.... <br>
![Desktop View](/assets/img/crto/2.png)
Ten en cuenta que en Black Friday y algunos dias concretos caen descuentos o que existe el Bundle de CRTO y CRTOII que puede ser interesante también. <br>
Con todo esto las condiciones generales de los nuevos cursos son las siguientes:
- Intentos de examen ilimitados.
- Bloqueo de 7 días del examen después de la compra para prevenir abusos.
- Sin fecha de expiración del laboratorio.
- Una compra incluye curso, examen y laboratorios; sin microtransacciones ni compras por separado.
- 3 usos de cada laboratorio por laboratorio cada 24 horas para CRTO / 1 uso de cada laboratorio por laboratorio cada 24 horas para CRTO II.
- Cool-down de 7 días para el examen (el período de enfriamiento comienza cuando inicias el examen).
- Soporte solo en horario laboral entre el lunes a las 09:00 y el jueves a las 16:00 hora del Reino Unido (excluyendo festivos).
<br>

Creo que es obvio que las condiciones son buenisimas en comparación con otros cursos del mercado con contenidos similares o inferiores.... entre el precio, intentos de examen ilimitados, contenido de por vida (y que se va actualizando) y los laboratorios... la verdad que me parece dificil tener algo calidad precio mejor. Yo personalmente estoy bastante contento tras realizarla, creo que sé más (cuantitativamente y cualitativamente hablando) y tengo mi [**certificación CRTO**](https://badges.parchment.eu/public/assertions/p5pFRJ94TDaTixY-0eMmdA) . 

![Desktop View](/assets/img/crto/cert.png)

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


