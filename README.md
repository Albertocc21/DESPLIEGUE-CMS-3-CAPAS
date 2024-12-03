# DESPLIEGUE-CMS-3-CAPAS

<b>En esta práctica realizaremos un despliegue en 3 capas la cuál contará con lo siguiente:</b>

Capa 1: Una capa pública donde se encontrará el balanceador de carga.

Capa 2: Una capa privada donde estarán los servidores de Backend y el  servidor NFS.

Capa 3: Y otra capa privada donde se alojará el servidor de BBDD.

<b>También tenemos las siguientes condiciones:</b>

Solo se accederá a la capa pública desde el exterior, no habrá conectividad en entre la capa 1 y 3 y se configurarán grupos de seguridad para proteger las máquinas.


<b>Y además las máquinas deberán de contar con:</b>

Capa 1: Servidor Apache que actúe como balanceador de carga.

Capa 2: Dos servidores Apache que serán los backend y un servidor NFS que tendrá los recursos necesarios del CMS.

Capa 3: Servidor BBDD que tendrá instalado MariaDB o MySQL.
