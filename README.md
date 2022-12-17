# SERVIDOR DHCP

## Imágenes necesarias

Para montar el servidor DHCP utilizaremos ya una imagen montada que incluye dicho servicio llamada [dhcpd](https://hub.docker.com/r/networkboot/dhcpd/) al igual que utilizaré como soporte la documentación que nos trae a mayores, bastante completa y útil ya que nos indica todos los parámetros de configuración.

## Docker-Compose

Para poder levantar el servicio primeramente debemos crear y generar nuestro [```docker-compose.yml```](https://github.com/ndiazdossantos/servidor-DHCP/blob/master/docker-compose.yml).


```yml

services:
  asir_dhcp_server:
    container_name: asir_dhcp_server
    network_mode: "host"
    image: networkboot/dhcpd
    volumes:
      - ./data:/data

```

En el especificamos:

* **Services** : Declaramos el servicio el cual vamos a generar.

* **asir_dhcp_server** : Asignamos un nombre al servicio que vamos a "levantar".

* **container_name** : Definimos el nombre del contenedor que vamos a "levantar".

* **network_mode** : Debemos especificar de que manera vamos a levantar dicho servicio, para que cuando levantemos el servicio este no esté aislado de la red de nuestro anfitrión como pasa en un contenedor de manera default.

* **image** : Indicamos el nombre de la imagen a utilizar, si no la tenemos descargada esta se descargará automáticamente.

* **volumes** : Mapeamos el volumen de la imagen que estamos descargando con un directorio de nuestro local para poder realizar nuestras configuraciones propias.

## Mapeo carpeta data

Localmente en el directorio del proyecto debemos crear la carpeta [```data```](https://github.com/ndiazdossantos/servidor-DHCP/tree/master/data)

Esta carpeta ya la tenemos mapeada en el [```docker-compose.yml```](https://github.com/ndiazdossantos/servidor-DHCP/blob/master/docker-compose.yml).


## Configuración del servicio

Primeramente deberemos ejecutar en el terminal desde el directorio de dicho servicio el comando:

```
docker-compose up
```

De esta manera en nuestra carpeta [```data```](https://github.com/ndiazdossantos/servidor-DHCP/tree/master/data) se generarán diferentes ficheros, entre ellos el que más nos interesa que es [```dhcpd.conf```](https://github.com/ndiazdossantos/servidor-DHCP/blob/master/data/dhcpd.conf) donde en el podremos especificar la configuración de nuestro servidor DHCP.



*En caso de tener un error al levantar el servicio de que el fichero [```dhcpd.leases```](https://github.com/ndiazdossantos/servidor-DHCP/blob/master/data/dhcpd.leases) no existe podremos generarlo nosotros manualmente. En podemos observar las IPS asignadas y el DUID de nuestro servicio*


```yml
subnet 10.0.2.0 netmask 255.255.255.0 {    
    range 10.0.2.100 10.0.2.200;    
    option routers 10.0.2.1;    
    option domain-name-servers 8.8.8.8;    
}

subnet 192.168.76.0 netmask 255.255.255.0 {
    range 192.168.76.100 192.168.76.200;
    option routers 192.168.70.1;
    option domain-name-servers 8.8.8.8;
}

subnet 10.0.9.0 netmask 255.255.255.0 {
    range 10.0.9.200 10.0.9.202

    option domain-name-servers 8.8.8.8, 193.146.96.2, 193.146.96.3;
    option domain-name "asir.tv";
    option routers 10.0.0.9.254;
    option subnet-mask 255.255.255.0;
    option broadcast-address 10.0.9.255;

    default-lease-time 86400;
    max-lease-time 172800;

    group{

        default-lease-time 604800;
        max-lease-time 691200;
        host pache {
            hardware ethernet 00:10:5a:f1:35:87;
            fixed-address 10.0.9.37;
        }

    }

}

```

En el estamos indicando:

* 2 subredes diferentes


En el campo concreto de nuestra subred *10.0.0.9* con máscara *255.255.255.0* especificamos:

* **range**: Estamos especificando el rango de IPS que podemos asignar, es decir, desde la IP "x" puedes asignar IPS hasta la IP "y".

* **option domain-name-servers** : Especificamos con que IPS queremos que resuelva nuestro DNS, de manera default indicaremos la de google y luego nuestras IPS internas de la red.

* **option domain-name** : En el especificamos nuestro nombre de dominio.

* **option routers** :  Indicamos la IP de nuestro router como puerta de salida.

* **option subnet-mask** : Señalamos la máscara de nuestra subred.

* **option broadcast-address** : Indicamos nuestra IP de broadcast.

* **default-lease-time** : Tiempo mínimo o por defecto de refresco de la IP que asignamos.

* **max-lease-time**: Tiempo máximo establecido para refresco de las IP.


## Asignación IP y configuración para equipos concretos


Si nosotros queremos asignar IPS de manera automática a ciertos equipos en concreto o queremos indicar un rango específico para dichos equipos sin ser todos aquellos que pertenecen a la red haremos incapíe en la siguiente parte del códgo de configuración de [```dhcpd.conf```](https://github.com/ndiazdossantos/servidor-DHCP/blob/master/data/dhcpd.conf).


```yml

 group{

        default-lease-time 604800;
        max-lease-time 691200;
        host pache {
            #hardware ethernet 00:10:5a:f1:35:87;
            #fixed-address 10.0.9.37;
        }

```

En este caso estamos especificando para la asignación los tiempos de refresco de las IP, a mayores de que estamos definiendo una asignación concreta de una IP a "x" equipo de manera que cuando identifique la MAC de este equipo solicitando la asignación de una IP este le establecerá la que hemos especificado.

Para ello:

```yml

        host pespecificamosNombre {
            hardware ethernet #indicamosLaMAC;
            fixed-address #indicamosLaIP;
        }


```


## Como realizar la comprobación

En un caso de un entorno de desarrollo podemos crear una nueva máquina mediante VirtualBox, cuando creamos la máquina si vamos al apartado de tarjetas de red deberemos tener:

* **Adaptador 1 en NAT**

_![IMAGEN_NAT](https://i.imgur.com/EuHLQU8.png)_

* **Adaptador 2 en Puente**

_![IMAGEN_ADAPTADOR_PUENTE](https://i.imgur.com/gMJac1y.png)_

De esta manera nosotros ya conocemos la dirección MAC  a poder especificar, de todas formas podemos consultarlo de manera interna con el comando *ip addr* y desde ahi consultar dicha información.

Para eliminar la **IP** que tenemos asignada utilizamos el comando:

```
sudo dhclient -r
```

Para que solicite una nueva ip ejecutamos el comando:

```
sudo dhclient
```

Si nos encontramos en **Windows** y realizamos la misma configuración en VirtualBox en principio ejecutando el comando *ipconfig* y haciendo la asignación de la IP por DHCP esta se refrescaría automáticamente, en caso de no hacerlo podemos ejecutar el comando *ipconfig/renew*.


*Dicha configuración fue realizara para desplegar en el aula del centro Daniel Castelao, por circunstancias médicas no he podido realizar el despliegue y por lo tanto mostrar las comprobaciones de asignaciones de IP a los dispositivos, si está realizada la comprobación del funcionamiento del servidor DHCP, queda pendiente de añadir el restante de la información, pero todos los pasos están indicados.*


[README.md](README.md) de Noé Díaz Dos Santos para el repositorio [Servidor DHCP](https://github.com/ndiazdossantos/servidor-DHCP)
