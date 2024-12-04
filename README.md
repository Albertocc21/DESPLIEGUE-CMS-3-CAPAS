# DESPLIEGUE-CMS-3-CAPAS

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
   - *[Comprobación](#33-comprobación)*

## 1. Diseño de la infraestructura

### 1.1 Planificación de la Arquitectura
  - **Capa 1 (Capa Pública):** Balanceador de carga accesible desde el exterior.
  - **Capa 2 (Capa Privada - Backend):** Servidores que procesan las solicitudes, con un servidor NFS.
  - **Capa 3 (Capa Privada - Base de Datos):** Servidor de base de datos MySQL o MariaDB.
  
- **Asignación de nombres (hostnames):**
  - Cada máquina debe tener un nombre como por ejemplo (BalanceadorAlbertoCalvo)

### 1.2 Configuración de acceso y seguridad
- **Restricciones:**
  - Solo se permitirá el acceso desde el exterior a la Capa 1.
  - No debe haber conectividad entre la Capa 1 y la Capa 3.
  
- **Grupos de seguridad:**
  - Configurar grupos de seguridad en las máquinas para garantizar que solo las conexiones necesarias estén permitidas entre las capas.

### 1.3 Scripts de aprovisionamiento
- Crear un script que automatice el la instalación de servicios para cada máquina.


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
## 3.1 Creación de la VPC.
**Lo primero es crear la VPC**
![Captura de pantalla 2024-12-03 195724](https://github.com/user-attachments/assets/4130821c-f0c1-44b9-a89f-de5f20a37f66)
![Captura de pantalla 2024-12-03 195811](https://github.com/user-attachments/assets/f7123218-4aa5-49f1-89e5-8359674dc9c8)

## 3.2 Lanzar las instancias
**Ahora creamos las instancias (repetimos el mismo proceso para todas, con sus respectivas configuraciones)**
![Captura de pantalla 2024-12-03 200713](https://github.com/user-attachments/assets/57966129-e994-4d03-8f5c-557d11cd9ea4)
![Captura de pantalla 2024-12-04 121032](https://github.com/user-attachments/assets/7f16d695-08e7-4e53-acc0-56b8730bcbaf)
![Captura de pantalla 2024-12-04 121107](https://github.com/user-attachments/assets/4b6cbf1c-fd5f-4df7-a8a7-bfcbcf30a881)
![Captura de pantalla 2024-12-04 121140](https://github.com/user-attachments/assets/5120bc18-7d64-425d-925c-1ebbde8323da)
**Comprobamos que se crea correctamente**
![Captura de pantalla 2024-12-04 121246](https://github.com/user-attachments/assets/36e8cbe3-dba1-4a6d-9868-8a6260452c72)

## 3.3 Comprobación














