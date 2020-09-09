---
layout: post
title:  "Cómo configurar ip estática en Ubuntu Server"
date:   2020-06-17 00:00:00 +0200
tipue_search_active: true
excerpt_separator: <!--end_excerpt-->
tags: OS
---
Me pasa bastante tener que levantar VM con ubuntu, que es un poco peculiar y he decidido hacer este pequeño paso a paso para configurarlo. 

<!--end_excerpt-->

# Configurar IP estática en Ubuntu Server

En Ubuntu, si utilizas una versión igual o superior a 18.04, verás que la forma de configurar tus tarjetas de red ha cambiado ligeramente. Ahora, la configuración es algo mas sencilla y se puede hacer con un fichero YAML de la siguente manera:

## Identificar el nombre de las interfaces de red disponibles

```bash
enrique@ubuntuk8sarc:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:01:f3:61 brd ff:ff:ff:ff:ff:ff
```

> NOTA: En este caso, quiero configurar eth0

## Configurar eth0

En este caso, la configuración se realiza con el YAML de configuración de [netplan](https://netplan.io/examples/) , que básicamente se encuentra en _/etc/netplan/_

```bash
enrique@ubuntuk8sarc:~$ cd /etc/netplan/
enrique@ubuntuk8sarc:/etc/netplan$ ls
00-installer-config.yaml
```

Editamos el fichero que tenemos

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

y le añadimos la configuración que queramos. En mi caso es esta:

```yaml
# This is the network config 
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.100.91/24
      gateway4: 192.168.100.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

> NOTA: Es bastante autoexplicativo, pero si quieres mas información entra en la [ayuda de netplan](https://netplan.io/examples/)

## Aplica los cambios

Ahora solo te queda aplicar cambios

```bash
sudo netplan apply
```

## Comprueba los cambios

Verifica que efectivamente todo funciona como debe

```bash
ip addr show dev eth0
```