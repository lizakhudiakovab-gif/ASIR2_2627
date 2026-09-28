# Configuración de Ubuntu y acceso remoto por SSH

En estos apuntes se recoge paso a paso el proceso realizado con la máquina virtual Ubuntu: preparación del sistema, instalación de SSH, Guest Additions, configuración de red, IP estática, pruebas con ping, Firewall de Windows y conexión remota por SSH.

---

## 1. Preparar Ubuntu

Antes de instalar SSH actualizamos el sistema.

Primero actualizamos la lista de paquetes disponibles:

```bash
sudo apt update
```

Después actualizamos los paquetes instalados:

```bash
sudo apt upgrade
```

También instalamos `bzip2`:

```bash
sudo apt install bzip2
```

`bzip2` sirve para comprimir y descomprimir archivos y puede ser necesario durante algunos procesos de instalación.

---

## 2. Instalar y activar SSH

Instalamos el servidor OpenSSH:

```bash
sudo apt install openssh-server -y
```

SSH permite conectarnos de forma remota a Ubuntu desde otro ordenador.

Activamos SSH para que se inicie automáticamente al arrancar Ubuntu:

```bash
sudo systemctl enable ssh
```

Iniciamos el servicio:

```bash
sudo systemctl start ssh
```

También se pueden hacer las dos cosas con:

```bash
sudo systemctl enable --now ssh
```

Comprobamos que SSH funciona:

```bash
sudo systemctl status ssh
```

Si aparece:

```text
active (running)
```

significa que el servicio está funcionando correctamente.

Si tenemos UFW activado, permitimos las conexiones SSH:

```bash
sudo ufw allow ssh
```

---

## 3. Instalar Guest Additions

En VirtualBox vamos a:

```text
Dispositivos > Insertar imagen de CD de las Guest Additions
```

Las Guest Additions mejoran la integración entre Ubuntu y el ordenador físico.

Por ejemplo, mejoran:

- La resolución de pantalla.
- El funcionamiento del ratón.
- El portapapeles compartido.
- La integración entre la máquina virtual y el equipo real.

---

## 4. Comprobar la dirección IP

Para ver las interfaces de red y sus direcciones IP usamos:

```bash
ip a
```

Nuestra interfaz de red es:

```text
enp0s3
```

Cuando utilizábamos NAT teníamos una IP parecida a:

```text
10.0.2.15/24
```

También podemos ver rápidamente las direcciones IP con:

```bash
hostname -I
```

---

# 5. Configurar la red de VirtualBox

Para cambiar el tipo de conexión de la máquina virtual:

1. Apagamos la máquina virtual.
2. Entramos en VirtualBox.
3. Seleccionamos la máquina.
4. Vamos a `Configuración > Red > Adaptador 1`.
5. Elegimos el modo de red.
6. Guardamos los cambios.
7. Volvemos a iniciar Ubuntu.

## NAT

Es el modo que suele venir por defecto.

La máquina virtual puede acceder a Internet utilizando la conexión del ordenador físico.

Es fácil de configurar, pero otros equipos de nuestra red no pueden conectarse directamente a la máquina virtual.

## Adaptador puente

La máquina virtual se conecta directamente a la misma red que el ordenador físico.

Funciona como si fuese otro ordenador conectado a la red y recibe su propia dirección IP.

```text
Router
  |
  |---- Windows
  |
  |---- Ubuntu (MV)
```

Este modo es útil para conectarnos directamente a Ubuntu mediante SSH.

Al seleccionar Adaptador puente debemos elegir la tarjeta de red que está utilizando nuestro PC, por ejemplo Wi-Fi o Ethernet.

## Red interna

Crea una red privada entre máquinas virtuales.

Las máquinas virtuales pueden comunicarse entre ellas, pero quedan aisladas del ordenador físico y de Internet.

## Adaptador sólo-anfitrión

Permite crear una red entre el ordenador físico y las máquinas virtuales.

```text
Windows <----> Ubuntu
```

Normalmente no proporciona acceso a Internet por sí solo.

## Red NAT

Es parecida a NAT, pero permite tener varias máquinas virtuales dentro de la misma red.

Las máquinas virtuales pueden comunicarse entre ellas y también acceder a Internet.

## Controlador genérico

Se utiliza para configuraciones de red más específicas o avanzadas.

## Red en la nube

Permite conectar la máquina virtual a una infraestructura de red en la nube.

Se utiliza principalmente en configuraciones avanzadas.

## No conectado

El adaptador existe, pero VirtualBox simula que el cable de red está desconectado.

Por lo tanto, no tendremos conexión mediante ese adaptador.

---

# 6. Configuración IPv4 en Ubuntu

Ubuntu permite configurar la dirección IP de forma automática o manual.

Vamos a:

```text
Configuración > Red > Cableada > IPv4
```

## Automático (DHCP)

El servidor DHCP proporciona automáticamente los datos necesarios:

- Dirección IP.
- Máscara de red.
- Puerta de enlace.
- DNS.

Es más cómodo, pero la dirección IP puede cambiar.

## Manual

Nosotros introducimos los datos de red.

Tenemos que configurar:

- Dirección IP.
- Máscara.
- Puerta de enlace.
- DNS.

Esto permite tener una IP fija, algo útil para trabajar con SSH.

---

# 7. Configurar una IP estática con Netplan

También podemos configurar una IP fija mediante comandos.

Primero comprobamos el nombre de nuestra interfaz:

```bash
ip a
```

En nuestro caso es:

```text
enp0s3
```

Podemos comprobar qué archivos de Netplan tenemos con:

```bash
ls /etc/netplan/
```

Después abrimos nuestro archivo de configuración. En nuestro caso:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Configuramos la IP:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 172.16.5.150/24
      routes:
        - to: default
          via: 172.16.0.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

### Explicación

```text
enp0s3
```

Es nuestra interfaz de red.

```yaml
dhcp4: false
```

Desactiva DHCP porque queremos utilizar una dirección IP fija.

```yaml
addresses:
  - 172.16.5.150/24
```

Establece nuestra dirección IP.

El `/24` corresponde a la máscara:

```text
255.255.255.0
```

La ruta:

```yaml
routes:
  - to: default
    via: 172.16.0.1
```

establece la puerta de enlace utilizada para salir de nuestra red.

Los DNS:

```yaml
nameservers:
  addresses:
    - 8.8.8.8
    - 8.8.4.4
```

se utilizan para resolver nombres de dominio.

---

# 8. Guardar y aplicar la configuración

Después de modificar el archivo en Nano guardamos con:

```text
Ctrl + O
```

Pulsamos:

```text
Enter
```

Y salimos con:

```text
Ctrl + X
```

Aplicamos los cambios:

```bash
sudo netplan apply
```

---

# 9. Comprobar la nueva IP

Volvemos a ejecutar:

```bash
ip a
```

Buscamos:

```text
enp0s3
```

Y comprobamos que aparece nuestra IP:

```text
inet 172.16.5.150/24
```

También podemos usar:

```bash
hostname -I
```

---

# 10. Comprobar la puerta de enlace

Ejecutamos:

```bash
ip route
```

Este comando muestra las rutas de nuestra máquina y nos permite comprobar la puerta de enlace que estamos utilizando.

---

# 11. Comprobar la conexión con ping

`ping` sirve para comprobar si existe comunicación entre dos equipos.

Utilizamos:

```bash
ping IP_DEL_OTRO_EQUIPO
```

Por ejemplo:

```bash
ping 172.16.5.199
```

Si aparecen respuestas parecidas a:

```text
64 bytes from 172.16.5.199
```

significa que existe comunicación.

Para detener el ping:

```text
Ctrl + C
```

---

# 12. Hacer ping entre Ubuntu y Windows

Queremos comprobar la comunicación entre:

```text
Máquina virtual Ubuntu <----> Máquina física Windows
```

Primero comprobamos la IP de Windows.

Abrimos CMD o PowerShell y ejecutamos:

```powershell
ipconfig
```

Buscamos la dirección IPv4 del adaptador que estamos utilizando.

Después, desde Ubuntu hacemos:

```bash
ping IP_DE_WINDOWS
```

Por ejemplo:

```bash
ping 172.16.5.199
```

---

# 13. Permitir ping en el Firewall de Windows

Aunque la configuración de red sea correcta, Windows puede bloquear las solicitudes de `ping`.

Para permitirlas abrimos **PowerShell como administrador** y ejecutamos:

```powershell
Enable-NetFirewallRule -Name "FPS-ICMP4-ERQ-In"
```

Este comando habilita la regla del Firewall de Windows que permite recibir solicitudes ICMPv4.

ICMP es el protocolo utilizado por `ping`.

Después volvemos a Ubuntu y repetimos:

```bash
ping IP_DE_WINDOWS
```

Si recibimos respuesta, existe comunicación entre Ubuntu y Windows.

---

# 14. Conectarnos por SSH desde Windows

Una vez que tenemos:

- SSH instalado y funcionando.
- La red configurada.
- La IP de Ubuntu.
- Comunicación entre los equipos.

Podemos intentar conectarnos desde Windows.

Abrimos CMD, PowerShell o Terminal.

## Con Adaptador puente

Ejecutamos:

```bash
ssh USUARIO_UBUNTU@IP_DE_UBUNTU
```

Por ejemplo, si nuestra IP es `172.16.5.150`:

```bash
ssh usuario@172.16.5.150
```

Tenemos que sustituir `usuario` por el nombre real de nuestro usuario de Ubuntu.

La primera vez puede aparecer una pregunta para confirmar que confiamos en el equipo.

Escribimos:

```text
yes
```

Después introducimos la contraseña de nuestro usuario de Ubuntu.

Si la conexión funciona, podremos utilizar la terminal de Ubuntu desde Windows.

---

# 15. SSH utilizando NAT y reenvío de puertos

Si utilizamos NAT en vez de Adaptador puente, podemos configurar un reenvío de puertos en VirtualBox.

Por ejemplo:

```text
Puerto del PC: 2222
        |
        v
Puerto SSH de Ubuntu: 22
```

Después nos conectamos desde Windows con:

```bash
ssh usuario@127.0.0.1 -p 2222
```

`127.0.0.1` hace referencia al propio ordenador físico.

La opción:

```text
-p 2222
```

indica que queremos utilizar el puerto `2222` del ordenador físico, que VirtualBox reenviará al puerto SSH de la máquina virtual.

---

# 16. Comandos principales utilizados

Ubuntu:

```bash
sudo apt update
sudo apt upgrade
sudo apt install bzip2
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
sudo ufw allow ssh
ip a
hostname -I
ip route
ls /etc/netplan/
sudo nano /etc/netplan/00-installer-config.yaml
sudo netplan apply
ping IP
```

Windows:

```powershell
ipconfig
```

Para permitir ping:

```powershell
Enable-NetFirewallRule -Name "FPS-ICMP4-ERQ-In"
```

Para conectarnos por SSH:

```bash
ssh usuario@IP_DE_UBUNTU
```

---

# Resumen

Durante esta práctica hemos preparado nuestra máquina virtual Ubuntu para poder acceder a ella de forma remota.

Primero actualizamos el sistema, instalamos SSH y las Guest Additions. Después estudiamos los diferentes modos de red de VirtualBox y configuramos la máquina para poder comunicarnos con ella desde Windows.

También aprendimos a configurar una dirección IP estática mediante Netplan. En nuestro caso utilizamos:

```text
IP: 172.16.5.150/24
Puerta de enlace: 172.16.0.1
DNS: 8.8.8.8 y 8.8.4.4
Interfaz: enp0s3
```

Finalmente utilizamos `ping` para comprobar la comunicación entre Ubuntu y Windows, configuramos el Firewall de Windows para permitir ICMP y probamos el acceso remoto mediante SSH.
