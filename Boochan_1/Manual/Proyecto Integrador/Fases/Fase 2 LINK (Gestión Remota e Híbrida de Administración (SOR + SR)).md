<hr style="border: none; height: 5px; background: linear-gradient(to right, rgb(34, 139, 34), rgb(218, 165, 32), transparent); margin-bottom: 20px;">
<div style="float: right; margin-left: 20px; margin-bottom: 10px; pointer-events: none;"> <img src="link.png" style="width: 180px; filter: brightness(1.1) drop-shadow(10px 10px 10px rgba(0,0,0,0.3));"> </div>

1. **Instalar OpenSSH Server:**
	-  Para actualizar la información del sistema sobre los paquetes disponibles `sudo apt update`
		- Para instalar el paquete necesario para utilizar ssh `sudo apt install openssh-server -y`.
		- Tenemos que entrar en la configuración del ssh para ello entraremos en el fichero con el comando `sudo nano /etc/ssh/sshd_config`. Una vez dentro buscamos una línea en la que pone `Port 22` y la descontentamos quitándole la #. Nos dirigimos al final del fichero y escribimos los siguientes parámetros:
			```
			ListenAddress  (Tu_ip) #en mi caso en la fase 1 le he asignado a la maquina la 192.168.0.200
			
			PermitRootLogin no
			
			AllowUsers (Tu_Usuario) #Los usuarios que hay en la maquina y queremos que accedan tenemos que poner su nombre importante: si no se pone el nombre tal cual lo tiene el servidor no va a funcionar.
			
			PubkeyAuthentication no
			
			PasswordAuthentication yes #La contraseña para acceder es la misma que la de tu servidor
			```
		- Guardamos el fichero txt editado con nano con `Crtl + O`, `Enter`, luego `Crtl +X`.
		- Para comprobar la sintaxis ponemos el comando `sudo sshd -t` (Si funciona bien no devuelve nada). ==Si aparece el mensaje de Missing privilege separation directory: /run/sshd = Significa que le faltan permisos asi que realizamos los siguientes comandos:==
			- Crear el directorio con permisos correctos:
				`sudo mkdir -p /run/sshd`
				`sudo chmod 755 /run/sshd`
			<div style="float: right; margin-left: 20px; margin-bottom: 10px; pointer-events: none;"> <img src="link2.webp" style="width: 220px; filter: brightness(1.1) drop-shadow(10px 10px 10px rgba(0,0,0,0.3));"> </div>
			- Volver a comprobar la configuración y reiniciar:
				`sudo sshd -t`
			- Nos dara un error asi que tendremos que generar las keys, en caso de que no salga ningun error ignorar este comando:
				->`sudo ssh-keygen- A`
				`sudo sshd -t`
				`sudo systemctl restart ssh`
		- Para comprobar que todo funciona nos dirigimos al sistema anfitrión abrimos el cmd o terminal y escribimos `ssh usuario@IP_Servidor`.
		
1. **Instalar xfce4:**
	- Para instalar un entorno grafico que nos facilite el manejo del servidor actualizamos la información del sistema sobre los paquetes `sudo apt update` y después `sudo apt upgrade -y`.
	- La instalación del paquete (básica y ligera) `sudo apt install xfce4 -y` para complementar la instalación le pondremos "sabor Xubuntu" `sudo apt install xubuntu-desktop -y`.
	- Para el login gráfico `sudo apt install lightdm -y`.
	- Para arrancar el XFCE pondremos el comando `sudo reboot`.

2. **Instalar el software AnyDesk, configurar contraseña de acceso no presencial en AnyDesk en el sistema anfitrión:**
	-  Entramos a nuestro buscador y escribimos AnyDesk descargamos la versión para nuestro sistema operativo.
	- Una vez descargada entramos en AnyDesk y seleccionamos las tres rayas de arriba a la derecha > Configuración.
	- Estando dentro de la configuración bajaremos hasta el apartado de Seguridad > Permisos.
	- Dentro de Permisos veremos la opción de Perfiles de permiso, seleccionamos Añadir perfil y lo seleccionaremos, a continuación activaremos todas las opciones disponibles para habilitar, establecemos una contraseña y ya tendremos acceso a nuestro ordenador desde fuera de nuestra red en caso de algún fallo.
<div style="clear: both;"></div>

<hr style="border: none; height: 5px; background: linear-gradient(to left, rgb(34, 139, 34), rgb(218, 165, 32), transparent); margin-bottom: 20px;">

Para ir a la siguiente Fase Click aqui →→ [[Fase 3 MARIO (Servidores de Contenido y Transferencia Local (SR + SI))]]