---
title: "Configuración de NAT"
date: 2026-04-04T10:00:00+01:00
description: "Primero voy a conectar las máquinas a un switch donde voy a conectar una nube NAT para tener salida a Internet y poder actualizar el sistema e instalar los paquetes que vamos a necesitar para el…"
menu:
  sidebar:
    name: "Configuración de NAT"
    identifier: par-configuracion-nat
    parent: par
    weight: 260404
hero: cover.png
tags: ["NAT", "SNAT", "DNAT", "DHCP", "systemd", "PostgreSQL", "MariaDB"]
categories: ["Prácticas"]
draft: false
---

> 📚 **1º ASIR · Planificación y Administración de Redes** · Tema 5: Nivel de red · Curso 2025/2026

## NAT en Linux

### Preparación escenario

Primero voy a conectar las máquinas a un switch donde voy a conectar una nube NAT para tener salida a Internet y poder actualizar el sistema e instalar los paquetes que vamos a necesitar para el escenario. Este proceso lo voy a repetir con todas las máquinas que vamos a necesitar.

![Captura 1](img/captura-001.png)

![Captura 2](img/captura-002.png)

![Captura 3](img/captura-003.png)

![Captura 4](img/captura-004.png)

![Captura 5](img/captura-005.png)

![Captura 6](img/captura-006.png)

![Captura 7](img/captura-007.png)

![Captura 8](img/captura-008.png)

![Captura 9](img/captura-009.png)

![Captura 10](img/captura-010.png)

![Captura 11](img/captura-011.png)

![Captura 12](img/captura-012.png)

#### Servidor PostgreSQL

<debian@debian>:~\$ sudo apt install postgresql postgresql-contrib -y

![Captura 13](img/captura-013.png)

![Captura 14](img/captura-014.png)

Una vez que lo hemos instalado voy a acceder como administrador y voy a crear un usuario y una base de datos con todos los privilegios.

![Captura 15](img/captura-015.png)

Voy a activar el servicio para que se arranque automáticamente al encender la máquina.

![Captura 16](img/captura-016.png)

Y voy a cambiar la configuración del fichero para poder entrar a los usuarios sin contraseña (no recomendado en entornos de producción).

![Captura 17](img/captura-017.png)

Una vez hechos los cambios vamos a reiniciar el servicio para que se apliquen los cambios.

![Captura 18](img/captura-018.png)

Accedemos con el usuario y vamos a crear un par de tablas en mi caso voy a usar el esquema SCOTT.

![Captura 19](img/captura-019.png)

Al insertar los datos y las tablas me dió este fallo y al consultar el diccionario de datos me di cuenta que no tenía los permisos suficientes para crear tablas. f: false.

![Captura 20](img/captura-020.png)

Entre como usuario administrador y le di los permisos para poder insertar los datos.

![Captura 21](img/captura-021.png)

Y si volvemos a consultar el diccionario ya tenemos los permisos suficientes.

![Captura 22](img/captura-022.png)

Aquí vemos las tablas ya creadas.

![Captura 23](img/captura-023.png)

#### Servidor ssh

![Captura 24](img/captura-024.png)

![Captura 25](img/captura-025.png)

![Captura 26](img/captura-026.png)

<debian@debian>:~\$ sudo apt install openssh-server -y

![Captura 27](img/captura-027.png)

![Captura 28](img/captura-028.png)

#### Servidor MariaDB

![Captura 29](img/captura-029.png)

![Captura 30](img/captura-030.png)

![Captura 31](img/captura-031.png)

<debian@debian>:~\$ sudo apt install mariadb-server -y

![Captura 32](img/captura-032.png)

Una vez instalado iniciamos el servicio y lo hacemos persistente en el arranque.

![Captura 33](img/captura-033.png)

![Captura 34](img/captura-034.png)

Ahora vamos a proceder con el script de configuración en el cuál vamos a poner una contraseña para root.

![Captura 35](img/captura-035.png)

Y también vamos a poder borrar usuarios anónimos, eliminar la base de datos de prueba y deshabilitar el inicio de sesión remoto de root.

![Captura 36](img/captura-036.png)

Tambiénm voy a borrar las bases de datos de test y después de esto voy a recargar los privilegios de las tablas.

![Captura 37](img/captura-037.png)

Una vez configurado todo voy a acceder comoo root y voy a crear un usuario y una base de datos para posteriormente hacer las pruebas del correcto funcionamiento de NAT.

![Captura 38](img/captura-038.png)

Una vez creado correctamente voy a probar que podemos acceder correctamente al usuario.

![Captura 39](img/captura-039.png)

Aquí vemos como las tablas están creadas con contenido (esto es desde el servidor, más tarde al configurar NAT lo realizaremos desde el cliente).

![Captura 40](img/captura-040.png)

#### Acceso remoto PostgreSQL

Tendremos que editar un fichero de configuración, la ruta dependerá de la versión que tengamos instalada, en mi caso es la siguiente:

<debian@debian>:~\$ sudo nano /etc/postgresql/15/main/postgresql.conf

![Captura 41](img/captura-041.png)

Y tendremos que editar la línea de listen address y yo en mi caso voy a poner un asterisco para permitir todas las direcciones IP. Luego de hacer los cambios en el fichero de configuración voy a reiniciar el servicio para que los cambios tengan efecto.

![Captura 42](img/captura-042.png)

Una vez que hemos editado este fichero con las direcciones que va a escuchar el servidor tenemos que indicar en el siguente fichero a que usuario en concreto vamos a permitir el acceso remoto y lo podemos hacer indicando una IP en concreto o cualquier dirección IP (no recomendado en entornos de producción). Y voy a usar md5 para que el acceso sea por contraseña.

debian@servidor2:~\$ sudo nano /etc/postgresql/15/main/pg_hba.conf

![Captura 43](img/captura-043.png)

#### Acceso remoto MariaDB

<debian@debian>:~\$ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

![Captura 44](img/captura-044.png)

Para MariaDB editamos el siguiente fichero cambiando el bind address e indicando 0.0.0.0 para permitir cualquier IP. Y una vez hecho los cambios reiniciamos el servicio.

Tras el reinicio del servicio voy a acceder como administrador a la base de datos para crear un usuario para la dirección IP remota y darle permisos sobre la base de datos que tenemosl (En este caso yo he permitido solo la red 103.0.0.1 que es la red que sale de casa por lo tanto toda petición que el servidor le llegue con otra IP origen la va a denegar. Para que el servidor reciba toda las peticiones sin importar la IP habría que poner en le creación del usuario lo siguiente:)

MariaDB \[(none)\]\> CREATE USER ‘nombreusuario’@’%’ IDENTIFIED BY ‘password’

![Captura 45](img/captura-045.png)

### Instalación clientes 

Para la configuración de los clientes simplemente voy a realizar lo mismo voy a conectarlos a un switch con una nube NAT y voy a actualizar el sistema e instalar los clientes necesarios (SSH, MariaDB, PostgreSQL y curl).

![Captura 46](img/captura-046.png)

![Captura 47](img/captura-047.png)

![Captura 48](img/captura-048.png)

![Captura 49](img/captura-049.png)

<debian@debian>:~\$ sudo apt install openssh-client

![Captura 50](img/captura-050.png)

<debian@debian>:~\$ sudo apt install mariadb-client

![Captura 51](img/captura-051.png)

Una vez instalado vamos a comprobar que se haya instalado correctamente.

![Captura 52](img/captura-052.png)

<debian@debian>:~\$ sudo apt install postgresql-client

![Captura 53](img/captura-053.png)

Y nuevamente una vez instalado comprobamos que se haya instalado correctamente.

![Captura 54](img/captura-054.png)

![Captura 55](img/captura-055.png)

![Captura 56](img/captura-056.png)

![Captura 57](img/captura-057.png)

### Servidor DHCP

<debian@debian>:~\$ sudo apt install isc-dhcp-server -y

![Captura 58](img/captura-058.png)

Al acabar la instalación saltara un código de error, esto ocurre ya que no hay una configuración valida en el servicio.

![Captura 59](img/captura-059.png)

### Direccionamiento

#### Router casa

Para las direcciones IP privadas voy a usar en todo momento la red 10.0.3.0/24 para comprobar que el NAT está haciendo su trabajo mientras que para las direcciones IP públicas voy a usar como base 100.0.0.0/16 en adelante.

![Captura 60](img/captura-060.png)

![Captura 61](img/captura-061.png)

#### Internet2

![Captura 62](img/captura-062.png)

![Captura 63](img/captura-063.png)

#### Internet 1

![Captura 64](img/captura-064.png)

![Captura 65](img/captura-065.png)

#### Empresa1

![Captura 66](img/captura-066.png)

![Captura 67](img/captura-067.png)

#### Empresa2

![Captura 68](img/captura-068.png)

![Captura 69](img/captura-069.png)

### Persistencia bit de forwarding

En Debian 12 para hacer persistente el bit de forwarding es decir que se comporte como un router hay que editar este fichero de configuración y ponerlo en 1. Una vez editado este fichero de configuración con el comando sysctl -p vamos a recargar la configuración del kernel de la red mediante este fichero de configuración. (Este proceso se va a repetir en todos los routers).

![Captura 70](img/captura-070.png)

![Captura 71](img/captura-071.png)

![Captura 72](img/captura-072.png)

![Captura 73](img/captura-073.png)

![Captura 74](img/captura-074.png)

![Captura 75](img/captura-075.png)

![Captura 76](img/captura-076.png)

![Captura 77](img/captura-077.png)

![Captura 78](img/captura-078.png)

![Captura 79](img/captura-079.png)

### Tablas de enrutamiento

#### Internet 1

![Captura 80](img/captura-080.png)

#### Internet 2

![Captura 81](img/captura-081.png)

#### Empresa1

![Captura 82](img/captura-082.png)

#### Empresa3

![Captura 83](img/captura-083.png)

#### Router casa

![Captura 84](img/captura-084.png)

#### Persistencia tablas de enrutamiento mediante creación de un servicio (systemd)

Cuando hacemos un ip route add en el siguiente reinicio se nos borra las entradas que hayamos metido para ello voy a crear un servicio de systemd para cada router el cual va a ejecutar un script en el que va a estar en cada router las rutas necesarias para que el escenario sea funcional.

![Captura 85](img/captura-085.png)

Una vez hecho el script lo voy a mover a los binarios del sistema y le voy a dar permisos de ejecución.

![Captura 86](img/captura-086.png)

Ahora nos faltaría crear el servicio de systemd para ello creamos un fichero en la ruta:

/etc/systemd/system/nombre-servicio.service.

En mi caso le he dado el nombre de rutas-estaticas.service y tendría que quedar con las siguientes líneas:

![Captura 87](img/captura-087.png)

Una vez realizado el servicio vamos a refrescar la lista de servicios y vamos a activarlo en el arranque e iniciar dicho servicio. Este proceso se va a repetir en todos los routers para que todas las rutas sean persistentes y no la perdamos en el siguiente reinicio.

![Captura 88](img/captura-088.png)

Internet 2

![Captura 89](img/captura-089.png)

![Captura 90](img/captura-090.png)

![Captura 91](img/captura-091.png)

![Captura 92](img/captura-092.png)

Empresa 1

![Captura 93](img/captura-093.png)

![Captura 94](img/captura-094.png)

![Captura 95](img/captura-095.png)

![Captura 96](img/captura-096.png)

Empresa 3

![Captura 97](img/captura-097.png)

![Captura 98](img/captura-098.png)

![Captura 99](img/captura-099.png)

![Captura 100](img/captura-100.png)

Casa

![Captura 101](img/captura-101.png)

![Captura 102](img/captura-102.png)

![Captura 103](img/captura-103.png)

![Captura 104](img/captura-104.png)

### Configuración servidor DHCP

Para el servidor DHCP voy a configurar la subred 10.0.3.0/24 que como comente anteriormente es el rango que estoy usando en las redes privadas. Esta configuración IP va a tener gateway, máscara de red, dirección de broadcast y servidores DNS. El lease time por defecto (tiempo de concesión) será de 600 segundos.

![Captura 105](img/captura-105.png)

También voy a reservar la 10.0.3.1 que es la interfaz del router para que no asigne esta IP.

![Captura 106](img/captura-106.png)

Y en el siguiente fichero tendremos que indicar la interfaz para que asigne IPS ya que sino el servicio no funcionará. (En mi caso enp2s1).

![Captura 107](img/captura-107.png)

Una vez hecho todos estos cambios reiniciaremos el servicio para que se lleven a cabo los cambios.

![Captura 108](img/captura-108.png)

Si ahora en la red de casa pedimos IP por DHCP en los 2 clientes que tenemos vemos como nos da IP en el rango del servidor DHCP.

![Captura 109](img/captura-109.png)

![Captura 110](img/captura-110.png)

Para comprobar que realmente es el servidor DHCP el que está dando las direcciones IP tenemos un fichero en la siguiente ruta donde podemos ver más detalladamente todo sobre las concesiones del servidor DHCP incluyendo dirección MAC.

debian@Casa:~\$ cat /var/lib/dhcp/dhcpd.leases

![Captura 111](img/captura-111.png)

### SNAT y DNAT

Para hacer SNAT simplemente añadimos la red de origen (-s), la tarjeta de salida (-o) y hacia que dirección IP va a ir “hacia fuera” (en caso de que fuesen IPS dinámicas se usaría MASQUERADE).

![Captura 112](img/captura-112.png)

Si ponemos las reglas iptable en la línea de comandos con el siguiente reinicio se perderán, por ello para hacerlas persistentes tendremos que poner en la interfaz las reglas iptables con up y down (down para cuando caiga la interfaz y borre la regla).

![Captura 113](img/captura-113.png)

Aquí vemos la regla se ha aplicado correctamente.

![Captura 114](img/captura-114.png)

Voy a realizar una prueba para ver que el SNAT en el router de casa ya es funcional para ello voy a iniciar una captura justo en este punto para ver que al hacer un ping desde casa cambia la IP origen por la pública.

![Captura 115](img/captura-115.png)

![Captura 116](img/captura-116.png)

Y aquí vemos como la IP origen ha cambiado por la pública.

![Captura 117](img/captura-117.png)

Empresa1

En este router tenemos servidores por lo que hay que hacer SNAT y DNAT (para que los servidores sean accesibles desde fuera). Primero voy a realizar el DNAT del servidor web y luego de PostgreSQL, para ello indicamos el puerto, la interfaz de entrada y la dirección destino a la que redirige la petición. Se usa PREROUTING ya que no nos interese que mira la tabla de enrutamiento (es decir queremos que antes que la mira el paquete haga DNAT).

![Captura 118](img/captura-118.png)

![Captura 119](img/captura-119.png)

Aquí vemos que al hacer una petición desde un cliente de la red de casa a la última IP pública antes de llegar a la red privada del servidor web nos contesta.

![Captura 120](img/captura-120.png)

Vamos a verificar que tengamos configurado correctamente el acceso remoto.

![Captura 121](img/captura-121.png)

DNAT postgresql

Y vamos a proceder con el DNAT de PostgreSQL para ello simplemente tenemos que cambiar el puerto por el 5432 y la IP de destino ya que son servidores diferentes es decir son dos máquinas, no está todo en una máquina.

![Captura 122](img/captura-122.png)

Tras hacer la regla de DNAT si pobramos a acceder al servidor desde uno de los clientes con el parámetro -h (host) indicando la IP podremos acceder sin problemas y veremos que están los datos con sus respectivas tablas.

![Captura 123](img/captura-123.png)

![Captura 124](img/captura-124.png)

SNAT Empresa2

![Captura 125](img/captura-125.png)

Tras aplicar la regla correspondiente de SNAT voy a realizar una captura entre empresa3 e internet1 para ver como cambia la IP origen.

![Captura 126](img/captura-126.png)

![Captura 127](img/captura-127.png)

Tras hacer el ping vemos como la IP de origen ha cambiado por la de empresa2.

![Captura 128](img/captura-128.png)

DNAT empresa2

![Captura 129](img/captura-129.png)

![Captura 130](img/captura-130.png)

Tras hacer las reglas correspondientes DNAT si desde un cliente accedemos a la base de datos remota con el parámetro -h (host) podemos acceder y tenemos las tablas disponibles con los datos.

![Captura 131](img/captura-131.png)

![Captura 132](img/captura-132.png)

![Captura 133](img/captura-133.png)

## NAT en Cisco

### Escenario inicial

Vamos a sustituir únicamente los routers Linux por routers Cisco. Para el direccionamiento voy a usar las mismas redes.

![Captura 134](img/captura-134.png)

### Asignación interfaces

Router Casa

![Captura 135](img/captura-135.png)

![Captura 136](img/captura-136.png)

Internet 2

![Captura 137](img/captura-137.png)

![Captura 138](img/captura-138.png)

Internet 1

![Captura 139](img/captura-139.png)

![Captura 140](img/captura-140.png)

Empresa 1

![Captura 141](img/captura-141.png)

![Captura 142](img/captura-142.png)

R3

![Captura 143](img/captura-143.png)

![Captura 144](img/captura-144.png)

### Tablas de enrutamiento

![Captura 145](img/captura-145.png)

![Captura 146](img/captura-146.png)

![Captura 147](img/captura-147.png)

![Captura 148](img/captura-148.png)

![Captura 149](img/captura-149.png)

![Captura 150](img/captura-150.png)

### Prueba conectividad

Hacemos ping a las diferentes redes de internet y como vemos llegan. Si pobrasemos a hacer un ping a una red privada que no sea la de casa desde este router no vamos a llegar nunca porque los routers de internet no enrutan IPS privadas.

![Captura 151](img/captura-151.png)

![Captura 152](img/captura-152.png)

### Servidor DHCP

Para crear el servidor DHCP crearemos un pool, y vamos a excluir la IP 10.0.3.1 que es la interfaz del router y sobre este pool vamos a definir la subred con le gateway.

![Captura 153](img/captura-153.png)

![Captura 154](img/captura-154.png)

Una vez configurado el servidor DHCP hay que iniciarlo para que empiece a conceder direcciones.

![Captura 155](img/captura-155.png)

Una vez que tenemos todo configurado vamos a irnos hacia los clientes y vamos a pedir IP por DHCP para ver que el servidor esté funcionando correctamente.

![Captura 156](img/captura-156.png)

<debian@debian>:~\$ sudo systemctl restart networking.service

Tras reiniciar el servicio veremos como hemos recibido una IP dentro de la subred del DHCP.

![Captura 157](img/captura-157.png)

Para comprobar que verdaremente nos ha concedido la IP vamos a ver las estadísticas del servidor y como vemos tenemos la IP con la hora de concesión y más parámetros relevantes.

![Captura 158](img/captura-158.png)

Aquí vemos las peticiones que le han llegado al servidor y como vemos tenemos 1 Offer, y 1 de ACK es decir que el servidor ha dado 1 configuración IP y el cliente la ha recibido.

![Captura 159](img/captura-159.png)

Repetimos el mismo proceso para el otro cliente.

![Captura 160](img/captura-160.png)

![Captura 161](img/captura-161.png)

Y aquí veremos que tanto los OFFER como los ACK han incrementado en 1 ya que ha dado una configuración IP a otro cliente.

![Captura 162](img/captura-162.png)

### SNAT

Casa

Para hacer SNAT en Cisco hay que crear una ACL básica indicando la red que va a hacer SNAT y la vamos a asignar a la interfaz de dentro es decir la de la red privada.

![Captura 163](img/captura-163.png)

Sobre esto tendremos que crear un pool de direcciones públicas e indicar que interfaz es la de dentro es decir la red privada y la interfaz de fuera es decir a la interfaz que va a salir.

Casa(config)#ip nat pool dir_publica 103.0.0.1 103.0.0.1 netmask 255.255.0.0

![Captura 164](img/captura-164.png)

Una vez configurado el SNAT voy a iniciar un captura de Wireshark en este punto para ver como cambia la IP de origen y destino tras realizar un ping.

![Captura 165](img/captura-165.png)

![Captura 166](img/captura-166.png)

Aquí vemos como tras lanzar un ping desde un cliente como la IP que se está “usando” es la IP pública es decir la f0/0.

![Captura 167](img/captura-167.png)

Para ver que realmente nuestras reglas NAT están funcionando tendremos que mirar el parámetro hits con la salida del comando que tenemos. Si en este parámetro no tenemos valor es decir 0 nuestra NAT no está funcionando.

![Captura 168](img/captura-168.png)

Empresa 1

Vamos a repetir el proceso para el router empresa1.

![Captura 169](img/captura-169.png)

Tras configurar NAT nuevamente vamos a verificar nuevamente el proceso para ello inicio una captura lanzo un ping y veremos como cambia la IP origen.

![Captura 170](img/captura-170.png)

![Captura 171](img/captura-171.png)

Tras hacer el proceso vemos como la IP cambia.

![Captura 172](img/captura-172.png)

Y nuevamente vemos los hits para comprobar que las reglas están funcionando.

![Captura 173](img/captura-173.png)

Empresa 2

Repetimos el proceso en el router de la empresa2. No voy a explicar el proceso para no ser repetitivo ya que lo he explicado anteriormente.

![Captura 174](img/captura-174.png)

![Captura 175](img/captura-175.png)

![Captura 176](img/captura-176.png)

![Captura 177](img/captura-177.png)

![Captura 178](img/captura-178.png)

Vemos como ha cambiado la IP tras hacer NAT.

![Captura 179](img/captura-179.png)

![Captura 180](img/captura-180.png)

Problema DHCP

Cuando añadimos la ACL para hacer NAT todo lo que añadamos a la ACL va a hacer NAT por lo que voy a borrar la ACL básica en la que voy a denegar los paquetes DHCP para que dichos paquetes no hagan NAT. Bootps (Servidor DHCP – Puerto 67), Bootpc (Cliente DHCP – Puerto 68). Con esta ACL conseguimos que las peticiones DHCP no hagan NAT ya que esta ACL se está usando para hacer NAT. Ahora las peticiones DHCP llegan y no salen hacia el exterior por lo que los clientes reciben IP.

![Captura 181](img/captura-181.png)

Cambiamos la ACL del NAT.

![Captura 182](img/captura-182.png)

### DNAT

Para hacer DNAT la sintaxis es la siguiente:

ip nat inside source static \[protocolo\] \[IP_privada\] \[puerto\] \[IP_pública\] \[puerto\]

![Captura 183](img/captura-183.png)

Tras hacer DNAT si enviamos una petición al servidor web vemos como nos responde.

![Captura 184](img/captura-184.png)

Ahora vamos a realizar el DNAT para el servidor PostgreSQL que usa el puerto 5432.

![Captura 185](img/captura-185.png)

Tras realizar la regla DNAT vamos a comprobar desde el cliente que tenemos conectividad con el servidor para ello vamos a usar el parámetro -h para indicar la IP del servidor y los parámetros para el usuario y la base de datos. Tras conectarnos vemos como tenemos las tablas disponibles que creamos en el servidor. Vemos que al ser una conexión remota usa conexión con certificado SSL.

![Captura 186](img/captura-186.png)

Ahora vamos a pasar al DNAT de la empresa 2 para ello vamos a empezar con el servidor SSH para ello usamos la misma sintaxis y ponemos la IP del servidor.

![Captura 187](img/captura-187.png)

Tras hacer el DNAT vemos como podemos conectarnos al servidor.

![Captura 188](img/captura-188.png)

Por último vamos a realizar la regla DNAT para el servidor MariaDB para ello seguimos usando la misma sintaxis y el puerto 3306 (MariaDB).

![Captura 189](img/captura-189.png)

Tras hacer la regla DNAT vemos como nos podemos conectar con el parámetro -h al servidor remoto y tenemos las tablas disponibles que introducimos en el servidor.

![Captura 190](img/captura-190.png)

Si hacemos una consulta sencilla para ver que datos hay en la tabla vemos que tenemos los datos totalmente disponibles.

![Captura 191](img/captura-191.png)

Comprobación servidor web

Aunque ya haya hecho las comprobaciones y todo funcione voy a realizar una captura con Wireshark para ver los paquetes.

![Captura 192](img/captura-192.png)

![Captura 193](img/captura-193.png)

Aquí vemos como he iniciado la captura dentro en la red privada la IP destino ha cambiado por la privada mediante el DNAT.

![Captura 194](img/captura-194.png)

También lo podemos comprobar mediante un firefox y vemos como tenemos la página por defecto de apache.

![Captura 195](img/captura-195.png)

Si preguntamos por las IPS privadas no nos va contestar en la vida ya que son routers de internet y en el mundo hay millones de IPS privadas y dichos routers no enrutan redes privadas.

![Captura 196](img/captura-196.png)

### Depuración NAT final

![Captura 197](img/captura-197.png)

![Captura 198](img/captura-198.png)

![Captura 199](img/captura-199.png)

Para ver los paquetes en tiempo real con el siguiente comando lo podemos ver:

Casa#debug ip nat

![Captura 200](img/captura-200.png)

![Captura 201](img/captura-201.png)

### Bibliografía

<https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/13778-9.html>

<https://netfilter.org/documentation/HOWTO/es/NAT-HOWTO-7.html>

<https://blog.junglacode.org/software/software-libre/configurar-acceso-remoto-en-mariadb/>

<https://help.clouding.io/hc/es/articles/4403784968594-Acceso-remoto-a-PostgreSQL>

## 📎 Ficheros de la práctica

<details>
<summary><code>Enrutamiento.txt</code></summary>

```bash
Internet 1

ip route default via 102.0.0.1 dev enp2s2
 
Internet 2

ip route add default via 102.0.0.2 dev enp2s0
 
Casa

ip route add default via 103.0.0.2 dev enp2s0

Empresa 1

ip route add default via 100.0.0.1 dev enp2s1

Empresa 2

ip route add default via 101.0.0.2 dev enp2s0

-- Cisco

Empresa 1

ip route 0.0.0.0 0.0.0.0 100.0.0.1 

Internet 1

ip route 0.0.0.0 0.0.0.0 102.0.0.1 

R3

ip route 0.0.0.0 0.0.0.0 101.0.0.1

Internet 2

ip route 0.0.0.0 0.0.0.0 102.0.0.2 
 
Casa

ip route 0.0.0.0 0.0.0.0 103.0.0.2
```

</details>

