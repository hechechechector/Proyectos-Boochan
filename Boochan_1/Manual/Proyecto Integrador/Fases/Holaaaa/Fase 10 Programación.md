<hr style="border: none; height: 5px; background: linear-gradient(to right, rgb(120, 0, 255), rgb(0, 166, 255), transparent); margin-bottom: 20px;">

<div style="float: right; margin-right: 80px; margin-left: 20px; pointer-events: none;">
  <img src="drstrange.gif" style="width: 120px;">
</div>

> [!abstract] **OBJETIVOS DE LA FASE DR STRANGE (PROG)**
> 
> 1. **Procesamiento de datos reales:** Leer un log del servidor y extraer eventos relevantes.
> 2. **Evidencias en texto:** Generar ficheros `.txt` con un registro de eventos y un resumen por ejecución.
> 3. **Procesar solo lo nuevo:** Evitar releer el log completo cada vez (control de estado con `.state`).


## 🧙‍♂️ Fase 10 - DR STRANGE - Gestor de Eventos del Servidor con Python (PROG)

En las fases anteriores ya se ha configurado el servidor y se han puesto en marcha servicios que generan información útil (logs).  
En esta fase vas a construir una herramienta sencilla y práctica:

✅ Lee un log (real o de pruebas).  
✅ Detecta eventos. Puedes registrar solamente los intentos SSH fallidos o accesos correctos. Estos eventos deben tener también la fecha y hora del momento en el que se ha producido.  
✅ Guarda evidencias acumuladas en ficheros de texto que tengan nombres claros de la información almacenada.  
✅ Recuerda por dónde iba, para procesar **solo las líneas nuevas** en cada ejecución.


### 🔎 S10.1 — Investigación mínima (antes de programar)

Investiga y toma apuntes breves sobre:

- **Lectura y escritura de ficheros de texto** (`open`, `r`, `a`, `encoding`).
- **Guardar estado de lectura** con `tell()` / `seek()` y un fichero `.state`.
- **Filtrado de texto** con `in` (búsqueda simple en líneas).
- **Fecha y hora** con `datetime.now()` + `strftime()`.
- **Control de errores** con `try/except` si el log no existe o no se puede leer.


### 🧩 S10.2 — Enunciado

El equipo Boochan necesita una herramienta que deje evidencias de lo que está ocurriendo en el servidor.  
El objetivo no es “mirar a mano” los logs, sino **registrar eventos importantes** en un formato claro.

Vas a crear un script llamado:

- `event_watcher.py`

que analice un log y genere dos evidencias en texto:

- `evidencias/eventos.txt`  → registro acumulativo de eventos detectados.
- `evidencias/resumen_eventos.txt` → resumen por ejecución (cuántos eventos nuevos hay).

Además, debe existir:

- `evidencias/.state` → guarda el “punto de lectura” para no repetir líneas ya procesadas.


### 🧪 S10.3 — Log a analizar (elige 1 opción)

#### ✅ Opción A (log real)
Intenta usar:
- `/var/log/auth.log`

> Nota: Si no tienes permisos o no existe, usa la opción B.

#### ✅ Opción B (log de pruebas)
Crea un fichero en la carpeta del proyecto:
- `auth_pruebas.log`

Contenido de ejemplo:

- Jan 12 12:00:01 ubuntu sshd[1111]: Failed password for root from 203.0.113.10 port 5555 ssh2  
- Jan 12 12:00:08 ubuntu sshd[1112]: Failed password for invalid user admin from 203.0.113.11 port 5556 ssh2  
- Jan 12 12:01:10 ubuntu sshd[1113]: Accepted password for iker from 192.168.0.50 port 51234 ssh2  
- Jan 12 12:02:10 ubuntu sshd[1114]: Failed password for iker from 203.0.113.12 port 5557 ssh2  
- Jan 12 12:05:10 ubuntu sshd[1115]: Accepted password for iker from 192.168.0.51 port 51235 ssh2


### 🧱 S10.4 — Preparación de estructura (carpetas y ficheros)

En la carpeta del proyecto, crea la estructura:

- `event_watcher.py`
- `evidencias/`
  - `eventos.txt`
  - `resumen_eventos.txt`
  - `.state`


### 🛠️ S10.5 — Reglas de detección (mínimo obligatorio)

Tu script debe detectar 2 tipos de evento:

- **SSH_FAIL** → si la línea contiene: `Failed password`
    
- **SSH_OK** → si la línea contiene: `Accepted password`
    

> (Opcional: extraer usuario e IP, pero NO es obligatorio.)


### 🧠 S10.6 — Requisito clave: procesar SOLO lo nuevo (estado con `.state`)

Cada vez que ejecutes el script:

1. Lee la última posición guardada en `evidencias/.state`
    
2. Haz `seek(posicion)`
    
3. Lee líneas nuevas desde ahí
    
4. Al terminar, guarda la nueva posición con `tell()`
    

📌 Si `.state` está vacío o no existe, empieza desde el inicio del log.


### ✍️ S10.7 — Evidencias en texto

#### ✅ A) Registro de eventos (acumulativo)

Archivo: `evidencias/eventos.txt` (modo `a` para añadir al final del archivo)

Por cada evento detectado, añade una línea con:

- `timestamp_ejecucion | tipo_evento | linea_original`
    

Ejemplo:

- 2026-01-12 19:20:00 | SSH_FAIL | Jan 12 12:00:01 ubuntu sshd[1111]: Failed password...
- 2026-01-12 19:20:00 | SSH_OK   | Jan 12 12:01:10 ubuntu sshd[1113]: Accepted password...`

#### ✅ B) Resumen por ejecución (acumulativo)

Archivo: `evidencias/resumen_eventos.txt` (modo `a` para añadir al final del archivo)

Cada ejecución debe añadir:

		[2026-01-12 19:20:00] 
		Nuevos SSH_FAIL: 2 
		Nuevos SSH_OK: 


### 🧪 S10.8 — Pruebas (cómo demostrar que funciona)

#### ✅ Prueba 1: Primera ejecución

- Ejecuta el script
    
- Comprueba que:
    
    - se han añadido líneas en `eventos.txt`
        
    - se ha añadido un bloque en `resumen_eventos.txt`
        
    - `.state` ya contiene un número (posición)
        

#### ✅ Prueba 2: Segunda ejecución sin cambios

- Ejecuta el script otra vez sin modificar el log
    
- El resumen debería indicar:
    
    - `SSH_FAIL: 0`
        
    - `SSH_OK: 0`
        

#### ✅ Prueba 3: Nuevos eventos

- Añade 1–2 líneas nuevas al log de pruebas (`auth_pruebas.log`)
    
- Ejecuta el script
    
- Debe registrar SOLO esas líneas nuevas


## ENTREGABLES

- `event_watcher.py`
    
- `evidencias/eventos.txt` (con eventos acumulados)
    
- `evidencias/resumen_eventos.txt` (con varias ejecuciones)
    
- `evidencias/.state` (para demostrar que se procesa solo lo nuevo)
    
- (si aplica) `auth_pruebas.log`
    
- `README.md` (muy breve):
    
    - cómo ejecutar
        
    - qué detecta
        
    - qué ficheros genera