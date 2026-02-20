# Práctica 04 – Implantación de Aplicaciones Web (IAW)

Repositorio para la práctica 04 del módulo de IAW. En esta práctica configuramos un servidor web seguro habilitando HTTPS mediante la creación de un certificado SSL autofirmado y forzando la redirección del tráfico HTTP a HTTPS.

## Tecnologías Utilizadas

![Bash](https://img.shields.io/badge/Bash-121011?style=for-the-badge&logo=gnu-bash&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=Apache&logoColor=white)
![OpenSSL](https://img.shields.io/badge/OpenSSL-721412?style=for-the-badge&logo=openssl&logoColor=white)



## Creación del Certificado y Configuración de Apache

El siguiente script en Bash se encarga de automatizar la creación del certificado autofirmado y de aplicar las configuraciones necesarias en Apache2.

```bash
#!/bin/bash
set -ex

# 1. Enlazamos con el fichero de variables de entorno
source .env

# 2. Creamos el certificado autofirmado
openssl req \
  -x509 \
  -nodes \
  -days 365 \
  -newkey rsa:2048 \
  -keyout /etc/ssl/private/apache-selfsigned.key \
  -out /etc/ssl/certs/apache-selfsigned.crt \
  -subj "/C=$OPENSSL_COUNTRY/ST=$OPENSSL_PROVINCE/L=$OPENSSL_LOCALITY/O=$OPENSSL_ORGANIZATION/OU=$OPENSSL_ORGUNIT/CN=$OPENSSL_COMMON_NAME/emailAddress=$OPENSSL_EMAIL"

# 3. Copiamos el archivo default-ssl.conf al servidor
cp ../conf/default-ssl.conf /etc/apache2/sites-available

# 4. Configuramos y habilitamos nuestro apache2 con el sitio SSL
a2ensite default-ssl.conf

# 5. Habilitamos el módulo ssl en Apache
a2enmod ssl

# 6. Copiamos el archivo de configuración de apache para reenviar de http a https
# Este archivo ya tiene la redirección configurada
cp ../conf/000-default.conf /etc/apache2/sites-available

# 7. Habilitamos el módulo rewrite de apache2 para que funcione la redirección
a2enmod rewrite

# 8. Reiniciamos el servicio para aplicar los cambios
systemctl restart apache2
```

# Resolución de DNS en la Máquina Anfitrión
Modificamos el archivo 000-default.conf para redirigir el tráfico HTTP a HTTPS. Este paso lo tendremos que hacer en nuestro archivo de configuración.

Y tendremos que en nuestra máquina anfitrión modificar la manera en la que nos resuelve la IP, para ello tendremos que acceder como root dentro de nuestro fichero de configuración (archivo hosts) y poner la IP privada/elástica que tenga nuestra máquina.

![imagen del cmd de mi máquina anfitrión](images/imagen%20captura%20pokemon.png)


Como podemos ver, ponemos nuestro nombre junto a la IP elástica de nuestra máquina virtual para poder resolver sin problemas.

![entrada limpia a con resolución de ip](images/captura%20practica%2004.png)

## Comprobaciones de Funcionamiento

Aquí podemos ver que nos entra en la IP elástica, aunque también resuelve por el nombre por la redirección creada.

![entrada limpia con resolución de nombre](images/practica%20https%2004.png)

Aquí podemos ver cómo entra con el nombre, por eso el cambio que hemos hecho en nuestra máquina anfitrión es muy importante, ya que sin el cambio de ese parámetro no sería posible la resolución del nombre.

![certificado que no es verificado por ninguna entidad](images/practica%2004%20cert.png)


Con esto podemos ver cómo tenemos el certificado aplicado pero nos sigue diciendo que "no es seguro" ya que es autofirmado y no está verificado por ninguna entidad certificadora como Let's Encrypt.