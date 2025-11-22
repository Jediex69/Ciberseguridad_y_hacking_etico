# 🚨 Informe de Vulnerabilidad Detectada en Sistema Linux

**Autor:** Jesús Díaz
**Tipo de Documento:** Informe CTF / Auditoría de Seguridad

-----

## 📄 1. Informe Ejecutivo

### 1.1. Introducción 🎯

Se ha identificado una posible brecha de seguridad en uno de los sistemas Linux de la organización. A través de una prueba de acceso (pentest), fue posible entrar en una sección del sistema no protegida, permitiendo la obtención de credenciales de acceso. Esto demuestra una debilidad en la configuración que podría ser explotada por un atacante, poniendo en riesgo la información.

### 1.2. Alcance 🔍

El análisis implicó un acceso no autorizado que permitió visualizar y extraer información sensible desde un panel de administración oculto.

  * Se obtuvieron credenciales de usuario, otorgando capacidad de actuación dentro del sistema con permisos válidos.
  * Este acceso compromete la confidencialidad y podría usarse para escalar privilegios o afectar la disponibilidad de servicios.

### ⚠️ Riesgos Asociados

  * **Robo de información:** Las credenciales obtenidas podrían utilizarse para acceder a otros servicios internos.
  * **Alteración de sistemas:** El acceso administrativo podría permitir la modificación o borrado de archivos críticos.
  * **Persistencia:** Si no se detecta, este acceso podría mantenerse en el tiempo sin ser advertido.
  * **Falta de visibilidad:** La ausencia de documentación sobre este acceso indica posibles deficiencias en la monitorización.

### 1.3. Problemas Encontrados 🛑

Durante el análisis se detectaron fallos que, combinados, permiten el acceso no autorizado:

1.  **Acceso no documentado:** Existe una vía de entrada al sistema que no está registrada ni protegida adecuadamente.
2.  **Falta de filtros de red:** El sistema Linux permite conexiones exteriores sin restricciones, facilitando su descubrimiento y explotación.
3.  **Panel oculto sin autenticación:** Se halló un panel de administración accesible sin protección, facilitando el acceso a información sensible.
4.  **Credenciales en texto claro:** Las claves obtenidas no estaban cifradas, facilitando su extracción y posterior utilización.
5.  **Ausencia de monitorización:** No existen sistemas de registro de actividad ni alertas que avisen de conexiones anómalas.

### 1.4. Soluciones y Recomendaciones ✅

Para mitigar estos riesgos y establecer una cultura de seguridad preventiva, se propone: 

  * **1. Cierre y revisión de accesos no documentados:** Cualquier acceso no autorizado debe ser eliminado inmediatamente.
  * **2. Aplicación de un firewall 🧱:** Implementar reglas que limiten las conexiones entrantes a los servicios necesarios, reduciendo la superficie de ataque.
  * **3. Fortalecimiento de la autenticación 🔑:** Proteger interfaces de administración con autenticación robusta (ej. 2FA) y almacenar credenciales de forma segura.
  * **4. Monitorización y alertas de seguridad 🔔:** Configurar sistemas de registro de actividad (logs) y herramientas de monitorización para detectar accesos anómalos.
  * **5. Auditorías de seguridad periódicas 🔄:** Realizar revisiones regulares para identificar vulnerabilidades antes de que puedan ser explotadas.
  * **6. Documentación y control de cambios 📝:** Mantener un registro actualizado de todas las configuraciones y servicios activos.
  * **7. Formación al personal 🧑‍🏫:** Capacitar y concienciar al personal sobre ciberseguridad básica y detección de configuraciones inseguras.

-----

## 🛠️ 2. Informe Técnico 

### 2.1. Introducción

Este informe técnico documenta el análisis realizado sobre el sistema Linux tras detectarse un tráfico de red anómalo que permitió descubrir una vía de entrada insegura. El objetivo fue identificar el origen de la brecha y las vulnerabilidades explotadas.

### 2.2. Herramientas Empleadas 🧰

  * **Kali Linux:** Distribución basada en Debian, utilizada como entorno principal de trabajo por su versatilidad y amplio conjunto de herramientas para análisis de seguridad.
  * **Wireshark:** Herramienta de análisis de protocolos de red para capturar y examinar el tráfico en detalle. Clave para detectar el tráfico anómalo.
  * **Netcat:** Utilidad versátil de red que permite leer y escribir datos a través de conexiones TCP o UDP. Se utilizó para conectarse directamente al servicio expuesto y simular el comportamiento del atacante.

### 2.3. Proceso de Explotación 💥

El objetivo fue simular un ataque paso a paso hasta obtener credenciales válidas.

#### Paso 1: Análisis de Tráfico (Wireshark) 🚦

Se detectaron comunicaciones sospechosas entre el sistema Linux (`10.0.3.4`) y la máquina Kali (`10.0.3.6`).

  * **Hallazgo:** Envío periódico de paquetes desde la IP `10.0.3.4` por el puerto **6666** hacia la IP `10.0.3.6` por el puerto **3333**.
  * Esto sugirió la existencia de un servicio activo expuesto sin medidas de protección.

#### Paso 2: Descubrimiento del Servicio Oculto 🕵️

Se utilizó Netcat para escuchar en el puerto **3333** de la máquina Kali, confirmando el servicio.

```bash
$ nc -lp 3333
listening on [any] 3333 ...
connect to [10.0.3.6] from (UNKNOWN) [10.0.3.4] 6666
Panel de administración en el puerto 4444
```

*Resultado:* Se confirmó la existencia de un panel de administración en el puerto **4444** del sistema Linux.

#### Paso 3: Interacción con el Servicio usando Netcat 💻

Se estableció conexión manual al puerto 4444 sin que se solicitara autenticación.

```bash
$ nc 10.0.3.4 4444
Bienvenido al panel de administracion s3cr3t0:
```

Al probar con el comando `help`, se listaron los comandos disponibles: `help, adduser, getdinosaur, getpassword`.

#### Paso 4: Acceso a Credenciales 🔓

Se ejecutó el comando `getpassword`, revelando la clave en texto plano.

```text
getpassword
La constraseña de administrador es: noteladigo
```

  * **Credenciales obtenidas:**
      * **Usuario:** `administrador`
      * **Contraseña:** `noteladigo`

Se probó el inicio de sesión con éxito en el sistema Linux. El sistema afectado es **Ubuntu 20.04.3 LTS**.

### Resultado Final de la Explotación

  * Se obtuvo acceso remoto no autorizado al sistema.
  * Se accedió a un panel de administración sin protección.
  * Se extrajeron credenciales válidas almacenadas en texto plano.
  * El sistema Linux objetivo no generó alertas ni bloqueos.

[cite\_start]La explotación fue exitosa y pone de manifiesto la urgencia de aplicar medidas correctivas[cite: 297].

