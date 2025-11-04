````{"id":"92746","variant":"standard","title":"README - CTF 02 Reto Python"}
# CTF 02 — Reto Python

**Autor:** Jesús Díaz  
**Bootcamp:** Cyberskills Red Team 2025  
**Fecha:** 22/05/2025

---

## Descripción breve
Reto tipo *CTF* (laboratorio controlado) diseñado para practicar análisis de scripts en Python, decodificación, razonamiento lógico y escalada de privilegios en Linux. Se parte de un usuario inicial (`user1`) y, mediante la resolución de 10 niveles basados en scripts Python, se progresa hasta obtener una shell `root`.

> ⚠️ Este reto se realizó en un entorno virtual seguro y controlado. Todas las cuentas, contraseñas e IP son ficticias y solo se usaron con fines educativos.

---

## Estructura del repositorio
- `solucion_reto.pdf` — Informe completo del reto (paso a paso, evidencias y conclusiones).
- `evidencias/` — Directorio con salidas y archivos recogidos (si procede).
- `scripts/` — Copias de los `nivel*.py` analizados (si procede).
- `README.md` — Este fichero.

---

## Objetivos del reto
- Analizar el comportamiento de scripts Python.
- Aplicar técnicas de decodificación (Base64, hex, reverse).
- Identificar lógicas vulnerables o predecibles (hardcoding, dependencias externas, parámetros de línea de comandos).
- Practicar enumeración en Linux y escalada de privilegios (uso de `sudo` y revisión de permisos).
- Documentar y preservar evidencias del proceso de intrusión controlada.

---

## Resumen rápido de la progresión (contraseñas obtenidas)
| Nivel | De → A | Técnica clave | Contraseña obtenida |
|---:|:---:|:---|:---|
| 1 | user1 → user2 | Ejecución directa del script | `S0Easy&Gu1d3d` |
| 2 | user2 → user3 | Lectura de código (comentario) | `N0C0mm3nts!` |
| 3 | user3 → user4 | Base64 (hardcoded) | `H4rdC0de` |
| 4 | user4 → user5 | Argumentos de línea de comandos | `P@ramet3rs` |
| 5 | user5 → user6 | Recurso externo (HTTP) — clave en gist | `Ext3rnalK3Y` |
| 6 | user6 → user7 | Manipulación de fichero (`.registered_users.db`) | `=0neM1llion` |
| 7 | user7 → user8 | Cliente/servidor (OTP local por sockets) | `TCP!S0ck3t` |
| 8 | user8 → user9 | Lógica determinista dependiente de la hora | `N0tS0Rand0m!` |
| 9 | user9 → user10 | Hex + reverse (encoded pass) | `ModifyPrivs&Win` |
|10 | user10 → root | `user10` pertenece al grupo `sudo` → `sudo su` | *root shell* |

---

## Cómo reproducir (resumen)
> Ejecuta TODO esto **solo** en un laboratorio controlado.

1. Conéctate al host de laboratorio:
```bash
ssh user1@10.0.3.9
# password: St@rt1ng
```

2. En cada directorio `~/Documentos` de cada usuario:
   - Leer el script (`cat nivelX.py`) y/o ejecutarlo (`python3 nivelX.py`) siguiendo las instrucciones del reto.
   - Aplicar las técnicas descritas en el informe para extraer la contraseña del siguiente usuario.
   - Cambiar de usuario con `su userN` y repetir.

3. Para el nivel 6 (ejemplo práctico) crear un archivo con 999 999 líneas:
```bash
yes "you@example.com" | head -n 999999 > .registered_users.db
python3 nivel6.py
# introducir el email final y obtener la contraseña
```

4. En el último paso, `user10` tiene permisos `sudo`:
```bash
groups user10        # muestra 'sudo'
sudo su              # introducir la contraseña de user10 → shell root
```

---

## Checklist mínimo de evidencias (para adjuntar al informe)
Generar y guardar las salidas de los siguientes comandos (cada uno en su fichero con timestamp y hash SHA256):

- `hostname`, `id`, `date`, `uname -a`, `/etc/os-release`  
- `cat /etc/passwd`, extracto de `/etc/shadow` (solo root)  
- `groups user10`, `sudo -l` (o `sudo -l -U user10`)  
- Registro del paso de escalada (comando `sudo su` y `whoami`)  
- Copia de los scripts `nivel1.py` ... `nivel9.py` analizados  
- Salidas de `find / -perm /6000 -type f` (SUID/SGID)  
- Logs relevantes: `/var/log/auth.log`, `/var/log/syslog` (extractos)  
- Índice con hashes SHA256 de todos los ficheros de evidencia

(Para un checklist completo y guía de comandos vea `solucion_reto.pdf`).

---

## Observaciones de seguridad (lecciones aprendidas)
- **Hardcoding** de credenciales en scripts o comentarios es una mala práctica grave.  
- **Ofuscación por Base64 o hex** no protege secretos; es reversible trivialmente.  
- **Dependencias externas** (peticiones HTTP para obtener claves) introducen fragilidad y vectores de manipulación.  
- **Lógicas deterministas** (ej. basadas en la hora) son predecibles y explotables.  
- **Permisos `sudo` amplios** en cuentas de usuario (como `user10`) conllevan riesgo de escalada completa; se recomienda restringir comandos permitidos y auditar uso.

---

## Recomendaciones de mitigación
- No almacenar contraseñas en texto plano ni en comentarios; usar vaults o gestores seguros.  
- Evitar `NOPASSWD` y limitar comandos permitidos en `/etc/sudoers.d/`.  
- Validar y sanitizar entradas de usuarios; no confiar en recursos externos.  
- Registrar y auditar accesos y uso de `sudo`.  
- Revisar permisos de ficheros y revisar SUID/SGID periódicamente.

---

## Licencia y aviso legal
Este repositorio y el contenido asociado son **material educativo**. No usar ni ejecutar estos scripts en sistemas de producción o redes públicas. El autor no respalda ni aprueba actividades no autorizadas.

---

## Contacto
Jesús Díaz — Bootcamp Cyberskills Red Team 2025

---
