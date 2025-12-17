# CTF05 - Reto infraestructura🛡️

Este repositorio documenta un ejercicio de análisis ofensivo (Ethical Hacking) realizado sobre una red interna, enfocándose en el compromiso de una máquina objetivo Linux Lubuntu. El proyecto demuestra la identificación y explotación de vulnerabilidades críticas debidas a configuraciones inseguras y tráfico no cifrado.

> **⚠️ DISCLAIMER:** Este material ha sido creado con fines puramente académicos y educativos. Las técnicas demostradas se realizaron en un entorno de laboratorio controlado y autorizado. El acceso no autorizado a sistemas informáticos es ilegal.

## 📋 Resumen Ejecutivo

El objetivo de la auditoría fue evaluar el nivel de exposición de una máquina **Lubuntu (10.0.3.16)** dentro de una red interna. A través de un enfoque de **Caja Negra**, se logró el compromiso total del sistema (acceso root/usuario) explotando una cadena de vulnerabilidades que incluyó la interceptación de credenciales y el abuso de recursos compartidos mal configurados.

#### 🎯 Alcance
* **Atacante:** Kali Linux (10.0.3.6).
* **Víctima:** Lubuntu Linux (10.0.3.16).
* **Cliente secundario:** Máquina "Retillo" (10.0.3.15) utilizada para generar tráfico.

## 🛠️ Herramientas Utilizadas

Se emplearon herramientas estándar de la industria para las fases de reconocimiento y explotación:
* **Nmap:** Escaneo de puertos y detección de servicios.
* **Arp-scan:** Descubrimiento de hosts en la red.
* **Ettercap:** Ejecución de ataques Man-in-the-Middle (MITM) vía ARP Poisoning.
* **Wireshark:** Análisis de tráfico y captura de credenciales.
* **Knock:** Cliente para realizar secuencias de Port Knocking.
* **OpenSSH & Mount:** Acceso a servicios remotos y sistemas de archivos.

## ⚔️ Kill Chain (Paso a Paso)

#### 1. Reconocimiento y Escaneo
Se identificaron los hosts activos mediante `arp-scan`. Posteriormente, un escaneo con `nmap` reveló que la máquina objetivo solo mostraba inicialmente el puerto **32013 (Apache HTTP)** abierto, ocultando servicios críticos.

#### 2. Ataque MITM y Captura de Credenciales
Al detectar tráfico entre el cliente "Retillo" y el servidor "Lubuntu", se ejecutó un ataque de **ARP Spoofing** con Ettercap.
* **Hallazgo:** El tráfico de login viajaba en texto plano (HTTP).
* **Captura:** Se interceptaron las credenciales `usuario=admin` y `palabra_secreta=LaBarbacoa` mediante Wireshark.

#### 3. Acceso Web y Port Knocking
Con las credenciales capturadas, se accedió al panel web en el puerto 32013. Se encontró una pista visual ("Knocking on heavens door") y una serie de números: **7003, 8004, 9005**.
* **Técnica:** Port Knocking (Evasión de defensas).
* **Ejecución:**

```
knock 10.0.3.16 7003 8004 9005
```

* **Resultado:** El firewall abrió los puertos **22 (SSH)**, **111 (RPC)** y **2049 (NFS)**.

#### 4. Explotación de NFS y Robo de Claves
El servicio SSH rechazó las credenciales web. Se procedió a enumerar recursos NFS, encontrando `/mnt/nfs_share` exportado sin restricciones.
* **Acción:** Se montó el recurso compartido en la máquina atacante.
* **Extracción:** Se localizó una clave privada SSH (`id_rsa`) en el directorio oculto `.ssh` dentro del recurso compartido.

#### 5. Acceso Inicial y Control
Utilizando la clave privada robada, se estableció una conexión SSH exitosa como el usuario `ubuntu`, logrando acceso total al sistema y capacidad de ejecución de comandos.

```
ssh -i sshkey ubuntu@10.0.3.16
````

## 📊 Mapeo de Vulnerabilidades

#### MITRE ATT&CK Matrix

Las técnicas observadas se alinean con el marco MITRE ATT&CK de la siguiente manera:

| **Táctica**           | **Técnica**             | **ID**    | **Descripción**                                    |
| --------------------- | ----------------------- | --------- | -------------------------------------------------- |
| **Credential Access** | Network Sniffing        | T1040     | Captura de credenciales HTTP plano vía MITM.       |
| **Defense Evasion**   | Port Knocking           | T1205.001 | Ocultación de puertos SSH hasta recibir secuencia. |
| **Discovery**         | Network Share Discovery | T1135     | Enumeración de carpetas NFS abiertas.              |
| **Credential Access** | Unsecured Credentials   | T1552.004 | Robo de claves SSH privadas en disco compartido.   |
| **Lateral Movement**  | Remote Services: SSH    | T1021.004 | Uso de clave robada para pivotar al servidor.      |

#### OWASP Top 10 (2021)

Se identificaron fallos correspondientes a las siguientes categorías críticas2:

- **A02: Cryptographic Failures:** Login sin HTTPS.   
- **A01: Broken Access Control:** NFS sin restricciones de IP/usuario.    
- **A06: Vulnerable and Outdated Components:** Configuración insegura de servicios.    

## 🛡️ Medidas de Mitigación Recomendadas

Basado en los hallazgos, se recomiendan las siguientes acciones correctivas inmediatas:

1. **Cifrado de Tránsito:** Implementar HTTPS para todos los paneles de autenticación y evitar protocolos de texto plano como HTTP o Telnet.    
2. **Hardening de NFS:** Restringir el acceso a los recursos compartidos NFS únicamente a direcciones IP de confianza y evitar compartir directorios sensibles (como `/home` o `.ssh`).    
3. **Gestión de Claves:** Proteger las claves privadas SSH con _passphrases_ robustas y asegurar que los permisos de archivo sean estrictos (600).    
4. **Segmentación de Red:** Aislar servicios críticos y monitorear el tráfico interno para detectar ataques de ARP Spoofing.

