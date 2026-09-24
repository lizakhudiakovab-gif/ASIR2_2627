# Acceso remoto - SSH

## 1. Preparar Ubuntu e instalar SSH

Antes de instalar SSH actualizamos el sistema:

sudo apt update
sudo apt upgrade

Instalamos el servidor SSH:

sudo apt install openssh-server -y

Activamos SSH para que se inicie automáticamente al arrancar Ubuntu:

sudo systemctl enable ssh

Iniciamos SSH:

sudo systemctl start ssh

También instalamos bzip2, necesario para trabajar con archivos comprimidos y para la instalación de las Guest Additions:

sudo apt install bzip2


## 2. Instalar Guest Additions

En VirtualBox vamos a:

Dispositivos > Insertar imagen de CD de las Guest Additions

Las Guest Additions mejoran la integración entre Ubuntu y nuestro ordenador, por ejemplo la resolución de pantalla, el ratón y el portapapeles.


## 3. Comprobar la dirección IP

Ejecutamos:

ip a

Este comando muestra las interfaces de red y sus direcciones IP.

En nuestro caso la interfaz es:

enp0s3

Y con NAT obtuvimos la IP:

10.0.2.15/24


## 4. Configuración de red en VirtualBox

Vamos a:

Configuración > Red > Adaptador 1

Desde aquí podemos elegir cómo se conecta nuestra máquina virtual a la red.


### NAT

Es la opción que viene normalmente por defecto.

La máquina virtual tiene acceso a Internet usando la conexión del ordenador físico.

Es fácil de configurar, pero otros equipos de la red no pueden conectarse directamente a la máquina virtual.


### Adaptador puente

La máquina virtual se conecta directamente a la misma red que el ordenador físico.

Recibe su propia IP dentro de esa red, como si fuese otro ordenador.

Es útil para conectarnos a la máquina virtual mediante SSH desde otro equipo.


### Red interna

Crea una red privada entre máquinas virtuales.

Las máquinas virtuales pueden comunicarse entre ellas, pero quedan aisladas del ordenador físico y de Internet.


### Adaptador sólo-anfitrión

Crea una red entre el ordenador físico y las máquinas virtuales.

Permite la comunicación:

Ordenador físico <----> Máquina virtual

Normalmente no tiene acceso a Internet por sí solo.


### Controlador genérico

Se utiliza para configuraciones de red más específicas o avanzadas.


### Red NAT

Es parecida a NAT, pero permite conectar varias máquinas virtuales en la misma red.

Las máquinas virtuales pueden comunicarse entre ellas y tener acceso a Internet.


### Red en la nube

Permite conectar la máquina virtual a una infraestructura de red en la nube.

Se utiliza principalmente en configuraciones avanzadas.


### No conectado

El adaptador existe, pero VirtualBox simula que el cable de red está desconectado.

La máquina virtual no tendrá conexión mediante ese adaptador.


## 5. Configuración IPv4 en Ubuntu

Vamos a:

Configuración > Red > Cableada > IPv4

Aquí podemos elegir cómo obtiene Ubuntu su dirección IP.


### Automático (DHCP)

La IP, máscara, puerta de enlace y otros datos se obtienen automáticamente desde un servidor DHCP.


### Manual

Nos permite configurar nosotros mismos:

- Dirección IP
- Máscara de red
- Puerta de enlace
- DNS

Se utiliza para poner una IP fija a la máquina, algo útil cuando usamos servicios como SSH y queremos que la dirección IP no cambie.
