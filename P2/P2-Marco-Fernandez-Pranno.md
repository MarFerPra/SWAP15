## Práctica 2 - Servidores Web de Altas Prestaciones (2015)
### *Marco Manuel Fernández Pranno*

**1. Prueba de funcionamiento de la copia de archivos por ssh:**

El envío de un fichero por ssh se realiza mediante la compresión del mismo primero y después con un cauce hacia la ejecución del ssh con un cat como argumento. La orden a introducir en el terminal sería: 

` tar czf - directorio_envio_comprimido | ssh root@192.168.24.129 'cat > ~/tar.tgz' `

Posteriormente introducimos la contraseña del usuario seleccionado para el login, en este caso el root. Y podemos ver como se ha enviado correctamente mediante el listado de elementos en el fichero del otro servidor.

![alt text][envio-por-ssh]

**2. Clonado de una carpeta entre las dos máquinas**

Para el clonado de un fichero entre dos máquinas he utilizado la orden rsync, sincronizando el directorio home de el servidor 1 en el directorio home del el servidor 2. Para hacerlo, he utilizado la siguiente orden lanzada desde el servidor 2:

` rsync -avc -c ssh user01@192.168.24.128:/home/user01/ /home/user02/ `

![alt text][envio-fichero]

**3. Configuración de ssh para acceder sin que solicite contraseña.**

Debido a que me ha sido imposible configurar la conexión sin contraseña por ssh con usuario root, he realizado el ejercicio mediante un usuario normal. Por otro lado, los pasos a seguir en ambos casos han sido los mismos:

  * Generación de claves en ambas máquinas mediante la orden ` ssh-keygen -t dsa ` tal y como se muestra en las siguientes capturas:
  
  Como usuario:
  ![alt text][generacion-claves] 
  Como root:
  ![alt text][generacion-claves-root]

  
  * Posteriormente he llevado a cabo el envío de las claves públicas de una máquina a otra con la orden 
    ` ssh-copy-id -i nombre-de-la-clave.pub nombre-usuario@ip-de-la-maquina-receptora `. Tal y como se refleja en las      capturas:
    
    Como usuario:
    ![alt text][envio-claves]

    Como root:
    ![alt text][envio-claves-root]

 Una vez hecho esto, en cada directorio home de los usuarios (y del root), en .ssh/authorized_keys aparecen reflejadas las claves permitidas para el login sin contraseña.
 Debido a que no podía realizarlo como root, me he asegurado de que el fichero de configuración de ssh en ambos servidores estuviese correctamente, en ambos casos consta "PermitRootLogin yes". Tal y como se refleja en la captura:
 
 ![alt txt][permitir-acceso-root]
 
 

**4. Establecer una tarea en cron que se ejecute cada hora para mantener actualizado el contenido del directorio /var/www entre las dos máquinas.**

Para realizar este ejercicio basta con agregar una linea en el fichero crontab con la sintaxis correcta y la ejecución de una orden o script en concreto.
Para facilitar el manejo de más acciones cada hora he optado por escribir un script y ejecutar el script en lugar de incluir en la sentencia crontab la orden de copia directamente.
La orden es la siguiente:

` rsyn -avz -e ssh usuario@ip-maquina-remota:/var/www/ /var/www `

En el caso de la captura se refleja el usuario root ya que fue el primero con el que probé la ejecución, pero como ya he mencionado anteriormente, es necesario escribir contraseña en cada conexión.
La sentencia agregada a crontab es la siguiente:

` 01 *  * * *   root  /root/www-updater `

En la captura se ve lo descrito en ambos casos:

![alt text][update-www]

[envio-claves]: https://github.com/MarFerPra/SWAP15/blob/master/P2/imagenes/envio-claves.png?raw=true
[envio-claves-root]: https://github.com/MarFerPra/SWAP15/blob/master/P2/imagenes/envio-claves-root.png?raw=true
[envio-por-ssh]: https://github.com/MarFerPra/SWAP15/blob/master/P2/imagenes/envio-por-ssh.png?raw=true
[envio-fichero]: https://github.com/MarFerPra/SWAP15/blob/master/P2/imagenes/envio_fichero.png?raw=true
[generacion-claves]: https://github.com/MarFerPra/SWAP15/blob/master/P2/imagenes/generacion-claves.png?raw=true
[generacion-claves-root]: https://github.com/MarFerPra/SWAP15/blob/master/P2/imagenes/generacion-claves-root.png?raw=true
[permitir-acceso-root]: https://github.com/MarFerPra/SWAP15/blob/master/P2/imagenes/permitir-acceso-root.png?raw=true
[update-www]: https://github.com/MarFerPra/SWAP15/blob/master/P2/imagenes/update-www-de-02.png?raw=true
