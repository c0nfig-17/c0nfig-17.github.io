---
title: Técnicas de hacking en MS Teams
description: En este post vamos a profundizar en técnicas de hacking que rodean a MS Teams
date: 2025-11-15
categories: [AD]
tags: [ad, domain, email, opsec, takeover]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/teams/teams.png

---
## 1. ¿Hacking en MS Teams?
Cada cierto tiempo sale una noticia o un video por ahí sobre una manera en la que se ha usado MS Teams para realizar alguna acción maliciosa o que se ha descubierto alguna manera de abusar de Teams para obtener información. Normalmente no se es muy transparente con este tipo de cosas, por lo que he invertido un tiempo en investigar que se está realizando hoy día y algunos recursos para realizar ejercicios de red team en los que se pueda emular estas capabilities de atacantes reales. La pregunta es ¿Que capabilities? bueno pues un poco de todo, como siempre... Tenemos phishing dirigido a empleados utilizando Teams como ha reportado [**Incibe**](https://www.incibe.es/empresas/avisos/phishing-traves-de-microsoft-teams-que-descarga-malware) en alguna ocasión, aunque también puedes encontrar Teams implicado en la distribución de Ransomware como reporta la propia Microsoft en este [**research sobre la distribución por Storm-1811 de Black Basta mediante MS Teams y Quick Assit de MS**](https://www.microsoft.com/en-us/security/blog/2024/05/15/threat-actors-misusing-quick-assist-in-social-engineering-attacks-leading-to-ransomware/) o como ha reportado [**Sophos**](https://news.sophos.com/en-us/2025/01/21/sophos-mdr-tracks-two-ransomware-campaigns-using-email-bombing-microsoft-teams-vishing/) mediante vishing. <br>
Como decía cada x tiempo aparece algo relaccionado con Teams pero no existe mucha documentación centralizada para lo importante que es la herramienta. Recuerda que la mayoría de organizaciones trabajan activamente con Microsoft y tienen todo su software desplegado de mejor o peor manera, independientemente de si hacen un uso activo o no de él (me he encontrado muchos sitios que lo tienen pero no lo usan porque prefieren comunicarse con otra cosa o por la facilidad que sea). Existen muchas posibilidades de que un vector de entrada pueda ser Teams con un phishing, pero también muchas otras cosas ¿no? Al final está conectado con O365 (muy conectado con Sharepoint, Drive y el resto de la suite) y los usuarios comparten un volumen de información muy considerable en MS Teams. 


## 2. MS Teams Attack Matrix 
Como punto inicial para comprender un poco los puntos en los que vamos a entrar más adelante creo que existe una recomendación necesaria, el blog [**cyberdom.blog**](https://cyberdom.blog/inside-the-microsoft-teams-attack-matrix-unpacking-the-the-frontier-in-collaboration-threats/) de Elli Shlomo supone una base importante para entender las diferentes acciones que podemos realizar como atacantes sobre MS Teams. El propio Elli genera la siguiente matriz de ataque con las diferentes acciones que se podrían llegar a realizar.

![Desktop View](/assets/img/teams/1.png)

En el blog proporciona bastantes vias, sobre todo de enumeración, bastante interesantes para ejecutar acciones sobre el entorno de MS Teams.  


## 3. Phishing para Teams
El siguiente reporte de [**eye.security**](https://www.eye.security/blog/microsoft-teams-chat-the-rising-phishing-threat-and-how-to-stop-it) sobre la killchain de un loader de DarkGate nos da una idea de cual puede ser el método de distribución e impacto. Normalmente el correo no tiene los mismos filtros que teams y podemos como atacantes usar esa vía de entrada.
![Desktop View](/assets/img/teams/2.png)

Existen muchas maneras de enviar un phishing a través de MS Teams, a mi personlamnete me gusta el proyecto [**TeamsPhisher**](https://github.com/Octoberfest7/TeamsPhisher) el cual nos puede ayudar a implementarlo de manera sencilla. Tened en cuenta que normalmente encontrareís dos controles 1)El propio envio del mensaje puede ser "reportado" como anómalo por Microsoft, aunque si trabajas con proveedores y trabajadores externos es habitual que salte el mensaje y 2)Si que es bastante fácil el que exista una política en Teams aplicada sobre compartir ficheros a desde correos que no son del dominio... aquí te tocará originalidad y probablemente una URL maliciosa. En este caso juegas con la ventaja de que teams te permite introducir links en texto desde el editor, por lo que puedes mostrar una url que parezca real y luego por debajo que no lo sea. <br>

Revisando he encontrado un bypass sencillo al mensaje que indica que una persona externa a tu dominio te está escribiendo, es bastante sencillo os dejo la [**explicación aquí**](https://posts.inthecyber.com/leveraging-microsoft-teams-for-initial-access-42beb07f12c4) .


## 4. Webhook phishing para Teams
En este punto la verdad que tenemos mucha información gracias a la gente de [**Black Hills InfoSec**](https://www.blackhillsinfosec.com/). Una explicación clara, sencilla, paso a paso y con muchisimos recursos de como abusar de webhooks en MS Teams para enviar phishing a usuarios... Existen muchos sitios donde webhooks, conectores e integraciones están a la orden del día y el nivel de detalle que proporciona Matthew Eidelberg es genial. Os dejo todos los recursos:
- [**Post Wishing: Webhook Phishing in Teams**](https://www.blackhillsinfosec.com/wishing-webhook-phishing-in-teams/#chap6)
- [**Video y explicación en Youtube**](https://www.youtube.com/watch?v=kMMZrd9intI)
- [**Slides de la presentación**](https://www.blackhillsinfosec.com/wp-content/uploads/2024/03/SLIDES_Webhooks.pdf)


## 4. MS Teams como C2
Quizás has visto el [**Video de John Hammond**](https://www.youtube.com/watch?v=FqZIm6vP7XM) donde hace unos meses conseguia usar teams como un C2. Esto funciona basicamente gracias al proyecto [**convoC2**](https://github.com/cxnturi0n/convoC2) la idea al parcer nace del siguiente post [**“GIFShell” — Covert Attack Chain and C2 Utilizing Microsoft Teams GIFs**](https://medium.com/@bobbyrsec/gifshell-covert-attack-chain-and-c2-utilizing-microsoft-teams-gifs-1618c4e64ed7) y paece que con un poco de cariño puede ayudarte en un entorno en la que la salida a internet sea complicada. 

![Desktop View](/assets/img/teams/3.png)


## 5. Impersonación y robo de cookies + BOF para hacerlo
Hace apenas unas semanas algunos researchers de [**Checkpoint**](https://research.checkpoint.com/2025/microsoft-teams-impersonation-and-spoofing-vulnerabilities-exposed/) publicaban un artículo en el que exponian tanto el abuso de algunas vulnerabilidades que han estado presentes en Teams así como algunos comportamientos maliciosos basados en la impersonación de usuarios. <br>
Un mes antes ya tenemos algo de documentación técnica sobre como hacerlo mediante este [**artículo**](https://blog.randorisec.fr/ms-teams-access-tokens/) y el equipo de [**tierzerosecurity**](https://tierzerosecurity.co.nz/) ya ha redactado este [**post**](https://tierzerosecurity.co.nz/2025/11/03/teams-cookies-bof.html) sobre el desarrollo de un BOF para extraer las cookies directamente desde tu C2. Han creado el repositorio [**teams-cookies-bof**](https://github.com/TierZeroSecurity/teams-cookies-bof) en el que tienes el bof disponible para tu uso. 
Te puede ser de utilidad este recurso de [**mr.d0x**](https://mrd0x.com/stealing-tokens-from-office-applications/) sobre el robo de tokens en aplicaciones office.

![Desktop View](/assets/img/teams/4.png)


## 6. Técnicas adicionales
Para finalizar creo que es importante recomendar este post de [**mr.d0x**](https://mrd0x.com/microsoft-teams-abuse/) donde realiza algunos pasos adicionales o bypasses que pueden ser bastante útiles complementandolos con todo lo anterior o simplemente durante los propios ejercicios. 


## 7. Recursos recomendados para securizar MS Teams
- [**cyberdom**](https://cyberdom.blog/protect-microsoft-teams-with-cloud-app-security-2/) <br>
- [**Guia de seguridad MS Teams**](https://learn.microsoft.com/en-us/microsoftteams/teams-security-guide) <br>
- [**Políticas Zero Trust en Teams**](https://learn.microsoft.com/en-us/security/zero-trust/zero-trust-identity-device-access-policies-workloads#microsoft-teams-recommendations-for-zero-trust)

![Desktop View](/assets/img/teams/5.jpg){: width="972" height="589" }

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
