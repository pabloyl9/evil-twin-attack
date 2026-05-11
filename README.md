# PoC: Ataque Evil Twin (Captive Portal)

**Autor:** Pablo A.

**Asignatura:** Hacking Ético

**Fecha:** Mayo 2026

**Vídeo:** " "

---

## 1. Descripción del Proyecto.
Este proyecto demuestra una **Prueba de Concepto (PoC)** sobre la vulnerabilidad de las redes inalámbricas (WPA2-PSK) frente a ataques de suplantación de identidad, concretamente un ataque **Evil Twin (Gemelo Malvado) con Portal Cautivo**. 

El ataque consiste en forzar la desconexión de los clientes legítimos de un punto de acceso Wi-Fi y levantar un "Rogue AP" (Punto de Acceso falso) con el mismo nombre. Esto provoca que los dispositivos víctimas se conecten a nuestra red controlada, donde se les presenta un portal web fraudulento que les solicita la contraseña del router, permitiendo robar la credencial en texto plano sin necesidad de recurrir a ataques de fuerza bruta.

<br/>

## 2. Entorno y Herramientas.

* **Máquina Atacante:** Kali Linux (VMware)
    * `airgeddon` (Framework principal de auditoría)
    * `mdk4` / `aireplay-ng` (Herramientas de desautenticación)
    * `dnsmasq` (Servidor DHCP/DNS falso)
    * `lighttpd` / `php-cgi` (Servidor web para el portal cautivo)
* **Adaptador Inalámbrico:** Alfa Network AWUS036ACH (Chipset RTL8812AU)
* **Máquina Víctima:** Apple iPhone (iOS)
* **Red Objetivo:** `S25 de Pabloo` (Cifrado WPA2-PSK)

<br/>

---

## 3. Procedimiento del Ataque.

### Paso 1: Preparación del Atacante.
Primero, instalamos los controladores necesarios para que Kali Linux reconozca el chipset de nuestra tarjeta de red y las dependencias opcionales que requiere el portal cautivo de Airgeddon.

#### 1. Instalación de dependencias y drivers:
```bash
sudo apt update && sudo apt install realtek-rtl88xxau-dkms dnsmasq lighttpd php-cgi mdk4 -y
```

#### 2. Ejecución de Airgeddon y Modo Monitor:
Iniciamos el framework y ponemos nuestra interfaz de red (`wlan0`) en modo monitor para poder inyectar paquetes.
```bash
sudo airgeddon
```

<br/>

### Paso 2: Reconocimiento y Captura del Handshake.
Para que el portal cautivo pueda verificar matemáticamente que la contraseña introducida por la víctima es correcta, necesitamos capturar primero un saludo de conexión cifrado (WPA Handshake).

#### 1. Exploración y Desautenticación:
Entramos en el menú de ataques *Evil Twin* (Opción 7), exploramos las redes cercanas (Opción 4) y fijamos nuestro objetivo (`S25 de Pabloo`). Lanzamos un ataque de desautenticación mediante `aireplay-ng` para forzar a un dispositivo a reconectarse y capturar el Handshake.

#### 2. Configuración del Portal Cautivo:
Una vez obtenido el Handshake, seleccionamos el ataque de Portal Cautivo (Opción 9), le indicamos el archivo capturado y configuramos el idioma de la página web falsa (Español). Airgeddon levanta automáticamente 6 terminales encargadas del enrutamiento, DNS, servidor web y el AP falso.

<br/>

---

## 4. Captura de Credenciales (PoC).
Para demostrar el riesgo crítico de esta técnica, observamos cómo se comporta el dispositivo víctima y cómo el atacante recibe la información.

### Paso 1: Infección y Redirección.
En la máquina víctima (iPhone): Al ser desconectada de la red legítima, se conecta automáticamente a nuestro Gemelo Malvado (que está abierto). El servidor DNS del atacante intercepta las peticiones y fuerza a iOS a abrir una ventana emergente de "Wi-Fi Cautiva".

<img width="425" height="566" alt="Captive Portal Pop-up" src="AQUI_PONES_EL_LINK_DE_LA_FOTO_1" />

<br/>

### Paso 2: Simulación de envío de credenciales.
La víctima, creyendo que es un proceso legítimo del router para restaurar su conexión a internet, introduce su contraseña real (`1234Abcd`).

<img width="425" height="566" alt="Introduciendo contraseña" src="AQUI_PONES_EL_LINK_DE_LA_FOTO_2" />

<br/>

### Paso 3: Validación del lado de la víctima.
Airgeddon recibe la contraseña, la compara instantáneamente con el Handshake capturado previamente y, al ser correcta, muestra un mensaje de éxito en la pantalla del usuario, cerrando el portal para no levantar sospechas.

<img width="425" height="566" alt="Validación exitosa" src="AQUI_PONES_EL_LINK_DE_LA_FOTO_3" />

<br/>

### Resultado Exitoso (Atacante).
Como se observa en la siguiente captura de la máquina atacante, el panel de control de Airgeddon detiene el ataque y revela la contraseña interceptada en texto plano (`1234Abcd`), guardándola en un archivo de texto en el sistema local:

<img width="425" height="566" alt="Contraseña capturada en Kali" src="AQUI_PONES_EL_LINK_DE_LA_FOTO_4" />

<br/>

---

## 5. Mitigación y Soluciones.
Para protegerse contra ataques de Gemelo Malvado y desautenticación en redes Wi-Fi:

### 1. Implementar PMF (Protected Management Frames): 
El estándar 802.11w cifra los paquetes de gestión de la red, impidiendo que un atacante externo pueda enviar paquetes de desautenticación (Deauth) falsificados para desconectar a los clientes.

### 2. Transición a WPA3: 
WPA3 utiliza el protocolo SAE (Simultaneous Authentication of Equals), que es inmune a la captura tradicional de Handshakes y a la posterior validación offline o ataques de diccionario.

### 3. Educación y Concientización (Capa 8): 
Instruir a los usuarios para que desconfíen de redes Wi-Fi conocidas que de repente aparecen como "Abiertas" (sin candado) y que soliciten contraseñas a través del navegador web, ya que los routers legítimos solicitan la clave directamente a través de la interfaz del sistema operativo.

### 4. Sistemas WIDS/WIPS: 
Utilizar sistemas de detección y prevención de intrusiones inalámbricas para monitorizar la aparición de puntos de acceso con MACs duplicadas (BSSID Spoofing) o nombres idénticos (ESSID Spoofing) en el entorno corporativo.

<br/>

---
Este proyecto ha sido realizado con fines educativos en un entorno controlado.
