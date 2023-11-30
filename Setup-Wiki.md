# Setup Aldea-Wiki

## 0. Configuraciones preliminares

### Configuración del nombre del host y archivo de hosts

Ahora, procedemos a configurar el nombre del host del sistema, así como a editar el archivo de hosts. Utilizamos el editor de texto `nano` para realizar estas tareas:

```bash
sudo nano /etc/hostname
```

Luego también procedemos a cambiar el archivo hosts.

```bash
sudo nano /etc/hosts
```

Este comando abre el archivo `/etc/hosts` en el editor `nano`. Aquí, puedes asignar la dirección IP del servidor al nombre del host. Asegúrate de que la línea correspondiente a la dirección IP `127.0.1.1` incluya el nombre del host que configuraste en el paso anterior.

### Creación de un nuevo usuario

Como recomendación de seguridad no es apropiado usar el usuario root directamente en las sesiones SSH. Por ello creamos un nuevo usuario llamado "ejemplo" utilizando el comando `adduser`:

```bash
adduser ejemplo
```

Este comando solicitará información adicional, como la contraseña del nuevo usuario y detalles opcionales como nombre completo, número de habitación, etc. Proporcione la información solicitada según sea necesario.

### Actualización del sistema

Antes de proceder con la configuración e instalación de MediaWiki, es importante mantener el sistema actualizado. Utilizamos los siguientes comandos para actualizar y mejorar los paquetes del sistema:

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Asignación de permisos sudo al nuevo usuario

A continuación, otorgamos privilegios de sudo al usuario "aldeano" para que pueda ejecutar comandos administrativos. En sistemas basados en Debian (como Ubuntu), utilizamos el comando `usermod`:

```bash
sudo usermod -aG sudo ejemplo
```

Este comando agrega el usuario "ejemplo" al grupo sudo, permitiéndole ejecutar comandos con privilegios administrativos.

## 1. Instalación dependencias

### Instalación dependencias Apache, MariaDB y PHP para MediaWiki

Ahora procedemos a instalar los componentes necesarios, incluyendo el servidor web Apache, el servidor de base de datos MariaDB y varias extensiones de PHP requeridas por MediaWiki. Utilizamos el siguiente comando:

```bash
sudo apt-get install apache2 mariadb-server php php-intl php-mbstring php-xml php-apcu php-curl php-mysql
```

Este comando instala los siguientes paquetes:

- **apache2**: El servidor web Apache.
- **mariadb-server**: El servidor de base de datos MariaDB.
- **php**: El lenguaje de scripting PHP.
- **php-intl**: Extensión para el soporte internacional en PHP.
- **php-mbstring**: Extensión para el manejo de cadenas de caracteres multibyte en PHP.
- **php-xml**: Extensión para el manejo de XML en PHP.
- **php-apcu**: Extensiones para la caché de PHP.
- **php-curl**: Extensión para el soporte de cURL en PHP.
- **php-mysql**: Extensión para la integración de MySQL en PHP.

### Descarga de MediaWiki

Una vez en el directorio home de nuestro usuario, continuamos descargando la última versión (1.40.1 en nuestro caso) de MediaWiki desde el sitio oficial. Utilizamos el comando `wget` para descargar el archivo y `tar` para descomprimirlo:

```bash
cd /home/ejemplo
wget https://releases.wikimedia.org/mediawiki/1.40/mediawiki-1.40.1.tar.gz
tar -xzvf mediawiki-*.tar.gz
```

Puedes encontrar la versión actual en este [link](https://www.mediawiki.org/wiki/Download). En la página debes copiar la dirección de la url y aplicarlo como argumento para wget. 

Para organizar los archivos de MediaWiki, siguiendo los comandos proporcionados:

```bash
sudo mkdir /var/lib/aldea-wiki
sudo mv mediawiki-1.40.1 /var/lib/aldea-wiki/
```

- `cd /home/usuario`: Cambia al directorio del usuario. Asegúrate de ajustar "usuario" al nombre real del usuario.
- `sudo mkdir /var/lib/aldea-wiki`: Crea un directorio en el sistema para almacenar los archivos de MediaWiki. Puedes elegir un nombre diferente en lugar de "aldea-wiki" si lo prefieres.
- `sudo mv mediawiki-1.40.1 /var/lib/aldea-wiki/`: Mueve la carpeta de la instalación de MediaWiki al nuevo directorio que acabas de crear.

### Configuración segura de MariaDB (MySQL)

Es una buena práctica ejecutar la configuración segura de MariaDB (equivalente a MySQL) después de su instalación. Este proceso te guiará a través de la mejora de la seguridad del servidor de base de datos. Utilizamos el siguiente comando:

```bash
sudo mysql_secure_installation
```

Este comando inicia un script interactivo que te hará las siguientes preguntas:

1. **Enter current password for root (enter for none):** Presiona Enter, ya que aún no se ha establecido una contraseña.
2. **Set root password? [Y/n]:** Elige si deseas establecer una contraseña para el usuario root de MariaDB.
3. **Remove anonymous users? [Y/n]:** Responde "Y" para eliminar el acceso anónimo, mejorando la seguridad.
4. **Disallow root login remotely? [Y/n]:** Responde "Y" para evitar que el usuario root inicie sesión de forma remota.
5. **Remove test database and access to it? [Y/n]:** Responde "Y" para eliminar la base de datos de prueba, que no es necesaria en un entorno de producción.
6. **Reload privilege tables now? [Y/n]:** Presiona "Y" para recargar las tablas de privilegios y aplicar los cambios.

### Instalación adicional de extensiones de PHP para MediaWiki

Para garantizar que MediaWiki tenga todas las extensiones de PHP necesarias, instalamos algunas más utilizando el siguiente comando:

```bash
sudo apt-get install php-apcu imagemagick php-curl php-intl
```

### Reboot para actualizar los cambios

```bash
sudo reboot
```

## 2. Configuración Base de datos

### Acceso a la consola de MySQL (MariaDB)

Para acceder a la consola de MySQL (MariaDB) y realizar operaciones directamente en la base de datos, utilizamos el siguiente comando:

```bash
sudo mysql
```

Este comando abrirá la consola de MySQL (MariaDB) y te permitirá interactuar directamente con la base de datos. Asegúrate de tener los privilegios adecuados para realizar las operaciones que planeas ejecutar.

### Creación usuarios, permisos y base de datos

Ahora, configuramos la base de datos necesaria para MediaWiki utilizando la consola de MySQL. A continuación, se encuentran los comandos que debes ejecutar.

Crea un nuevo usuario de la base de datos con el nombre de usuario 'ejemplo-usuario' y la contraseña especificada. Asegúrate de sustituir 'ejemplo-usuario' y 'contraseña' con los valores reales que deseas utilizar.

```sql
CREATE USER 'ejemplo-usuario'@'localhost' IDENTIFIED BY 'contraseña';
```

Crea una nueva base de datos llamada 'aldeawiki_db'. Puedes cambiar el nombre de la base de datos si lo prefieres.

```sql
CREATE DATABASE aldeawiki_db;
```

Concede todos los privilegios en la base de datos recién creada al usuario 'aldea-wiki' para las conexiones locales.

```sql
GRANT ALL ON aldeawiki_db.* TO 'aldea-wiki'@'localhost';
```

Salimos con el siguiente comando 

```sql
\q
```

 ## 3. Configuración del servidor

A continuación, realizamos algunos pasos adicionales para configurar MediaWiki y aplicar cambios en la configuración de PHP y Apache. Sigue estos comandos:

```bash
sudo ln -s /var/lib/aldea-wiki /var/www/html/aldea-wiki
```

Este comando crea un enlace simbólico entre el directorio de instalación de MediaWiki y el directorio web de Apache. Esto facilitará el acceso a MediaWiki a través de la URL correspondiente.

```bash
sudo phpenmod mbstring xml
```

Aquí, habilitamos las extensiones de PHP necesarias para MediaWiki, como `mbstring` y `xml`.

```bash
sudo systemctl restart apache2
```

Finalmente, reiniciamos el servidor Apache para aplicar las configuraciones y asegurarnos de que MediaWiki esté disponible. Y verificamos el estado actual del servidor Apache con el siguiente comando: `

```bash
sudo systemctl status apache2
```

### Crear servidor de nombres para la wiki

A continuación, creamos y editamos el archivo de configuración del sitio virtual para MediaWiki:

Crea una copia del archivo de configuración predeterminado de Apache para el nuevo sitio de MediaWiki.

```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/aldea-wiki.conf
```

Abre el archivo de configuración del sitio virtual en el editor `nano` para realizar ediciones.

```bash
sudo nano /etc/apache2/sites-available/aldea-wiki.conf
```

Asegúrate de ajustar la configuración según tus necesidades, especialmente los directorios y las rutas específicas de tu instalación de MediaWiki. Una vez abierto el archivo `aldea-wiki.conf` deberás cambiar el  párametro de la variable `DocumentRoot`a la siguiente dirección.

```bash
DocumentRoot /var/www/html/aldea-wiki
```

Luego debes agregar la variable `ServerName`al archivo y especificar la dirección de dominio.

```bash
ServerName www.example.com
```

### Habilitación del Sitio Virtual y Recarga de Apache

Continuamos con la habilitación del nuevo sitio virtual de MediaWiki y la deshabilitación del sitio predeterminado. 

```bash
sudo a2ensite aldea-wiki
sudo a2dissite 000-default
```

Además, recargamos el servidor Apache y verificamos su estado:

```bash
sudo systemctl reload apache2
sudo systemctl status apache2
```

Si todo está correcto debería decir en `running` en el estado del servicio.

## 4. Configuración generación archivo de configuración de mediawiki.

1. Después de haber realizado todos los pasos anteriores, abre tu navegador web y accede al sitio de MediaWiki. Puedes hacerlo ingresando la dirección correspondiente en la barra de direcciones del navegador. Por ejemplo, si has configurado todo según las instrucciones anteriores, podrías acceder a MediaWiki mediante la URL http://tudominio/aldea-wiki.
2. En la página de inicio de MediaWiki, haz clic en el enlace que generalmente dice "Please set up the wiki first" o una opción similar. Esto te llevará al asistente de configuración inicial.
3. El asistente de configuración te guiará a través de varios pasos para configurar tu wiki. Estos pasos incluyen la configuración del idioma, la conexión a la base de datos, la configuración del nombre del wiki, y otros detalles importantes.
4. Durante este proceso, se te pedirá que ingreses información como el nombre de la base de datos, el nombre de usuario y la contraseña de la base de datos que configuraste previamente. También se te pedirá que establezcas una contraseña para la cuenta de administrador de MediaWiki.
5. Después de completar la configuración, el asistente generará un archivo llamado `LocalSettings.php`. Este archivo contiene la configuración específica de tu instalación de MediaWiki.
6. Debes descargar este archivo `LocalSettings.php`. Puedes hacerlo a través del enlace proporcionado o accediendo directamente al servidor y copiándolo desde la ubicación donde se ha generado.

Después de descargar el archivo `LocalSettings.php` desde el asistente de configuración de MediaWiki, necesitas transferir ese archivo a tu servidor remoto, específicamente al directorio home del usuario `aldeano`. Puedes utilizar `scp` (Secure Copy Protocol) para realizar esta transferencia de forma segura. Aquí hay un resumen de los pasos:

1. Abre una terminal en tu máquina local (donde has descargado `LocalSettings.php`).

2. Utiliza el siguiente comando `scp` para copiar el archivo al directorio home del usuario `aldeano` en tu servidor remoto. Asegúrate de reemplazar `ruta/local/de/LocalSettings.php` con la ruta completa del archivo `LocalSettings.php` en tu máquina local y `usuario@direccion-IP-del-servidor:/home/aldeano/` con la dirección y el nombre de usuario del servidor remoto:

    ```bash
    scp ruta/local/de/LocalSettings.php usuario@direccion-IP-del-servidor:/home/aldeano/
    ```

    Por ejemplo:

    ```bash
    scp /ruta/local/de/LocalSettings.php aldeano@192.168.1.100:/home/aldeano/
    ```

3. Cuando se te solicite, ingresa la contraseña del usuario en el servidor remoto para completar la transferencia.

Este comando `scp` copiará el archivo `LocalSettings.php` desde tu máquina local al directorio home del usuario `aldeano` en el servidor remoto. Luego, este archivo estará listo para ser utilizado por tu instalación de MediaWiki en el servidor.

Cambia la propiedad del archivo `LocalSettings.php` para que sea propiedad del usuario y grupo de Apache (usualmente `www-data`):

```bash
sudo chown www-data:www-data LocalSettings.php
```

Ajusta los permisos del archivo para garantizar la seguridad:

```bash
sudo chmod 600 LocalSettings.php
```

Mueve el archivo `LocalSettings.php` al directorio adecuado para que MediaWiki pueda acceder a él:

```bash
sudo mv LocalSettings.php /var/www/html/aldea-wiki/
```

## 5. Backup

*Por hacer*

## Referencia

[Ver el tutorial de instalación de MediaWiki en YouTube](https://www.youtube.com/watch?v=FN9VdCm0FrU)





