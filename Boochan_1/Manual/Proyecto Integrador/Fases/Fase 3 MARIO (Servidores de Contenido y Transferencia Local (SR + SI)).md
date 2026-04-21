<hr style="border: none; height: 5px; background: linear-gradient(to right, rgb(255, 0, 0), rgb(0, 0, 255), transparent); margin-bottom: 20px;">

- 1. **Servidor Web Nginx:**
    
    - Instalamos Nginx como motor de alto rendimiento: `sudo apt install nginx -y`.
        
    - Verificamos que el servicio esté activo: `sudo systemctl status nginx`.
        
    - **Validación:** Al introducir `http://192.168.0.200` en el navegador del anfitrión,         <div style="float: right; margin-right: 180px; margin-left: 20px; margin-top: 30px; pointer-events: none;"> <img src="mario2.png" style="width: 50px;"> </div>visualizamos la página de bienvenida de Nginx.

- 2. **Servidor de Archivos ProFTPD con Enjaulado:** 
    
    - Instalamos el servicio: `sudo apt install proftpd -y`.
        
    - Para evitar que los usuarios escalen directorios, aplicamos el **Hardening** en `sudo nano /etc/proftpd/proftpd.conf`:
        
        - Descomentamos (Le quitamos el #) `DefaultRoot ~` para que cada usuario quede atrapado en su `/home`.
            
- **3. Implementación de Cifrado SSL/TLS (FTPS)**

	En esta etapa, transformamos el servidor en un entorno seguro mediante el uso de certificados criptográficos y la configuración de módulos específicos de ProFTPD.

	 **A. Instalación de Paquetes y Módulos**

	Primero, instalamos las dependencias necesarias para el manejo de criptografía y certificados:

	- `sudo apt install proftpd-basic -y`
	    
	- `sudo apt install proftpd-mod-crypto -y`
	    

	 **B. Generación del Certificado de Seguridad**

	Creamos un certificado RSA de 2048 bits auto-firmado que será válido por un año:
	
	Bash
	
	```
	sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/proftpd.key -out /etc/ssl/certs/proftpd.crt
	```
	
	- Al meter el comando anterior, nos saldrán una lista de cosas que poner, en general no tiene importancia pero hay una que tiene mucha importancia y es el Common Name:
		- **Country Name:** ES (para España).
		
		- **Locality Name:** Tu ciudad.
		
		- **Organization Name:** El nombre de tu proyecto (ej: "MiHomeLab").
		
		- **Common Name (CN):** 
			- Tienes que poner tu **IP** (ej: `192.168.0.100`).
	
		- **Email Address:** Puedes dejarlo en blanco
	
	#### **C. Configuración del Sistema de Módulos**
	
	Para que ProFTPD pueda usar TLS, debemos activar el módulo correspondiente:
	
	1. **Archivo:** `sudo nano /etc/proftpd/modules.conf`
	    
	    - Quitar el `#` en la línea: `LoadModule mod_tls.c`
	        
	2. **Archivo:** `sudo nano /etc/proftpd/proftpd.conf`
	    
	    - Quitar el `#` en la línea: `Include /etc/proftpd/tls.conf`
	        
	    - **Hardening:** Añadir un `#` (comentar) en la línea `TLSRenderCondition standard` si apareciera en este archivo.
	        
	
	#### **D. Configuración Detallada de Seguridad (`tls.conf`)**
	
	Accedemos a `sudo nano /etc/proftpd/tls.conf` y aplicamos los siguientes cambios quitando el `#` o añadiéndolo según corresponda:
	
	|**Parámetro**|**Acción**|**Función**|
	|---|---|---|
	|`TLSEngine on`|Quitar `#`|Activa el motor TLS.|
	|`TLSLog /var/log/proftpd/tls.log`|Quitar `#`|Registra intentos de conexión segura.|
	|`TLSProtocol SSLv23`|Quitar `#`|Define la compatibilidad de protocolos.|
	|`TLSRSACertificateFile`|Quitar `#`|Ruta al archivo `.crt` generado.|
	|`TLSRSACertificateKeyFile`|Quitar `#`|Ruta a la clave privada `.key`.|
	|`TLSRSNRequired off`|Añadir `#`|Desactiva requerimientos de nombre de sitio.|
	|`TLSRenderCondition standard`|Añadir `#`|Evita conflictos de renderizado SSL.|
	
	#### **E. Permisos, Validación y Reinicio**
	
	Aseguramos la integridad de los archivos de configuración para que solo el sistema pueda leerlos correctamente:
	
	1. **Ajuste de permisos:**
	    
	    - `sudo chmod 644 /etc/proftpd/modules.conf`
	        
	    - `sudo chmod 644 /etc/proftpd/tls.conf`
	        
	    - `sudo chmod 644 /etc/proftpd/proftpd.conf`
	        
	2. **Comprobación de errores:** `sudo proftpd -t`
	    
	3. **Reinicio del servicio:** `sudo systemctl restart proftpd`

- 4. **Software para uso de FTP y comprobaciones:**
    - **Validación Final:** Descargando el programa de FileZilla Client nos conectamos desde  usando **FTP explícito sobre TLS**. El sistema nos pide aceptar nuestro certificado y el icono del candado confirma que la comunicación es privada, la configuración de FileZilla será la siguiente.<div style="margin-left: 0px; margin-bottom: 50px; pointer-events: none;"> <img src="mario3.webp" style="width: 100px; filter: drop-shadow(-5px 5px 10px rgba(255,0,0,0.3));"> </div>
      
      <div style="pointer-events: none;"> <img src="filezilla.png" style="width: 500px;"> </div>

<hr style="border: none; height: 5px; background: linear-gradient(to left, rgb(255, 0, 0), rgb(0, 0, 255), transparent); margin-bottom: 20px;">

FASE 4 Redireccion de puertos (A) →→ [[Fase 4 PORTAL Versión A (Publicación WAN y Configuración Perimetral (SR))]]
FASE 4 Cloudflare tunnel (B) →→ [[Fase 4 PORTAL Versión B (Infraestructura Cloudlare Tunnel (Zero Trust) (SR))]]

