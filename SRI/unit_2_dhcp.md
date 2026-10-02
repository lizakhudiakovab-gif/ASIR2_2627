# Configuración de DHCP desde la máquina cliente Ubuntu

En esta práctica utilizamos dos máquinas virtuales Ubuntu:

- **José:** servidor DHCP.
- **Liza:** máquina cliente desde la que realizamos la configuración.

La configuración final de la red es:

```text
Servidor José: 172.16.5.150/24
Cliente Liza: 172.16.5.151/24
Red: 172.16.5.0/24
```

Una parte importante de la práctica es que la instalación y configuración del servidor DHCP de José se realiza **remotamente desde Liza mediante SSH**.

De esta forma, trabajamos físicamente desde Liza, nos conectamos por SSH a José y configuramos desde allí el servidor DHCP.

---

## 1. Configurar las máquinas en VirtualBox

Las dos máquinas necesitan estar conectadas entre sí mediante una red interna de VirtualBox.

La red interna utilizada es:

```text
red-dhcp
```

En el servidor José utilizamos:

```text
Adaptador 1: Red interna
Nombre: red-dhcp

Adaptador 2: NAT
```

El primer adaptador permite la comunicación entre José y Liza.

El segundo adaptador permite que José tenga también conexión a Internet.

En Liza utilizamos también un adaptador conectado a:

```text
Red interna
Nombre: red-dhcp
```

Es importante que las dos máquinas utilicen exactamente el mismo nombre de red interna.

---

# Fase 1: Configuración del servidor José

## 2. Configurar la IP del servidor

José debe tener una dirección IP fija.

Su dirección es:

```text
172.16.5.150/24
```

La interfaz utilizada para la red interna es:

```text
enp0s3
```

Abrimos el archivo de Netplan:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

La configuración de `enp0s3` queda con una estructura como esta:

```yaml
network:
  version: 2
  renderer: networkd
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
dhcp4: false
```

Indica que José no obtiene automáticamente su dirección IPv4 mediante DHCP.

```text
addresses:
  - 172.16.5.150/24
```

Establece `172.16.5.150` como dirección IP fija.

```text
routes:
  - to: default
    via: 172.16.0.1
```

Configura la ruta por defecto.

Los DNS utilizados son:

```text
8.8.8.8
8.8.4.4
```

Después guardamos:

```text
Ctrl + O
Enter
Ctrl + X
```

Aplicamos la configuración:

```bash
sudo netplan apply
```

Y comprobamos la IP:

```bash
ip -4 addr show enp0s3
```

Debe aparecer:

```text
inet 172.16.5.150/24
```

---

# Fase 2: Configuración del cliente Liza

## 3. Configurar `enp0s3` en Liza

En Liza configuramos finalmente la interfaz `enp0s3` con la dirección:

```text
172.16.5.151/24
```

Primero comprobamos los archivos existentes:

```bash
ls -l /etc/netplan/
```

En nuestro caso aparecieron varios archivos de configuración:

```text
00-installer-config.yaml
01-network-manager-all.yaml
90-NM-1eef7e45-3b9d-3043-bee3-fc5925c90273.yaml
```

El archivo `90-NM-...yaml` contenía una configuración anterior de `enp0s3`.

Esto provocaba un conflicto al ejecutar:

```bash
sudo netplan apply
```

y aparecía un error relacionado con:

```text
Conflicting default route declarations
```

Esto ocurría porque había más de una configuración intentando establecer una ruta por defecto para la misma interfaz.

---

## 4. Solucionar el conflicto de Netplan

Antes de modificar el archivo antiguo hicimos una copia de seguridad:

```bash
sudo cp /etc/netplan/90-NM-1eef7e45-3b9d-3043-bee3-fc5925c90273.yaml /etc/netplan/90-NM-backup.yaml.bak
```

Después desactivamos el archivo antiguo cambiándole la extensión:

```bash
sudo mv /etc/netplan/90-NM-1eef7e45-3b9d-3043-bee3-fc5925c90273.yaml /etc/netplan/90-NM-1eef7e45-3b9d-3043-bee3-fc5925c90273.yaml.disabled
```

No eliminamos el archivo.

Simplemente dejamos de utilizarlo como archivo de configuración de Netplan.

Después comprobamos:

```bash
ls -l /etc/netplan/
```

Quedando:

```text
00-installer-config.yaml
01-network-manager-all.yaml
90-NM-1eef7e45-3b9d-3043-bee3-fc5925c90273.yaml.disabled
90-NM-backup.yaml.bak
```

---

## 5. Configuración final de Liza

Editamos:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

La configuración de `enp0s3` quedó:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 172.16.5.151/24
      routes:
        - to: default
          via: 172.16.0.1
          on-link: true
```

### Explicación

```text
dhcp4: false
```

Indica que la dirección de `enp0s3` está configurada manualmente.

```text
172.16.5.151/24
```

Es la dirección asignada a Liza.

```text
via: 172.16.0.1
```

Indica la puerta de enlace configurada.

```text
on-link: true
```

Permite utilizar esa puerta de enlace aunque Netplan no la considere directamente dentro de la misma subred.

---

## 6. Comprobar y aplicar Netplan

Antes de aplicar los cambios comprobamos que la configuración fuera válida:

```bash
sudo netplan generate
```

Si el comando vuelve al terminal sin mostrar errores, la sintaxis es correcta.

Después aplicamos:

```bash
sudo netplan apply
```

En nuestro caso se aplicó correctamente y sin errores.

Comprobamos `enp0s3`:

```bash
ip -4 addr show enp0s3
```

El resultado mostró:

```text
inet 172.16.5.151/24
```

Por tanto, Liza quedó configurada correctamente con:

```text
172.16.5.151/24
```

---

# Fase 3: Comprobar la comunicación

## 7. Hacer ping desde Liza a José

Desde Liza comprobamos que podemos comunicarnos con el servidor:

```bash
ping -c 4 172.16.5.150
```

Obtuvimos:

```text
4 packets transmitted, 4 received, 0% packet loss
```

Esto demuestra que existe conectividad entre:

```text
Liza  ->  José
.151      .150
```

---

# Fase 4: Entrar al servidor mediante SSH

## 8. Conectarnos desde Liza a José

Desde Liza ejecutamos:

```bash
ssh jose@172.16.5.150
```

Antes de conectarnos aparece:

```text
liza@liza-VirtualBox:~$
```

Después de entrar mediante SSH aparece:

```text
jose@jose-VirtualBox:~$
```

Esto significa que seguimos utilizando físicamente la máquina Liza, pero ahora los comandos se ejecutan remotamente en José.

Además, anteriormente configuramos las claves SSH, por lo que podemos entrar al servidor sin introducir la contraseña del usuario José.

En el servidor también aparece que la conexión procede de:

```text
172.16.5.151
```

que corresponde a Liza.

---

# Fase 5: Instalar DHCP desde Liza

## 9. Instalar DHCP en José mediante SSH

Una vez conectados mediante SSH aparece:

```text
jose@jose-VirtualBox:~$
```

Aunque estamos utilizando Liza, ahora estamos controlando remotamente José.

Actualizamos los repositorios:

```bash
sudo apt update
```

Instalamos el servidor DHCP:

```bash
sudo apt install isc-dhcp-server -y
```

El paquete:

```text
isc-dhcp-server
```

queda instalado realmente en José.

La diferencia es que hemos realizado la instalación remotamente desde Liza utilizando SSH.

---

## 10. Indicar la interfaz que utilizará DHCP

Seguimos dentro de José mediante SSH.

Abrimos:

```bash
sudo nano /etc/default/isc-dhcp-server
```

Configuramos:

```text
INTERFACESv4="enp0s3"
INTERFACESv6=""
```

Esto indica que el servidor DHCP trabajará sobre:

```text
enp0s3
```

que es la interfaz conectada a la red `172.16.5.0/24`.

Guardamos:

```text
Ctrl + O
Enter
Ctrl + X
```

---

## 11. Configurar el servidor DHCP

Abrimos:

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

### Explicación

```text
subnet 172.16.5.0
```

Indica la red sobre la que trabaja DHCP.

```text
netmask 255.255.255.0
```

Es la máscara correspondiente a `/24`.

```text
range 172.16.5.151 172.16.5.200
```

Define el rango de direcciones que el servidor puede entregar.

```text
option routers 172.16.5.150
```

Indica la dirección configurada como router para los clientes.

```text
option domain-name-servers 8.8.8.8, 8.8.4.4
```

Establece los servidores DNS.

```text
default-lease-time 600
```

Establece una concesión normal de 600 segundos.

```text
max-lease-time 7200
```

Establece el tiempo máximo de una concesión.

---

# Fase 6: Comprobar DHCP

## 12. Comprobar la sintaxis

Antes de iniciar el servicio comprobamos que el archivo no tenga errores:

```bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
```

Si vuelve al terminal sin mostrar errores, la sintaxis es correcta.

---

## 13. Reiniciar DHCP

Ejecutamos:

```bash
sudo systemctl restart isc-dhcp-server
```

Después comprobamos:

```bash
sudo systemctl status isc-dhcp-server
```

Obtuvimos:

```text
Active: active (running)
```

También comprobamos que DHCP estaba escuchando en:

```text
enp0s3
```

sobre la red:

```text
172.16.5.0/24
```

Por tanto, el servicio DHCP está funcionando correctamente.

Para salir de la pantalla de estado pulsamos:

```text
q
```

---

## 14. Comprobar las concesiones DHCP

Para comprobar las direcciones que el servidor DHCP había concedido ejecutamos en José:

```bash
sudo cat /var/lib/dhcp/dhcpd.leases
```

En el archivo aparecieron concesiones como:

```text
lease 172.16.5.152
```

y también una concesión correspondiente a:

```text
lease 172.16.5.151
```

con el nombre del cliente:

```text
client-hostname "liza-VirtualBox";
```

Esto permite comprobar que el servidor DHCP ha realizado concesiones de direcciones IP a clientes de la red.

---

# Fase 7: Comprobaciones finales

## 15. Comprobar la IP de Liza

En Liza ejecutamos:

```bash
ip -4 addr show enp0s3
```

Obtenemos:

```text
inet 172.16.5.151/24
```

En nuestra configuración final esta dirección está establecida manualmente mediante Netplan.

---

## 16. Comprobar la comunicación

Desde Liza:

```bash
ping -c 4 172.16.5.150
```

Resultado:

```text
4 packets transmitted, 4 received, 0% packet loss
```

La comunicación entre cliente y servidor funciona correctamente.

---

## 17. Comprobar SSH

Desde Liza:

```bash
ssh jose@172.16.5.150
```

La conexión se realiza correctamente y entramos en:

```text
jose@jose-VirtualBox:~$
```

Además, la conexión mediante claves SSH permite acceder sin introducir la contraseña de José.

---

## 18. Resultado final

La estructura final de la práctica queda:

```text
CLIENTE LIZA
172.16.5.151/24
      |
      | ping correcto
      | SSH sin contraseña
      |
      v
SERVIDOR JOSÉ
172.16.5.150/24
      |
      | isc-dhcp-server
      | interfaz enp0s3
      |
      v
RED 172.16.5.0/24
```

Hemos comprobado que:

1. José tiene la dirección `172.16.5.150/24`.
2. Liza tiene la dirección `172.16.5.151/24`.
3. Las dos máquinas se comunican correctamente mediante ping.
4. Liza puede entrar remotamente a José mediante SSH.
5. La conexión SSH funciona mediante claves y no solicita la contraseña de José.
6. `isc-dhcp-server` está instalado en José.
7. DHCP utiliza la interfaz `enp0s3`.
8. El servicio aparece como `active (running)`.
9. El servidor tiene registradas concesiones DHCP en `dhcpd.leases`.
10. La instalación y configuración de DHCP en José se ha realizado remotamente desde Liza mediante SSH.

Por tanto, tenemos comunicación entre las dos máquinas, acceso remoto mediante SSH y el servicio DHCP configurado y funcionando en el servidor José.


#Configuración de un servidor DHCP en Windows Server

En esta práctica configuramos **Windows Server como servidor DHCP** para que pueda asignar automáticamente direcciones IP a los equipos clientes que se encuentren en la misma red interna.

La configuración utilizada es:

```text
Red:                172.16.5.0/24
Windows Server:     172.16.5.159
Rango DHCP:         172.16.5.150 - 172.16.5.200
Máscara:            255.255.255.0
DNS:                8.8.8.8 y 8.8.4.4
```

El cliente Ubuntu Liza tiene inicialmente la IP fija:

```text
172.16.5.151/24
```

Al final comprobaremos que puede mantener esta IP fija y solicitar además una dirección IP dinámica al servidor DHCP.

---

## 1. Configurar la red de VirtualBox

Para que el servidor DHCP y los clientes puedan comunicarse, las máquinas virtuales tienen que estar conectadas a la **misma Red interna de VirtualBox**.

Por ejemplo:

```text
Red interna: red-dhcp
```

Por tanto, Windows Server y los clientes Ubuntu deben utilizar la misma red interna.

Esto permite que los mensajes DHCP enviados por los clientes puedan llegar al Windows Server.

---

## 2. Configurar una IP fija en Windows Server

Un servidor DHCP debe tener una dirección IP fija para que su dirección no cambie.

En nuestro caso configuramos:

```text
IP:       172.16.5.159
Máscara:  255.255.255.0
DNS:      8.8.8.8
          8.8.4.4
```

Podemos comprobar la configuración desde PowerShell con:

```powershell
ipconfig /all
```

Debe aparecer la dirección:

```text
Dirección IPv4: 172.16.5.159
```

y DHCP debe aparecer deshabilitado en la propia interfaz del servidor, ya que el servidor utiliza una IP configurada manualmente.

---

# 3. Instalar el rol de servidor DHCP

Abrimos:

```text
Administrador del servidor
```

Seleccionamos:

```text
Agregar roles y características
```

En **Tipo de instalación** seleccionamos:

```text
Instalación basada en características o en roles
```

Seleccionamos nuestro Windows Server como servidor de destino.

En **Roles de servidor** marcamos:

```text
Servidor DHCP
```

Aceptamos también las herramientas adicionales que Windows necesite instalar.

Continuamos con:

```text
Siguiente → Siguiente → Instalar
```

Esperamos hasta que finalice la instalación.

---

# 4. Completar la configuración de DHCP

Después de instalar el rol aparece la opción:

```text
Completar configuración de DHCP
```

La seleccionamos.

Windows crea los grupos necesarios para administrar DHCP, entre ellos:

```text
Administradores de DHCP
Usuarios de DHCP
```

Pulsamos:

```text
Confirmar
```

Cuando aparezca:

```text
Creando grupos de seguridad → Listo
```

podemos cerrar el asistente.

---

# 5. Comprobar que el servicio DHCP está funcionando

Abrimos PowerShell como administrador y ejecutamos:

```powershell
Get-Service DHCPServer
```

El resultado debe mostrar:

```text
Status    Name
------    ----
Running   DHCPServer
```

`Running` significa que el servicio DHCP está ejecutándose correctamente.

---

# 6. Abrir la consola DHCP

Desde el Administrador del servidor entramos en:

```text
Herramientas → DHCP
```

Desplegamos:

```text
DHCP
 └── Servidor
      └── IPv4
```

Sobre **IPv4** creamos un:

```text
Ámbito nuevo
```

Un ámbito determina qué direcciones IP puede entregar nuestro servidor DHCP.

---

# 7. Crear el ámbito DHCP

Le damos un nombre al ámbito.

En nuestro caso:

```text
Red DHCP
```

Después configuramos el intervalo de direcciones.

Utilizamos:

```text
Dirección IP inicial: 172.16.5.150
Dirección IP final:   172.16.5.200
Longitud:             24
Máscara:              255.255.255.0
```

Por tanto, nuestro servidor DHCP trabajará con el rango:

```text
172.16.5.150 - 172.16.5.200
```

---

# 8. Configurar exclusiones

Una **exclusión** sirve para indicar al DHCP que determinadas direcciones del rango no deben entregarse automáticamente.

En nuestra práctica añadimos como exclusión:

```text
172.16.5.159
```

Esta dirección pertenece al propio Windows Server, por lo que no queremos que DHCP se la entregue a otro equipo.

La exclusión queda:

```text
172.16.5.159 - 172.16.5.159
```

De esta forma evitamos un conflicto de direcciones IP.

---

# 9. Configurar la duración de la concesión

Después aparece la duración de la concesión.

Dejamos el valor predeterminado:

```text
8 días
```

Una concesión indica durante cuánto tiempo un cliente puede utilizar una dirección IP que le ha proporcionado el servidor DHCP.

---

# 10. Configurar las opciones DHCP

Seleccionamos:

```text
Configurar estas opciones ahora
```

## Puerta de enlace

En nuestra práctica utilizamos:

```text
172.16.0.1
```

La añadimos en el apartado:

```text
Enrutador (puerta de enlace predeterminada)
```

## Servidores DNS

Configuramos:

```text
8.8.8.8
8.8.4.4
```

El dominio primario se puede dejar vacío si no estamos utilizando uno.

## Servidores WINS

No necesitamos WINS para esta práctica, por lo que dejamos esta pantalla vacía y continuamos.

---

# 11. Activar el ámbito

Finalizamos el asistente y activamos el ámbito.

Podemos comprobar su estado desde PowerShell:

```powershell
Get-DhcpServerv4Scope
```

En nuestro caso obtuvimos:

```text
ScopeId       172.16.5.0
SubnetMask    255.255.255.0
Name          Red DHCP
State         Active
StartRange    172.16.5.150
EndRange      172.16.5.200
```

Lo más importante es:

```text
State: Active
```

Esto significa que el ámbito está activo y puede entregar direcciones IP.

---

# 12. Comprobar conectividad con el cliente

Antes de probar DHCP comprobamos que Windows Server puede comunicarse con Ubuntu Liza.

Liza tiene inicialmente:

```text
172.16.5.151
```

Desde Windows Server ejecutamos:

```powershell
ping 172.16.5.151
```

Obtuvimos:

```text
Enviados = 4
Recibidos = 4
Perdidos = 0
```

Esto demuestra que Windows Server y Ubuntu Liza pueden comunicarse correctamente a través de la red interna.

---

# 13. Configuración fija del cliente Ubuntu

El cliente Liza mantiene su configuración fija en Netplan.

El archivo utilizado es:

```bash
/etc/netplan/00-installer-config.yaml
```

Podemos abrirlo con:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

La configuración fija es:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 172.16.5.151/24
      routes:
        - to: default
          via: 172.16.0.1
          on-link: true
```

Aplicamos los cambios con:

```bash
sudo netplan apply
```

Si ejecutamos:

```bash
ip -4 addr show enp0s3
```

aparece:

```text
inet 172.16.5.151/24
valid_lft forever
preferred_lft forever
```

`forever` indica que esta dirección pertenece a la configuración fija y no tiene el tiempo de concesión propio de una dirección obtenida mediante DHCP.

---

# 14. Instalar el cliente DHCP en Ubuntu

Para solicitar manualmente otra dirección IP al servidor DHCP utilizamos `dhclient`.

Si el comando no está instalado:

```bash
sudo apt update
```

Después:

```bash
sudo apt install isc-dhcp-client -y
```

Esto instala el cliente DHCP necesario para realizar la solicitud.

---

# 15. Solicitar una IP al Windows Server

Sin eliminar nuestra dirección fija `172.16.5.151`, ejecutamos:

```bash
sudo dhclient enp0s3
```

`dhclient` envía una solicitud DHCP a través de la interfaz:

```text
enp0s3
```

El Windows Server recibe la solicitud y busca una dirección disponible dentro de su ámbito:

```text
172.16.5.150 - 172.16.5.200
```

En nuestro caso Windows Server asignó:

```text
172.16.5.152
```

---

# 16. Comprobar la IP recibida por DHCP

En Ubuntu ejecutamos:

```bash
ip -4 addr show enp0s3
```

El resultado final fue:

```text
inet 172.16.5.151/24 ... enp0s3
    valid_lft forever
    preferred_lft forever

inet 172.16.5.152/24 ... secondary dynamic enp0s3
    valid_lft ...
    preferred_lft ...
```

Ahora la interfaz tiene **dos direcciones IP**.

### IP fija

```text
172.16.5.151
```

Es nuestra dirección configurada manualmente.

Por eso aparece:

```text
valid_lft forever
```

### IP dinámica

```text
172.16.5.152
```

Es la dirección que ha entregado automáticamente el Windows Server.

Aparece:

```text
secondary dynamic
```

La palabra:

```text
dynamic
```

es especialmente importante porque demuestra que esa dirección se ha obtenido dinámicamente y no se ha configurado manualmente.

---

# 17. Comprobar la concesión desde Windows Server

También podemos comprobar desde el propio servidor que ha entregado la dirección.

Entramos en:

```text
DHCP
 └── IPv4
      └── Ámbito [172.16.5.0] Red DHCP
           └── Concesiones de direcciones
```

En **Concesiones de direcciones** aparece:

```text
172.16.5.152
```

Esto confirma desde el lado del servidor que Windows Server ha concedido esa dirección al cliente.

---

# 18. Funcionamiento final

El funcionamiento completo queda así:

```text
WINDOWS SERVER
IP: 172.16.5.159
Servidor DHCP
Rango: 172.16.5.150 - 172.16.5.200
        |
        |
        | DHCP
        v
UBUNTU LIZA
enp0s3
        |
        ├── 172.16.5.151 → IP fija
        |
        └── 172.16.5.152 → IP dinámica recibida por DHCP
```

Por tanto, al final de la práctica podemos demostrar dos cosas:

1. El cliente mantiene su dirección fija:

```text
172.16.5.151
```

2. Al ejecutar:

```bash
sudo dhclient enp0s3
```

el cliente solicita una dirección al servidor DHCP y Windows Server le entrega una dirección disponible dentro del rango configurado.

En nuestro caso:

```text
172.16.5.152
```

La comprobación final en Ubuntu es:

```bash
ip -4 addr show enp0s3
```

y debemos encontrar algo parecido a:

```text
inet 172.16.5.151/24
inet 172.16.5.152/24 ... secondary dynamic
```

Mientras que en Windows Server la misma dirección dinámica debe aparecer en:

```text
IPv4
→ Ámbito [172.16.5.0] Red DHCP
→ Concesiones de direcciones
→ 172.16.5.152
```

De esta forma queda demostrado que **el servidor DHCP de Windows Server está funcionando y está asignando direcciones IP automáticamente a los clientes dentro del rango que hemos configurado**.
