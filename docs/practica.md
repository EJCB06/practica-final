# Práctica 01 – Implantación de Aplicaciones Web (IAW)

Repositorio para la práctica 01 del módulo de IAW. En este proyecto automatizamos el despliegue de una pila LAMP completa y la instalación de varias herramientas de administración y monitorización web.

## Tecnologías Utilizadas

![Bash](https://img.shields.io/badge/Bash-121011?style=for-the-badge&logo=gnu-bash&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=Apache&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-00000F?style=for-the-badge&logo=mysql&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)
![GoAccess](https://img.shields.io/badge/GoAccess-FFA500?style=for-the-badge&logo=goaccess&logoColor=white)


## Parte 1: Instalación de la Pila LAMP

El siguiente script en Bash (`#!/bin/bash`) se encarga de dejar el servidor web listo con Apache, PHP y MySQL.

> **Nota:** *La actualización de repositorios la tenemos que hacer siempre que vayamos a instalar cualquier cosa.*

```bash
set -ex

# 1. Actualizamos los repositorios y paquetes del sistema
apt update
apt upgrade -y

# 2. Instalamos el servidor web Apache
apt install apache2 -y

# 3. Habilitamos el módulo rewrite de Apache
a2enmod rewrite

# 4. Copiamos nuestro archivo de configuración y reiniciamos
# Cada vez que toquemos el archivo de conf, tendremos que reiniciar el servicio
cp ../conf/000-default.conf /etc/apache2/sites-available
systemctl restart apache2

# 5. Instalamos PHP y sus módulos
apt install php libapache2-mod-php php-mysql -y
systemctl restart apache2

# 6. Copiamos el index y ajustamos permisos
cp ../php/index.php /var/www/html
chown -R www-data:www-data /var/www/html/* # Cambiamos el propietario para que no sea root

# 7. Instalamos MySQL Server
apt install mysql-server -y
```

## Parte 2: Instalación de Herramientas y Monitorización
Para esta parte tendríamos que haber ejecutado ya el script de arriba, es decir, ya tendríamos que tener la pila LAMP totalmente funcional.

### Preparación del entorno
Importamos el archivo .env. Este archivo nos sirve para guardar variables y poder llamarlas después dentro de nuestro script.

```Bash
#!/bin/bash
set -ex

source .env
# Instalación de phpMyAdmin
# Configuramos las respuestas automáticas para que la instalación no nos pida datos por pantalla.

echo "phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2" | debconf-set-selections
echo "phpmyadmin phpmyadmin/dbconfig-install boolean true" | debconf-set-selections
echo "phpmyadmin phpmyadmin/mysql/app-pass password $PHPMYADMIN_APP_PASSWORD" | debconf-set-selections
echo "phpmyadmin phpmyadmin/app-password-confirm password $PHPMYADMIN_APP_PASSWORD" | debconf-set-selections

apt update
apt install phpmyadmin php-mbstring php-zip php-gd php-json php-curl -y
Con esto podemos comprobar que la instalación de phpMyAdmin se ha hecho correctamente:

# Instalación de Adminer

mkdir -p /var/www/html/adminer

# Descargamos el archivo de Adminer y lo renombramos a index.php
wget [https://github.com/vrana/adminer/releases/download/v5.4.1/adminer-5.4.1-mysql.php](https://github.com/vrana/adminer/releases/download/v5.4.1/adminer-5.4.1-mysql.php) -P /var/www/html/adminer
mv /var/www/html/adminer/adminer-5.4.1-mysql.php /var/www/html/adminer/index.php
Comprobación de Adminer:

# Creación de Base de Datos de Ejemplo
# Esto no es realmente necesario pero al hacerlo podemos tener una primera toma de contacto con las variables de nuestro archivo .env.


# Creamos la BD
mysql -u root -e "DROP DATABASE IF EXISTS $DB_NAME;"
mysql -u root -e "CREATE DATABASE $DB_NAME;"

# Creamos un usuario/contraseña y le damos privilegios
mysql -u root -e "DROP USER IF EXISTS '$USER'@'%';"
mysql -u root -e "CREATE USER '$USER'@'%' IDENTIFIED BY '$DB_PASSWORD'"
mysql -u root -e "GRANT ALL PRIVILEGES ON $DB_NAME.* TO '$USER'@'%';"

# Monitorización con GoAccess
# Vamos a generar estadísticas en tiempo real protegiendo el directorio con autenticación básica (htpasswd).

apt update
apt install goaccess -y

# Creamos el directorio y generamos el HTML en tiempo real
mkdir -p /var/www/html/stats
goaccess /var/log/apache2/access.log -o /var/www/html/stats/index.html --log-format=COMBINED --daemonize --real-time-html

# Autenticación básica
# Copiamos la conf de Apache que contiene las directivas para /stats
cp ../conf/000-default-stats.conf /etc/apache2/sites-available/000-default.conf

# Creamos las credenciales
htpasswd -bc /etc/apache2/.htpasswd $STATS_USERNAME $STATS_PASSWORD
cp ../htaccess/.htaccess /var/www/html/stats

systemctl restart apache2
```
