# [cite\_start]🚨 Informe de Vulnerabilidad Detectada en Sistema Linux [cite: 1]

[cite\_start]**Autor:** Jesús Díaz [cite: 2]
**Tipo de Documento:** Informe CTF / Auditoría de Seguridad

-----

## [cite\_start]📄 1. Informe Ejecutivo [cite: 5]

### 1.1. Introducción 🎯

[cite\_start]Se ha identificado una posible brecha de seguridad en uno de los sistemas Linux de la organización[cite: 7]. [cite\_start]A través de una prueba de acceso (pentest), fue posible entrar en una sección del sistema no protegida, permitiendo la obtención de credenciales de acceso[cite: 8]. [cite\_start]Esto demuestra una debilidad en la configuración que podría ser explotada por un atacante, poniendo en riesgo la información[cite: 9].

### 1.2. Alcance 🔍

[cite\_start]El análisis implicó un acceso no autorizado que permitió visualizar y extraer información sensible desde un panel de administración oculto[cite: 12].

  * [cite\_start]Se obtuvieron credenciales de usuario, otorgando capacidad de actuación dentro del sistema con permisos válidos[cite: 13].
  * [cite\_start]Este acceso compromete la confidencialidad y podría usarse para escalar privilegios o afectar la disponibilidad de servicios[cite: 14].

### [cite\_start]⚠️ Riesgos Asociados [cite: 15]

  * [cite\_start]**Robo de información:** Las credenciales obtenidas podrían utilizarse para acceder a otros servicios internos[cite: 16].
  * [cite\_start]**Alteración de sistemas:** El acceso administrativo podría permitir la modificación o borrado de archivos críticos[cite: 17].
  * [cite\_start]**Persistencia:** Si no se detecta, este acceso podría mantenerse en el tiempo sin ser advertido[cite: 19].
  * [cite\_start]**Falta de visibilidad:** La ausencia de documentación sobre este acceso indica posibles deficiencias en la monitorización[cite: 20].

### 1.3. Problemas Encontrados 🛑

[cite\_start]Durante el análisis se detectaron fallos que, combinados, permiten el acceso no autorizado[cite: 25]:

1.  [cite\_start]**Acceso no documentado:** Existe una vía de entrada al sistema que no está registrada ni protegida adecuadamente[cite: 27, 28].
2.  [cite\_start]**Falta de filtros de red:** El sistema Linux permite conexiones exteriores sin restricciones, facilitando su descubrimiento y explotación[cite: 29].
3.  [cite\_start]**Panel oculto sin autenticación:** Se halló un panel de administración accesible sin protección, facilitando el acceso a información sensible[cite: 30].
4.  [cite\_start]**Credenciales en texto claro:** Las claves obtenidas no estaban cifradas, facilitando su extracción y posterior utilización[cite: 31].
5.  [cite\_start]**Ausencia de monitorización:** No existen sistemas de registro de actividad ni alertas que avisen de conexiones anómalas[cite: 32].

### 1.4. Soluciones y Recomendaciones ✅

[cite\_start]Para mitigar estos riesgos y establecer una cultura de seguridad preventiva, se propone: [cite: 35]

  * **1. [cite\_start]Cierre y revisión de accesos no documentados:** Cualquier acceso no autorizado debe ser eliminado inmediatamente[cite: 36, 38].
  * **2. [cite\_start]Aplicación de un firewall 🧱:** Implementar reglas que limiten las conexiones entrantes a los servicios necesarios, reduciendo la superficie de ataque[cite: 40, 41].
  * **3. [cite\_start]Fortalecimiento de la autenticación 🔑:** Proteger interfaces de administración con autenticación robusta (ej. 2FA) y almacenar credenciales de forma segura[cite: 42, 43, 44].
  * **4. [cite\_start]Monitorización y alertas de seguridad 🔔:** Configurar sistemas de registro de actividad (logs) y herramientas de monitorización para detectar accesos anómalos[cite: 45, 46].
  * **5. [cite\_start]Auditorías de seguridad periódicas 🔄:** Realizar revisiones regulares para identificar vulnerabilidades antes de que puedan ser explotadas[cite: 47, 48].
  * **6. [cite\_start]Documentación y control de cambios 📝:** Mantener un registro actualizado de todas las configuraciones y servicios activos[cite: 49, 50].
  * **7. [cite\_start]Formación al personal 🧑‍🏫:** Capacitar y concienciar al personal sobre ciberseguridad básica y detección de configuraciones inseguras[cite: 52, 53].

-----

## [cite\_start]🛠️ 2. Informe Técnico [cite: 55]

### 2.1. Introducción

[cite\_start]Este informe técnico documenta el análisis realizado sobre el sistema Linux tras detectarse un tráfico de red anómalo que permitió descubrir una vía de entrada insegura[cite: 57]. [cite\_start]El objetivo fue identificar el origen de la brecha y las vulnerabilidades explotadas[cite: 58].

### 2.2. Herramientas Empleadas 🧰

  * [cite\_start]**Kali Linux:** Distribución basada en Debian, utilizada como entorno principal de trabajo por su versatilidad y amplio conjunto de herramientas para análisis de seguridad[cite: 61, 62, 64].
  * [cite\_start]**Wireshark:** Herramienta de análisis de protocolos de red para capturar y examinar el tráfico en detalle[cite: 65, 66]. [cite\_start]Clave para detectar el tráfico anómalo[cite: 68].
  * [cite\_start]**Netcat:** Utilidad versátil de red que permite leer y escribir datos a través de conexiones TCP o UDP[cite: 69, 70]. [cite\_start]Se utilizó para conectarse directamente al servicio expuesto y simular el comportamiento del atacante[cite: 72, 73].

### 2.3. Proceso de Explotación 💥

[cite\_start]El objetivo fue simular un ataque paso a paso hasta obtener credenciales válidas[cite: 75].

#### Paso 1: Análisis de Tráfico (Wireshark) 🚦

[cite\_start]Se detectaron comunicaciones sospechosas entre el sistema Linux (`10.0.3.4`) y la máquina Kali (`10.0.3.6`)[cite: 78, 203].

  * [cite\_start]**Hallazgo:** Envío periódico de paquetes desde la IP `10.0.3.4` por el puerto **6666** hacia la IP `10.0.3.6` por el puerto **3333**[cite: 203, 204].
  * [cite\_start]Esto sugirió la existencia de un servicio activo expuesto sin medidas de protección[cite: 205].

#### Paso 2: Descubrimiento del Servicio Oculto 🕵️

Se utilizó Netcat para escuchar en el puerto **3333** de la máquina Kali, confirmando el servicio.

```bash
$ nc -lp 3333
listening on [any] 3333 ...
connect to [10.0.3.6] from (UNKNOWN) [10.0.3.4] 6666
Panel de administración en el puerto 4444
```

[cite\_start]*Resultado:* Se confirmó la existencia de un panel de administración en el puerto **4444** del sistema Linux[cite: 211, 218].

#### Paso 3: Interacción con el Servicio usando Netcat 💻

[cite\_start]Se estableció conexión manual al puerto 4444 sin que se solicitara autenticación[cite: 221, 222].

```bash
$ nc 10.0.3.4 4444
Bienvenido al panel de administracion s3cr3t0:
```

[cite\_start]Al probar con el comando `help`, se listaron los comandos disponibles: `help, adduser, getdinosaur, getpassword`[cite: 235, 237].

#### Paso 4: Acceso a Credenciales 🔓

[cite\_start]Se ejecutó el comando `getpassword`, revelando la clave en texto plano[cite: 252, 253].

```text
getpassword
La constraseña de administrador es: noteladigo
```

  * **Credenciales obtenidas:**
      * [cite\_start]**Usuario:** `administrador` [cite: 256]
      * [cite\_start]**Contraseña:** `noteladigo` [cite: 257]

Se probó el inicio de sesión con éxito en el sistema Linux. [cite\_start]El sistema afectado es **Ubuntu 20.04.3 LTS**[cite: 258, 261].

### Resultado Final de la Explotación

  * [cite\_start]Se obtuvo acceso remoto no autorizado al sistema[cite: 292].
  * [cite\_start]Se accedió a un panel de administración sin protección[cite: 293].
  * [cite\_start]Se extrajeron credenciales válidas almacenadas en texto plano[cite: 294].
  * [cite\_start]El sistema Linux objetivo no generó alertas ni bloqueos[cite: 295].

[cite\_start]La explotación fue exitosa y pone de manifiesto la urgencia de aplicar medidas correctivas[cite: 297].

