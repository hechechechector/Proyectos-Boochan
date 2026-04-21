## 1. Preparación del servidor (Windows Server)

Para montar el servidor VPN, comenzamos configurando el programa base en la máquina de Windows Server.

- Descarga el instalador de **SoftEther VPN Server** para Windows desde la página oficial y ejecútalo.​

![[Pasted image 20260224192931.png]]

- En el asistente, elige instalar **SoftEther VPN Server**.​
    
- Deja las opciones y rutas de instalación por defecto hasta finalizar el asistente. Se iniciará automáticamente la herramienta de gestión **SoftEther VPN Server Manager**.​
    
- Al abrir el _Server Manager_, haz doble clic en **localhost (This server)** para conectarte a tu propia máquina.​
    
- Te pedirá inmediatamente que configures una **Contraseña de administrador** para el servidor. Introduce una clave segura y guárdala.​
    

## 2. Configuración del Virtual Hub y Usuarios

El _Virtual Hub_ es el espacio virtual donde se conectarán los usuarios. A través del asistente "Easy Setup" se realiza la configuración inicial.

- Marca la opción **Remote Access VPN Server** y avanza en el asistente.​
    
- Ponle un nombre a tu Virtual Hub (por ejemplo, `VPN WinServer`) y acepta.​
    
- En la pantalla de DNS Dinámico (DDNS), simplemente dale a **Exit**. Ya que usaremos VPN Azure, este paso no es relevante.
    
- Desactiva cualquier otra configuración de L2TP/IPsec que salte en el asistente (déjalo desmarcado y dale a OK).​
    
- En la pantalla de tareas rápidas, haz clic en **Create Users** (Crear usuarios).[](https://support.antibex.com/knowledgebase/softether-vpn-server-installation-and-setup/)​
    
- Introduce un **User Name** (ej. `hector`) y en el tipo de autenticación por contraseña (`Password Authentication`), escribe y confirma la contraseña de este usuario.[](https://support.antibex.com/knowledgebase/softether-vpn-server-installation-and-setup/)​
    
- Guarda el usuario y cierra el asistente.[](https://support.antibex.com/knowledgebase/softether-vpn-server-installation-and-setup/)​
    

## 3. Activación de SecureNAT y VPN Azure

Para que la conexión funcione sin abrir puertos en Azure ni tener conflictos de NAT, se activan dos herramientas integradas de SoftEther.

- En la pantalla principal del _Server Manager_, haz doble clic en tu Virtual Hub para abrir su panel de administración.[](https://www.softether.org/4-docs/1-manual/3/3.7)​
    
- A la derecha, haz clic en el botón **Virtual NAT and Virtual DHCP Server (SecureNAT)**.[](https://www.softether.org/4-docs/1-manual/3/3.7)​
    
- Haz clic en **Enable SecureNAT**. Esto encenderá el servidor DHCP interno para que SoftEther asigne automáticamente direcciones IP a los clientes que se conecten.[](https://www.softether.org/4-docs/1-manual/3/3.7)​
    
- Sal de ese menú y vuelve a la pantalla principal del _Server Manager_.
    
- Haz clic en el botón grande inferior derecho llamado **VPN Azure Settings**.[](https://www.softether.org/4-docs/2-howto/6.vpn_server_behind_nat_or_firewall/2.vpn_azure)​
    
- Marca la casilla **Enable VPN Azure**. Esto generará un nombre de host especial terminado en `.vpnazure.net` (ej. `mivpn.vpnazure.net`). Cópialo, ya que será la dirección que usarás para conectarte.[](https://www.softether.org/4-docs/2-howto/6.vpn_server_behind_nat_or_firewall/2.vpn_azure)​
    

## 4. Configuración del cliente (Windows 11)

En lugar de usar el cliente integrado de Windows, se emplea el software oficial de SoftEther para aprovechar el túnel SSL por el puerto 443, evitando cualquier bloqueo.

- Descarga e instala el **SoftEther VPN Client** en tu Windows 11 desde la web oficial.​
    
- Abre el **SoftEther VPN Client Manager** y haz doble clic en **Add VPN Connection** (Agregar conexión VPN).​
    
- El programa te pedirá crear un **Virtual Network Adapter** (Adaptador de red virtual). Acéptalo y espera unos segundos a que se instale en Windows.​
    
- En la ventana de propiedades de la nueva conexión, rellena los siguientes campos:​
    
    - **Host Name:** Pega la dirección completa que obtuviste de VPN Azure (la que acaba en `.vpnazure.net`).[](https://www.softether.org/4-docs/2-howto/6.vpn_server_behind_nat_or_firewall/2.vpn_azure)​
        
    - **Port Number:** Déjalo en `443`.[](https://www.softether.org/4-docs/2-howto/6.vpn_server_behind_nat_or_firewall/2.vpn_azure)​
        
    - **Virtual Hub Name:** Escribe el nombre del hub que creaste en el servidor (ej. `VPN WinServer`).​
        
    - **User Authentication Setting:** Selecciona autenticación por contraseña e introduce tu usuario (`hector`) y tu clave.​
        
- Haz clic en **OK** para guardar la conexión.​
    

## 5. Conexión y verificación

Para iniciar y verificar que todo funciona correctamente:

- Haz doble clic sobre la conexión VPN recién creada en el _SoftEther VPN Client Manager_.
    
- El estado pasará a "Connected" (Conectado) e inmediatamente el SecureNAT te proporcionará una IP (desaparecerá el mensaje "Requesting IP").
    
- Abre una ventana de comandos (`cmd`) en tu Windows 11 y escribe `ipconfig`. Verás que el "VPN Client Adapter" tiene una dirección IP asignada (por ejemplo, del rango `192.168.30.X`).[](https://www.softether.org/4-docs/1-manual/3/3.7)​
    
- Entra en tu navegador a cualquier web para ver tu IP pública (como whatismyip.com). Aparecerá la dirección IP del servidor de Azure, confirmando que tu tráfico está saliendo exitosamente por la VPN.
    

Al utilizar VPN Azure junto con el cliente oficial de SoftEther, el servidor crea una conexión de salida por TCP/443 hacia la nube de SoftEther, lo que permite saltarse las reglas de entrada del Network Security Group (NSG) de Azure, el CGNAT de las redes móviles y los conflictos internos de Windows Server con el protocolo IPsec.[](https://www.softether.org/4-docs/2-howto/6.vpn_server_behind_nat_or_firewall/2.vpn_azure)​