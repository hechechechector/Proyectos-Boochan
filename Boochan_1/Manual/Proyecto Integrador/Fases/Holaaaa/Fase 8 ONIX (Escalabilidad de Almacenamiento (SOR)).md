
<hr style="border: none; height: 5px; background: linear-gradient(to right, rgb(44, 62, 80), rgb(0, 210, 255), transparent); margin-bottom: 20px;">

## 🛠️ Versión A: Gestión vía Live-ISO (VirtualBox + GParted)

_Este método utiliza la interfaz gráfica cargando la ISO de GParted antes de iniciar el sistema._

### 1. Preparación del Hardware y Booteo

- **Ampliacion del disco:** En VirtualBox, vamos a Archivo -> Herramientas -> Media -> Propiedades -> Seleccionamos el disco que querramos ampliar y aumentamos el espacio hasta **100 GB**.
    
- **Carga de ISO:** En Configuracion -> Almacenamiento, insertamos la ISO de **gparted-live-xxx.iso** en controlador IDE como disco optico.
    
- **Arranque:** Entramos en configuración y desactivamos todos los arranques menos el de la unidad optica, para darle prioridad maxima al gparted, si salta error, simplemente elegir gparted y darle al boton de reiniciar que salta.
- Seleccionamos la opción Other Modes of GParted Live -> Gàrted Ñove (Safe Graphic settings)	 <div style="float: center; pointer-events: none;"> <img src="gparted1.png" style="width: 550px;"> </div> <div style="float: center; pointer-events: none;"> <img src="gparted2.png" style="width: 550px;"> </div>

Al entrar, este sera el orden que tenemos que meter:
	1. Don't touch keymap
	2. 25 # Para poner el idioma en español
	3. 0
### 2. Particionado Visual

<div style="float: right; margin-right: 50px; margin-left: 20px; pointer-events: none;"> <img src="onix.png" style="width: 150px;"> </div>

- Seleccionamos la unidad de **100GB** (normalmente `/dev/sdb`).
    
- **Tabla de particiones:** `Device` -> `Create Partition Table` -> **GPT**.
    
- **/dev/sda3:**(O como se llame el disco principal) Click derecho y redimensionar al maximo.
        
- **Aplicar:** Clic en el botón redimensionar.

- **Confirmacion:** En la barra de arriba, darle click en el ✔️ verde. 
    
- **Cierre:** Apagamos la VM y retiramos la ISO para iniciar Ubuntu normalmente.
    

---

## 💻 Versión B: Gestión desde el Servidor (CLI / Terminal)

_Este método se realiza directamente con el servidor encendido, ideal para el ODROID-C2 o gestión remota._

### 1. Detección del Disco

Identificamos el nuevo volumen de 100GB (que aparecerá como espacio sin particionar):

Bash

```
sudo fdisk -l   # Localizamos el dispositivo (ej: /dev/sdb)
```

### 2. Particionado y Formateo por Comandos

Utilizamos la potencia de la terminal para realizar el proceso sin entorno gráfico:

<div style="float: right; margin-right: 600px; pointer-events: none;"> <img src="Porygon.gif" style="width: 80px;"> </div>

Bash

```
# Crear la tabla de particiones GPT
sudo parted /dev/sdb mklabel gpt

# Crear la partición primaria (0% al 100% del disco)
sudo parted /dev/sdb mkpart primary ext4 0% 100%

# Formatear la partición resultante (/dev/sdb1) con Etiqueta
sudo mkfs.ext4 -L BACKUPS /dev/sdb1
```

---

## 🔗 S8.3: Protocolo de Montaje Persistente (Común)

Una vez preparado el disco (por el método A o B), debemos asegurar que el servidor lo reconozca siempre:

1. **Punto de anclaje:** `sudo mkdir -p /mnt/backups`
    
2. **Permisos:** `sudo chown -R $USER:$USER /mnt/backups` <div style="float: right; margin-right: 50px; margin-left: 20px; pointer-events: none;"> <img src="steelix.png" style="width: 200px;"> </div>
    
3. **Edición del FSTAB:** `sudo nano /etc/fstab` 
    
    - Añadimos la línea: `LABEL=BACKUPS /mnt/backups ext4 defaults 0 2`
        
4. **Validación:**
    

Bash

```
sudo mount -a
df -h | grep BACKUPS
```


> **Estado Final:** Unidad de 100GB operativa y vinculada. La infraestructura está lista para recibir los repositorios de **Borg Backup** de la Fase 7.

<hr style="border: none; height: 5px; background: linear-gradient(to left, rgb(44, 62, 80), rgb(0, 210, 255), transparent); margin-bottom: 20px;">

FASE 8 Almacenamiento →→ [[Fase 9 DARK SOULS (Auditoría de Seguridad y Pentesting (SI))]]


![[gparted1.png]]