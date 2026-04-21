<div style="border: none; height: 5px; background: linear-gradient(to right, rgb(0, 166, 255), rgb(255, 145, 0), transparent); margin-bottom: 20px;"></div>

Esta fase describe la apertura manual de comunicaciones desde una red externa (WAN) hacia nuestra red interna (LAN). El objetivo es permitir que los servicios configurados en la Fase 3 sean accesibles desde internet mediante el mapeo de puertos en el router hacia la IP estática del servidor.
<div style="float: right; margin-right: 100px; margin-left: 20px; margin-top: 30px; margin-bottom: 30px;pointer-events: none;"> <img src="portalend.png" style="width: 150px"> </div>

- 1. **Mapeo de Puertos en el Router (NAT/PAT):**        

    
    - Accedemos a la interfaz de administración del router (puerta de enlace: `192.168.0.1` o `192.168.1.1`).
        
    - Configuramos las reglas de reenvío (Port Forwarding) apuntando a la IP de nuestro servidor `192.168.0.200`:
        
        - **Puerto 80 (TCP):** Redirige el tráfico Web hacia Nginx.
            
        - **Puerto 22 (TCP):** Redirige el tráfico para administración remota SSH y SFTP.
            

En todos los casos, en el puerto WAN añadiremos numeros para hacerlo algo mas segura, aunque realmente no nos asegura nada pero para proteger el puerto un poco mas, ejemplo:
<div style="margin-right: 100px; margin-left: 20px; margin-top: 30px; margin-bottom: 30px;pointer-events: none;"> <img src="reedirecciondepuertos.png" style="width: 300px"> </div>

- 2. **Configuración del Cortafuegos Interno (UFW):**
        
	- Instalación de ufw con `sudo apt install ufw`
	
    - Para que Ubuntu acepte las peticiones externas redirigidas por el router, debemos abrir los puertos en el sistema:
        
        - `sudo ufw allow 80/tcp` (Web).
            
        - `sudo ufw allow 22/tcp` (SSH/SFTP).
            
            
    - Aplicamos y verificamos con: `sudo ufw enable` y `sudo ufw status`.
        
- 3. **Configuración de Escucha en Servicios:**
        
    
    - Nos aseguramos de que los servicios no solo escuchen en `127.0.0.1`, sino en todas las interfaces (`0.0.0.0`) para recibir tráfico WAN.
        
    - Verificamos con el comando: `ss -tlpn`.
        <div style="float: right; margin-right: 100px; margin-left: 20px; margin-top: 30px; pointer-events: none;"> <img src="portalnether.png" style="width: 150px;"> </div>
- 4. **Ajustes de Seguridad para Exposición Pública:**
        
    
    - Al exponer el puerto 22 a internet, reforzamos el servicio SSH en `sudo nano /etc/ssh/sshd_config`:
	    ```
        MaxAuthTries 3  #Limita los intentos de acceso para prevenir fuerza bruta.
            
        LoginGraceTime 60  #Define el tiempo máximo para completar el login.
        ```
    - **Hardening FTP:** En el fichero `sudo nano /etc/proftpd/proftpd.conf`, configuramos la directiva `MasqueradeAddress` con nuestra IP pública actual para que el modo pasivo funcione correctamente desde fuera de la red local. Para saber nuestra IP pública actual accederemos a https://www.cual-es-mi-ip.net y la copiaremos.
        
- 5. **Validación de Conectividad WAN:**
        
    
    - **Prueba Web:** Accedemos a nuestra IP pública desde un dispositivo fuera de la red (datos móviles) para confirmar que Nginx responde. Necesitaremos poner nuestra IP y nuestro puerto WAN colocado anteriormente (ej: Si pusiste para el puerto http: LAN 80 WAN 8000, la pagina web será: ippublica:8000)
        
    - **Prueba SFTP/FTP:** Utilizamos Bitvise configurando la dirección del servidor con nuestra IP pública, accediendo a traves del puerto 22 y ya teniendo acceso al protocolo FTPS y SSH.
        

<div style="clear: both;"></div>

<div style="border: none; height: 5px; background: linear-gradient(to left, rgb(0, 166, 255), rgb(255, 145, 0), transparent); margin-bottom: 20px;"></div>

Para ir a la siguiente Fase Click aqui →→ [[Fase 5 DITTO (Identidad Digital y Automatización DDNS (SOR + SR))]]