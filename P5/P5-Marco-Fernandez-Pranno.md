## Práctica 5 - Servidores Web de Altas Prestaciones (2015)
### *Marco Manuel Fernández Pranno*

**Creación de la tabla en la base de datos - Servidor Maestro:**

La creación de la tabla se realiza primeramente en el servidor que va a actuar como maestro, con las siguientes órdenes:

`mysql -u root -p`

`create database contactos;`

`create table datos(nombre varchar(100),tlf int);`

Insertamos una entrada con: 

`insert into datos(nombre,tlf) values ("K.Mitnick",13371337);`

El resultado es el siguiente:

![alt text][creacion-tabla]

**Copia de seguridad manual entre máquinas de la base de datos:**

La copia de seguridad la realizamos con las siguientes órdenes (en la máquina Maestro), siguiendo los pasos indicados en el guión de prácticas:

`mysqldump contactos -u root -p > /root/contactos.sql`

Antes de la órden anterior, hay que salvar las tablas con un lock de lectura para que los datos no se alteren en el proceso de salvado. Y al finalizar la ejecución de mysqldump, desbloquearlas.
En mysql:

`mysql> FLUSH TABLES WITH READ LOCK;`

`mysql> UNLOCK TABLES;`

Ahora, desde la máquina esclavo, accedemos por ssh (usando scp) a la máquina maestro para copiar el archivo contactos.sql. Antes de ésto, es necesario crear la base de datos:

`mysql> create database contactos;`

Y por último:

`mysql -u root -p contactos < /root/contactos.sql`

Tal y como se refleja en las siguientes capturas:

* En el Maestro:

![alt text][manual-master]

* En el Esclavo:

![alt text][manual-slave]

**Configuración Maestro-Esclavo para la sincronización de base de datos y backup:**

En primer lugar, en el servidor Maestro accedemos al fichero de configuración de mysql en /etc/mysql/my.sql y editamos los siguientes apartados:

  1. Comentar: `(#)bind-address 127.0.0.1`
  2. Configurar (viene por defecto): `log_error = /var/log/mysql/error.log`
  3. `server-id = 1 `
  4. `log_bin = /var/log/mysql/bin.log`

Guardamos la configuración y reiniciamos el servicio con: `/etc/init.d/mysql restart`.

Repetimos estos mismos pasos en el fichero de configuración del Esclavo. Seguidamente, en éste último, accedemos a la consola de mysql y tecleamos lo siguiente:

`mysql> CHANGE MASTER TO MASTER_HOST="192.168.24.128";`

`mysql> MASTER_USER="esclavo", MASTER_PASSWORD="esclavo";` 

`mysql> MASTER_LOG_FILE="mysql-bin.000001", MASTER_LOG_POS=501, MASTER_PORT=3306;`

Y por último, en el Maestro:

`mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';`

`mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';`

`mysql> FLUSH PRIVILEGES;`

`mysql> FLUSH TABLES;`

Una vez hecho esto, tenemos la configuración preparada para la sincronización, solo queda ejecutar la orden en el servidor esclavo:

`mysql> start slave;`

**Problemas encontrados y resolución:**

El ejecutar la última sentencia, y las órdenes para visualizar el estado en las dos máquinas:

`mysql> show master status;`

`mysql> show slave status \G`

Me encontré con el problema de que Seconds_Behind_Master estaba a NULL, y Slave_IO_running en NO.

En el último log de error me encontré lo siguiente:

`Got fatal error 1236 from master when reading data from binary log: "Could not find first log file name in binary log index file"`

Es decir, los ficheros binarios de logs no estaban sincronizados entre las máquinas y utilizaban uno distinto. Después de indagar un poco con las opciones ` purge binary logs [opciones]` y sus capacidades, vi más sencillo utilizar un ` reset master` y borrar todos los logs. 

Una vez hecho ésto, la sincronización  funcionó perfectamente.

**Capturas de pantalla y muestra de funcionamiento:**

Izquierda Maestro - Derecha Esclavo.

![alt text][automatico-master-slave]

![alt text][automatico-prueba]

[creacion-tabla]: https://github.com/MarFerPra/SWAP15/blob/master/P5/imagenes/creacion-tabla.png?raw=true
[manual-slave]: https://github.com/MarFerPra/SWAP15/blob/master/P5/imagenes/manual-slave.png?raw=true
[manual-master]: https://github.com/MarFerPra/SWAP15/blob/master/P5/imagenes/manual-master.png?raw=true
[automatico-master-slave]: https://github.com/MarFerPra/SWAP15/blob/master/P5/imagenes/automatico-master-slave.png?raw=true
[automatico-prueba]: https://github.com/MarFerPra/SWAP15/blob/master/P5/imagenes/automatico-prueba.png?raw=true

