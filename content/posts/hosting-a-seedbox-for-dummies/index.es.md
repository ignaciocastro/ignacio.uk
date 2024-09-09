+++
title = 'Hosteando un seedbox a Seedbox para principiantes'
date = 2024-09-09T01:24:56-03:00
draft = false
showSummary = true
summary = 'Configurando un seedbox privado, desde cero'
+++

Con la ola de gente queriendo configurar sus propios seedboxes sin ser molestados por sus compañias de internet por descargar torrents, entra a la mente mantener un servidor privado para descargar. Esta guia tocará los pasos básicos, empezando en como usar la terminal en Windows, escogiendo un proveedor de VPS e instalando (y configurando) Swizzin.
## Parte 1: Configurando Windows
### Paso 1: Instalando un buen terminal

Usar el simbolo del sistema es antiguo, aburrido y feo. Powershell no es muy distinto, hora de actualizar. Para esto, usaremos Windows Terminal. Puedes descargarlo gratis desde la [Microsoft Store](https://apps.microsoft.com/detail/9n0dx20hk701) or [instalarlo con un método alternativo](https://github.com/microsoft/terminal#installing-and-running-windows-terminal). Cmder, Tabby son buenas alternativas, pero nos iremos por la opción más fácil.

#### ¿Por que no usar PuTTY, KiTTY o *insertar cliente SSH aquí*?
Si estas siguiendo esta guia, se tiene en consideración que no tienes conocimientos profundos en configurar un servidor de Linux. Para aprender, tendrás que aprender a trabajar en una terminal completa en vez de usar un programa que haga la primera parte por ti.

### Paso 2: Instalando Git for Windows
cmd apesta, Powershell apesta, Cygwin está estancado en la nada. Instalaremos Git for Windows, que incluye la emulación de BASH integrada, para ayudarte a familiarizarte antes con el bash de Linux.

Descarga la versión 64 bits de Git for Windows [aquí](https://git-scm.com/download/win). Al instalar, recuerda marcar la opción "(NEW!) Add a Git Bash Profile to Windows Terminal" en la parte _Select Components_. Para facilitarte la vida, en _Choosing the default editor used by Git_, desplázate hacia arriba en la lista y selecciona "nano". No es necesario cambiar nada más en el proceso de instalación.

Abre Windows Terminal y deberías ver una opción _Git_ cuando hagas clic en la flecha junto al botón de nueva pestaña. **Usarás esta opción a partir de este punto hasta el final del tutorial.** También puedes abrir la configuración (`CTRL + ,` o flecha hacia abajo > Configuración) y seleccionar _Git_ como perfil predeterminado. Recuerda hacer clic en **Guardar** al final. Cierra tu terminal y abre otra, ya estás listo.

## Parte 2: Obteniendo una clave SSH
Vamos a crear una clave que reemplazará el inicio de sesión con contraseña. Abre una nueva ventana de terminal y pega el siguiente comando, reemplazando _your-key-filename_ con el nombre que realmente deseas.

```
ssh-keygen -t ed25519 -f ~/.ssh/your-key-filename
```

Te pedirá una passphrase, puedes ingresar tu contraseña aquí (o mejor que tengas una [buena passphrase](https://xkcd.com/936/)) o dejarla en blanco (no recomendado) presionando ENTER dos veces.

Después de terminar, si revisas la carpeta .ssh (está en tu carpeta de usuario, es posible que debas habilitar "Ver carpetas ocultas"), verás dos archivos, uno con el nombre de tu clave y otro con el mismo nombre, pero con .pub al final. El primero es tu clave privada, **no la compartas nunca** con nadie más, tu proveedor de VPS nunca te la pedirá. El segundo es tu clave pública, esta es la que enviarás a tu servidor.

## Parte 3: Eligiendo un proveedor de servidor
Deberías elegir un proveedor de servidor que te proporcione:
- Una dirección IPv4 para ti (IPv4 compartida sirve, pero tendrás que configurar un dominio para ello)
- Tanto SSD como HDD, con espacio HDD de al menos 500GB (solo espacio HDD también está bien)
- Al menos 1GB de RAM
- Te permita sembrar desde trackers públicos / privados (es mejor abrir un ticket y preguntar directamente)
He estado usando el VPS de almacenamiento híbrido de Hostbrr durante casi un año, y funciona perfectamente para mi uso. Puedes revisar sus diversas ofertas [aquí](https://my.hostbrr.com/order/main/index/storage?a=NjEw) ([aquí, sin código de afiliado](https://my.hostbrr.com/order/main/index/storage)), o preguntar si su oferta personalizada aún está disponible (estoy usando esta):

```
2 vCore AMD EPYC 7502
30 GB NVMe
2 TB HDD
6 TB de ancho de banda @ 1Gbps
IPv4+IPv6
Helsinki, Finlandia
$5.5/mes
```

Como alternativa, ServaRica tiene su oferta de almacenamiento Polar Bear ([enlace, sin af](https://clients.servarica.com/store/black-friday-2023/polar-bear-storage-offer)) con:
```
2 núcleos compartidos
2GB RAM
2TB HDD
Ancho de banda ilimitado @ 250Mbps + 1Mbps de aumento diario, límite 1Gbps O
12TB de ancho de banda @ 1Gbps
IPv4 + IPv6 (a solicitud)
Montreal, Canadá
$5/mes
```

## Parte 4: Configurando y asegurando tu servidor
### Elegir un sistema operativo
Personalmente uso Ubuntu 22.04 para todo. Te recomiendo que hagas lo mismo.

### Asegurándote de poder iniciar sesión
En una nueva ventana de terminal, escribe lo siguiente, reemplazando *root* en caso de que tu host te dé un usuario diferente (como *ubuntu* o *debian* o *user*) y reemplazando *your_server_ip* con la dirección IP de tu servidor.

{{< alert "circle-info" >}}
**Importante** 22 es el puerto predeterminado para SSH. Si elegiste un servidor con IPv4 NAT (IP compartida), esto podría ser diferente, así que asegúrate de cambiar 22 a lo que se haya seleccionado para ti.
{{< /alert >}}

```console
ssh root@your_server_ip -p 22
```

Si te dice "La autenticidad del host \[...\] no se puede establecer", es seguro escribir "sí". Ahora deberías estar dentro de tu servidor.

### Creando un nuevo usuario y agregándolo a sudoers
Siempre es una buena idea moverse del usuario predeterminado a uno nuevo creado por ti. En tu ventana de terminal, pega lo siguiente, cambiando "*ignacio*" a cualquier usuario que desees.

{{< alert "circle-info" >}}
**Importante** Si estás en el usuario "root", obtendrás el error "*sudo: comando no encontrado*", recuerda eliminar "sudo" en los comandos de este paso.
{{< /alert >}}

```console
sudo adduser ignacio
```

Escribe tu (segura) contraseña dos veces y completa la información (o presiona enter varias veces para dejarla en blanco), luego escribe "Y" para aceptar. Ahora, necesitas agregar este nuevo usuario al grupo *sudo*, dándole permisos de administrador básicamente.
```console
sudo usermod -aG sudo ignacio
```

Para verificar si se agregó correctamente, ejecuta el siguiente comando:
```console
sudo groups ignacio
```

y deberías ver algo similar a esto: `ignacio : ignacio sudo`.

### Asegurándote de poder iniciar sesión como tu nuevo usuario
Envía el comando `exit` en tu terminal (o ciérralo y ábrelo de nuevo) y haz el mismo comando de inicio de sesión SSH, pero cambiando *root* por tu nombre de usuario recién creado.
```console
ssh ignacio@your_server_ip -p 22
```

Proporciona la contraseña y, si tienes éxito, deberías haber iniciado sesión nuevamente, esta vez con tu nuevo usuario. ¡Genial!

### Enviando tu clave pública a tu servidor
Iniciar sesión con una contraseña es antiguo y propenso a ataques de diccionario de contraseñas, enviaremos tu clave pública a tu servidor, como el primer paso para deshabilitar los inicios de sesión con contraseña por completo.

En una nueva ventana de terminal, ejecuta el siguiente comando. Recuerda reemplazar ignacio con tu usuario real.
```console
ssh-copy-id -i ~/.ssh/your-key-filename.pub -p 22 ignacio@your_server_ip
```

Te pedirá tu contraseña, proporciónala. Si tienes éxito, verás lo siguiente:
```bash
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ignacio@your_server_ip'"
and check to make sure that only the key(s) you wanted were added.
```

¡Genial! Ahora iniciemos sesión con tu archivo de clave.
```console
ssh ignacio@your_server_ip -p 22 -i ~/.ssh/your-key-filename
```

Proporciona la frase de paso / contraseña de tu archivo de clave (en caso de que hayas usado una) y deberías haber iniciado sesión en el servidor sin usar la contraseña real de tu usuario.

### Habilitando el firewall con UFW
Verifica el estado de UFW con `sudo ufw status`, debería responder con `Status: inactive`.
Antes de habilitar, asegúrate de no bloquearnos del acceso, por lo que necesitamos permitir que el puerto 22 (o tu puerto SSH) esté abierto.
```console
sudo ufw allow 22
```

La respuesta a eso debería ser `Rules updated` y `Rules updated (v6)`. Ahora podemos habilitar el firewall con:
```console
sudo ufw enable
```

Acepta la advertencia de interrupción de SSH escribiendo "y" y presionando enter. Ahora, `sudo ufw status` debería mostrar reglas de permiso para el puerto 22.

### Cambiando tu puerto SSH a uno aleatorio
Sí, acabamos de permitir el puerto 22 a través de nuestro firewall, pero cambiar el puerto a uno menos obvio reducirá los intentos de inicio de sesión por parte de bots a casi cero, y múltiples intentos fallidos pueden hacer que las máquinas con menos recursos sean más lentas.

{{< alert "circle-info" >}}
**Importante** Si estás en IPv4 NAT o tenías un puerto SSH diferente desde el principio, puedes omitir esta sección.
{{< /alert >}}

Elige un puerto aleatorio, para este ejemplo usaré el puerto 566. Permite el tráfico a través del puerto 566 en UFW:
```console
sudo ufw allow 566
```

Ahora cambia el puerto SSH a tu puerto elegido con esta línea, recuerda cambiar 566 a tu puerto preferido:
```console
sudo sed -i 's/#Port 22/Port 566/' /etc/ssh/sshd_config
```

Y reinicia el servicio SSH:
```console
sudo systemctl restart sshd
```

En caso de que quieras verificar, haz `nano /etc/ssh/sshd_config` y debería verse similar a esto:
```bash
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

Port 566
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
(...)
```

Abre una nueva ventana y verifica si puedes iniciar sesión con tu nuevo puerto en lugar del predeterminado:
```console
ssh ignacio@your_server_ip -p 566 -i ~/.ssh/your-key-filename
```

Si tuviste éxito, ahora puedes denegar el puerto SSH predeterminado con:
```console
sudo ufw deny 22
```

### Deshabilitando los inicios de sesión con contraseña y los inicios de sesión como root
Primero copia y pega este comando que deshabilitará el inicio de sesión como root:
```console
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
```

Luego copia y pega este comando que deshabilitará el inicio de sesión con una contraseña, permitiendo así solo el inicio de sesión con el archivo de clave correcto:
```console
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
```

Una vez más, reinicia el servicio SSH:
```console
sudo systemctl restart sshd
```

Ahora, el inicio de sesión como root está deshabilitado y solo podrás iniciar sesión con tu archivo de clave.

### Creando un alias SSH en tu dispositivo local
Tu comando de inicio de sesión SSH se está volviendo insanamente largo. Podemos configurar un archivo llamado "config" dentro de ~/.ssh en tu máquina, agregando todos esos comandos adicionales a un alias que lo hará mucho más fácil.

Crea un archivo en blanco en .ssh/ dentro de tu carpeta de usuario llamado "*config*", y pega lo siguiente, reemplazando "*seedbox*" con el alias que desees, "*ignacio*" con tu usuario elegido, *566* con el puerto SSH que elegiste y *"your-key-filename*" con el nombre de tu clave privada.
```console
Host *
    ServerAliveInterval 40

Host seedbox
  HostName your_server_ip
  User ignacio
  Port 566
  IdentityFile ~/.ssh/your-key-filename
```

Guárdalo, cierra tu terminal y abre una nueva. En lugar de hacer todo el comando SSH con tu usuario, IP, puerto y archivo de clave, simplemente haz:
```console
ssh seedbox
```

y habrás iniciado sesión.

### Habilitando fail2ban y configurándolo al extremo
fail2ban es un programa útil que mantendrá una lista de IP que fallaron al iniciar sesión. Puedes personalizarlo a tu gusto, desde cuántos intentos hasta cuántos minutos / horas de prohibición, etc. Dado que ya tenemos todo configurado para iniciar sesión con un comando simple, configuraremos fail2ban para:
- Prohibir permanentemente a cualquiera que intente iniciar sesión como root
- Prohibir permanentemente a cualquiera que no proporcione un archivo de clave
- Prohibir durante 24 horas a cualquiera que proporcione el archivo de clave incorrecto

Actualiza la lista de paquetes de tu sistema:
```console
sudo apt-get update
```

Actualiza cualquier paquete desactualizado. Si obtienes una pantalla rosa preguntándote qué servicios deseas reiniciar, simplemente presiona enter.
```console
sudo apt-get dist-upgrade
```

Instala fail2ban:
```console
sudo apt-get install fail2ban
```

Crea un nuevo archivo con nano llamado jail.local:
```console
sudo nano /etc/fail2ban/jail.local
```

Pega lo siguiente, recuerda cambiar 566 a tu puerto SSH real:
```console
[sshd]
enabled = true
port = 566
filter = sshd
logpath = /var/log/auth.log
maxretry = 1
bantime = -1

[sshd-root]
enabled = true
port = 566
filter = sshd[mode=aggressive]
logpath = /var/log/auth.log
maxretry = 1
bantime = -1

[sshd-nokeyfile]
enabled = true
port = 566
filter = sshd-nokeyfile
logpath = /var/log/auth.log
maxretry = 1
bantime = -1

[sshd-wrongkeyfile]
enabled = true
port = 566
filter = sshd-wrongkeyfile
logpath = /var/log/auth.log
maxretry = 1
bantime = 86400
```

Para guardar, presiona `CTRL + X`, presiona `Y` y luego presiona enter.
Crea el siguiente filtro haciendo `sudo nano /etc/fail2ban/filter.d/sshd-nokeyfile.conf` y pega el contenido:
```console
[Definition]
failregex = ^.*Did not receive identification string from <HOST>$
ignoreregex =
```

Guarda, y crea el último filtro haciendo `sudo nano /etc/fail2ban/filter.d/sshd-wrongkeyfile.conf` y pega lo siguiente:
```console
[Definition]
failregex = ^.*error: Authentication failed for .* from <HOST>$
ignoreregex =
```

Guarda, luego inicia y habilita fail2ban:
```console
sudo systemctl start fail2ban
```

```console
sudo systemctl enable fail2ban
```

Para verificar el estado de todas las cárceles (para verificar una cárcel individual, pon el nombre de la cárcel después de "status"):
```console
sudo fail2ban-client status
```

Para desbloquear una IP:
```console
sudo fail2ban-client unban <IP_ADDRESS>
```

### Habilitando actualizaciones automáticas
Instala estos paquetes:
```console
sudo apt-get install unattended-upgrades update-notifier-common
```

Habilita las actualizaciones automáticas (En la pantalla rosa, asegúrate de que "Yes" esté seleccionado y presiona enter):
```console
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Haz `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades` y asegúrate de que se vea así:
```shell
// Automatically upgrade packages from these (origin:archive) pairs
//
// Note that in Ubuntu security updates may pull in new dependencies
// from non-security sources (e.g. chromium). By allowing the release
// pocket these get automatically pulled in.
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
        // Extended Security Maintenance; doesn't necessarily exist for
        // every release and this system may not have it installed, but if
        // available, the policy for updates is such that unattended-upgrades
        // should also install from here by default.
        "${distro_id}ESMApps:${distro_codename}-apps-security";
        "${distro_id}ESM:${distro_codename}-infra-security";
//      "${distro_id}:${distro_codename}-updates";
//      "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
};
```

También, verifica `sudo nano /etc/apt/apt.conf.d/20auto-upgrades`, asegúrate de que tenga estas líneas:
```shell
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

Habilita y arranca el servicio:
```console
sudo systemctl enable unattended-upgrades
```

```console
sudo systemctl start unattended-upgrades
```

## Parte 5: ¿Ya podemos instalar Swizzin?
### Instalando Swizzin

Habilita el acceso a los puertos 80 (HTTP) y 443 (HTTPS):
```console
sudo ufw allow 80 && sudo ufw allow 443
```

Cambia a root:
```console
sudo -i
```

Ejecuta el asistente de instalación de Swizzin:
```console
bash <(curl -sL s5n.sh) && . ~/.bashrc
```

Presiona "Yes" cuando te diga que configures, luego ingresa tu nombre de usuario. Recibirás una notificación como esta:
```shell
INFO    The user ignacio already appears to be present on this machine; however, it does not appear to be configured as a swizzin user.
        The user will be added to swizzin.
INPUT   Continue setting up user?
(Y/n) >
```

Ingresa "Y" y presiona enter para confirmar. En la contraseña, reescribe la contraseña que ya configuraste para tu usuario.

En "Instalar software", selecciona lo siguiente (Mueve con las flechas arriba/abajo, selecciona con la barra espaciadora):
- nginx
- qBittorrent (También puedes seleccionar deluge o rtorrent, aunque)
- panel

Presiona la tecla TAB para seleccionar `<Ok>` y presiona ENTER.
En la pantalla "Make some more choices", desplázate hacia abajo y selecciona `letsencrypt`. Cualquier otra cosa puede instalarse más tarde. Nuevamente, TAB, selecciona `<Ok>` y presiona ENTER.

Selecciona tu versión preferida de qBittorrent. Elegiré 4.6 porque uso feeds RSS. TAB y continúa. Esto comenzará a instalar todas tus aplicaciones seleccionadas. En servidores con menos RAM, puede haber algunas partes donde parece que todo está congelado. **No te preocupes**, solo déjalo y déjalo cargar hasta que termine.

Mientras termina, continuemos con el siguiente paso.

### Configurando tu dominio con Cloudflare
Usaremos Cloudflare para esto. Los dominios son baratos el primer año, pero si no quieres gastar dinero, [Afraid's FreeDNS](https://freedns.afraid.org/), [DuckDNS](https://www.duckdns.org/), [deSEC](https://desec.io/) o [eu.org](https://nic.eu.org/) son alternativas gratuitas para obtener un subdominio.

Primero en Cloudflare: Selecciona tu dominio, ve a SSL/TLS > Overview > Configure y cambia Custom SSL/TLS a "Full".
Ahora, en DNS > Records > Add Record:
- Selecciona tipo `A`
- En el nombre, si deseas usar un subdominio (tee.yourdomain.com), entonces escribe "tee". Si deseas usar el dominio completo, solo pon `@`
- IPV4 Address: Tu IP de seedbox
- Proxy status: Déjalo como Proxied (Nube naranja)

Luego haz clic en "Save".

Finalmente, en la esquina superior derecha, selecciona el ícono de perfil > Mi perfil, selecciona API Tokens (O [haz clic aquí](https://dash.cloudflare.com/profile/api-tokens)), desplázate hacia abajo y haz clic en "View" en Global API Key, cópialo; lo usarás ahora.

### Configurando tu dominio en tu servidor
Regresa a tu terminal, deberías ver lo siguiente:
```shell
SUCCESS Panel installed
INFO    Installing letsencrypt
DOCS    Further reference: https://swizzin.ltd/applications/letsencrypt
INPUT   Enter domain name to secure with LE
```

Ingresa tu dominio (o subdominio), cuando se te pregunte *"Do you want to apply this certificate to your swizzin default conf?"* escribe "y". Lo mismo con *"Is your DNS managed by CloudFlare?"* y *"Does the record for this subdomain already exist?"*. Cuando se te pida tu clave API de CF, pega la clave api global que copiaste antes, luego ingresa tu correo electrónico de la cuenta de Cloudflare.

Finalmente, verás esto en tu pantalla:
```shell
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2297    0  2297    0     0   7098      0 --:--:-- --:--:-- --:--:--  7089
...     Performing apt update
        ✔   Done
...     Performing installation of 1 apt packages
        (socat)
        ✔   Apt install complete
...     Installing ACME script
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1032    0  1032    0     0   7709      0 --:--:-- --:--:-- --:--:--  7701
        ✔   Done
...     Registering certificates
        ✔   Certificate acquired
...     Installing certificate
[Sun  8 Sep 05:47:14 UTC 2024] The domain "tee.yourdomain.com" seems to already have an ECC cert, let\'s use it.
[Sun  8 Sep 05:47:14 UTC 2024] Installing CA to: /etc/nginx/ssl/tee.yourdomain.com/chain.pem
[Sun  8 Sep 05:47:14 UTC 2024] Installing key to: /etc/nginx/ssl/tee.yourdomain.com/key.pem
[Sun  8 Sep 05:47:14 UTC 2024] Installing full chain to: /etc/nginx/ssl/tee.yourdomain.com/fullchain.pem
[Sun  8 Sep 05:47:14 UTC 2024] Running reload cmd: systemctl reload nginx
[Sun  8 Sep 05:47:14 UTC 2024] Reload successful
        ✔   Certificate installed
SUCCESS Letsencrypt installed
INFO    Package installation took 129 minutes and 16 seconds


SUCCESS Swizzin installation complete!
INFO    Seedbox can be accessed at https://server.ip
INFO    You can now use the box command to manage swizzin features, e.g. `box install nginx panel`
DOCS    Further reference: https://swizzin.ltd/getting-started/box-basics
INFO    Shell completions for the box command have been installed. They will be applied the next time you log into a bash shell
...     Executing post-install commands
        ✔   Post-install commands finished
```

En tu navegador, abre tu dominio (o subdominio), inicia sesión y deberías ver el panel de Swizzin. Recuerda salir del usuario sudo simplemente escribiendo `exit` en tu terminal y presionando enter.

### Abriendo tu puerto de torrent
En tu panel, selecciona qBittorrent en la barra lateral izquierda (o ve directamente a `https://tee.yourdomain.com/qbittorrent`), inicia sesión con la misma información que antes. Haz clic en el ícono de configuración (ícono de engranaje), selecciona la pestaña "Connection" y copia el puerto listado en "Port used for incoming connections" (o reemplázalo por tu preferido). En mi caso, mi puerto es 2549.

Desplázate hacia abajo, haz clic en "Save" y en tu terminal haz:
```console
sudo ufw allow 2549
```

Verifica si tu puerto está abierto con cualquier verificador de puertos como [Dynu's Port Check](https://www.dynu.com/NetworkTools/PortCheck), espera hasta que aparezca el mensaje "SUCCESS!". ¡Ya terminaste! Ahora tienes un seedbox funcional disponible para ti.

### Configuraciones opcionales para qBittorrent

#### Deshabilitando límites de torrent
Ve al ícono de configuración > Connection y **deselecciona** lo siguiente:
- Global maximum number of connections
- Maximum number of connections per torrent
- Global maximum number of upload slots
- Maximum number of upload slots per torrent
#### Configurando tu seedbox para uso solo de tracker privado
Ve al ícono de configuración > Bittorrent y **deselecciona** lo siguiente:
- Enable DHT (decentralised network) to find more peers
- Enable Peer Exchange (PeX) to find more peers
- Enable Local Peer Discovery to find more peers}
### Mantenimiento general (Bueno hacer cada pocos días / cada semana)
#### Actualizando tus listas de paquetes
```console
sudo apt-get update
```

#### Actualizando paquetes desactualizados
```console
sudo apt-get dist-upgrade
```

#### Actualizando Swizzin a la última versión
```console
sudo box update
```