# SCRIPTS DE APROVISONAMIENTO

## SCRIPT PARA BALANCEADOR
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

## SCRIPT PARA SERVIDORES WEB
```bash
apt update -y
apt install apache2 -y
apt install nfs-common -y
apt install php libapache2-mod-php php-mysql php-curl php-gd php-xml php-mbstring php-xmlrpc php-zip php-soap php -y
a2enmod rewrite
#servidores web
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

## SCRIPT PARA NFS
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

## SCRIPT PARA SGBD
```bash
apt install mysql-server -y
apt update -y
apt install -y mysql-server
sudo apt install -y phpmyadmin
sed -i "s/^bind-address\s*=.*/bind-address = 192.168.50.152/" /etc/mysql/mysql.conf.d/mysqld.cnf
sudo systemctl restart mysql

mysql <<EOF
CREATE DATABASE db_wordpress;
CREATE USER 'alberto'@'192.168.50.%' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON db_wordpress.* TO 'alberto'@'192.168.50.%';
FLUSH PRIVILEGES;
EOF
```
