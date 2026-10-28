---
title: "The Hunger Games - Sinsajo: NAT y cortafuegos con iptables"
date: 2026-04-27T10:00:00+01:00
description: "Antes de realizar ninguna configuración voy a activar el bit de forwarding para que la máquina reenvíe tráfico y se comporte como un router para ello en Debian 12 es el siguiente fichero de…"
menu:
  sidebar:
    name: "The Hunger Games - Sinsajo"
    identifier: par-the-hunger-games-sinsajo
    parent: par
    weight: 260427
hero: cover.png
tags: ["iptables", "Firewall", "NAT", "Enrutamiento", "systemd"]
categories: ["Prácticas"]
draft: false
---

> 📚 **1º ASIR · Planificación y Administración de Redes** · Tema 5: Nivel de red · Curso 2025/2026

## Bit de forwarding

Antes de realizar ninguna configuración voy a activar el bit de forwarding para que la máquina reenvíe tráfico y se comporte como un router para ello en Debian 12 es el siguiente fichero de configuración y una vez que lo tengamos activado vamos a recargar la configuración con sudo sysctl -p. (Esta máquina es una plantilla que he realizado y a partir de esta las voy a ir duplicando)

<debian@debian>:~\$ sudo nano /etc/sysctl.conf

![Captura 1](img/captura-001.png)

![Captura 2](img/captura-002.png)

## Preparación escenario

Para preparar los servicios necesarios para el escenario voy a conectar a un switch una máquina que va a ser el nodo base a partir del cuál voy a ir duplicando máquinas y a dicho switch también hay un NAT conectado para tener salida a Internet y poder instalar paquetes.

![Captura 3](img/captura-003.png)

Tras poner una configuración por DHCP voy a reiniciar el servicio para recibir IP por DHCP.

debian@debian:~\$ sudo systemctl restart networking.service

Voy a realizar una actualización de los repositorios.

![Captura 4](img/captura-004.png)

Y voy a instalar todos los servicios necesarios para este escenario. (Plantilla masculina).

![Captura 5](img/captura-005.png)

Para la plantilla femenina va a ser algo similar primeramente vamos a conectarlo a dicho switch para poder salir a Internet. (No lo voy a explicar el proceso ya que es lo mismo nuevamente).

debian@debian:~\$ sudo nano /etc/network/interfaces

![Captura 6](img/captura-006.png)

debian@debian:~\$ sudo systemctl restart networking.service

![Captura 7](img/captura-007.png)

La instalación de MariaDB tiene dos pasos uno que es instalar este paquete y luego tiene un script de instalación para desplegar la base de datos.

![Captura 8](img/captura-008.png)

Ejecutamos el script de instalación de MariaDB.

![Captura 9](img/captura-009.png)

El acceso remoto de root lo voy a tener desactivado por motivos de seguridad.

![Captura 10](img/captura-010.png)

Y tras hacer esto recargamos los permisos delas tablas.

![Captura 11](img/captura-011.png)

### Acceso remoto MariaDB

Para permitir el acceso remoto en MariaDB lo vamos a realizar mediante el siguiente fichero de configuración y en el bind address con la IP que tenemos vamos a permitir la conexión desde cualquier host.

debian@debian:~\$ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

![Captura 12](img/captura-012.png)

Y tras hacer este cambio vamos a reiniciar el servicio para que los cambios tengan efecto.

debian@debian:~\$ sudo systemctl restart mariadb.service

Una vez instalado voy a acceder a la base de datos como root.

![Captura 13](img/captura-013.png)

Una vez dentro voy a crear una base de datos de prueba para el escenario, y voy a crear un usuario poniendo % que esto indica que vamos a permitir la conexión con este usuario desde cualquier host y en mi caso le voy a dar todos los privilegios que esto no se recomienda en entornos de producción.

![Captura 14](img/captura-014.png)

Una vez creado el usuario voy a acceder para meter tablas.

![Captura 15](img/captura-015.png)

En mi caso he usado el script Scott.

![Captura 16](img/captura-016.png)

Para el distrito 13 voy a instalar el servidor PostgreSQL ya que ssh y apache ya están instalados porque es la plantilla femenina. (Más adelante le cambio el nombre a la máquina).

<debian@Thresh>:~\$ sudo apt install postgresql postgresql-contrib

![Captura 17](img/captura-017.png)

Cuando accedemos al usuario administrador de postgresql con la configuración por defecto que trae el paquete nos sale este error.

![Captura 18](img/captura-018.png)

Para ello hay que editar el siguiente fichero de configuración y en el usuario postgres y en los demás podemos poner trust (no recomendado en entornos de producción ya que no pide contraseña) o también tenemos la opción de md5 que nos pide contraseña para el login.

debian@Thresh:~\$ sudo nano /etc/postgresql/15/main/pg_hba.conf

![Captura 19](img/captura-019.png)

Tras hacer este cambio en el fichero y reiniciar el servicio ya podremos acceder con el usuario administrador de postgreSQL.

![Captura 20](img/captura-020.png)

### Acceso remoto PostgreSQL

Para configurar el acceso remoto en PostgreSQL es mediante el siguiente fichero de configuración y las direcciones de escucha ponemos el asterisco para permitir conexiones desde cualquier host.

debian@Thresh:~\$ sudo nano /etc/postgresql/15/main/postgresql.conf

![Captura 21](img/captura-021.png)

Y tras realizar este cambio reiniciamos el servicio para que los cambios tengan efecto.

debian@Thresh:~\$ sudo systemctl restart postgresql.service

Una vez que tenemos configurado el acceso remoto vamos a configurar el usuario remoto para ello mediante el siguiente fichero de configuración y voy a permitir conexiones desde cualquier host pero con acceso por contraseña. Esta métrica es la de método (md5).

debian@Thresh:~\$ sudo nano /etc/postgresql/15/main/pg_hba.conf

![Captura 22](img/captura-022.png)

Una vez configurado el acceso remoto voy a introducir nuevamente el esquema scott pero por defecto en Debian 12 al introducir datos nos da este error. Básicamente es por tema de administración de permisos.

![Captura 23](img/captura-023.png)

Para ello nos metemos en la base de datos que hemos creado pero con el usuario administrador y le vamos a dar permisos para que pueda insertar datos.

![Captura 24](img/captura-024.png)

Tras realizar esto listamos las tablas y ya tenemos el esquema introducido.

![Captura 25](img/captura-025.png)

## Direccionamiento

Para configurar las interfaces simplemente es configurar cada IP con su máquina correspondiente y tras configurar el fichero de configuración reiniciar el servicio de red. (No voy a explicar este paso por cada máquina ya que siempre es el mismo proceso).

debian@Marvel:~\$ sudo nano /etc/network/interfaces

debian@Marvel:~\$ sudo systemctl restart networking.service

Marvel

![Captura 26](img/captura-026.png)

![Captura 27](img/captura-027.png)

Glimmer

![Captura 28](img/captura-028.png)

![Captura 29](img/captura-029.png)

Thresh

![Captura 30](img/captura-030.png)

![Captura 31](img/captura-031.png)

Rue

![Captura 32](img/captura-032.png)

![Captura 33](img/captura-033.png)

Cato

![Captura 34](img/captura-034.png)

![Captura 35](img/captura-035.png)

Clove

![Captura 36](img/captura-036.png)

![Captura 37](img/captura-037.png)

Peeta

![Captura 38](img/captura-038.png)

![Captura 39](img/captura-039.png)

Katniss

![Captura 40](img/captura-040.png)

![Captura 41](img/captura-041.png)

## Direccionamiento routers

R1

![Captura 42](img/captura-042.png)

![Captura 43](img/captura-043.png)

R2

![Captura 44](img/captura-044.png)

![Captura 45](img/captura-045.png)

R3

![Captura 46](img/captura-046.png)

![Captura 47](img/captura-047.png)

R4

![Captura 48](img/captura-048.png)

![Captura 49](img/captura-049.png)

R5

![Captura 50](img/captura-050.png)

![Captura 51](img/captura-051.png)

![Captura 52](img/captura-052.png)

## Tablas de enrutamiento

R1

![Captura 53](img/captura-053.png)

R2

![Captura 54](img/captura-054.png)

R3

![Captura 55](img/captura-055.png)

R4

![Captura 56](img/captura-056.png)

R5

![Captura 57](img/captura-057.png)

## Prueba conectividad

Para probar la efectividad de las tablas de enrutamiento simplemente desde R3 voy a realizar ping a las IPS públicas ya que son routers de Internet y no enrutan IPS privadas y además no está configurado el NAT.

![Captura 58](img/captura-058.png)

## Persistencia tablas de enrutamiento (systemd)

Para hacer las tablas de enrutamiento persistentes voy a crear un servicio de systemd el cual va a ejecutar un script con todos las tablas de enrutamiento.

R1

![Captura 59](img/captura-059.png)

![Captura 60](img/captura-060.png)

R2

![Captura 61](img/captura-061.png)

![Captura 62](img/captura-062.png)

![Captura 63](img/captura-063.png)

R3

![Captura 64](img/captura-064.png)

![Captura 65](img/captura-065.png)

![Captura 66](img/captura-066.png)

R4

![Captura 67](img/captura-067.png)

![Captura 68](img/captura-068.png)

![Captura 69](img/captura-069.png)

R5

![Captura 70](img/captura-070.png)

![Captura 71](img/captura-071.png)

![Captura 72](img/captura-072.png)

## Reglas SNAT y DNAT persistentes

Para hacer las reglas iptables persistentes voy a crear un script con todas las reglas, las primeras líneas de este script va a borrar las reglas anteriores y reinicia los contadores de las matches. Una vez que tenga el script con todas las reglas creo un servicio de systemd para que el script se ejecute automáticamente. (Este proceso se va a repetir en todos los routers).

Para hacer SNAT simplemente añadimos la red de origen (-s) hacia que dirección IP va a ir “hacia fuera” (en caso de que fuesen IPS dinámicas se usaría MASQUERADE).

Para el caso de DNAT usaremos el destino y el puerto y en prerouting ya que si no al ver el router que tiene como IP destino su IP se queda con el paquete y no lo redirecciona al servidor.

R1

SNAT y DNAT

![Captura 73](img/captura-073.png)

Servicio de systemd

![Captura 74](img/captura-074.png)

![Captura 75](img/captura-075.png)

![Captura 76](img/captura-076.png)

R2

![Captura 77](img/captura-077.png)

![Captura 78](img/captura-078.png)

![Captura 79](img/captura-079.png)

![Captura 80](img/captura-080.png)

R3

![Captura 81](img/captura-081.png)

![Captura 82](img/captura-082.png)

![Captura 83](img/captura-083.png)

![Captura 84](img/captura-084.png)

R4

![Captura 85](img/captura-085.png)

![Captura 86](img/captura-086.png)

![Captura 87](img/captura-087.png)

![Captura 88](img/captura-088.png)

R5

![Captura 89](img/captura-089.png)

![Captura 90](img/captura-090.png)

![Captura 91](img/captura-091.png)

![Captura 92](img/captura-092.png)

## Comprobación NAT

![Captura 93](img/captura-093.png)

![Captura 94](img/captura-094.png)

![Captura 95](img/captura-095.png)

![Captura 96](img/captura-096.png)

DNAT

Etag Servidores Web

Para cuando hagamos un curl y sepamos que máquina me está respondiendo he configurado una métrica de apache para saber en todo momento que máquina me responde. Para ello hay que activar el etag y una vez activado voy a exportar una variable la cuál va a leer el hostname del sistema. Esta variable la voy a asignar a un fichero para que cuando hagamos un curl nos devuelva el nombre. (Este proceso lo voy a realizar en todos los servidores web).

Marvel

![Captura 97](img/captura-097.png)

![Captura 98](img/captura-098.png)

![Captura 99](img/captura-099.png)

Una vez que reiniciemos el servicio si comprobamos hacer curl vemos como en el parámetro etag nos responde con el nombre de la máquina.

![Captura 100](img/captura-100.png)

Thresh

![Captura 101](img/captura-101.png)

![Captura 102](img/captura-102.png)

![Captura 103](img/captura-103.png)

![Captura 104](img/captura-104.png)

![Captura 105](img/captura-105.png)

Cato

![Captura 106](img/captura-106.png)

![Captura 107](img/captura-107.png)

![Captura 108](img/captura-108.png)

![Captura 109](img/captura-109.png)

Peeta

![Captura 110](img/captura-110.png)

![Captura 111](img/captura-111.png)

![Captura 112](img/captura-112.png)

![Captura 113](img/captura-113.png)

Servidor Web secreto (Distrito 13)

![Captura 114](img/captura-114.png)

![Captura 115](img/captura-115.png)

![Captura 116](img/captura-116.png)

![Captura 117](img/captura-117.png)

DNAT comprobación

En este punto voy a comprobar todos los servidores de todas las redes (OpenSSH Server, MariaDB, PostgreSQL y Apache2).

R1

SSH

![Captura 118](img/captura-118.png)

ServidorWEB

![Captura 119](img/captura-119.png)

MariaDB

![Captura 120](img/captura-120.png)

R2

![Captura 121](img/captura-121.png)

![Captura 122](img/captura-122.png)

![Captura 123](img/captura-123.png)

R3

![Captura 124](img/captura-124.png)

![Captura 125](img/captura-125.png)

![Captura 126](img/captura-126.png)

R4

![Captura 127](img/captura-127.png)

![Captura 128](img/captura-128.png)

![Captura 129](img/captura-129.png)

R5

![Captura 130](img/captura-130.png)

![Captura 131](img/captura-131.png)

![Captura 132](img/captura-132.png)

## Política por defecto

Instalar iptables en cada uno de los routers Linux, estableciendo una política DROP por defecto para todas las cadenas. Toma las medidas necesarias para conservar el acceso a los routers por ssh y para que la configuración de iptables sea persistente. Demuestra que los cortafuegos han interrumpido todas las comunicaciones.

Voy a establecer en mi script de reglas la política por defecto para cortar todas las comunicaciones.

R1

![Captura 133](img/captura-133.png)

R2

![Captura 134](img/captura-134.png)

R3

![Captura 135](img/captura-135.png)

R4

![Captura 136](img/captura-136.png)

R5

![Captura 137](img/captura-137.png)

## Permitir acceso por SSH

R1

![Captura 138](img/captura-138.png)

![Captura 139](img/captura-139.png)

R2

![Captura 140](img/captura-140.png)

![Captura 141](img/captura-141.png)

R3

![Captura 142](img/captura-142.png)

![Captura 143](img/captura-143.png)

R4

![Captura 144](img/captura-144.png)

![Captura 145](img/captura-145.png)

R5

![Captura 146](img/captura-146.png)

![Captura 147](img/captura-147.png)

Comprobación de como los cortafuegos han interrumpido todas las comunicaciones.

![Captura 148](img/captura-148.png)

![Captura 149](img/captura-149.png)

## Reglas de Firewall

### Regla a

a\) Los indigentes pueden acceder al servidor de bases de datos de los ricos.

Para que esto ocurra he contemplado dos posibles caminos uno que sería con R5 encendido y otro con R5 apagado. He usado el módulo state para asegurarme que no se inician conexiones nuevas al servidor solo las que ya han entablado una conexión previa.

#### Reglas

![Captura 150](img/captura-150.png)

![Captura 151](img/captura-151.png)

![Captura 152](img/captura-152.png)

![Captura 153](img/captura-153.png)

#### Funcionamiento

Para realizar la prueba de estas reglas he usado la dirección IP remota 193.168.1.2 porque es hacia la cuál mi enrutamiento sabe llegar según mis tablas de enrutamiento. Para comprobar el otro camino posible habría apagr R5 y cambiar las tablas de enrutamiento.

![Captura 154](img/captura-154.png)

#### Matches

Tras realizar esta conexión al servidor vemos como los matches han subido con la regla que hemos implementado. En R2 los matches no han incrementado porque no hemos usado esa ruta.

![Captura 155](img/captura-155.png)

![Captura 156](img/captura-156.png)

![Captura 157](img/captura-157.png)

### Regla b

b\) Los indigentes y los superpijos pueden acceder al servidor ssh existente en el distrito 13

En este regla solo hay un camino posible ya que si R5 se apaga, la LAN que tiene conectada a el donde están los diferentes servidores ya no estaría.

#### Reglas

![Captura 158](img/captura-158.png)

![Captura 159](img/captura-159.png)

![Captura 160](img/captura-160.png)

#### Funcionamiento

Una vez implementada las reglas voy a realizar la conexión desde los indigentes y los superpijos para comprobar que ambas cosas funcionan.

Indigentes

Como vemos tras realizar SSH accedemos al servidor de la red interna de R5 que es el servidor secreto.

![Captura 161](img/captura-161.png)

![Captura 162](img/captura-162.png)

Superpijos

![Captura 163](img/captura-163.png)

![Captura 164](img/captura-164.png)

#### Matches

Tras conectarnos vemos como los matches asociado a esas reglas han incrementado.

![Captura 165](img/captura-165.png)

![Captura 166](img/captura-166.png)

![Captura 167](img/captura-167.png)

### Regla c

c\) Los pobres pueden acceder al servidor web de Thresh, pero solo de 08 a 15 horas.

Para esta regla hay que implementar un cortafuegos de nodo ya que cuando los pobres intenten acceder al servidor web de Thresh ese mensaje pasa por el switch pero no llega al router porque ya están en la red interna. Por lo tanto hay que implementar un cortafuegos de nodo en Thresh.

#### Reglas

Para ello las primeras líneas del script es establecer una política DROP por defecto, la cadena de FORWARD no la he establecido por defecto ya que no tiene sentido porque está máquina no tiene el bit de forwarding es decir no reenvía paquetes si el destinatario no es él. Para controlar las horas voy a usar el módulo time indicando fecha de inicio y fin.

![Captura 168](img/captura-168.png)

Tras crear el script con las reglas necesarias voy a seguir el mismo procedimiento que anteriormente. Damos permisos de ejecución a dicho script.

![Captura 169](img/captura-169.png)

Y voy a crear un servicio de systemd para que el script de las reglas se ejecute automáticamente y además sea persistente (tanbién se puede con iptables-persistant).

![Captura 170](img/captura-170.png)

Tras crear el servicio voy a recargar la lista de servicios disponibles y lo voy a hacer persistente en el arranque del sistema y también lo voy a iniciar.

![Captura 171](img/captura-171.png)

#### Funcionamiento

Tras implementar estas reglas si desde una máquina de los pobres hacemos un curl fuera del rango horario que hemos definido en las reglas no vamos a poder acceder.

![Captura 172](img/captura-172.png)

#### Matches

Vemos como no está haciendo match con ninguna regla por lo que la política por defecto drop tira el paquete.

![Captura 173](img/captura-173.png)

Si probamos a realizar un curl dentro del rango horario vamos a poder realizarlo.

![Captura 174](img/captura-174.png)

Aquí vemos como si ha hecho match con la regla que hemos implementado.

![Captura 175](img/captura-175.png)

### Regla d

d\) Clover puede acceder a los servidores de los superpijos, excepto al ssh.

Para esta regla nuevamente he contemplado con R4 apagado y con R5 encendido. Pero como he comentado anteriormente siguiendo mis tablas de enrutamiento el camino más corto es por R5 por lo que para comprobar dichas reglas con R5 apagado habría que cambiar las tablas de enrutamiento. Hay que permitir el servidor web y el servidor de base de datos (MariaDB).

#### Reglas

![Captura 176](img/captura-176.png)

![Captura 177](img/captura-177.png)

![Captura 178](img/captura-178.png)

![Captura 179](img/captura-179.png)

![Captura 180](img/captura-180.png)

#### Funcionamiento

Tras implementar las reglas voy a realizar la comprobación que puedo acceder a los diferentes servicios desde Clove.

![Captura 181](img/captura-181.png)

![Captura 182](img/captura-182.png)

#### Matches

Una vez comprobada la efectividad de las reglas vemos como los matches han incrementado (R2 y R4 no tienen matches ya que mi enrutamiento coge el camino más corto por R5).

![Captura 183](img/captura-183.png)

![Captura 184](img/captura-184.png)

![Captura 185](img/captura-185.png)

### Regla e

e\) Marvel decide que Peeta pueda entrar en su servidor web, que recibe peticiones por el puerto 8080 en lugar de por el 80 para mayor seguridad.

Para ello lo que primero vamos a realizar es configurar el servidor web para que escuche en el puerto 8080 para ello editamos el siguiente fichero de configuración:

debian@Marvel:~\$ sudo nano /etc/apache2/ports.conf

![Captura 186](img/captura-186.png)

También habrá que editar el siguiente fichero de configuración e indicarle el puerto 8080 ya que si lo dejamos este fichero con el puerto 80 el servidor no va a saber que página html servirle al cliente. Gracias a este fichero de configuración podemos tener varios sitios web mediante los virtual hosts.

debian@Marvel:~\$ sudo nano /etc/apache2/sites-available/000-default.conf

![Captura 187](img/captura-187.png)

Una vez hecho los cambios voy a reiniciar el servicio para que tengan efecto en el servicio.

![Captura 188](img/captura-188.png)

#### Reglas

![Captura 189](img/captura-189.png)

![Captura 190](img/captura-190.png)

![Captura 191](img/captura-191.png)

![Captura 192](img/captura-192.png)

Al cambiar el puerto de escucha del servidor web también hay que cambiar la regla de DNAT ya que ahora mismo esta en el puerto 80 y habría que cambiarla por el puerto 8080 para que el router redireccione la petición mediante ese puerto. También habría que cambiar el puerto en las reglas que permiten el tráfico hacia este servidor, en todos los routers que tengan reglas hacia este servidor y simplemente cambiamos el puerto 80 por el 8080.

![Captura 193](img/captura-193.png)

#### Funcionamiento

Como el servidor está escuchando en el puerto 8080 al hacer la petición hay que indicarle ese puerto ya que no es el puerto por defecto.

![Captura 194](img/captura-194.png)

#### Matches

![Captura 195](img/captura-195.png)

![Captura 196](img/captura-196.png)

![Captura 197](img/captura-197.png)

### Regla f

f\) Se ha producido una fuga de información militar de la base de datos del distrito 13. Los pijos han conseguido acceder al servidor Postgres y realizar una consulta a la tabla Misiles. Añade las reglas necesarias para realizar dicho acceso y muestra el momento de la consulta de la información.

Para realizar esta consulta de carácter militar he añadido una tabla en la base de datos del distrito 13. Cabe destacar que en este punto podemos añadir reglas para que se pueda acceder desde las 4 interfaces posibles de R5 pero yo bajo mi punto de visto no lo veo seguro ya que el punto de ataque es más grande por lo que yo he añadido las reglas necesarias para que los pijos puedan acceder a la base de datos del distrito 13 desde la IP más cercana. También si añadimos las reglas que he comentado anteriormente no sería lo más efectivo ya que tal y como está el escenario montado los pijos está pegado a R5 por lo que no tiene sentido dar la vuelva a todo el escenario cuando ya el router está al lado.

![Captura 198](img/captura-198.png)

#### Reglas

![Captura 199](img/captura-199.png)

![Captura 200](img/captura-200.png)

#### Funcionamiento

Aquí vemos el momento de la consulta (fuga militar).

![Captura 201](img/captura-201.png)

#### Matches

![Captura 202](img/captura-202.png)

![Captura 203](img/captura-203.png)

### Regla g

g\) Katniss ha conseguido incorporar reglas que le permiten (solo a ella) acceder a todos los servidores ssh de todos los distritos. Una vez dentro de ellos puede acceder a los servidores de bases de datos, al haber conseguido infiltrarse en esas redes. Demuestra que puede hacerlo.

Primero voy a añadir las reglas que permiten el acceso a SSH desde Katniss a todas las redes y después comprobaremos como nos conectamos por SSH y estando en la red interna nos conectamos al servidor de base de datos. Para esto solo hace falta añadir las reglas de SSH ya que una vez que estamos en la red interna para acceder a la base de datos, ese mensaje no llega al router, simplemente llega al switch y el switch lo envía al servidor por lo que podríamos de alguna manera aprovechar esta regla como posibles ataques.

R4 a R2

![Captura 204](img/captura-204.png)

![Captura 205](img/captura-205.png)

![Captura 206](img/captura-206.png)

![Captura 207](img/captura-207.png)

![Captura 208](img/captura-208.png)

R4 a R3

![Captura 209](img/captura-209.png)

![Captura 210](img/captura-210.png)

![Captura 211](img/captura-211.png)

![Captura 212](img/captura-212.png)

![Captura 213](img/captura-213.png)

R4 a R5

![Captura 214](img/captura-214.png)

![Captura 215](img/captura-215.png)

![Captura 216](img/captura-216.png)

![Captura 217](img/captura-217.png)

![Captura 218](img/captura-218.png)

#### Reglas

![Captura 219](img/captura-219.png)

![Captura 220](img/captura-220.png)

![Captura 221](img/captura-221.png)

![Captura 222](img/captura-222.png)

![Captura 223](img/captura-223.png)

#### Matches

![Captura 224](img/captura-224.png)

![Captura 225](img/captura-225.png)

![Captura 226](img/captura-226.png)

#### Funcionamiento

Una vez implementada las reglas voy a probar a conectarme a cada servidor ssh de cada red para posteriormente conectarme a la base de datos de la red interna.

![Captura 227](img/captura-227.png)

![Captura 228](img/captura-228.png)

![Captura 229](img/captura-229.png)

Ya que en Thresh anteriormente configuramos un cortafuegos de nodo y tiene una política DROP por defecto hay que configurar esta máquina para que reciba las peticiones y mande respuesta el servidor de base de datos el cuál tiene un cortafuegos de nodo, para ello usamos input ya que es un mensaje que le llega a él (petición) y output el mensaje que manda al aceptar la conexión (respuesta).

![Captura 230](img/captura-230.png)

Una vez implementado ya podremos conectarnos al servidor de base de datos.

![Captura 231](img/captura-231.png)

### Regla h

h\) Los superpijos han planeado un ataque de denegación de servicios saturando de peticiones el servidor Postgres del distrito 13, configura el cortafuegos del R5 para rechazar el ataque, demostrando la eficacia de dicha defensa.

Actualmente con las reglas que tenemos los superpijos no pueden acceder al servidor de base de datos del distrito 13 por lo que primero voy a añadir las reglas para que esto sea posible.

![Captura 232](img/captura-232.png)

Matches

![Captura 233](img/captura-233.png)

![Captura 234](img/captura-234.png)

Una vez que tenemos configurado el acceso normal vamos a proceder a evitar un ataque de denegación de servicios.

R5

Primero estas reglas las vamos a borrar porque vamos a sustituir añadiendo un módulo de iptables para limitar las conexiones.

![Captura 235](img/captura-235.png)

Añadimos un módulo en el cual solo vamos a permitir una conexión simultánea y uso reject porque he estado realizando pruebas y corta más rápido el tráfico que DROP. Estas reglas las tenemos que añadir al principio ya que si llega a la regla con la que hace la conexión no va a seguir leyendo las reglas por lo que primero tienen que estar dichas reglas para que no haga la conexión con la regla sin el módulo de connlimit.

![Captura 236](img/captura-236.png)

Una vez tenemos configuradas las reglas voy a proceder con el ataque de denegación de servicios para ello antes de nada para no tener que indicar cada conexión la contraseña he creado un fichero que tiene que tener el siguiente formato:  
IP:puerto:basededatos:usuario:contraseña

<debian@debian>:~\$ echo "193.168.1.1:5432:pruebajesus:jesusfigueroa:acceso" \> ~/.pgpass

Tendrá que tener permisos de lectura y escritura solo para el usuario.

![Captura 237](img/captura-237.png)

Y una vez que tenemos la contraseña en un fichero podemos proceder al ataque. Este comando genera 10000 iteracciones, ejecutando hasta 50 procesos en paralelo.

seq 10000 \| xargs -P 50 -I{} psql -h 193.168.1.1 -U jesusfigueroa -d distrito13 -c "SELECT 1;"

Una vez ejecutado el ataque voy a ir monitorizando cada segundo los matches de las reglas que he añadido para prevenir este ataque.

watch -n1 sudo iptables -L FORWARD -n -v --line-numbers

Como vemos ahí tenemos los matches que están bloqueando el ataque.

![Captura 238](img/captura-238.png)

Y simultáneamente si probamos a acceder podemos acceder sin problema.

![Captura 239](img/captura-239.png)

![Captura 240](img/captura-240.png)

## Servidor DHCP

5\. Configurar el servicio DHCP que tenían los superpijos en las mismas condiciones que tenían cuando se usaban routers CISCO. Comprueba que siguen funcionando las reglas d), e), g) y h)

He configurado el servidor DHCP de R1 en las mismas condiciones que los routers Cisco.

![Captura 241](img/captura-241.png)

Para que cuando implemente el servidor DHCP y esté funcionando y no haya problemas con las direcciones IP de los servidores voy a reservar la IP del gateway de la red interna, y además la dirección IP de ambos servidores de la red. Ya que si no reservamos la IP va a estar constantemente cambiado y además habría que ir cambiado las reglas de DNAT por lo que es mucho más cómodo reservar IPS para los servidores de nuestra red.

![Captura 242](img/captura-242.png)

![Captura 243](img/captura-243.png)

Una vez que tenemos la configuración vamos a definir la interfaz hacia “fuera” que va a dar direcciones IPS el servidor DHCP.

![Captura 244](img/captura-244.png)

Tras realizar todos estos cambios voy a reiniciar el servicio y lo voy a hacer persistente en el arranque.

![Captura 245](img/captura-245.png)

Para que el tráfico DHCP llegue al router hay que añadir estas dos reglas.

![Captura 246](img/captura-246.png)

Una vez implementado el servidor DHCP voy a configurar Marvel y Glimmer para que reciban IP por DHCP.

![Captura 247](img/captura-247.png)

![Captura 248](img/captura-248.png)

Comprobación reglas

Tras implementar el servidor DHCP en R1 no vamos a tener ningun problema con las reglas ya que he realizado la reserva de la IP de los servidores que hay en dicha red.

d\)

![Captura 249](img/captura-249.png)

![Captura 250](img/captura-250.png)

Matches

![Captura 251](img/captura-251.png)

![Captura 252](img/captura-252.png)

![Captura 253](img/captura-253.png)

e\)

![Captura 254](img/captura-254.png)

Matches

![Captura 255](img/captura-255.png)

![Captura 256](img/captura-256.png)

![Captura 257](img/captura-257.png)

g\)

![Captura 258](img/captura-258.png)

![Captura 259](img/captura-259.png)

Matches

![Captura 260](img/captura-260.png)

![Captura 261](img/captura-261.png)

![Captura 262](img/captura-262.png)

h\)

![Captura 263](img/captura-263.png)

Matches

![Captura 264](img/captura-264.png)

## Bibliografía

<https://docs.redhat.com/es/documentation/red_hat_enterprise_linux/6/html/security_guide/sect-security_guide-firewalls-using_iptables>

<https://plataforma.josedomingo.org/pledin/cursos/mv2006/files/IPtables_pello.pdf>

<https://sysadblog.onrender.com/posts/ejercicio1_cortafuegos_personal_iptables/>

<https://www.ibm.com/docs/es/qsip/7.5.0?topic=tasks-advanced-iptables-rules-examples>

<https://rincondelsistema.home.blog/2021/02/26/configuracion-iptables/>

## 📎 Ficheros de la práctica

<details>
<summary><code>Enrutamiento/Enrutamiento.txt</code></summary>

```bash
172.23.0.0 
172.24.0.0
172.25.0.0
193.168.1.0
193.168.2.0
193.168.11.0
193.168.12.0

R1

sudo ip route add 172.24.0.0/24 via 173.23.0.2 dev enp2s2
sudo ip route add 172.25.0.0/24 via 173.23.0.2 dev enp2s2
sudo ip route add default via 193.168.1.1

R2

sudo ip route add 193.168.1.0/24 via 173.23.0.1
sudo ip route add default via 173.24.0.2

R3
 
sudo ip route add 172.24.0.0/24 via 173.25.0.2 dev enp2s1
sudo ip route add default via 193.168.2.1

R4

sudo ip route add 172.23.0.0/24 via 173.24.0.1 dev enp2s1 
sudo ip route add 193.168.11.0/24 via 173.24.0.1 dev enp2s1
sudo ip route add default via 193.168.12.1

R5

sudo ip route add 173.23.0.0/24 via 193.168.1.2 dev enp2s1
sudo ip route add 173.24.0.0/24 via 193.168.11.2 dev enp2s0
sudo ip route add 173.25.0.0/24 via 193.168.2.2 dev enp2s3


Persistencia

sudo nano /etc/systemd/system/rutas-estaticas.service

[Unit]
Description=rutas estáticas Jesús Figueroa Roldán
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/rutas.sh
RemainAfterExit=yes
TimeoutStartSec=15

[Install]
WantedBy=multi-user.target

sudo nano /usr/local/bin/rutas.sh

#!/usr/bin/env bash

# Rutas estáticas Jesús Figueroa Roldán

sleep 5

sudo chmod +x /usr/local/bin/rutas.sh

sudo systemctl daemon-reload
sudo systemctl enable rutas-estaticas.service
sudo systemctl start rutas-estaticas.service
```

</details>

<details>
<summary><code>Enrutamiento/Servicio systemd.txt</code></summary>

```bash
sudo nano /usr/local/bin/iptables.sh

sudo chmod +x /usr/local/bin/iptables.sh

sudo nano /etc/systemd/system/iptables-persis.service


[Unit]
Description=iptables persistente Jesús Figueroa Roldán
After=network-pre.target
Before=network.target
Wants=network-pre.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/iptables.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target


sudo systemctl daemon-reload
sudo systemctl enable iptables-persis.service
sudo systemctl start iptables-persis.service
```

</details>

<details>
<summary><code>NAT/DNAT.txt</code></summary>

```bash
R1

DNAT

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.23.0.1 -j DNAT --to 10.0.1.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.1.2 -j DNAT --to 10.0.1.2

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.23.0.1 -j DNAT --to 10.0.1.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.1.2 -j DNAT --to 10.0.1.2

# Servidor MariaDB

sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.23.0.1 -j DNAT --to 10.0.1.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 193.168.1.2 -j DNAT --to 10.0.1.3

R2

DNAT

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.23.0.2 -j DNAT --to 10.0.11.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.11.2 -j DNAT --to 10.0.11.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.24.0.1 -j DNAT --to 10.0.11.2

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.23.0.2 -j DNAT --to 10.0.11.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.11.2 -j DNAT --to 10.0.11.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.24.0.1 -j DNAT --to 10.0.11.2

# Servidor MariaDB

sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.23.0.2 -j DNAT --to 10.0.11.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 193.168.11.2 -j DNAT --to 10.0.11.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.24.0.1 -j DNAT --to 10.0.11.3

R3

DNAT

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.25.0.1 -j DNAT --to 10.0.2.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.2.2 -j DNAT --to 10.0.2.2

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.25.0.1 -j DNAT --to 10.0.2.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.2.2 -j DNAT --to 10.0.2.2

# Servidor MariaDB

sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.25.0.1 -j DNAT --to 10.0.2.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 193.168.2.2 -j DNAT --to 10.0.2.3

R4

DNAT

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.24.0.2 -j DNAT --to 10.0.12.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.12.2 -j DNAT --to 10.0.12.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.25.0.2 -j DNAT --to 10.0.12.2

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.24.0.2 -j DNAT --to 10.0.12.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.12.2 -j DNAT --to 10.0.12.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.25.0.2 -j DNAT --to 10.0.12.2

# Servidor MariaDB

sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.24.0.2 -j DNAT --to 10.0.12.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 193.168.12.2 -j DNAT --to 10.0.12.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.25.0.2 -j DNAT --to 10.0.12.3

R5

DNAT

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.1.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.11.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.12.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.12.1 -j DNAT --to 10.0.13.2

# Servidor PostgreSQL

sudo iptables -t nat -A PREROUTING -p tcp --dport 5432 -d 193.168.1.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 5432 -d 193.168.11.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 5432 -d 193.168.12.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 5432 -d 193.168.12.1 -j DNAT --to 10.0.13.2

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.1.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.11.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.12.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.2.1 -j DNAT --to 10.0.13.2
```

</details>

<details>
<summary><code>NAT/SNAT.txt</code></summary>

```bash
R1

SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o ens1 -j SNAT --to 193.168.1.2
sudo iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o enp2s2 -j SNAT --to 173.23.0.1

R2

SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.11.0/24 -o ens1 -j SNAT --to 193.168.11.2
sudo iptables -t nat -A POSTROUTING -s 10.0.11.0/24 -o enp2s3 -j SNAT --to 173.23.0.2
sudo iptables -t nat -A POSTROUTING -s 10.0.11.0/24 -o enp2s2 -j SNAT --to 173.24.0.1

R3

SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.2.0/24 -o ens1 -j SNAT --to-source 173.25.0.1
sudo iptables -t nat -A POSTROUTING -s 10.0.2.0/24 -o enp2s2 -j SNAT --to-source 193.168.2.2

R4

SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.12.0/24 -o ens1 -j SNAT --to-source 173.24.0.2
sudo iptables -t nat -A POSTROUTING -s 10.0.12.0/24 -o enp2s2 -j SNAT --to-source 193.168.12.2
sudo iptables -t nat -A POSTROUTING -s 10.0.12.0/24 -o enp2s3 -j SNAT --to-source 173.25.0.2

R5

SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.13.0/24 -o enp2s0 -j SNAT --to-source 193.168.11.1
sudo iptables -t nat -A POSTROUTING -s 10.0.13.0/24 -o ens1 -j SNAT --to-source 193.168.1.1
sudo iptables -t nat -A POSTROUTING -s 10.0.13.0/24 -o enp2s2 -j SNAT --to-source 193.168.12.1
sudo iptables -t nat -A POSTROUTING -s 10.0.13.0/24 -o enp2s3 -j SNAT --to-source 193.168.2.1
```

</details>

<details>
<summary><code>Reglas iptables finales/Cortafuegos.txt</code></summary>

```bash
a)

R4

# Petición

sudo iptables -A FORWARD -s 10.0.12.0/24 -d 193.168.1.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.12.0/24 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 193.168.1.2 -d 10.0.12.0/24 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.23.0.1 -d 10.0.12.0/24 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT 

R5

# Petición

sudo iptables -A FORWARD -s 193.168.12.2 -d 193.168.1.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 193.168.1.2 -d 193.168.12.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT 

R1

# Petición

sudo iptables -A FORWARD -s 173.24.0.2 -d 10.0.1.3 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.12.2 -d 10.0.1.3 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta 

sudo iptables -A FORWARD -s 10.0.1.3 -d 173.24.0.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.3 -d 193.168.12.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT 

R2

# Petición

sudo iptables -A FORWARD -s 173.24.0.2 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 173.23.0.1 -d 173.24.0.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT

b) Los indigentes y los superpijos pueden acceder al servidor ssh existente en el distrito 13

R4

# Petición

sudo iptables -A FORWARD -s 10.0.12.0/24 -d 193.168.12.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta 

sudo iptables -A FORWARD -s 193.168.12.1 -d 10.0.12.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

R5

# Petición indigentes

sudo iptables -A FORWARD -s 193.168.12.1 -d 10.0.13.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta indigentes

sudo iptables -A FORWARD -d 10.0.13.2 -d 193.168.12.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Petición superpijos

sudo iptables -A FORWARD -s 193.168.1.1 -d 10.0.13.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta superpijos

sudo iptables -A FORWARD -s 10.0.13.2 -d 193.168.1.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

R1

# Petición

sudo iptables -A FORWARD -s 10.0.1.0/24 -d 193.168.1.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 193.168.1.1 -d 10.0.1.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

d) Clover puede acceder a los servidores de los superpijos, excepto al ssh.

R3

# Petición (por R5)

sudo iptables -A FORWARD -s 10.0.2.3 -d 193.168.1.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.2.3 -d 193.168.1.2 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta (por R5)

sudo iptables -A FORWARD -s 193.168.1.2 -d 10.0.2.3 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.1.2 -d 10.0.2.3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT

# Petición (sin R5)

sudo iptables -A FORWARD -s 10.0.2.3 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.2.3 -d 173.23.0.1 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta (sin R5)

sudo iptables -A FORWARD -s 173.23.0.1 -d 10.0.2.3 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.23.0.1 -d 10.0.2.3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT

R5

# Petición

sudo iptables -A FORWARD -s 193.168.2.2 -d 193.168.1.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.2.2 -d 193.168.1.2 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 193.168.1.2 -d 193.168.2.2 -p tcp --sport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.1.2 -d 193.168.2.2 -p tcp --sport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

R1

# Petición

sudo iptables -A FORWARD -s 193.168.2.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.2.2 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.25.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.25.0.1 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 10.0.1.3 -d 193.168.2.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.3 -d 193.168.2.2 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.2 -d 173.25.0.1 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.2 -d 173.25.0.1 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT


Con R5 off

R4

sudo iptables -A FORWARD -s 173.25.0.1 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.23.0.1 -d 173.25.0.1 -p tcp --dport 80 -m state --state ESTABLISHED -j ACCEPT

R2

sudo iptables -A FORWARD -s 173.25.0.1 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.23.0.1 -d 173.25.0.1 -p tcp --dport 80 -m state --state ESTABLISHED -j ACCEPT



e)

R4

# Petición

sudo iptables -A FORWARD -s 10.0.12.2 -d 193.168.1.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -d 193.168.1.2 -s 10.0.12.2 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

R5

# Petición

sudo iptables -A FORWARD -s 193.168.12.2 -d 193.168.1.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -d 193.168.12.2 -s 193.168.1.2 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

R1

# Petición

sudo iptables -A FORWARD -s 193.168.12.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 10.0.1.2 -d 193.168.12.2 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

Sin R5

R4

# Petición

sudo iptables -A FORWARD -s 10.0.12.2 -d 173.23.0.1 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -d 10.0.12.2 -s 173.23.0.1 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

R2

# Petición

sudo iptables -A FORWARD -s 173.24.0.2 -d 173.23.0.1 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -d 173.24.0.2 -s 173.23.0.1 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

R1

# Petición 

sudo iptables -A FORWARD -s 173.24.0.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 10.0.1.2 -d 173.24.0.2 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

f)

R3

# Petición

sudo iptables -A FORWARD -s 10.0.2.0/24 -d 193.168.2.1 -p tcp --dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 193.168.2.1 -d 10.0.2.0/24 -p tcp --sport 5432 -m state --state ESTABLISHED -j ACCEPT

R5

# Petición

sudo iptables -A FORWARD -s 193.168.2.2 -p tcp --dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 10.0.13.2 -d 193.168.2.2 -p tcp --sport 5432 -m state --state ESTABLISHED -j ACCEPT

g) Katniss ha conseguido incorporar reglas que le permiten (solo a ella) acceder a todos los servidores ssh de todos los distritos. Una vez dentro de ellos puede acceder a los servidores de bases de datos, al haber conseguido infiltrarse en esas redes. Demuestra que puede hacerlo.

R4

SSH a R2

sudo iptables -A FORWARD -s 10.0.12.3 -d 173.24.0.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -d 173.24.0.2 -s 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

R2

sudo iptables -A FORWARD -s 173.24.0.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -s 10.0.11.2 -d 173.24.0.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT


R4 a R3

Reglas R4

sudo iptables -A FORWARD -s 10.0.12.3 -d 173.25.0.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -s 173.25.0.1 -d 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

R3

sudo iptables -A FORWARD -s 173.25.0.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.2.2 -d 173.25.0.1 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

R4 a R5

R4

sudo iptables -A FORWARD -s 10.0.12.3 -d 193.168.12.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.12.1 -d 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

R5

sudo iptables -A FORWARD -s 193.168.12.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.13.2 -d 193.168.12.2 -p tcp --sport 22 -m state ESTABLISHED -j ACCEPT


R4 a R1

R4

sudo iptables -A FORWARD -s 10.0.12.3 -d 193.168.1.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.1.2 -s 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Con R5 apagado

sudo iptables -A FORWARD -s 10.0.12.3 -d 173.23.0.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.23.0.1 -d 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

R2

sudo iptables -A FORWARD -s 173.24.0.2 -d 173.23.0.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -d 173.24.0.2 -s 173.23.0.1 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

R1

sudo iptables -A FORWARD -s 193.168.12.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.2 -d 193.168.12.2 -p tcp --sport 22 -m state ESTABLISHED -j ACCEPT

# Con R5 apagado

sudo iptables -A FORWARD -s 173.24.0.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.2 -d 173.24.0.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT


Cortafuegos nodo Thresh

sudo iptables -A OUTPUT -d 10.0.11.3 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -s 10.0.11.3 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT


h)

R1


sudo iptables -A FORWARD -s 10.0.1.0/24 -d 193.168.1.1 -p tcp --dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -d 10.0.1.0/24 -s 193.168.1.1 -p tcp --sport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT


R5

sudo iptables -A FORWARD -s 193.168.1.2 -p tcp --dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.13.2 -d 193.168.1.2 -p tcp --sport 5432 -m state --state ESTABLISHED -j ACCEPT




for i in $(seq 10000); do psql -h 193.168.1.1 -U jesusfigueroa -c "SELECT 1;" & done


echo "193.168.1.1:5432:pruebajesus:jesusfigueroa:acceso" > ~/.pgpass
chmod 600 ~/.pgpass

seq 10000 | xargs -P 50 -I{} psql -h 193.168.1.1 -U jesusfigueroa -d distrito13 -c "SELECT 1;"

seq 2 | xargs -P 50 -I{} psql -h 193.168.1.1 -U jesusfigueroa -d pruebajesus -c "SELECT 1;"


  host Marvel {
    hardware ethernet 08:00:27:XX:XX:XX; # MAC de Marvel
    fixed-address 10.0.1.2;
  }
  host Glimmer {
    hardware ethernet 08:00:27:YY:YY:YY; # MAC de Glimmer
    fixed-address 10.0.1.3;
  }
}
```

</details>

<details>
<summary><code>Reglas iptables finales/iptables-R1.sh</code></summary>

```bash
#!/usr/bin/env bash

# iptables Jesús Figueroa Roldán

iptables -F
iptables -t nat -F
iptables -Z
iptables -t nat -Z

# SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o ens1 -j SNAT --to-source 193.168.1.2
sudo iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o enp2s2 -j SNAT --to-source 173.23.0.1

# DNAT

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -d 173.23.0.1 -j DNAT --to 10.0.1.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -d 193.168.1.2 -j DNAT --to 10.0.1.2

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.23.0.1 -j DNAT --to 10.0.1.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.1.2 -j DNAT --to 10.0.1.2

# Servidor MariaDB

sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.23.0.1 -j DNAT --to 10.0.1.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 193.168.1.2 -j DNAT --to 10.0.1.3

# Política por defecto

sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP

# Acceso SSH red interna JFR

sudo iptables -A INPUT -p tcp -s 10.0.1.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# a) Jesús Figueroa Roldán (Indigentes - ricos MariaDB)

# Petición

sudo iptables -A FORWARD -s 173.24.0.2 -d 10.0.1.3 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.12.2 -d 10.0.1.3 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta 

sudo iptables -A FORWARD -s 10.0.1.3 -d 173.24.0.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.3 -d 193.168.12.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT 

# b) Jesús Figueroa Roldán (Indigentes - superpijos SSH Distrito 13)

# Petición

sudo iptables -A FORWARD -s 10.0.1.0/24 -d 193.168.1.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 193.168.1.1 -d 10.0.1.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# d) Jesús Figueroa Roldán Clover - Superpijos (excepto SSH)

# Petición

sudo iptables -A FORWARD -s 193.168.2.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.2.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.25.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.25.0.1 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 10.0.1.3 -d 193.168.2.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.2 -d 193.168.2.2 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.3 -d 173.25.0.1 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.2 -d 173.25.0.1 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

# e) Jesús Figueroa Roldán Marvel - Peeta (Puerto 8080 SWeb)

# Petición

sudo iptables -A FORWARD -s 193.168.12.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 10.0.1.2 -d 193.168.12.2 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

# Con R5 apagado

# Petición 

sudo iptables -A FORWARD -s 173.24.0.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 10.0.1.2 -d 173.24.0.2 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

# Katniss - superpijos

sudo iptables -A FORWARD -s 193.168.12.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.2 -d 193.168.12.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -s 193.168.12.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.2 -d 193.168.12.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Con R5 apagado

sudo iptables -A FORWARD -s 173.24.0.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.1.2 -d 173.24.0.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# g) Superpijos - Distrito 13 (PostgreSQL)

sudo iptables -A FORWARD -s 10.0.1.0/24 -d 193.168.1.1 -p tcp --dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -d 10.0.1.0/24 -s 193.168.1.1 -p tcp --sport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT

# Tráfico DHCP Jesús Figueroa Roldán

iptables -A INPUT -i enp2s0 -p udp --dport 67 --sport 68 -j ACCEPT
iptables -A OUTPUT -o enp2s0 -p udp --dport 68 --sport 67 -j ACCEPT
```

</details>

<details>
<summary><code>Reglas iptables finales/iptables-R2.sh</code></summary>

```bash
#!/usr/bin/env bash

# iptables Jesús Figueroa Roldán

iptables -F
iptables -t nat -F
iptables -Z
iptables -t nat -Z

# SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.11.0/24 -o ens1 -j SNAT --to 193.168.11.2
sudo iptables -t nat -A POSTROUTING -s 10.0.11.0/24 -o enp2s2 -j SNAT --to 173.24.0.1
sudo iptables -t nat -A POSTROUTING -s 10.0.11.0/24 -o enp2s3 -j SNAT --to 173.23.0.2

# DNAT

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.23.0.2 -j DNAT --to 10.0.11.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.11.2 -j DNAT --to 10.0.11.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.24.0.1 -j DNAT --to 10.0.11.2

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.23.0.2 -j DNAT --to 10.0.11.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.11.2 -j DNAT --to 10.0.11.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.24.0.1 -j DNAT --to 10.0.11.2

# Servidor MariaDB

sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.23.0.2 -j DNAT --to 10.0.11.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 193.168.11.2 -j DNAT --to 10.0.11.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.24.0.1 -j DNAT --to 10.0.11.3

# Política por defecto

sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP

# Acceso SSH red interna JFR

sudo iptables -A INPUT -s 10.0.11.0/24 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# a) Jesús Figueroa Roldán (Indigentes - ricos MariaDB)

# Petición

sudo iptables -A FORWARD -s 173.24.0.2 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 173.23.0.1 -d 173.24.0.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT

# d) Jesús Figueroa Roldán Clover - Superpijos (excepto SSH)

sudo iptables -A FORWARD -s 173.25.0.1 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -d 173.25.0.1 -s 173.23.0.1 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -s 173.23.0.1 -d 173.25.0.1 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -d 173.23.0.1 -s 173.25.0.1 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT

# e) Jesús Figueroa Roldán Marvel - Peeta (Puerto 8080 SWeb)

# Petición

sudo iptables -A FORWARD -s 173.24.0.2 -d 173.23.0.1 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -d 173.24.0.2 -s 173.23.0.1 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

# g) Jesús Figueroa Roldán (Katniss - Pobres)

# Petición

sudo iptables -A FORWARD -s 173.24.0.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 10.0.11.2 -d 173.24.0.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Katniss - superpijos

sudo iptables -A FORWARD -s 173.24.0.2 -d 173.23.0.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -d 173.24.0.2 -s 173.23.0.1 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

</details>

<details>
<summary><code>Reglas iptables finales/iptables-R3.sh</code></summary>

```bash
#!/usr/bin/env bash

# iptables Jesús Figueroa Roldán

iptables -F
iptables -t nat -F
iptables -Z
iptables -t nat -Z

# SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.2.0/24 -o ens1 -j SNAT --to 173.25.0.1
sudo iptables -t nat -A POSTROUTING -s 10.0.2.0/24 -o enp2s2 -j SNAT --to 193.168.2.2

# DNAT

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.25.0.1 -j DNAT --to 10.0.2.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.2.2 -j DNAT --to 10.0.2.2

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.25.0.1 -j DNAT --to 10.0.2.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.2.2 -j DNAT --to 10.0.2.2

# Servidor MariaDB

sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.25.0.1 -j DNAT --to 10.0.2.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 193.168.2.2 -j DNAT --to 10.0.2.3

# Política por defecto

sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP

# Acceso SSH red interna JFR

sudo iptables -A INPUT -s 10.0.2.0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# d) Jesús Figueroa Roldán Clover - Superpijos (excepto SSH)

# Petición (por R5)

sudo iptables -A FORWARD -s 10.0.2.3 -d 193.168.1.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.2.3 -d 193.168.1.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta (por R5)

sudo iptables -A FORWARD -s 193.168.1.2 -d 10.0.2.3 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.1.2 -d 10.0.2.3 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

# Petición (sin R5)

sudo iptables -A FORWARD -s 10.0.2.3 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.2.3 -d 173.23.0.1 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta (sin R5)

sudo iptables -A FORWARD -s 173.23.0.1 -d 10.0.2.3 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.23.0.1 -d 10.0.2.3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT

# f) Jesús Figueroa Roldán (Fuga militar)

# Petición

sudo iptables -A FORWARD -s 10.0.2.0/24 -d 193.168.2.1 -p tcp --dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 193.168.2.1 -d 10.0.2.0/24 -p tcp --sport 5432 -m state --state ESTABLISHED -j ACCEPT

# Katniss - ricos

sudo iptables -A FORWARD -s 173.25.0.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.2.2 -d 173.25.0.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

</details>

<details>
<summary><code>Reglas iptables finales/iptables-R4.sh</code></summary>

```bash
#!/usr/bin/env bash

# iptables Jesús Figueroa Roldán

iptables -F
iptables -t nat -F
iptables -Z
iptables -t nat -Z

# SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.12.0/24 -o ens1 -j SNAT --to-source 173.24.0.2
sudo iptables -t nat -A POSTROUTING -s 10.0.12.0/24 -o enp2s2 -j SNAT --to-source 193.168.12.2
sudo iptables -t nat -A POSTROUTING -s 10.0.12.0/24 -o enp2s3 -j SNAT --to-source 173.25.0.2

# DNAT

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.24.0.2 -j DNAT --to 10.0.12.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.12.2 -j DNAT --to 10.0.12.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 173.25.0.2 -j DNAT --to 10.0.12.2

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.24.0.2 -j DNAT --to 10.0.12.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.12.2 -j DNAT --to 10.0.12.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 173.25.0.2 -j DNAT --to 10.0.12.2

# Servidor MariaDB

sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.24.0.2 -j DNAT --to 10.0.12.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 193.168.12.2 -j DNAT --to 10.0.12.3
sudo iptables -t nat -A PREROUTING -p tcp --dport 3306 -d 173.25.0.2 -j DNAT --to 10.0.12.3

# Política por defecto

sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP

# Acceso SSH red interna JFR

sudo iptables -A INPUT -s 10.0.12.0/24 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# a) Jesús Figueroa Roldán (Indigentes - Pijos MariaDB)

# Petición

sudo iptables -A FORWARD -s 10.0.12.0/24 -d 193.168.1.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.12.0/24 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -d 10.0.12.0/24 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -d 10.0.12.0/24 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT 

# b) Jesús Figueroa Roldán (Indigentes - superpijos SSH Distrito 13)

# Petición

sudo iptables -A FORWARD -s 10.0.12.0/24 -d 193.168.12.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta 

sudo iptables -A FORWARD -s 193.168.12.1 -d 10.0.12.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# d) Jesús Figueroa Roldán Clover - Superpijos (excepto SSH)

sudo iptables -A FORWARD -s 173.25.0.1 -d 173.23.0.1 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -d 173.25.0.1 -s 173.23.0.1 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -s 173.23.0.1 -d 173.25.0.1 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -d 173.23.0.1 -s 173.25.0.1 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT

# e) Jesús Figueroa Roldán Marvel - Peeta (Puerto 8080 SWeb)

# Petición

sudo iptables -A FORWARD -s 10.0.12.2 -d 193.168.1.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 193.168.1.2 -d 10.0.12.2 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

# Con R5 apagado

# Petición

sudo iptables -A FORWARD -s 10.0.12.2 -d 173.23.0.1 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -d 10.0.12.2 -s 173.23.0.1 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

# g) Jesús Figueroa Roldán (Katniss - Pobres)

# Petición

sudo iptables -A FORWARD -s 10.0.12.3 -d 173.24.0.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 173.24.0.1 -d 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Katniss - Ricos

sudo iptables -A FORWARD -s 10.0.12.3 -d 173.25.0.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.25.0.1 -d 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Katniss - Distrito 13

sudo iptables -A FORWARD -s 10.0.12.3 -d 193.168.12.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.12.1 -d 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Katniss - Superpijos

sudo iptables -A FORWARD -s 10.0.12.3 -d 193.168.1.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.1.2 -d 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Con R5 apagado

sudo iptables -A FORWARD -s 10.0.12.3 -d 173.23.0.1 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 173.23.0.1 -d 10.0.12.3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

</details>

<details>
<summary><code>Reglas iptables finales/iptables-R5.sh</code></summary>

```bash
#!/usr/bin/env bash

# iptables Jesús Figueroa Roldán

iptables -F
iptables -t nat -F
iptables -Z
iptables -t nat -Z

# SNAT

sudo iptables -t nat -A POSTROUTING -s 10.0.13.0/24 -o enp2s0 -j SNAT --to-source 193.168.11.1
sudo iptables -t nat -A POSTROUTING -s 10.0.13.0/24 -o ens1 -j SNAT --to-source 193.168.1.1
sudo iptables -t nat -A POSTROUTING -s 10.0.13.0/24 -o enp2s2 -j SNAT --to-source 193.168.12.1
sudo iptables -t nat -A POSTROUTING -s 10.0.13.0/24 -o enp2s3 -j SNAT --to-source 193.168.2.1

# DNAT

# Servidor SSH

sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.1.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.2.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.11.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -d 193.168.12.1 -j DNAT --to 10.0.13.2

# Servidor PostgreSQL

sudo iptables -t nat -A PREROUTING -p tcp --dport 5432 -d 193.168.1.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 5432 -d 193.168.2.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 5432 -d 193.168.11.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 5432 -d 193.168.12.1 -j DNAT --to 10.0.13.2

# Servidor web

sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.1.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.11.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.12.1 -j DNAT --to 10.0.13.2
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -d 193.168.2.1 -j DNAT --to 10.0.13.2

# Política por defecto

sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP

# Acceso SSH red interna JFR

sudo iptables -A INPUT -s 10.0.13.0/24 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# a) Jesús Figueroa Roldán (Indigentes - ricos MariaDB)

# Petición

sudo iptables -A FORWARD -s 193.168.12.2 -d 193.168.1.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -d 193.168.12.2 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT 

# b) Jesús Figueroa Roldán (Indigentes - superpijos SSH Distrito 13)

# Petición indigentes

sudo iptables -A FORWARD -s 193.168.12.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta indigentes

sudo iptables -A FORWARD -s 10.0.13.2 -d 193.168.12.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Petición superpijos

sudo iptables -A FORWARD -s 193.168.1.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta superpijos

sudo iptables -A FORWARD -s 10.0.13.2 -d 193.168.1.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# d) Jesús Figueroa Roldán Clove - Superpijos (excepto SSH)

# Petición

sudo iptables -A FORWARD -s 193.168.2.2 -d 193.168.1.2 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.2.2 -d 193.168.1.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 193.168.1.2 -d 193.168.2.2 -p tcp --sport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.1.2 -d 193.168.2.2 -p tcp --sport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# e) Jesús Figueroa Roldán Marvel - Peeta (Puerto 8080 SWeb)

# Petición

sudo iptables -A FORWARD -s 193.168.12.2 -d 193.168.1.2 -p tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -d 193.168.12.2 -s 193.168.1.2 -p tcp --sport 8080 -m state --state ESTABLISHED -j ACCEPT

# f) Jesús Figueroa Roldán (Fuga militar)

# Petición

sudo iptables -A FORWARD -s 193.168.2.2 -p tcp --dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT

# Respuesta

sudo iptables -A FORWARD -s 10.0.13.2 -d 193.168.2.2 -p tcp --sport 5432 -m state --state ESTABLISHED -j ACCEPT

# Katniss - Distrito 13

sudo iptables -A FORWARD -s 193.168.12.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 10.0.13.2 -d 193.168.12.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Katniss - superpijos

sudo iptables -A FORWARD -s 193.168.12.2 -d 193.168.1.2 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -s 193.168.1.2 -d 193.168.12.2 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

# Ataque Dos Superpijos

sudo iptables -I FORWARD 1 -p tcp -s 193.168.1.2 -d 10.0.13.2 --dport 5432 -m connlimit --connlimit-above 10 -j REJECT
sudo iptables -I FORWARD 2 -p tcp -s 193.168.1.2 -d 10.0.13.2 --dport 5432 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -I FORWARD 3 -p tcp -s 10.0.13.2 -d 193.168.1.2 --sport 5432 -m state --state ESTABLISHED -j ACCEPT
```

</details>

