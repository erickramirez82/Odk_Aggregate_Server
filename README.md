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











