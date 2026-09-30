# Configuración de un servidor DHCP en Ubuntu

En estos apuntes se recoge paso a paso el proceso realizado para configurar una máquina Ubuntu como servidor DHCP y otra máquina Ubuntu como cliente. El servidor tendrá una IP fija y será el encargado de asignar automáticamente una dirección IP al cliente.

---

## 1. Configurar las tarjetas de red en VirtualBox

Antes de empezar configuramos las tarjetas de red de las dos máquinas virtuales.

En la máquina servidor José configuramos dos adaptadores.

El Adaptador 1 lo ponemos como **Red interna** y utilizamos el nombre:

```text
red-dhcp
```

Este adaptador será el que utilizaremos para comunicar el servidor con el cliente y para trabajar con DHCP.

El Adaptador 2 lo configuramos como:

```text
NAT
```

Este segundo adaptador permite que el servidor tenga conexión a Internet.

Por tanto, el servidor queda de la siguiente forma:

```text
Adaptador 1 → Red interna → red-dhcp
Adaptador 2 → NAT
```

En la máquina cliente Liza configuramos también el Adaptador 1 como:

```text
Red interna → red-dhcp
```

Es importante que las dos máquinas tengan exactamente el mismo nombre de red interna para que puedan comunicarse.

También podemos añadir un segundo adaptador NAT al cliente para que tenga conexión a Internet.

---

## 2. Comprobar las interfaces de red del servidor

Una vez iniciamos el servidor comprobamos las interfaces de red disponibles.

Ejecutamos:

```bash
ip a
```

En nuestro caso tenemos dos interfaces principales.

La interfaz:

```text
enp0s3
```

corresponde a la red interna `red-dhcp`.

La interfaz:

```text
enp0s8
```

corresponde al adaptador NAT que utilizamos para tener conexión a Internet.

---

## 3. Configurar una IP fija en el servidor

El servidor DHCP necesita tener una dirección IP fija.

En nuestro caso queremos utilizar:

```text
172.16.5.150/24
```

Abrimos el archivo de configuración de Netplan:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Configuramos las interfaces de esta forma:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 172.16.5.150/24
    enp0s8:
      dhcp4: true
```

En `enp0s3` ponemos:

```text
dhcp4: false
```

porque no queremos que el servidor reciba automáticamente una dirección IP. Queremos que siempre utilice la IP fija `172.16.5.150`.

En cambio, en `enp0s8` ponemos:

```text
dhcp4: true
```

porque esta interfaz corresponde al adaptador NAT y puede recibir automáticamente su configuración.

Guardamos el archivo con:

```text
Ctrl + O
Enter
Ctrl + X
```

Después aplicamos la nueva configuración:

```bash
sudo netplan apply
```

Comprobamos el resultado:

```bash
ip a
```

En `enp0s3` debe aparecer:

```text
172.16.5.150/24
```

---

## 4. Instalar el servidor DHCP

Primero actualizamos la lista de paquetes disponibles:

```bash
sudo apt update
```

Después instalamos el servidor DHCP:

```bash
sudo apt install isc-dhcp-server -y
```

`isc-dhcp-server` es el servicio que permitirá que nuestra máquina José entregue automáticamente direcciones IP a los clientes de la red.

---

## 5. Indicar la interfaz que utilizará DHCP

Tenemos que indicarle al servidor DHCP por qué interfaz debe trabajar.

Abrimos el archivo:

```bash
sudo nano /etc/default/isc-dhcp-server
```

Buscamos:

```text
INTERFACESv4=""
```

Y lo cambiamos por:

```text
INTERFACESv4="enp0s3"
```

También dejamos IPv6 vacío:

```text
INTERFACESv6=""
```

Utilizamos `enp0s3` porque es la interfaz que está conectada a nuestra red interna `red-dhcp`.

Guardamos:

```text
Ctrl + O
Enter
Ctrl + X
```

---

## 6. Configurar la red DHCP

Ahora tenemos que indicar qué red y qué direcciones IP puede entregar nuestro servidor.

Abrimos:

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Al final del archivo añadimos:

```conf
subnet 172.16.5.0 netmask 255.255.255.0 {
    range 172.16.5.151 172.16.5.200;
    option routers 172.16.5.150;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    default-lease-time 600;
    max-lease-time 7200;
}
```

La línea:

```conf
subnet 172.16.5.0 netmask 255.255.255.0
```

indica que vamos a trabajar con la red `172.16.5.0/24`.

La línea:

```conf
range 172.16.5.151 172.16.5.200;
```

indica el rango de direcciones que puede entregar DHCP.

Por tanto, podrá asignar direcciones desde:

```text
172.16.5.151
```

hasta:

```text
172.16.5.200
```

La línea:

```conf
option routers 172.16.5.150;
```

indica la dirección configurada como router para los clientes.

Los servidores DNS utilizados son:

```conf
option domain-name-servers 8.8.8.8, 8.8.4.4;
```

También configuramos el tiempo de las concesiones DHCP:

```conf
default-lease-time 600;
max-lease-time 7200;
```

Guardamos:

```text
Ctrl + O
Enter
Ctrl + X
```

---

## 7. Comprobar que la configuración DHCP no tiene errores

Antes de iniciar el servicio comprobamos que el archivo de configuración esté bien escrito.

Ejecutamos:

```bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
```

Si el comando termina y vuelve al terminal sin mostrar ningún error, significa que la configuración es correcta.

---

## 8. Reiniciar el servidor DHCP

Después de modificar la configuración reiniciamos el servicio:

```bash
sudo systemctl restart isc-dhcp-server
```

Comprobamos su estado:

```bash
sudo systemctl status isc-dhcp-server
```

Si aparece:

```text
Active: active (running)
```

significa que el servidor DHCP está funcionando correctamente.

Para salir de esta pantalla pulsamos:

```text
q
```

---

## 9. Configurar el cliente para utilizar DHCP

En la máquina cliente Liza configuramos el Adaptador 1 como:

```text
Red interna → red-dhcp
```

La configuración IPv4 del cliente debe estar en automático para que pueda solicitar una dirección al servidor DHCP.

Después comprobamos las direcciones del cliente:

```bash
ip a
```

Una dirección obtenida mediante DHCP aparece como `dynamic`.

Por ejemplo:

```text
inet 172.16.5.152/24 ... dynamic
```

Esto significa que el servidor José ha entregado automáticamente esa dirección al cliente.

---

## 10. Comprobar la comunicación entre cliente y servidor

Desde la máquina cliente comprobamos si podemos comunicarnos con José.

Ejecutamos:

```bash
ping -c 4 172.16.5.150
```

En nuestra prueba obtuvimos:

```text
4 packets transmitted, 4 received, 0% packet loss
```

Esto significa que el cliente puede comunicarse correctamente con el servidor.

También podemos utilizar:

```bash
ping 172.16.5.150
```

Para detener el ping pulsamos:

```text
Ctrl + C
```

---

## 11. Comprobar el acceso SSH

Desde la máquina cliente Liza nos conectamos al servidor José mediante SSH.

Ejecutamos:

```bash
ssh jose@172.16.5.150
```

Antes de conectarnos estamos en:

```text
liza@liza-VirtualBox:~$
```

Cuando la conexión se realiza correctamente pasamos a:

```text
jose@jose-VirtualBox:~$
```

Esto significa que hemos accedido remotamente desde el cliente al servidor.

Como anteriormente configuramos las claves SSH, podemos entrar sin tener que escribir la contraseña del usuario José.

Para salir del servidor:

```bash
exit
```

---

## 12. Comprobar el acceso a Internet

Para que el cliente pueda utilizar la red interna y también tener Internet añadimos un segundo adaptador en VirtualBox.

La configuración del cliente queda:

```text
Adaptador 1 → Red interna → red-dhcp
Adaptador 2 → NAT
```

Podemos comprobar las interfaces con:

```bash
ip a
```

La interfaz de la red interna tendrá una dirección `172.16.5.x`.

La interfaz NAT tendrá una dirección parecida a:

```text
10.0.3.x
```

Para comprobar la conexión a Internet podemos ejecutar:

```bash
ping -c 4 8.8.8.8
```

También comprobamos que funciona la resolución de nombres:

```bash
ping -c 4 google.com
```

---

## 13. Comprobar el resultado final

En el servidor José comprobamos las direcciones:

```bash
ip a
```

La configuración final del servidor es:

```text
enp0s3 → 172.16.5.150/24 → Red interna
enp0s8 → 10.0.3.x        → NAT
```

Comprobamos también el servidor DHCP:

```bash
sudo systemctl status isc-dhcp-server
```

Debe aparecer:

```text
Active: active (running)
```

En el cliente Liza comprobamos:

```bash
ip a
```

La dirección obtenida mediante DHCP aparecerá como `dynamic`.

Comprobamos la comunicación con el servidor:

```bash
ping -c 4 172.16.5.150
```

Comprobamos el acceso SSH:

```bash
ssh jose@172.16.5.150
```

Y comprobamos Internet:

```bash
ping -c 4 google.com
```

---

## Resultado final

Al terminar la práctica tenemos dos máquinas conectadas mediante una red interna de VirtualBox.

El servidor José utiliza la dirección fija:

```text
172.16.5.150
```

y ejecuta el servicio DHCP.

El cliente Liza solicita automáticamente una dirección IP al servidor mediante DHCP.

La comunicación queda de la siguiente forma:

```text
Servidor José
172.16.5.150
      |
      | Red interna: red-dhcp
      | DHCP
      |
      v
Cliente Liza
172.16.5.x
```

También hemos comprobado que existe comunicación entre las máquinas mediante `ping` y que podemos acceder desde Liza al servidor José mediante SSH.

Con el segundo adaptador NAT, las máquinas también pueden mantener conexión a Internet.
