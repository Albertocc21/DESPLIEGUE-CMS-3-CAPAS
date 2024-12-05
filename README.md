# DESPLIEGUE-CMS-3-CAPAS

# Requisitos
- Cuenta en AWS.
- Conexión SSH.

# Descripción de la práctica
En esta práctica desplegaremos una infraestructura en 3 capas la cuál tendrá un servidor Apache que actuará como balanceador de carga alojado en la primera capa y estará en una red pública, en la segunda capa tendremos dos servidores backend y un servidor NFS que pertenecerán a una red privada y por último tendremos un servidor de base de datos (MariaDB o MySQL) que estará alojado en la tercera capa y que también estará en una red privada.

# Índice

1. **Diseño de la Infraestructura**  
   - *[Planificación de la Arquitectura](#11-planificación-de-la-arquitectura)*  
   - *[Configuración](#12-configuración-de-acceso-y-seguridad)*  
   - *[Scripts](#13-scripts-de-aprovisionamiento)*  

2. **Instalación de servicios necesarios**  
   - *[Capa 1](#21-capa-1-balanceador-de-carga)*  
   - *[Capa 2](#22-capa-2-servidores-backend-y-nfs)*  
   - *[Capa 3](#23-capa-3-servidor-de-bbdd)*  

3. **Despliegue de la infraestructura** 
   - *[Creación de la VPC](#31-creación-de-la-vpc)*  
   - *[Lanzar las instancias](#32-lanzar-las-instancias)*
   - *[Configurar grupos de seguridad](#33-configurar-grupos-de-seguridad)*
   - *[Configurar las máquinas](#34-configurar-las-máquinas)*
   - *[Sitio seguro](#35-sitio-seguro)*

## 1. Diseño de la infraestructura

### 1.1 Planificación de la Arquitectura
  - **Capa 1 (Capa Pública):** Balanceador de carga accesible desde el exterior.
  - **Capa 2 (Capa Privada - Backend):** Servidores que procesan las solicitudes, con un servidor NFS.
  - **Capa 3 (Capa Privada - Base de Datos):** Servidor de base de datos MySQL o MariaDB.
  
- **Asignación de nombres (hostnames):**
  - Cada máquina debe tener un nombre como por ejemplo (BalanceadorAlbertoCalvo).

### 1.2 Configuración de acceso y seguridad
- **Restricciones:**
  - Solo se permitirá el acceso desde el exterior a la Capa 1.
  - No debe haber conectividad entre la Capa 1 y la Capa 3.
  
- **Grupos de seguridad:**
  - Configurar grupos de seguridad en las máquinas para garantizar que solo las conexiones necesarias estén permitidas entre las capas.

### 1.3 Scripts de aprovisionamiento
- Crear un script que automatice el la instalación de servicios para cada máquina.

#### BALANCEADOR
```bash
apt update -y
apt install -y apache2
a2enmod proxy
a2enmod proxy_http
a2enmod proxy_balancer
a2enmod lbmethod_byrequests
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/load-balancer.conf
sed -i '/DocumentRoot \/var\/www\/html/s/^/#/' /etc/apache2/sites-available/load-balancer.conf
sed -i '/:warn/ a \<Proxy balancer://mycluster>\n    # Server 1\n    BalancerMember http://192.168.50.134\n    # Server 2\n    BalancerMember http://192.168.50.133\n</Proxy>\nProxyPass / balancer://mycluster/' /etc/apache2/sites-available/load-balancer.conf
a2ensite load-balancer.conf
a2dissite 000-default.conf
systemctl restart apache2
systemctl reload apache2
```
apt update -y: actualiza los paquetes automáticamente.

apt install -y apache2: instala Apache de manera automática.

a2enmod proxy: habilita el módulo proxy que permite redirigir solicitudes HTTP.

a2enmod proxy_http: habilita el módulo proxy que permite redirigir solicitudes HTTP a servidores backend.

a2enmod proxy_balancer: habilita el módulo que permite que Apache funcione con balanceador.

a2enmod lbmethod_byrequests: habilita el módulo que distribuye de manera equitativa entre los servidores backend.

cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/load-balancer.conf: copia el archivo por defecto de Apache.

sed -i '/DocumentRoot \/var\/www\/html/s/^/#/' /etc/apache2/sites-available/load-balancer.conf: desactiva la configuración de la raíz de documentos estática del servidor web.

sed -i '/:warn/ a \<Proxy balancer://mycluster>\n    # Server 1\n    BalancerMember http://192.168.50.134\n    # Server 2\n    BalancerMember http://192.168.50.133\n</Proxy>\nProxyPass / balancer://mycluster/' /etc/apache2/sites-available/load-balancer.conf: inserta las configuraciones necesarias en el archivo copiado, define un grupo donde están los servidores backend y las solicitudes se redirigen al grupo usando ProxyPass.

a2ensite load-balancer.conf: habilita el fichero que hemos copiado.

a2dissite 000-default.conf: deshabilita el fichero por defecto.

systemctl restart apache2: reinicia Apache.

systemctl reload apache2: recarga Apache sin reiniciar el servicio por completo.

#### NFS
```bash
apt update -y
apt install nfs-kernel-server -y
apt install unzip -y
apt install curl -y
apt install php php-mysql -y
apt install mysql-client -y
mkdir /var/nfs/shared -p
chown -R nobody:nogroup /var/nfs/shared
sed -i '$a /var/nfs/shared    192.168.50.134(rw,sync,no_subtree_check)' /etc/exports
sed -i '$a /var/nfs/shared    192.168.50.133(rw,sync,no_subtree_check)' /etc/exports
curl -O https://wordpress.org/latest.zip
unzip -o latest.zip -d /var/nfs/shared/
chmod 755 -R /var/nfs/shared/
chown -R www-data:www-data /var/nfs/shared/*
systemctl restart nfs-kernel-server
```
apt update -y: actualiza la lista de paquetes automáticamente.

apt install nfs-kernel-server -y: isntala el servidor NFS, confirmando cualquier pregunta.

apt install unzip -y: instala la herramienta unzip, que descomprimirá el archivo de Wordpress.

apt install curl -y: instala la herramienta curl usada para descargar archivos de internet.

apt install php php-mysql -y: instala PHP y la extensión MySQL para que PHP interactúe con MySQL.

apt install mysql-client -y: instala el cliente de MySQL que permite conectarse a servidores de bases de datos MySQL.

mkdir /var/nfs/shared -p: crea el directorio que será compartido a través de NFS.

chown -R nobody:nogroup /var/nfs/shared: cambia el propietario del directorio a nobody:nogroup.

sed -i '$a /var/nfs/shared    192.168.50.134(rw,sync,no_subtree_check)' /etc/exports: agrega una línea al archivo para que el directorio creado sea accesible con el servidor que tiene esa ip con permisos: 

rw: lectura y escritura.

sync: las operaciones de escritura se realizan de forma sincronizada.

no_subtree_check: mejora el rendimiento y evita problemas con verificaciones innecesarias.

sed -i '$a /var/nfs/shared    192.168.50.133(rw,sync,no_subtree_check)' /etc/exports: lo mismo que antes pero para el otro servidor.

curl -O https://wordpress.org/latest.zip: descarga la última versión de Wordpress en formato comprimido.

unzip -o latest.zip -d /var/nfs/shared/: descomprime el archivo descargado.

chmod 755 -R /var/nfs/shared/: cambia los permisos al directorio y su contenido:

Lectura,escritura,ejecución para el propietario.

Lectura y ejecución para el grupo y otros.

chown -R www-data:www-data /var/nfs/shared/*: cambia el propietario de los directorios y de los archivos que hay dentro del direcotio /var/nfs/shared.

systemctl restart nfs-kernel-server: reinicia el servicio NFS para aplicar los cambios.

#### WEBS
```bash
apt update -y
apt install apache2 -y 
apt install nfs-common -y
apt install php libapache2-mod-php php-mysql php-curl php-gd php-xml php-mbstring php-xmlrpc php-zip php-soap php -y
a2enmod rewrite
sudo sed -i 's|DocumentRoot .*|DocumentRoot /nfs/shared/wordpress|g' /etc/apache2/sites-available/000-default.conf
sed -i '/<\/VirtualHost>/i \
<Directory /nfs/shared/wordpress>\
    Options Indexes FollowSymLinks\
    AllowOverride All\
    Require all granted\
</Directory>' /etc/apache2/sites-available/000-default.conf
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/websv.conf
mkdir -p /nfs/shared
mount 192.168.50.135:/var/nfs/shared /nfs/shared
a2dissite 000-default.conf
a2ensite websv.conf
echo "192.168.50.135:/var/nfs/shared /nfs/shared nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0" | sudo tee -a /etc/fstab
mount -a
systemctl restart apache2
systemctl reload apache2
systemctl status apache2
```
apt update -y: actualiza la lista de paquetes.

apt install apache2 -y: instala Apache.

apt install nfs-common -y: instala el cliente NFS.

apt install php libapache2-mod-php php-mysql php-curl php-gd php-xml php-mbstring php-xmlrpc php-zip php-soap php -y: instala PHP y las extensiones necesarias para WordPress:

php-mysql: comunicación con la base de datos MySQL.

php-curl: permite realizar peticiones HTTP desde PHP.

php-gd: manipulación de imágenes.

php-xml y php-xmlrpc: gestión de datos en XML.

php-mbstring: manejo de cadenas de caracteres multibyte.

php-zip: trabajo con archivos comprimidos.

php-soap: protocolo SOAP.

libapache2-mod-php: integra PHP con Apache.

a2enmod rewrite: habilita el módulo para que WordPress maneje enlaces amigables.

sudo sed -i 's|DocumentRoot .*|DocumentRoot /nfs/shared/wordpress|g' /etc/apache2/sites-available/000-default.conf: modifica el archivo por defecto para que la raíz apunte al directorio compartido NFS donde está wordpress.

sed -i '/<\/VirtualHost>/i \
<Directory /nfs/shared/wordpress>\
    Options Indexes FollowSymLinks\
    AllowOverride All\
    Require all granted\
</Directory>' /etc/apache2/sites-available/000-default.conf: inserta directivas necesarias para configurar permisos de acceso y habilitar el uso de archivos .htaccess en el directorio /nfs/shared/wordpress:

Options Indexes FollowSymLinks: Permite listar directorios y seguir enlaces simbólicos.

AllowOverride All: Permite que WordPress utilice .htaccess para personalizar configuraciones.

Require all granted: Da acceso a todos los usuarios.

cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/websv.conf: copia el sitio por defecto y crea uno nuevo.

mkdir -p /nfs/shared: crea el directorio /nfs/shared.

mount 192.168.50.135:/var/nfs/shared /nfs/shared: monta el directorio del servidos NFS en el directorio local.

a2dissite 000-default.conf: deshabilita el archivo por defecto.

a2ensite websv.conf: habilita el arcivo creado.

echo "192.168.50.135:/var/nfs/shared /nfs/shared nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0" | sudo tee -a /etc/fstab: manda la línea echo al fichero /etc/fstab.

mount -a: monta los recursos definido en /etc/fstab.

systemctl restart apache2: reinicia Apache.

systemctl reload apache2: recarga Apache.

systemctl status apache2: muestra el estado de Apache.

#### SQL
```bash
apt update -y
apt install mysql-server -y
sed -i "s/^bind-address\s*=.*/bind-address = 192.168.50.152/" /etc/mysql/mysql.conf.d/mysqld.cnf
sudo systemctl restart mysql

mysql <<EOF
CREATE DATABASE db_wordpress;
CREATE USER 'alberto'@'192.168.50.%' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON db_wordpress.* TO 'alberto'@'192.168.50.%';
FLUSH PRIVILEGES;
EOF
```
apt update -y: actualiza la lista de paquetes.

apt install mysql-server -y: instala el servidor de MySQL.

sed -i "s/^bind-address\s*=.*/bind-address = 192.168.50.152/" /etc/mysql/mysql.conf.d/mysqld.cnf: modifica el archivo de configuración de MySQL para que el servicio escuche en la dirección IP 192.168.50.152 en lugar de 127.0.0.1, esto permite que las conexiones externas desde la red puedan acceder al servidor.

sudo systemctl restart mysql: reinicia el servicio de MySQL.

mysql <<EOF
CREATE DATABASE db_wordpress;
CREATE USER 'alberto'@'192.168.50.%' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON db_wordpress.* TO 'alberto'@'192.168.50.%';
FLUSH PRIVILEGES;
EOF: crea la base de datos, un usuario y su contraseña, le da todos los privilegios a ese usuario y recarga los privilegios del servidor MySQL.


## 2. Servicios necesarios

### 2.1 Capa 1 Balanceador de carga
- **Instalar y configurar Apache** 
- **Configurar HTTPS**
- **Configuración de balanceo de carga**

### 2.2 Capa 2 Servidores backend y NFS
- **Servidores de Backend:**
  - Instalar Apache en los servidores de la Capa 2 para gestionar las solicitudes del balanceador.
  
- **Servidor NFS:**
  - Instalar y configurar el servidor NFS en la Capa 2 que contenga los recursos para WordPress y que los exporte a los servidores backend.

### 2.3 Capa 3 Servidor de BBDD
- **Instalar MySQL o MariaDB** en el servidor de la Capa 3 para almacenar la base de datos de WordPress.
- **Configurar el acceso** para que solo los servidores de la Capa 2 puedan interactuar con la base de datos.


## 3. Desarrollo
### 3.1 Creación de la VPC.
**Lo primero es crear la VPC**
![Captura de pantalla 2024-12-05 171813](https://github.com/user-attachments/assets/c2426a88-2c17-4f32-bd77-9fcfdf1cc444)
![Captura de pantalla 2024-12-03 195811](https://github.com/user-attachments/assets/f7123218-4aa5-49f1-89e5-8359674dc9c8)

### 3.2 Lanzar las instancias
**Ahora creamos las instancias (repetimos este proceso para todas las instancias, con sus respectivas configuraciones.)**
![Captura de pantalla 2024-12-03 200713](https://github.com/user-attachments/assets/57966129-e994-4d03-8f5c-557d11cd9ea4)
![Captura de pantalla 2024-12-04 162911](https://github.com/user-attachments/assets/3c901723-e880-4f23-acee-fd9db75d5098)
![Captura de pantalla 2024-12-04 121107](https://github.com/user-attachments/assets/4b6cbf1c-fd5f-4df7-a8a7-bfcbcf30a881)
![Captura de pantalla 2024-12-04 121140](https://github.com/user-attachments/assets/5120bc18-7d64-425d-925c-1ebbde8323da)
**Comprobamos que se crea correctamente**
![Captura de pantalla 2024-12-04 121246](https://github.com/user-attachments/assets/36e8cbe3-dba1-4a6d-9868-8a6260452c72)

### 3.3 Configurar grupos de seguridad
Cada grupo de seguridad debe tener unas restricciones:

**GRUPO DE SEGURIDAD DE BALANCEADOR**
![Captura de pantalla 2024-12-05 183053](https://github.com/user-attachments/assets/ca1fd440-0efc-4ed7-a040-9102efc3a8bf)

**GRUPOS DE SEGURIDAD NFS**
![image](https://github.com/user-attachments/assets/20a910f2-dab5-468a-a078-30609fc6d407)

**GRUPOS DE SEGURIDAD DE LOS WEBS**
![Captura de pantalla 2024-12-05 183259](https://github.com/user-attachments/assets/882610ee-e0c2-43ce-b6c8-8e59099f2a61)

**GRUPOS DE SEGURIDAD DEL SQGB**
![Captura de pantalla 2024-12-05 183506](https://github.com/user-attachments/assets/b1eab736-1708-4540-98c3-d88713d9d36a)


### 3.4 Configurar las máquinas
Una vez ya creadas todas las instancias, **_ejecutamos los scripts de aprovisionamientos y pasamos a la instalación de Wordpress._**

Para ello a la instancia del balanceador le tenemos que asignar una dirección ip elástica a través de la cuál accederemos a la página.

Una vez asignada, buscamos en el navegador http://dirección-elástica y procedemos con la instalación.

![Captura de pantalla 2024-12-04 224335](https://github.com/user-attachments/assets/79804889-8f51-4042-9e1d-eedcf0db2414)
![Captura de pantalla 2024-12-04 224449](https://github.com/user-attachments/assets/c1932781-74bb-4ac7-8032-87133a542f0d)
![foto](https://github.com/user-attachments/assets/bf1ec192-e0cb-4f8f-9321-705fdb456f55)
![Captura de pantalla 2024-12-05 131403](https://github.com/user-attachments/assets/f0d24867-ad41-40b1-8a78-2d01ff08a3d4)
![Captura de pantalla 2024-12-05 131449](https://github.com/user-attachments/assets/c64e58ea-8123-43bb-a592-6bba92fc0b59)

Una vez instalado Wordpress pasamos a estabecer el sitio seguro.

### 3.5 Sitio seguro
**Crear dominio e instalar certificado para ese dominio**

Para que nuestra página sea segura tenemos que instalar un certificado ssl para un dominio que crearemos y que le asignaremos la ip elástica.

Para ello creamos el dominio en [No IP](https://www.noip.com/es-MX/remote-access?msclkid=77fa27a2dd6a12aac4bd40f31b25bc18&utm_campaign=free-dynamic-dns&utm_medium=cpc&utm_source=bing) y le asociamos la ip elástica.

Para el sitio seguro debemos de ejecutar los siguientes comandos:
```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot
```
**Una vez hecho esto, podemos probar a entrar en nuestra página con https://dirección-ip-elástica**

![Captura de pantalla 2024-12-05 181746](https://github.com/user-attachments/assets/09277ad8-e8ea-4942-9e15-a0d7dca2040d)
