# Configuración de DHCP desde la máquina cliente

En esta práctica utilizamos dos máquinas virtuales Ubuntu.

La máquina José funcionará como servidor DHCP y tendrá la dirección IP fija:

```text
172.16.5.150
```

La máquina Liza funcionará como cliente.

La diferencia importante es que la instalación y configuración del servidor DHCP de José la realizaremos remotamente desde la máquina cliente Liza mediante SSH.

De esta forma trabajamos desde el cliente, entramos remotamente al servidor y desde ahí instalamos y configuramos DHCP.

---

## 1. Preparar la máquina cliente

Primero iniciamos la máquina Liza.

Comprobamos su configuración de red:

```bash
ip a
```

La interfaz que utilizamos para comunicarnos con José es:

```text
enp0s3
```

Las dos máquinas deben estar conectadas a la misma red interna de VirtualBox:

```text
red-dhcp
```

---

## 2. Comprobar la comunicación con el servidor

Antes de conectarnos por SSH comprobamos que Liza puede comunicarse con José.

Desde Liza ejecutamos:

```bash
ping -c 4 172.16.5.150
```

`172.16.5.150` es la dirección IP del servidor José.

Si recibimos respuestas como:

```text
64 bytes from 172.16.5.150
```

significa que existe comunicación entre las dos máquinas.

---

## 3. Conectarnos desde el cliente al servidor mediante SSH

Desde la terminal de Liza ejecutamos:

```bash
ssh jose@172.16.5.150
```

Antes de conectarnos, nuestro terminal muestra:

```text
liza@liza-VirtualBox:~$
```

Cuando entramos correctamente mediante SSH pasa a mostrar:

```text
jose@jose-VirtualBox:~$
```

Esto significa que seguimos utilizando físicamente la máquina cliente Liza, pero los comandos que escribamos a partir de este momento se ejecutarán en el servidor José.

Como anteriormente configuramos la autenticación mediante claves SSH, podemos entrar sin introducir la contraseña de José.

---

## 4. Instalar DHCP en el servidor desde el cliente

Una vez conectados mediante SSH veremos:

```text
jose@jose-VirtualBox:~$
```

Aunque estamos trabajando desde la máquina Liza, en este momento estamos controlando remotamente el servidor José.

Primero actualizamos la lista de paquetes del servidor:

```bash
sudo apt update
```

Después instalamos el servidor DHCP:

```bash
sudo apt install isc-dhcp-server -y
```

Por tanto, `isc-dhcp-server` se instala realmente en José, pero hemos realizado la instalación remotamente desde Liza utilizando SSH.

---

## 5. Indicar la interfaz que utilizará el servidor DHCP

Seguimos conectados desde Liza al servidor José mediante SSH.

Abrimos:

```bash
sudo nano /etc/default/isc-dhcp-server
```

Configuramos:

```text
INTERFACESv4="enp0s3"
INTERFACESv6=""
```

`enp0s3` es la interfaz de José conectada a la red interna `red-dhcp`.

Guardamos:

```text
Ctrl + O
Enter
Ctrl + X
```

---

## 6. Configurar DHCP desde el cliente

Seguimos trabajando desde Liza dentro de José mediante SSH.

Abrimos el archivo de configuración:

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Añadimos:

```conf
subnet 172.16.5.0 netmask 255.255.255.0 {
    range 172.16.5.151 172.16.5.200;
    option routers 172.16.5.150;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    default-lease-time 600;
    max-lease-time 7200;
}
```

Con esta configuración, José podrá entregar direcciones IP del rango:

```text
172.16.5.151 - 172.16.5.200
```

a los clientes de la red.

Guardamos:

```text
Ctrl + O
Enter
Ctrl + X
```

---

## 7. Comprobar la configuración desde el cliente

Como seguimos dentro del servidor mediante SSH, comprobamos que la configuración no tenga errores:

```bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
```

Si no aparecen errores, la configuración es correcta.

Reiniciamos DHCP:

```bash
sudo systemctl restart isc-dhcp-server
```

Y comprobamos su estado:

```bash
sudo systemctl status isc-dhcp-server
```

Debe aparecer:

```text
Active: active (running)
```

Esto confirma que el servidor DHCP de José está funcionando.

Para salir de la pantalla de estado:

```text
q
```

---

## 8. Salir del servidor y volver al cliente

Cuando terminamos la configuración escribimos:

```bash
exit
```

Antes estábamos dentro del servidor:

```text
jose@jose-VirtualBox:~$
```

Después de ejecutar `exit` volvemos a:

```text
liza@liza-VirtualBox:~$
```

Esto significa que hemos cerrado la sesión SSH y estamos otra vez trabajando directamente sobre el cliente.

---

## 9. Configurar Liza como cliente DHCP

Ahora Liza debe obtener automáticamente su dirección IP.

La configuración IPv4 de `enp0s3` debe estar configurada como:

```text
Automático (DHCP)
```

Comprobamos la dirección:

```bash
ip a
```

Una dirección entregada por DHCP aparecerá con la palabra:

```text
dynamic
```

Por ejemplo:

```text
inet 172.16.5.152/24 ... dynamic
```

Esto demuestra que Liza ha recibido una dirección IP automáticamente desde el servidor DHCP José.

---

## 10. Funcionamiento completo

El proceso que hemos realizado es:

```text
CLIENTE LIZA
     |
     | SSH
     v
SERVIDOR JOSÉ
172.16.5.150
     |
     | Instalamos isc-dhcp-server
     | Configuramos DHCP
     | Iniciamos DHCP
     |
     v
SERVIDOR DHCP FUNCIONANDO
     |
     | asigna una IP
     v
CLIENTE LIZA
172.16.5.x
```

Por tanto, la instalación del servidor DHCP se ha realizado desde la máquina cliente mediante una conexión SSH.

El paquete:

```text
isc-dhcp-server
```

está instalado en José, ya que José es el servidor DHCP.

Liza actúa como cliente DHCP y recibe automáticamente una dirección IP del servidor.

La diferencia es que toda la instalación y configuración de José la hemos realizado remotamente desde la terminal de Liza mediante SSH.
