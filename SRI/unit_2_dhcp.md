# Configuración de DHCP desde la máquina cliente

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
