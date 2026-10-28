---
title: "Enrutamiento dinámico con OSPF"
date: 2026-05-08T10:00:00+01:00
description: "b) Usando Zebra, Quagga o FRR como herramienta y OSPFv2 o BGP como protocolo, monta enrutamiento dinámico sobre los routers Linux del escenario de la práctica 14: The Hunger Games: Sinsajo con los…"
menu:
  sidebar:
    name: "Enrutamiento dinámico (OSPF)"
    identifier: par-enrutamiento-dinamico
    parent: par
    weight: 260508
hero: cover.png
tags: ["OSPF", "Enrutamiento dinámico", "Cisco", "Linux"]
categories: ["Prácticas"]
draft: false
---

> 📚 **1º ASIR · Planificación y Administración de Redes** · Tema 5: Nivel de red · Curso 2025/2026

## Escenario Linux

b\) Usando Zebra, Quagga o FRR como herramienta y OSPFv2 o BGP como protocolo, monta enrutamiento dinámico sobre los routers Linux del escenario de la práctica 14: The Hunger Games: Sinsajo con los costes asignados en el apartado anterior.

Debes explicar las rutas elegidas por el algoritmo en cada uno de los casos, para ilustrar esta explicación recuerda que puedes usar traceroute o mostrar las tablas de enrutamiento directamente.

Comprueba también que pasa cuando cambias el coste de una ruta o cuando apagas el Router 5.

### Preparación escenario

Lo primero que voy a realizar es poner la política ACCEPT pero solo para instalar la herramienta FRR, ya que al ser un entorno virtualizado para acceder a Internet GNS3 tiene un elemento llamado NAT que hace de “puente” entre la red privada y el exterior, en la realidad se implementarían las reglas necesarias para poder instalar paquetes. Una vez que instale esta herramienta vuelvo a ejecutar el script con todas las reglas iptables. Configuro una tarjeta de red por DHCP del NAT y a partir de ahí instalo el paquete. Este proceso se repite en todos los routers.

R1

![Captura 1](img/captura-001.png)

![Captura 2](img/captura-002.png)

![Captura 3](img/captura-003.png)

![Captura 4](img/captura-004.png)

R2

![Captura 5](img/captura-005.png)

![Captura 6](img/captura-006.png)

![Captura 7](img/captura-007.png)

![Captura 8](img/captura-008.png)

R3

![Captura 9](img/captura-009.png)

![Captura 10](img/captura-010.png)

![Captura 11](img/captura-011.png)

![Captura 12](img/captura-012.png)

R4

![Captura 13](img/captura-013.png)

![Captura 14](img/captura-014.png)

![Captura 15](img/captura-015.png)

![Captura 16](img/captura-016.png)

![Captura 17](img/captura-017.png)

R5

![Captura 18](img/captura-018.png)

![Captura 19](img/captura-019.png)

![Captura 20](img/captura-020.png)

![Captura 21](img/captura-021.png)

![Captura 22](img/captura-022.png)

### Permitir tráfico OSPF entre routers

Una vez que he instalado la herramienta de enrutamiento dinámico Frr al tener una política DROP por defecto en iptables hay que permitir el tráfico OSPF entre los routers del escenario. Para ello he implementado dos reglas (“petición” y “respuesta”) y el 89 no es el número de puerto sino el protocolo 89 que es el número asignado oficialmente a OSPF. Este proceso se va a repetir en todos los routers.

![Captura 23](img/captura-023.png)

![Captura 24](img/captura-024.png)

![Captura 25](img/captura-025.png)

![Captura 26](img/captura-026.png)

![Captura 27](img/captura-027.png)

### Configuración protocolos de enrutamiento

Para configurar los protocolos de enrutamiento he borrado el scripts con las tablas de enrutamiento a las diferentes redes y el servicio de systemd el cuál hacia persistente dichas rutas. Y a partir de esto voy a configurar OSPF como protocolo de enrutamiento. Como vemos tenemos más protocolos de enrutamiento don esta herramienta.

![Captura 28](img/captura-028.png)

R1

Tras hacer los cambios vamos a reiniciar el servicio y vamos a acceder a la consola de esta herramienta.

![Captura 29](img/captura-029.png)

Una vez en la consola vamos a definir un router id para ospf y las redes que tiene conectadas. En este caso vamos a trabajar en el area 0. (Este proceso se va a repetir en todos los routers pero cambiando la definición de las redes).

![Captura 30](img/captura-030.png)

R2

![Captura 31](img/captura-031.png)

![Captura 32](img/captura-032.png)

R3

![Captura 33](img/captura-033.png)

![Captura 34](img/captura-034.png)

R4

![Captura 35](img/captura-035.png)

![Captura 36](img/captura-036.png)

R5

![Captura 37](img/captura-037.png)

![Captura 38](img/captura-038.png)

### Rutas generadas

Tras definir las redes y esperar unos 40 segundos a que el protocolo mande los hello packets para que sepan que están usando dicho protocolo tendremos generadas las tablas de enrutamiento. Las rutas generadas sin asignar costes es el camino con el costo total más bajo, en este caso al no configurar costes el router mira la velocidad de los enlaces y asigna un número. Si para llegar a una red hay 2 caminos posibles y los 2 tienen el mismo coste OSPF añade los 2 caminos y a la hora de enrutar los paquetes hace un balanceo de carga en función del tráfico que haya.

R1

![Captura 39](img/captura-039.png)

R2

![Captura 40](img/captura-040.png)

R3

![Captura 41](img/captura-041.png)

R4

![Captura 42](img/captura-042.png)

R5

![Captura 43](img/captura-043.png)

#### Con R5 apagado

Una vez que apagamos R5 y los routers vean que contesta a los hello packets recalcularán las rutas.

![Captura 44](img/captura-044.png)

Aquí podremos ver el contenido de un paquete hello de OSPF.

![Captura 45](img/captura-045.png)

### Nuevas rutas sin R5

Tras apagar R5 se han generado nuevas rutas.

R1

![Captura 46](img/captura-046.png)

R2

![Captura 47](img/captura-047.png)

R3

![Captura 48](img/captura-048.png)

R4

![Captura 49](img/captura-049.png)

### Asignación de costes

Ahora vamos a proceder a asignar costes, lo voy a realizar de tal manera que los costes más caros sean tirando hacia R5 y los más baratos por los extremos.

R1

![Captura 50](img/captura-050.png)

![Captura 51](img/captura-051.png)

R2

![Captura 52](img/captura-052.png)

![Captura 53](img/captura-053.png)

R3

![Captura 54](img/captura-054.png)

![Captura 55](img/captura-055.png)

R4

![Captura 56](img/captura-056.png)

![Captura 57](img/captura-057.png)

R5

![Captura 58](img/captura-058.png)

![Captura 59](img/captura-059.png)

![Captura 60](img/captura-060.png)

### Rutas generadas con costes

El númro que vemos entre corchetes es el coste total de todos los enlaces para llegar a esa subred. En este caso si vemos las tablas de enrutamientos los caminos no suelen ir por R5 ya que he asignado costes más grandes que en los otros enlaces aunque se puede dar el caso.

R1

![Captura 61](img/captura-061.png)

R2

![Captura 62](img/captura-062.png)

R3

![Captura 63](img/captura-063.png)

R4

![Captura 64](img/captura-064.png)

R5

![Captura 65](img/captura-065.png)

### Prueba conectividad

Para poder comprobar que hay conectividad entre todas las redes tengo que añadir las reglas para permitir el tráfico ICMP. Este proceso se repite en todos los routers.

![Captura 66](img/captura-066.png)

![Captura 67](img/captura-067.png)

![Captura 68](img/captura-068.png)

![Captura 69](img/captura-069.png)

![Captura 70](img/captura-070.png)

### Prueba conectividad

![Captura 71](img/captura-071.png)

![Captura 72](img/captura-072.png)

### Traceroute

![Captura 73](img/captura-073.png)

![Captura 74](img/captura-074.png)

![Captura 75](img/captura-075.png)

![Captura 76](img/captura-076.png)

## Escenario Cisco

a\) Usando OSPFv2, monta enrutamiento dinámico sobre los routers CISCO del escenario de la práctica 12: The Hunger Games. Posteriormente, realiza el mismo ejercicio asignando costes a cada uno de los enlaces.

Debes explicar las rutas elegidas por el algoritmo en cada uno de los casos, para ilustrar esta explicación recuerda que puedes usar traceroute o mostrar las tablas de enrutamiento directamente.

Comprueba también que pasa cuando cambias el coste de una ruta o cuando apagas el Router 5.

### Configuración OSPF

Ahora vamos a realizar los mismos pasos y es definir las redes que tienen los routers conectadas a él para que el protocolo calcule las rutas. En este escenario no voy a tener problema con las ACLS ya que en la práctica 12 puse un permit ip any any para que funcione todo lo demás por lo que paras las ACLS no voy a necesitar configuracioón adicional.También es importante destacar que al configurar el enrutamiento dinámico en Cisco en lugar de usar la másca de red de la subred tendremos que usar la máscara de Wildcard (lo mismo que en ACL).

![Captura 77](img/captura-077.png)

![Captura 78](img/captura-078.png)

![Captura 79](img/captura-079.png)

![Captura 80](img/captura-080.png)

![Captura 81](img/captura-081.png)

![Captura 82](img/captura-082.png)

### Rutas generadas

Tras definir todas las redes el protocolo OSPF ha generado una ruta en función al algoritmo que usa. Las rutas con un 0 a la izquierda significa que son routers generadas por OSPF.

![Captura 83](img/captura-083.png)

![Captura 84](img/captura-084.png)

![Captura 85](img/captura-085.png)

![Captura 86](img/captura-086.png)

![Captura 87](img/captura-087.png)

### Prueba conectividad

![Captura 88](img/captura-088.png)

![Captura 89](img/captura-089.png)

![Captura 90](img/captura-090.png)

![Captura 91](img/captura-091.png)

![Captura 92](img/captura-092.png)

### Rutas con R5 apagado

Una vez apagado R5 el protocolo mandará los hello packets y al no recibir respuesta tras un tiempo procederá a cambiar las rutas a las diferentes redes.

![Captura 93](img/captura-093.png)

![Captura 94](img/captura-094.png)

![Captura 95](img/captura-095.png)

![Captura 96](img/captura-096.png)

![Captura 97](img/captura-097.png)

#### Prueba conectividad

![Captura 98](img/captura-098.png)

![Captura 99](img/captura-099.png)

![Captura 100](img/captura-100.png)

![Captura 101](img/captura-101.png)

![Captura 102](img/captura-102.png)

#### Traceroute

![Captura 103](img/captura-103.png)

![Captura 104](img/captura-104.png)

### Asignación de costes

Para la asignación de costes voy a seguir el mismo criterio que en el escenario de Linux es decir que las rutas más caras en cuanto a coste sean por R5.

![Captura 105](img/captura-105.png)

![Captura 106](img/captura-106.png)

![Captura 107](img/captura-107.png)

![Captura 108](img/captura-108.png)

![Captura 109](img/captura-109.png)

### Rutas generadas

En las rutas tenemos entre corchetes dos valores importantes, el primero que es la fiabilidad de la ruta que OSPF por defecto tiene 110 y el 2 valor que es el coste total para llegar a la red destino.

![Captura 110](img/captura-110.png)

![Captura 111](img/captura-111.png)

![Captura 112](img/captura-112.png)

![Captura 113](img/captura-113.png)

![Captura 114](img/captura-114.png)

#### Traceroute

![Captura 115](img/captura-115.png)

![Captura 116](img/captura-116.png)

![Captura 117](img/captura-117.png)

![Captura 118](img/captura-118.png)

![Captura 119](img/captura-119.png)

### Bibliografía

<https://www.ticarte.com/contenido/quagga-enrutamiento-ospf>

<https://frrouting.org/doc/>

<https://docs.oracle.com/cd/E63244_01/html/E63245/z40025071872988.html>
