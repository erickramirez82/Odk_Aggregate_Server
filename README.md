# Open Data Kit (ODK) 

ODK es una colección de herramientas para recopilar y administrar datos de encuestas utilizando dispositivos móviles android. Crea el formulario de encuesta, utiliza el formulario para recopilar datos de los dispositivos portátiles, almacena y publica los datos según la necesidad del usuario. ODK Build es una herramienta para crear un formulario. ODK Collect utiliza el formulario para recopilar y enviar datos a ODK Aggregate . ODK Aggregate recibe los datos y realiza varios tipos de gestión de datos, incluida la exportación a CSV y KML. Los usuarios pueden incluso utilizar MS Excel o Open Office para crear el formulario sin escribir un solo código. Todo esto es bastante simple, sigue leyendo para los pasos.


## Instalación y configuración Open Data Kit (ODk) en Centos

ODK necesita un lugar donde pueda implementar y personalizar su propio servidor y base de datos Tomcat.

1. Instalar el JAVA

```bash
yum update
yum clean all
yum search java | grep -i --color JDK
yum install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64
java -version
```

El paquete Java SDK se instalará en el /usr/lib/jvmdirectorio de forma predeterminada. Utilícelo para establecer JAVA_HOME
la variable de entorno. Puede configurar JAVA_HOME permanentemente agregando esta línea a "$HOME/.bashrc".

```bash
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.111-0.b15.el6_8.x86_64/
echo ${JAVA_HOME}
```
2. Configuración del TOMCAT

El último Tomcat estable es la versión 8.5.6 y se puede descargar desde el sitio web oficial.

```bash
cd /home/sr/odk
wget http://www-eu.apache.org/dist/tomcat/tomcat-8/v8.5.8/bin/apache-tomcat-8.5.8.tar.gz
tar -xvzf apache-tomcat-8.5.6.tar.gz
mv apache-tomcat-8.5.6 /opt/tomcat
```

Activamos el servicio de TOMCAT

```bash
systemctl enable tomcat
```

3. Instalación de motor de base de datos.

Ahora necesitamos instalar mysql-server y mysql en este servidor. Centos7 viene con MariaDB en lugar de MySQL. MariaDb es una fuente abierta equivalente a MySQL. Aunque podemos instalar Postgress es mas recomendable con la extensión Postgis.

## POSTGRES con Postgis

Para instarlar usamos comando

```bash
rpm -ivh http://yum.postgresql.org/9.5/redhat/rhel-7-x86_64/pgdg-centos95-9.5-2.noarch.rpm
yum list | grep pgdg95
yum install -y postgis2_95 postgresql95 postgresql95-server postgresql95-libs postgresql95-contrib \
postgresql95-devel gdal gdal-python geos python-imaging gcc-c++  \
python-psycopg2 libxml2 libxml2-devel libxml2-python libxslt libxslt-devel libxslt-python
```

Ahora que nuestro software está instalado, tenemos que realizar algunos pasos antes de poder usarlo.

Cree un nuevo clúster de bases de datos PostgreSQL:

```bash
postgresql-setup initdb
```

De forma predeterminada, PostgreSQL no permite la autenticación de contraseñas. Cambiamos esto editando su configuración de autenticación basada en host (HBA).

Abra la configuración del HBA con su editor de texto favorito. Usaremos vi:
```bash
vi /var/lib/pgsql/data/pg_hba.conf
```
Encuentre las líneas que se vean así, cerca de la parte inferior del archivo:

 <b> pg_hba.conf (original)</b>
```
host    all             all             127.0.0.1/32            ident
host    all             all             ::1/128                 ident
```

Luego reemplace "ident" por "md5", por lo que se parecen a esto:

<b> pg_hba.conf (actualizado)</b>
```
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Guardar y Salir. PostgreSQL está ahora configurado para permitir la autenticación de contraseñas.

Ahora inicie y habilite PostgreSQL:
```bash
systemctl start postgresql
systemctl enable postgresql
```
PostgreSQL ya está listo para ser utilizado.

## MySQL. MariaDb

```bash
yum -y instalar mariadb-server mariadb
systemctl enable mysqld
```

Establecer una contraseña aquí es una buena práctica de seguridad. Para administrar esto use este comando.

```bash
mysql_secure_installation
```

Establezca una contraseña. Continúe con otros ajustes con la escritura y.

```
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

Ahora vamos a comprobar si nuestro servidor MySQL funciona. Utilice su contraseña para iniciar sesión.

```bash
mysql -u root -p
```
4. Vamos instalar <a href="https://opendatakit.org/downloads/download-category/aggregate/" target="_blank">ODK Aggregate</a>

<img src="https://github.com/erickramirez82/Odk_Aggregate_Server/blob/master/Descargar_odkaggregate.png">

```bash
wget https://s3.amazonaws.com/opendatakit.downloads/ODK%20Aggregate%20v1.4.12%20linux-x64-installer.run
chmod +x ODK\ Aggregate\ v1.4.12\ linux-x64-installer.run
./ODK\ Aggregate\ v1.4.12\ linux-x64-installer.run
```

El instalador creará un archivo .wary .sqlbasado en las configuraciones que establezca. El .sqlarchivo tendrá los códigos necesarios para las bases de datos y el .wararchivo establecerá la aplicación web. Mantenga presionando la tecla Intro (Intro) para desplazarse por los archivos de configuración iniciales, probablemente tendrá siete o nueve devoluciones.

<img src="https://github.com/erickramirez82/Odk_Aggregate_Server/blob/master/instalar_Odk.png" />

Aceptamos las licencia

```
Do you accept this license? [y/n]: y
```

Los preguntas vamos encontrar durante la instalación

```
----------------------------------------------------------------------------
 Select an output parent directory
 
This "installer" does not actually install anything. Instead, it guides you
through configuring ODK Aggregate. As the last step of the installer, it will
either launch the upload tool for uploading to Google's AppEngine cloud services
or open a README.html containing the final installation instructions for MySQL
or PostgreSQL deployments.
 
Please select the parent directory under which an 'ODK Aggregate' directory will
be created to contain the configured software.
 
[]: /home/sr/odk
 
----------------------------------------------------------------------------
Choose Platform
 
Choose the data storage and execution environment on which you would like ODK Aggregate to run. 
If you choose MySQL or PostgreSQL, you will need to first install that database server and 
an Apache Tomcat 6 webserver.
 
[1] Google App Engine: Uses Google's cloud-based data storage and webserver services.
[2] MySQL: Uses a MySQL database and an Apache Tomcat 6 webserver that you install.
[3] PostgreSQL: Uses a PostgreSQL database and an Apache Tomcat 6 webserver that you install.
Please choose an option [1] : 2
 
----------------------------------------------------------------------------
Pre-installation Requirements - Apache Tomcat
 
Before continuing, you should download and install an Apache Tomcat 6 webserver
within which you will run ODK Aggregate (instructions on installing your chosen
database will be presented later).
 
Apache Tomcat 6 can be downloaded from the link below.
 
If you modify any of the Apache Tomcat defaults, please make note of them.
 
http://tomcat.apache.org/download-60.cgi [Y/n]: y
 
----------------------------------------------------------------------------
Apache Tomcat SSL configuration
 
SSL certificates allow a client to verify the identity of a webserver and to
establish secured (encrypted) communications with that webserver (https:). SSL
certificates are typically purchased from Verisign or another issuing authority;
that process can take weeks to complete.
 
If you do have an SSL certificate, please refer to the Apache Tomcat
documentation for how to install the certificate on your system. Within the
Tomcat 6.0 documentation, refer to the User Guide section 12: SSL.
 
http://tomcat.apache.org/tomcat-6.0-doc/ssl-howto.html [Y/n]: y
 
----------------------------------------------------------------------------
Apache Tomcat SSL configuration
 
Do you have an SSL certificate?
 
[1] No, I do not have an SSL certificate: - User logins ARE secure.
- Submitting and/or viewing form data MAY NOT BE secure.
- Changing passwords MAY NOT BE secure.
Security breaches are particularly likely over unsecured WiFi hotspots.
 
[2] Yes, I have an SSL certificate: - User logins ARE secure.
- Submitting and/or viewing form data IS secure.
- Changing passwords IS secure.
 
Please choose an option [1] : 1
 
----------------------------------------------------------------------------
Apache Tomcat Port Configuration
 
ODK Aggregate needs to know the port numbers and the Internet-visible IP address
or DNS name through which it can be reached in order to construct links back to
its content when publishing or exporting data files.
 
Follow the ODK Aggregate Tomcat Install instructions (link below) to make your
computer accessible on the web (using an Internet-visible IP address) and to
establish a DNS name.
 
http://opendatakit.org/use/aggregate/tomcat-install/#Configure_for_Network_Access [Y/n]: y
 
HTTP/1.1. Connector Port: [8080]: 8080
 
Internet-visible IP address or DNS name: [srv1.clubgis.net]: xxx.xxx.xx.xx
 
----------------------------------------------------------------------------
Pre-installation Database Requirements - MySQL
 
We have developed and tested against MySQL 5.1 and 5.5
 
Please download, install and configure MySQL from the link below.
 
Choose the Downloads (GA) tab, and select the MySQL Community Server edition.
When running the MySQL Server Instance Configuration Wizard, please make note of
your root password and any non-default values you have entered during the
installation process.
 
http://www.mysql.com/downloads/ [Y/n]: y
 
----------------------------------------------------------------------------
Pre-installation Database Requirements - MySQL (concluded)
 
You also must install a MySQL library on the Apache Tomcat instance.
Please download the MySQL Connector/J zip file from the link below.
 
Unzip or extract the contents of this file to a directory on your system.
The top-level directory should contain a file named:
 
mysql-connector-java-...-bin.jar
 
Copy this file into the /lib directory of your Apache Tomcat 6 installation. On
Windows, the default location for this is 'C:\Program Files\Apache Software
Foundation\Tomcat 6.0\lib'
 
Stop and restart your Apache webserver (so it can discover this new library).
 
http://www.mysql.com/downloads/connector/j/5.1.html [Y/n]: y
 
----------------------------------------------------------------------------
Database server settings
 
Specify the host and port number used to communicate with the database server.
 
If the database server is on the same host as the webserver, enter '127.0.0.1'
 
Database port number: [3306]: 3306
 
Database server hostname: [127.0.0.1]: 127.0.0.1
 
----------------------------------------------------------------------------
ODK Aggregate database authentication settings
 
The database server must be configured with an account (username and password)
that ODK Aggregate will employ for accessing the database server. This is not
an account that you will give to anyone; it is only for use by the ODK Aggregate
webserver. Username and password should be alphanumeric strings beginning with a
letter (no spaces or punctuation please!).
 
The installer will generate a script that can be run on the database server to
establish the required configuration.
 
Database username: [odk_user]: odk_user
 
Database password: :
Retype password: :
----------------------------------------------------------------------------
ODK Aggregate database datastore settings
 
The database server must be configured with a workspace identified by a database
name, and, if using Postgresql, a schema name. This workspace is where ODK
Aggregate will store uploaded forms, submissions and other information. The
database name (and schema name for Postgresql) can be any alphanumeric strings
beginning with a letter; underscores (_) are OK (no spaces or punctuation
please!).
 
The installer will generate a script that can be run on the database server to
create the workspace.
 
Database Name: [odk_prod]: odk_prod
 
----------------------------------------------------------------------------
ODK Aggregate Instance Name
 
The ODK Aggregate instance name will be displayed to your users when they log
into ODK Aggregate using their ODK Aggregate username and password. Including
the name of your organization in the instance name can help users confirm that
they have contacted the correct website.
 
NOTE: During subsequent upgrades, any changes to this text will invalidate all
the ODK Aggregate passwords configured on your webserver. To prevent
disruptions during upgrades, please make a record of the name you have chosen.
 
Please enter the name of your instance: []: odk_test
 
----------------------------------------------------------------------------
Super-user ODK Aggregate Username
 
The user with this account will always have full permissions on the system.
 
The first time ODK Aggregate runs, only this user is allowed to log onto the
system. Upon first logging in, they should change their password and complete
the configuration of ODK Aggregate by specifying additional users and what
permissions those users have on the system. This user's password will be set to
'aggregate' upon initial install or upon changing the ODK Aggregate instance
name.
 
Please enter an ODK Aggregate Username: []: admin
 
----------------------------------------------------------------------------
Setup is now ready to begin configuring ODK Aggregate.
 
Do you want to continue? [Y/n]: y
```

Si va a la carpeta de instalación ( /home/sr/odk), encontrará tres archivos.
```
ODKAggregate.war
README.html
create_db_and_user.sql
```








