# Configuración de Ubuntu 26.04 LTS

En estos apuntes voy guardando todo el proceso que hemos realizado con mi máquina virtual de Ubuntu 26.04 LTS: actualización del sistema, instalación de SSH, Guest Additions, configuración de red, IP estática, ping y Firewall de Windows.

---

# 1. Preparar Ubuntu

Antes de instalar los programas actualizamos la lista de paquetes:

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

# 2. Instalar SSH

Instalamos el servidor SSH:

```bash
sudo apt install openssh-server -y
```

SSH nos permite conectarnos de forma remota a nuestra máquina Ubuntu desde otro ordenador.

Activamos SSH para que se inicie automáticamente cuando arranque Ubuntu:

```bash
sudo systemctl enable ssh
```

Iniciamos el servicio:

```bash
sudo systemctl start ssh
```

Podemos comprobar su estado con:

```bash
sudo systemctl status ssh
```

Si aparece:

```text
active (running)
```

significa que SSH está funcionando correctamente.

---

# 3. Instalar Guest Additions

Para instalar las Guest Additions de VirtualBox vamos a:

**Dispositivos > Insertar imagen de CD de las Guest Additions**

Las Guest Additions sirven para mejorar la integración entre la máquina virtual Ubuntu y el ordenador real.

Por ejemplo, permiten mejorar:

- La resolución de pantalla.
- El funcionamiento del ratón.
- El portapapeles compartido.
- La integración entre la máquina virtual y el ordenador real.

---

# 4. Comprobar la dirección IP

Para ver las interfaces de red y las direcciones IP utilizamos:

```bash
ip a
```

Nuestra interfaz de red es:

```text
enp0s3
```

Cuando utilizábamos NAT, nuestra máquina tenía una dirección parecida a:

```text
10.0.2.15/24
```

También podemos consultar rápidamente las IP de nuestra máquina con:

```bash
hostname -I
```

---

# 5. Modos de red de VirtualBox

Para configurar la red de nuestra máquina virtual vamos a:

**Configuración > Red > Adaptador 1**

VirtualBox permite utilizar diferentes modos de red.

## NAT

Es el modo que normalmente viene configurado por defecto.

La máquina virtual puede acceder a Internet utilizando la conexión del ordenador real.

Es fácil de utilizar, pero otros equipos de la red no pueden conectarse directamente a la máquina virtual.

## Adaptador puente

La máquina virtual se conecta directamente a la misma red que el ordenador real.

Recibe su propia dirección IP, como si fuese otro ordenador conectado a la red.

Es útil cuando queremos conectarnos a la máquina virtual mediante SSH desde otro equipo.

## Red interna

Crea una red privada entre máquinas virtuales.

Las máquinas virtuales pueden comunicarse entre ellas, pero quedan aisladas del ordenador real y de Internet.

## Adaptador sólo-anfitrión

Crea una red entre el ordenador real y las máquinas virtuales.

Permite la comunicación:

```text
Ordenador real <----> Máquina virtual
```

Normalmente no proporciona acceso a Internet por sí solo.

## Controlador genérico

Se utiliza para configuraciones de red más específicas o avanzadas.

## Red NAT

Es parecida al modo NAT, pero permite tener varias máquinas virtuales dentro de la misma red.

Las máquinas virtuales pueden comunicarse entre ellas y también tener acceso a Internet.

## Red en la nube

Permite conectar la máquina virtual a una infraestructura de red en la nube.

Se utiliza principalmente para configuraciones más avanzadas.

## No conectado

El adaptador de red existe, pero VirtualBox simula que el cable de red está desconectado.

La máquina virtual no tendrá conexión mediante ese adaptador.

---

# 6. Configuración IPv4 desde la interfaz gráfica

En Ubuntu podemos configurar la red desde:

**Configuración > Red > Cableada > IPv4**

Aquí podemos elegir entre configuración automática o manual.

## Automático (DHCP)

La dirección IP y otros datos de red se obtienen automáticamente desde un servidor DHCP.

Por ejemplo:

- Dirección IP.
- Máscara de red.
- Puerta de enlace.
- DNS.

## Manual

Nos permite introducir nosotros mismos los datos de red.

Podemos configurar:

- Dirección IP.
- Máscara.
- Puerta de enlace.
- DNS.

Esto es útil cuando queremos tener una IP fija, por ejemplo para trabajar con SSH y evitar que nuestra dirección cambie.

---

# 7. Configurar una IP estática mediante comandos

También podemos configurar una IP estática desde la terminal utilizando Netplan.

Primero abrimos el archivo de configuración:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Dentro del archivo configuramos nuestra interfaz de red.

En mi caso quería utilizar la IP:

```text
172.16.5.150
```

La configuración utilizada fue:

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 172.16.5.150/24
      routes:
        - to: default
          via: 172.16.0.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
  version: 2
```

### Explicación

```yaml
dhcp4: false
```

Desactiva DHCP para que Ubuntu no obtenga automáticamente una dirección IP.

```yaml
addresses:
  - 172.16.5.150/24
```

Indica la dirección IP fija que queremos utilizar.

El `/24` corresponde a la máscara:

```text
255.255.255.0
```

```yaml
routes:
  - to: default
    via: 172.16.0.1
```

Indica la ruta por defecto y la puerta de enlace.

```yaml
nameservers:
  addresses: [8.8.8.8, 8.8.4.4]
```

Indica los servidores DNS que utilizará Ubuntu.

---

# 8. Guardar el archivo de Netplan

Después de modificar el archivo en `nano`, guardamos utilizando:

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

---

# 9. Aplicar la configuración de red

Después de guardar los cambios ejecutamos:

```bash
sudo netplan apply
```

Este comando aplica la nueva configuración de red.

---

# 10. Comprobar que la IP ha cambiado

Utilizamos:

```bash
ip a
```

Buscamos nuestra interfaz:

```text
enp0s3
```

Y comprobamos que aparece:

```text
inet 172.16.5.150/24
```

También podemos utilizar:

```bash
hostname -I
```

para ver rápidamente las direcciones IP de Ubuntu.

---

# 11. Comprobar la puerta de enlace

Podemos consultar las rutas de nuestra máquina con:

```bash
ip route
```

Aquí podemos comprobar cuál es nuestra puerta de enlace y por qué interfaz está saliendo el tráfico.

---

# 12. Comprobar la conexión con ping

El comando `ping` sirve para comprobar si existe comunicación entre dos equipos.

La estructura es:

```bash
ping IP_DEL_OTRO_EQUIPO
```

Por ejemplo:

```bash
ping 172.16.5.199
```

Si obtenemos respuestas parecidas a:

```text
64 bytes from 172.16.5.199
```

significa que existe comunicación con el otro equipo.

Para detener el ping utilizamos:

```text
Ctrl + C
```

---

# 13. Ping entre Ubuntu y Windows

Queríamos comprobar la comunicación entre:

```text
Máquina virtual Ubuntu <----> Máquina real Windows
```

Primero necesitamos conocer la IP del ordenador Windows.

En Windows podemos utilizar:

```powershell
ipconfig
```

Después, desde Ubuntu hacemos:

```bash
ping IP_DE_WINDOWS
```

Por ejemplo:

```bash
ping 172.16.5.199
```

---

# 14. Problema con el Firewall de Windows

Aunque las dos máquinas estén correctamente conectadas, Windows puede bloquear las solicitudes de `ping`.

Esto ocurre porque el Firewall de Windows puede bloquear las solicitudes ICMP entrantes.

Para permitirlas abrimos **PowerShell como administrador** en Windows.

Ejecutamos:

```powershell
Enable-NetFirewallRule -Name "FPS-ICMP4-ERQ-In"
```

Este comando activa la regla del Firewall de Windows que permite recibir solicitudes **ICMPv4**.

ICMP es el protocolo utilizado por herramientas como `ping` para comprobar la comunicación entre equipos.

Después volvemos a Ubuntu y hacemos:

```bash
ping IP_DE_WINDOWS
```

Si recibimos respuesta significa que la máquina virtual Ubuntu puede comunicarse correctamente con la máquina real Windows.

---

# 15. Comandos utilizados

Estos son los principales comandos que hemos utilizado durante todo el proceso:

```bash
sudo apt update
sudo apt upgrade
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
sudo apt install bzip2
ip a
hostname -I
ip route
sudo nano /etc/netplan/00-installer-config.yaml
sudo netplan apply
ping IP
```

En Windows hemos utilizado:

```powershell
ipconfig
```

y:

```powershell
Enable-NetFirewallRule -Name "FPS-ICMP4-ERQ-In"
```

---

# Resumen

Durante esta práctica hemos aprendido a preparar una máquina virtual Ubuntu y configurar su red.

Hemos instalado y activado SSH, instalado las Guest Additions de VirtualBox, visto los diferentes modos de red de VirtualBox y aprendido la diferencia entre utilizar DHCP y configurar una IP manualmente.

También hemos configurado una IP estática utilizando Netplan:

```text
172.16.5.150/24
```

Después hemos utilizado `ping` para comprobar la comunicación entre nuestra máquina virtual Ubuntu y nuestra máquina real Windows.

Finalmente, hemos configurado el Firewall de Windows para permitir las solicitudes ICMPv4 y poder realizar ping entre las dos máquinas.
