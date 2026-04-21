<hr style="border: none; height: 5px; background: linear-gradient(to right, rgb(168, 0, 255), rgb(0, 166, 255), transparent); margin-bottom: 20px;">

**🎯 S5.1: Justificación del Cambio (Evolución de 4-A a 5)**

En la **Fase 4-A**, se configuró la conectividad mediante el modelo tradicional de **Port Forwarding**. Aunque funcional, este modelo expone la IP pública del servidor y depende de protocolos obsoletos como FTPS, que son complejos de gestionar tras firewalls.

**La Fase 5 representa la profesionalización de la infraestructura:**

- **Seguridad Zero Trust:** Se sustituye la apertura de puertos por un túnel saliente cifrado.
    
- **Simplificación de Protocolos:** Se descarta FTPS en favor de **SFTP (SSH File Transfer Protocol)**, centralizando la gestión de archivos y comandos en el puerto 22.
    
- **Enmascaramiento Total:** La IP real del servidor queda oculta tras la red de Cloudflare.
    



**S5.2: Implementación Profesional del Túnel**

### 1. Instalación y Autenticación

Instalamos el agente `cloudflared` y creación de tunel :

 <div style="float: right; margin-right: 100px; margin-left: 520px; pointer-events: none;"> <img src="ditto2.gif" style="width: 90px;"> </div>

```
# Instalación del agente
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb
```

1. Accedemos a Zero Trust <div style="pointer-events: none;"> <img src="paso2.png" style="width: 300px;"> </div>

2. Entramos en Overview <div style="pointer-events: none;"> <img src="paso2t.png" style="width: 300px;"> </div>

3. View all Tunnels <div style="pointer-events: none;"> <img src="paso3t.png" style="width: 500px;"> </div>

4.  Create a tunnel <div style="pointer-events: none;"> <img src="paso4t.png" style="width: 500px;"> </div>

5. Cloudflared <div style="pointer-events: none;"> <img src="paso5t.png" style="width: 500px;"> </div>

6. Le ponemos el nombre que queramos y elegimos Debian como entorno, a continuacion copiamos los bloques de comandos con un click y los pegamos tal cual en la maquina virtual <div style="pointer-events: none;"> <img src="paso6t.png" style="width: 500px;"> </div>
7. La primera ruta que le pondremos sera la de ssh <div style="pointer-events: none;"> <img src="paso7t.png" style="width: 800px;"> </div>


### 2. Configuracion Hostnames
Para configurar los hostnames necesitaremos hacerlo dentro del propio tunel, así que le daremos a los 3 puntos encima de nuestro tunel creado en el anterior paso y le damos a editar, una vez dentro nos iremos al apartado Published application routes -> Add a published application route y añadiremos el de la pagina web (http).<div style="pointer-events: none;"> <img src="http.png" style="width: 800px;"> </div>
 
## 🔒 S5.3: El "Apagón" de Puertos y Hardening Perimetral

Una vez levantado el túnel, el servidor es accesible de forma saliente. Esto permite blindar el router y el sistema:

1. **En el Router:** Eliminar todas las reglas de _Port Forwarding_ y _DMZ_. Específicamente, cerrar los puertos **21 y 80**.
    
2. **En Ubuntu (UFW):**
    

    
    ```
    # Bloquear accesos directos externos
    sudo ufw deny 21/tcp
    sudo ufw deny 80/tcp
    # Solo permitir SSH para gestión local o vía túnel
    sudo ufw allow from 192.168.0.0/24 to any port 22
    ```
    

---

## 🤖 S5.4: Configuración de Accesos y Capa Zero Trust

 <div style="float: right; margin-right: 620px; pointer-events: none;"> <img src="ditto.webp" style="width: 90px;"> </div>


### Cloudflare Access (Identity Provider)

Para el subdominio `ssh.tudominio.com`, tenemos que configurar una  **Access Application** de tipo _Self-hosted_:

**Para crearla será:** 
1. Back to domains  <div style="pointer-events: none;"> <img src="paso1.png" style="width: 800px;"> </div>

2. Zero Trust<div style="pointer-events: none;"> <img src="paso2.png" style="width: 300px;"> </div>

3. Access Controls -> Applications <div style="pointer-events: none;"> <img src="paso3.png" style="width: 300px;"> </div>

4. Add an application <div style="pointer-events: none;"> <img src="paso4.png" style="width: 500px;"> </div>

5. Self-Hosted <div style="pointer-events: none;"> <img src="paso5.png" style="width: 500px;"> </div>

6. Configuración que poner <div style="pointer-events: none;"> <img src="paso6.png" style="width: 900px;"> </div>

7. Create new policy <div style="pointer-events: none;"> <img src="paso7.png" style="width: 500px;"> </div>

8. Configuración policy  <div style="pointer-events: none;"> <img src="paso8.png" style="width: 800px;"> </div>

9. Agregar policies <div style="pointer-events: none;"> <img src="paso9.png" style="width: 400px;"> </div>

10. Añadir policies<div style="pointer-events: none;"> <img src="paso10.png" style="width: 500px;"> </div>

11. Configurar Browser rendering settings<div style="pointer-events: none;"> <img src="paso11.png" style="width: 800px;"> </div> 
12. Next y save a todo hasta salir

La aplicación se verá algo tal que así: <div style="pointer-events: none;"> <img src="paso12.png" style="width: 900px;"> </div>




 <div style="float: right; margin-left: 520px; pointer-events: none;"> <img src="zorua.png" style="width: 200px;"> </div>

## 🔍 S5.5: Conexión web y SSH
Una vez configurada la infraestructura, el acceso se divide en tres niveles de seguridad según el servicio:

#### **1. Acceso Web Público (HTTP → HTTPS)**

Cualquier usuario puede acceder a la web de forma segura. Cloudflare gestiona el certificado SSL automáticamente.

- **URL:** `https://boochan.space`
    
- **Resultado:** Conexión cifrada con candado de seguridad, ocultando la IP real del servidor Apache.
    

#### **2. Acceso Administrativo vía Navegador (Clientless SSH)**

Ideal para gestiones rápidas desde cualquier dispositivo sin instalar software.

- Proceso: 1. Navegar a https://ssh.boochan.space.
    
    2. Aparecerá el portal de Cloudflare Access.
    
    3. Introducir el email autorizado y el código OTP recibido.
    
- **Resultado:** Se abre una terminal SSH funcional directamente en la pestaña del navegador.
    

#### **3. Acceso SSH Profesional (Túnel TCP)**

Para administración avanzada, uso de **VS Code**, **FileZilla (SFTP)** o terminal local. Este método es el más seguro ya que requiere un "apretón de manos" entre el PC cliente y el servidor.

**Paso A: Abrir el puente en el PC local (Windows/Mac/Linux)**

PowerShell

```
# Este comando crea un túnel secreto entre tu PC y Cloudflare
cloudflared access tcp --hostname ssh.boochan.space --listener localhost:2222
```

Paso B: Conectar por SSH tradicional

En una segunda terminal (o en tu cliente favorito), conecta al puerto que acabas de abrir:

Bash

```
# Conexión al 'puente' que hemos creado en el puerto 2222
ssh iker@localhost -p 2222
```

---

#### **Comparativa de Seguridad por Servicio**

|**Método**|**URL/Host**|**Autenticación**|**Cifrado**|
|---|---|---|---|
|**Web**|`boochan.space`|Ninguna (Público)|TLS 1.3|
|**Web SSH**|`ssh.boochan.space`|**Email OTP**|Zero Trust|
|**Terminal SSH**|`localhost:2222`|**Email OTP + SSH Key/User**|Túnel Cifrado Doble|



# EN CASO DE ERRORES

Durante el despliegue se identificaron y resolvieron los siguientes puntos críticos:

- **Conflicto de SSH:** Se desactivó `ssh.socket` para permitir que `ssh.service` gestionara exclusivamente el puerto 22 sin errores de "Address already in use".

**1. Resolución del Conflicto de SSH (Address already in use)**

El error ocurría porque en versiones modernas de Ubuntu, el sistema utiliza "sockets" para escuchar el puerto 22, lo que impedía que el servicio SSH tradicional tomara el control total.

- **Síntoma:** Al intentar reiniciar SSH, el log mostraba: `Error: Bind to port 22 failed - Address already in use`.
    
- **Solución Técnica:** Ejecutamos la desactivación del socket para liberar el puerto definitivamente:
    
    Bash
    
    ```
    # Desactivar y enmascarar el socket para que no vuelva a arrancar
    sudo systemctl stop ssh.socket
    sudo systemctl disable ssh.socket
    sudo systemctl mask ssh.socket
    
    # Reiniciar el servicio SSH estándar
    sudo systemctl restart ssh
    ```
    
- **Validación:** Al ejecutar `sudo ss -tulpn | grep :22`, ahora solo aparece el proceso `sshd`.
    

<hr style="border: none; height: 5px; background: linear-gradient(to left, rgb(168, 0, 255), rgb(0, 166, 255), transparent); margin-bottom: 20px;">

Para ir a la siguiente Fase Click aqui →→ [[Fase 6 BATMAN (Hardening y Blindaje de Seguridad (SI))]]