---
title: Notify -  Generar Webhooks para monitorizaciones masivas
description: Vemos un poco la herramienta Notify de project discovert
date: 2026-08-20
categories: [Automatización]
tags: [ad, domain, email, smb, redteam, passwords]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/notify/logo.jpg

---
## 1. De que va esto
Muchas veces me he encontrado la necesidad de realizar enumeraciones masivas y tener que dejarlas ejecutando durante mucho tiempo. Normalmente el ouput de este tipo de cosas suele ser muy dificil de trabajar y es dificil también estar enterado al momento cuanto tienes alertas de cosas que ha detectado un scanner o algo que has ejecutado. Personalmente llevo un tiempo buscando una solución que me sea útil para estos casos y he tratado muchas veces de hacer mis propias tools para notificarmen mediante webhooks. <br>

La situación modelo que necesitaba resolver es, enumero con un nuclei o lanzo una herramienta sobre un alcance que sé que por lo que sea voy a tener que esperar alomejor 10h en tener la salida. Y que además tengo que estar relativamente pendiente porque tengo que analizarlo a posteriori. Por eso la idea de tener webhooks me aparecía recurrentemente, ya que realmente lo que quiero es una notificación cuando encuentre algo. Ahí es donde aparece notify y soluciona mucho de los problemas que tenia.


## 2. Notify 
Me encontré con [**Notify**](https://github.com/projectdiscovery/notify) un poco por casualidad cotilleando proyectos de [**Project Discovery**](https://projectdiscovery.io/) y la verdad que desde lo he probado está metido en parte de mis procesos cuando tengo que analizar muchos activos o desplegar escanerés. <br>

La lógica de la herramienta es simple, le pasas el output de una herramienta y lo envia mediante el webhook que configures al servicio que configures. La forma en la que yo lo uso es la siguiente, simplemente tengo mi servidor desde el cual despliego un Nuclei por ejemplo a un alcance, tunelizo el ouput através de Notify y pido que en este caso me envie un webhook a Discord. Pongo el ejemplo de Discord pero puedes configurar lo que quieras: Teams, Slack, Telegram, SMTP.... la integración con Discord en concreto es muy muy sencilla y para proyectos personales es lo que he estado usando por esa facilidad. <br>

![Desktop View](/assets/img/notify/1.png)

Como se ve en el dibujo el uso es sencillo, abro diferentes sesiones con tmux. Ya sabes los comandos básicos de tmux vaya <br>

````bash
```
tmux list-sessions
tmux kill-session -t nombre_sesion
tmux new -s nombre_sesion
```
````

Posteriormente configuras tu provider config tal y como marca la docu. La puedes consultar en $HOME/.config/notify/config.yaml <br>

````bash
```
discord:
  - id: "crawl"
    discord_channel: "crawl"
    discord_username: "test"
    discord_format: "{{data}}"
    discord_webhook_url: "https://discord.com/api/webhooks/XXXXXXXX"
```
````

Después simplemente tienes que ejecutar lo que quieras imaginemos que tu idea es utilizar nuclei sobre un alcance determinado<br>

````bash
```
cat alcance.txt | nuclei -severity low,medium,high,critical | notify
```
````


> Tienes la flag -bulk para Notify. Dependiendo del uso que hagas puede ser útil o no. A mi me gusta que envie una notificación por hallazgo, pero claro, si derrepente llegan 500 notificaciones alomejor te da algo.
{: .prompt-warning }


En este caso recibirás una notificación de este estilo.<br>

![Desktop View](/assets/img/notify/2.png)

Como todo puedes personalizar más o menos, por ejemplo puede utilizar el ouput del proceso anterior como fichero y modificarlo. Ahi ya a gustos colores.<br>


## 3. Usos útiles
Aprovecho y detallo algunos usos que le estoy dando y me están aligerando mucho tiempo

### 3.1 Nuclei con desarrollo

Las herramientas de Project Discovery en general son bastante compatibles. En los ejemplos de Notify te dan este uso el cual puede ser de utilidad pero en mi opinión no tener ningún control sobre la enumeración previa al escaner y basarlo todo en subfinder.... no es lo que estoy buscando <br>

````bash
```
subfinder -d intigriti.com | httpx | nuclei -tags exposure -o output.txt; notify -bulk -data output.txt
```
````

En mi caso todo lo que sea relativo a sub dominios lo haré de otra manera pero bueno. Pongo un uso que si me ha parecido útil partiendo de la enumeración previa.

````bash
```
cat alcance.txt | httpx |nuclei -severity low,medium,high,critical -stats -H "X-Cabecera: Micabecera" -rl 5 | notify 
```
````

Como puedes ver el objetivo es leer el alcance que ya he estudiado. Después utilizo [**httpx**](https://github.com/projectdiscovery/httpx) para que Nuclei solo trabaje sobre los activos que respondan y así optimizar las consultas ya que pueden existir subdominios no activos. Aquí mi intención es que solo me traslade las vulnerabilidades low, medium, high y critical. Utilizo -stats para tener visibilidad durante la ejecución del estado. En caso de ser necesario pongo una cabezera custom y defino con -rl un máximo de request por seguro ya que es muy tipico no solo los bloqueos sino que por un rl muy alto no detectar algo. <br>

En general, ganar tiempo pudiendo hacer enumeraciones masivas en background es el uso que le doy. <br>

### 3.2 NetExec y Password Spraying con notify

Otra cosa super típica es tener que hacer password Spraying e invertir una barbaridad de tiempo en analizar o conectarme a revisar un output concreto.  <br>

````bash
```
nxc smb 192.168.1.101 -u user.txt -p password.txt --jitter 2 -t 50 --continue-on-success | grep '\[+\]' | notify
```
````

En este caso podría lanzar sobre un alcance grande un password spraying con 50 thereaths a la vez y un jitter en 2 de cara a poder lanzarlo durante mucho tiempo. Recibiría la notificación una vez acabe el proceso. <br>

### 3.3 Enumeración de Subdomain Takeover con notify

Otro caso en el que me ayuda Notify es con los [**Subdomain Takeover**](https://c0nfig17.com/posts/Subdomain-takeover/) . Por ejemplo con la herramienta [**Subzy**](https://github.com/PentestPad/subzy/) o con cualquier otra puedo realizar un reconocimiento de posibles subdomain takeover sobre un listado grande. Normalmente esto si son muchos activos puede llevar un tiempo en mi caso paso la notificación por notify. Como sé que esto suele tener muchisimos falsos postivios por subdomain takeovers que ya no estan activos utilizo -bulk para que cuando me traslade la información sea en bloques y yo la analice posteriormente. <br>

````bash
```
./subzy r --concurrency 30 --targets subdomains.txt --hide_fails | notify -bulk
```
````

En general como puedes ver algunas de mis aplicaciones serían algo así. Lo cual me facilita y agiliza algunos pasos y análisis simples. <br>

![Desktop View](/assets/img/notify/3.png)


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
