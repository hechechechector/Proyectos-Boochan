<hr style="border: none; height: 5px; background: linear-gradient(to right, rgb(0, 0, 0), rgb(255, 215, 0), transparent); margin-bottom: 20px;">


> [!abstract] **OBJETIVOS DE LA FASE BATMAN**
> 
> 1. **Defensa Reactiva:** Configurar Fail2Ban para bloqueo automático de intrusos. 
>     
> 2. **Criptografía Asimétrica:** Implementar Llaves RSA y desactivar contraseñas en SSH.
>     
> 3. **Cifrado Profesional:** Eliminar advertencias de seguridad mediante certificados de confianza.
>     

---

### 🛡️ S6.1: Defensa Reactiva (Fail2Ban)

El servidor ahora "aprende" de los ataques y bloquea las IPs maliciosas automáticamente.

 <div style="float: right; margin-right: 50px; margin-left: 520px; pointer-events: none;"> <img src="batman1.gif" style="width: 200px;"> </div>

**Comandos de implementación:**


```
# Instalación del servicio
sudo apt update && sudo apt install fail2ban -y

# Configuración de protección SSH
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### 🛡️ S6.1.1: Configuracion de Fail2Ban

La configuración la realizaremos en la carpeta local para que nuestras reglas tengan prioridad:
````
sudo nano /etc/fail2ban/jail.local

# Dentro de la carpeta, buscaremos el apartado sshd y copiaremos todo esto:
[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3              # A los 3 fallos, fuera.
bantime  = 24h            # Baneo de un día entero.
findtime = 10m            # Ventana de tiempo para los reintentos.
````


### 🔑 S6.2: Criptografía Asimétrica (Llaves RSA)
#### 🔑 S6.2.1: Creación de la key con el uso de CMD
Las contraseñas son vulnerables a fuerza bruta; las llaves RSA son virtualmente imposibles de romper.

**Pasos para crear la llave y meterla en el servidor (Con el uso de CMD):**
```
1. Generar llaves en el PC desde el que nos vayamos a conectar:
   ssh-keygen -t rsa -b 4096
	Enter file in which to save the key (C:\Users\yo/.ssh/id_rsa): (Enter)
	Enter passphrase (empty for no passphrase): (Contraseña que querais asignarle a la key)
    Enter same passphrase again: (La misma de antes)
    
2. Para copiar tu llave pública al Ubuntu de forma automática, usa este comando desde tu PC:
   ssh-copy-id iker@ssh.tudominio.com
	   Si usas Windows y no tienes ese comando, hazlo manualmente:

		Accede a la carpeta creada en tu usuario llamada .ssh.
		Accede al archivo creado nuevo que acaba en .pub editandolo con bloc de notas.
    
		Cópialo todo
	    
	    Accede al servidor por ssh y pega la llave dentro del documento authorized_keys:
			nano ~/.ssh/authorized_keys
    
3. Hardening del SSH (Desactivar contraseñas)
   Editar "/etc/ssh/sshd_config" y dejarlos como en las siguientes lineas
    - PasswordAuthentication no   
    - PubkeyAuthentication yes
```


En caso de que estemos trabajando con Bitvise, para agregar la key estos son los pasos:

#### 🔑 S6.2.2: Interoperabilidad y Gestión de Clientes (Bitvise/PuTTY)

**Conversión de Formatos (PPK)** Los clientes como **Bitvise** y **PuTTY** requieren a veces el formato propietario `.ppk` en lugar del formato OpenSSH estándar de Windows.
 
 **Procedimiento de Conversión:**
 
 1. **PuTTYgen:** Abrir la herramienta y pulsar en `Load` para cargar la llave privada (`id_rsa`), marcamos todos los archivos y elegimos al nuestra.
     
 2. **Exportación:** Pulsar en `Save private key` para generar el archivo `.ppk` compatible.
     
 3. **Importación en Bitvise:** > - Acceder a `Client key manager`.
     
     - Importar el archivo `.ppk`.
		- En el apartado Client key manager le damos a importar, metemos nuestra contraseña y elegimos
		 
		- En la pestaña `Login`, cambiar el método de autenticación a `publickey` y seleccionar el perfil importado.

#### 🔑 S6.2.3: Creación de la key con la herramienta Bitvise SSH
1. **Client key manager:** Accedemos a este apartado dentro de login
     
 2. **Generate New:** Al darle a generate new se nos abrira un menu, como Algorithm dejaremos RSA y en Size pondremos 4096, para la passphrase pondremos nuestra contraseña personal
     
 3. **Exportacion de Bitvise:** Aplicaremos el formato OpenSSH y exportaremos a nuestra carpeta .ssh
     
     1. Una vez tengamos la key publica en formato .pub, accede a la carpeta creada en tu usuario llamada .ssh.
	2. Accede al archivo creado nuevo que acaba en .pub editandolo con bloc de notas.
	3. Cópialo todo
	4. Accede al servidor por ssh y pega la llave dentro del documento authorized_keys:
			nano ~/.ssh/authorized_keys
	5. Seguidamente editaremos "/etc/ssh/sshd_config" y dejarlos como en las siguientes lineas
		```
		PasswordAuthentication no
		PubkeyAuthentication yes
		```
		 

		
**Ventajas del Acceso Asimétrico en Cliente**

- **Zero-Password:** Conexión instantánea sin intervención manual.
    
 - **Seguridad de Punto a Punto:** La llave privada nunca viaja por la red, solo se usa para firmar el desafío criptográfico localmente.     
- **Protección contra Keyloggers:** Al no teclear contraseñas, los virus que capturan pulsaciones de teclas quedan inutilizados.



### 🌐 S6.3: Cifrado Profesional (Let's Encrypt + Certbot)


#### 1. Preparación en el Panel de Cloudflare (API)

Antes de tocar la terminal, necesitamos generar el permiso:

1. Ve a **Profile**(Arriba a la derecha) -> **API Tokens** -> **Create Token**.
    
2. Usa la plantilla **"Edit zone DNS"**.
    
3. En **Zone Resources**, selecciona tu dominio: `tudominio.com`.

4. En **Permissions** añadir zona con DNS pero con la configuracion en Read, el resultado deberia de ser algo asi:
	 <div style="float: right; pointer-events: none;"> <img src="configuracionapi.png" style="width: 750px;"> </div>
    
5. Dale a continuar y copia el Token generado (es el código largo que usaremos ahora).
    


#### 2. Instalación de Herramientas en Ubuntu

Preparamos el sistema con el plugin necesario para que Ubuntu sepa hablar con Cloudflare:

Bash

```
sudo apt update
sudo apt install certbot python3-certbot-dns-cloudflare -y
```

#### 3. Configuración del Secreto (API Token)

Creamos el archivo donde el servidor guardará la "llave" de acceso a la nube:

Bash

```
# Creamos el archivo
sudo nano /root/.cloudflare_api_token

# --- DENTRO DEL ARCHIVO PEGAMOS TU TOKEN ---
dns_cloudflare_api_token = <TU_TOKEN_DE_CLOUDFLARE_AQUÍ>

# IMPORTANTE: Solo Root puede leer este archivo por seguridad
sudo chmod 600 /root/.cloudflare_api_token

# Creamos la carpeta por si no existe 
sudo mkdir -p /root/.secrets/certbot/ 

# Movemos tu archivo a la nueva ruta con el nombre correcto 
sudo mv /root/.cloudflare_api_token /root/.secrets/certbot/cloudflare.ini 

# Ajustamos los permisos por seguridad 
sudo chmod 600 /root/.secrets/certbot/cloudflare.ini
```

#### 4. Ejecución del Comando de Aceptación

Lanzamos el desafío DNS para obtener los certificados profesionales sin abrir puertos:

Bash

```
sudo certbot certonly --dns-cloudflare \ --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \ --dns-cloudflare-propagation-seconds 60 \ -d tudominio.com -d ssh.tudominio.com
```

#### 5. Verificación de Inmortalidad

Comprobamos que la renovación automática está activa para no tener que volver a hacerlo nunca:

Bash

```
sudo certbot renew --dry-run
```
 
<hr style="border: none; height: 5px; background: linear-gradient(to left, rgb(0, 0, 0), rgb(255, 215, 0), transparent); margin-bottom: 20px;">

Para ir a la siguiente Fase Click aqui →→ [[Fase 7 PHOENIX (Auditoría de Seguridad y Pentesting (SI))]]
<div style="float: right; margin-right: 100px; margin-left: 20px; margin-top: 30px; pointer-events: none;"> <img src="batman.gif" style="width: 500px;"> </div>

