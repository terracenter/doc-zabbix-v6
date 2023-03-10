# Capítulo 1: Instalando Zabbix y Empezando a Usar el Frontend

Zabbix 6 es como una continuación de Zabbix 5, ya que esta vez no estamos viendo grandes cambios en la interfaz de usuario. Sin embargo, en Zabbix 6, usted todavía encontrará un montón de mejoras, tanto en la interfaz de usuario y componentes básicos. Por ejemplo, la introducción de alta disponibilidad para el servidor Zabbix. Detallaremos todos los cambios importantes a lo largo del libro.

En este capítulo, vamos a instalar el servidor Zabbix y explorar la interfaz de usuario Zabbix para que se familiarice con ella. Vamos a ir sobre la búsqueda de sus hosts, triggers, dashboards, y más para asegurarse de que se sienta seguro de sumergirse en el material más profundo más adelante en este libro. La interfaz de usuario de Zabbix tiene un montón de opciones para explorar, así que si estás empezando, no te sientas abrumado. Está bastante estructurada y una vez que le cojas el truco, estoy seguro de que encontrarás tu camino sin problemas. Aprenderás todo sobre estos temas en las siguientes recetas:

* Instalación del servidor Zabbix
* Configuración del frontend Zabbix
* Habilitar la alta disponibilidad del servidor Zabbix
* Uso del frontend Zabbix
* Navegación por el frontend Zabbix

## Requisitos técnicos

Empezaremos este capítulo con una máquina (virtual) Linux vacía. Siéntete libre de elegir una distribución Linux basada en RHEL o Debian. A continuación, vamos a configurar un servidor Zabbix desde cero en este host.

Así que antes de empezar, asegúrate de tener tu máquina Linux lista.

## Instalación del servidor Zabbix

Antes de hacer nada dentro de Zabbix, necesitamos instalarlo y prepararnos para empezar a trabajar con él. En esta receta, vamos a descubrir cómo instalar el servidor Zabbix 6.

### Preparándonos

Antes de instalar el servidor Zabbix, vamos a necesitar cumplir con algunos requisitos previos. Vamos a utilizar MariaDB principalmente a lo largo de este libro. MariaDB es popular y hay mucha información disponible sobre su uso con Zabbix.

En este punto, deberías tener un servidor Linux preparado delante de ti ejecutando una distribución basada en RHEL o Debian. Yo instalaré CentOS y Ubuntu 20.04 en mi servidor; vamos a llamarlos `lar-book-centos` y `lar-book-ubuntu`.

Cuando tengas tu servidor listo, podemos empezar el proceso de instalación.

### Cómo hacerlo...

1. Comencemos añadiendo el repositorio Zabbix 6.0 a nuestro sistema.
   Para sistemas basados en RHEL:

   ```bash
   rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-6.0-1.el8.noarch.rpm
   dnf clean all
   ```

   Para sistemas Ubuntu:

   ```bash
   wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb
   dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb
   apt update
   ```

   Para Debian:

   ```bash
   wget https://repo.zabbix.com/zabbix/6.0/debian/pool/main/z/zabbix-release/zabbix-release_6.0-4%2Bdebian11_all.deb
   dpkg -i zabbix-release_6.0-4+debian11_all.deb
   apt update
   ```
2. Ahora que el repositorio está añadido, vamos a añadir el repositorio MariaDB en nuestro servidor:

   Según este libro, pero instala una versión no soportada por Zabbix, a la fecha

   ```bash
   wget https://downloads.mariadb.com/MariaDB/mariadb_repo_setup
   chmod +x mariadb_repo_setup
   ./mariadb_repo_setup
   ```

Nota:
Must not be higher than (10.10.xx). Use of supported database version is highly recommended
Override by setting AllowUnsupportedDBVersions=1 in Zabbix server configuration file at your own risk.

Para Debian 11 (estoy usando), como dice la documentacion https://mariadb.com/kb/en/mariadb-package-repository-setup-and-usage/

```bash
curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | bash -s -- --mariadb-server-version="mariadb-10.10" --os-type=debian  --os-version=11
```

3. A continuación, instálelo y habilítelo mediante los siguientes comandos:

   ```bash
   dnf install mariadb-server
   systemctl enable mariadb
   systemctl start mariadb
   ```

   Para sistemas Ubuntu/Debian:

   ```bash
   apt install -y zabbix-server-mysql zabbix-sql-scripts
   ```

   ```bash
   apt install -y mariadb-server
   systemctl enable mariadb
   systemctl start mariadb
   ```
4. Después de instalar MariaDB, asegúrese de proteger su instalación con el siguiente comando:

   ```bash
   /usr/bin/mariadb-secure-installation
   ```
5. Asegúrate de responder a las preguntas con un sí (Y) y configura una contraseña de root que sea segura.
6. Ejecute la configuración de instalación segura y asegúrese de guardar su contraseña en algún lugar. Es muy recomendable utilizar una bóveda de contraseñas.
7. Ahora, vamos a instalar nuestro servidor Zabbix con soporte MySQL.

   Para sistemas basados en RHEL:

   ```bash
   dnf install zabbix-server-mysql zabbix-sql-scripts
   ```

   Para sistemas basados en Debian / Ubuntu:

   ```bash
   apt install zabbix-server-mysql zabbix-sql-scripts
   ```
8. Con el servidor Zabbix instalado, estamos listos para crear nuestra base de datos Zabbix. Inicie sesión en MariaDB con lo siguiente:

   ```bash
   mysql -u root -p
   ```
9. Introduzca la contraseña que estableció durante la instalación segura y cree la base de datos Zabbix con los siguientes comandos. No olvide cambiar la `contraseña` en el segundo comando:

   ```bash
   create database zabbix character set utf8mb4 collate utf8mb4_bin;
   create user zabbix@localhost identified by 'password';
   grant all privileges on zabbix.* to zabbix@localhost identified by 'password';
   flush privileges;
   quit
   ```

   **Consejo**

   Para aquellos que lo necesiten, Zabbix también soporta `utf8mb4` ahora. Hemos cambiado `utf8` por `utf8mb4` en el comando anterior y todo funcionará. Para una referencia, consulte el ticket de soporte de Zabbix aquí: https://support.zabbix.com/browse/ZBXNEXT-3706
10. Ahora necesitamos importar nuestro esquema de base de datos Zabbix a nuestra recién creada base de datos Zabbix:

    Sistemas (Ubuntu y RedHad) ??

    ```bash
    zcat /usr/share/doc/zabbix-sql-scripts/mysql/server.sql.gz | mysql -u zabbix -p zabbix
    ```

    Debian

    ```bash
    zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -u zabbix -p zabbix
    ```

    **Nota importante**

    En este punto, puede parecer que está atascado y que el sistema no responde. No se preocupe, ya que sólo tardará un poco en importar el esquema SQL.

Ahora hemos terminado con los preparativos para nuestro lado MariaDB y estamos listos para pasar al siguiente paso, que será la configuración del servidor Zabbix:

1. El servidor Zabbix se configura utilizando el archivo de configuración del servidor Zabbix. Este archivo se encuentra en `/etc/zabbix/`. Vamos a abrir este archivo con nuestro editor favorito; Voy a utilizar **Vim** en todo el libro:

   ```bash
   vim /etc/zabbix/zabbix_server.conf
   ```
2. Ahora, asegúrese de que las siguientes líneas del archivo coinciden con el nombre de su base de datos, el nombre de usuario de la base de datos y la contraseña del usuario de la base de datos:

   ```bash
   DBName=zabbix
   DBUser=zabbix
   DBPassword=password
   ```

   **Consejo**

   Antes de iniciar el servidor Zabbix, debe configurar SELinux o AppArmor para permitir el uso del servidor Zabbix. Si se trata de una máquina de prueba, puede utilizar una postura permisiva para SELinux o desactivar AppArmor, pero se recomienda no hacer esto en producción.
3. Todo hecho; ahora estamos listos para iniciar nuestro servidor Zabbix:

   ```bash
   systemctl enable zabbix-server
   systemctl start zabbix-server
   ```
4. Comprueba si todo arranca como se espera con lo siguiente:

   ```bash
   systemctl status zabbix-server 
   ```
5. Alternativamente, supervise el archivo de registro, que proporciona una descripción detallada del proceso de inicio de Zabbix:

   ```bash
   tail -f /var/log/zabbix/zabbix_server.log 
   ```
6. La mayoría de los mensajes en este archivo están bien y pueden ser ignorados con seguridad, pero asegúrese de leer bien y ver si hay algún problema con el arranque de su servidor Zabbix.

### Cómo funciona...

El servidor Zabbix es el proceso principal de nuestra configuración Zabbix. Es responsable de nuestra monitorización, alertas de problemas, y muchas de las otras tareas descritas en este libro. Una pila completa de Zabbix consiste en al menos lo siguiente:

* Una base de datos (MySQL, PostgreSQL u Oracle)
* Un servidor Zabbix
* Apache o NGINX ejecutando el frontend Zabbix con PHP 7.2+, pero PHP 8 no está soportado actualmente

  **Nota**:

  A la fecha funciona con 7.2.5 o superior, 8.0 - 8.2, según documentación: https://www.zabbix.com/documentation/6.0/en/manual/installation/requirements

Podemos ver los componentes y cómo se comunican entre sí en la siguiente figura:

![Figura 1.1 - Diagrama de comunicaciones de la configuración de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_001.jpg)

Figura 1.1 - Diagrama de comunicaciones de la configuración de Zabbix

Acabamos de configurar el servidor Zabbix y la base de datos; ejecutando estos dos, estamos básicamente listos para empezar a monitorizar. El servidor Zabbix se comunica con la base de datos Zabbix para escribir en ella los valores recogidos.

Sin embargo, todavía hay un problema: no podemos configurar nuestro servidor Zabbix para hacer nada. Para ello, vamos a necesitar nuestro frontend Zabbix, que vamos a configurar en la siguiente receta.

## Configuración del frontend Zabbix

El frontend de Zabbix es la cara de nuestro servidor. Es donde vamos a configurar todos nuestros hosts, plantillas, cuadros de mando, mapas, y todo lo demás. Sin él, estaríamos ciegos a lo que está pasando, en el lado del servidor. Por lo tanto, vamos a configurar nuestro frontend Zabbix en esta receta.

### Preparándonos

Vamos a configurar el frontend de Zabbix usando Apache. Antes de comenzar con esta receta, asegúrese de que está ejecutando el servidor Zabbix en una distribución de Linux de su elección. Usaré los hosts `lar-book-centos` y `lar-book-ubuntu` en estas recetas para mostrar el proceso de configuración en CentOS 8 y Ubuntu 20.

### Cómo hacerlo...

1. Vamos a instalar el frontend. Ejecute el siguiente comando para empezar.
   Para sistemas basados en RHEL:

   ```bash
   dnf install zabbix-web-mysql zabbix-apache-conf 
   ```

   Para sistemas Debian / Ubuntu:

   **Nota**: Si deseas utilizar PHP 8.2 en Debian 11

   * Instale los paquetes temporales necesarios:

   ```bash
    apt update
    apt install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2
   ```

   * Añada el repositorio Surý Debian PPA a su sistema Debian.

     ```bash
     echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/sury-php.list
     ```
   * Importar paquetes firmando clave GPG;

     ```bash
     curl -fsSL  https://packages.sury.org/php/apt.gpg|  gpg --dearmor -o /etc/apt/trusted.gpg.d/sury-keyring.gpg
     ```
   * Confirme si el repositorio funciona descargando la información de los paquetes de todas las fuentes configuradas:

     ```bash
      apt update
     ```

   ```bash
    apt install zabbix-frontend-php zabbix-apache-conf 
   ```

   **Consejo**

   No olvide permitir los puertos 80 y 443 en su cortafuegos si está utilizando uno. Sin esto, no podrás conectarte al frontend.
2. Reinicie los componentes de Zabbix y asegúrese de que se inician al arrancar el servidor con lo siguiente.

   Para sistemas basados en RHEL:

   ```bash
   systemctl enable httpd php-fpm
   systemctl restart zabbix-server httpd php-fpm
   ```

   Para sistemas Ubuntu:

   ```bash
   systemctl enable apache2
   systemctl restart zabbix-server apache2
   ```
3. Ahora deberíamos ser capaces de navegar a nuestro frontend Zabbix sin ningún problema y comenzar los pasos finales para configurar el frontend Zabbix.
4. Vayamos a nuestro navegador y naveguemos hasta la IP de nuestro servidor. Debería verse así:

   ```bash
   http://<your_server_ip>/zabbix 
   ```
5. Ahora deberíamos ver la siguiente página web:
   ![Figura 1.2 - Pantalla de bienvenida de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_002.jpg)
   Figura 1.2 - Pantalla de bienvenida de Zabbix

   Si no ves esta página web, es posible que hayas omitido algunos pasos en el proceso de configuración. Vuelva sobre sus pasos y vuelva a comprobar sus archivos de configuración; incluso el más pequeño error tipográfico podría impedir que la página web se sirva.
6. Continuemos haciendo clic en **Siguiente paso** en esta página, que le servirá con la siguiente página:
   ![Figura 1.3 - Página de requisitos previos para la instalación de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_003.jpg)

   Figura 1.3 - Página de requisitos previos para la instalación de Zabbix
7. Todas y cada una de las opciones deberían aparecer **OK** ahora; si no es así, corrija el error que le está mostrando. Si todo está bien, puede continuar haciendo clic en Siguiente paso de nuevo, que le llevará a la página siguiente:
   ![Figura 1.4 - Página de conexión a la base de datos de instalación de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_004.jpg)

   Figura 1.4 - Página de conexión a la base de datos de instalación de Zabbix
8. Aquí, tenemos que decirle a nuestro frontend Zabbix donde se encuentra nuestra base de datos MySQL. Desde que la instalamos en localhost, sólo tenemos que asegurarnos de que publicamos el nombre correcto de la base de datos, el nombre de usuario de la base de datos, y la contraseña del usuario de la base de datos.
9. Esto debería hacer que el frontend Zabbix pueda comunicarse con la base de datos. Vamos a proceder haciendo clic en Siguiente paso de nuevo:
   ![Figura 1.5 - Página de detalles del servidor de instalación de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_005.jpg)

   Figura 1.5 - Página de detalles del servidor de instalación de Zabbix

   Lo siguiente es la configuración del servidor Zabbix. Asegúrese de nombrar a su servidor algo útil o algo fresco. Por ejemplo, he configurado un servidor de producción llamado Meeseeks porque cada vez que recibíamos una alerta, podíamos hacer que Zabbix dijera "Soy el Sr. Meeseeks mírame". Pero algo como zabbix.ejemplo.com también funciona.
10. Pongamos un nombre a nuestro servidor, configuremos la zona horaria para que coincida con la nuestra y pasemos al siguiente paso:
    ![Figura 1.6 - Página de resumen de la instalación de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_006.jpg)

    Figura 1.6 - Página de resumen de la instalación de Zabbix
11. Verifique su configuración y proceda a hacer clic en Siguiente paso una vez más.
    ![Figura 1.7 - Página de finalización de la instalación de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_007.jpg)

    Figura 1.7 - Página de finalización de la instalación de Zabbix
12. Ha instalado correctamente el frontend Zabbix. Ahora puede hacer clic en el botón Finalizar y podemos empezar a utilizar el frontend. Se le mostrará una página de inicio de sesión donde puede utilizar las siguientes credenciales por defecto:

    ```bash
    Username: Admin
    Password: zabbix
    ```

### Cómo funciona...

Ahora que hemos instalado nuestro frontend Zabbix, nuestra configuración de Zabbix está completa y estamos listos para empezar a trabajar con ella. Nuestro frontend Zabbix se conectará a nuestra base de datos para editar los valores de configuración de nuestra configuración, como podemos ver en la siguiente figura:
![Figura 1.8 - Diagrama de comunicaciones de configuración de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_008.jpg)

Figura 1.8 - Diagrama de comunicaciones de configuración de Zabbix

El frontend Zabbix también hablará con nuestro servidor Zabbix, pero esto es sólo para asegurarse de que el servidor Zabbix está en funcionamiento. Ahora que sabemos cómo configurar el frontend Zabbix, podemos empezar a usarlo. Vamos a comprobarlo después de la siguiente receta.

### Hay más...

Zabbix proporciona una guía de configuración muy práctica, que contiene muchos detalles sobre la instalación de Zabbix. Yo siempre recomendaría mantener esta página abierta durante una instalación de Zabbix, ya que contiene información como el enlace al último repositorio. Compruébalo aquí:

https://www.zabbix.com/download

---

## Habilitar la alta disponibilidad del servidor Zabbix

Zabbix 6 está aquí, con una de las características más esperadas de todos los tiempos. La alta disponibilidad llevará su configuración Zabbix al siguiente nivel, asegurándose de que si uno de sus servidores Zabbix tiene problemas, otro se hará cargo.

Una gran cosa acerca de esta implementación es que soporta una forma propietaria fácil de poner de uno a muchos servidores Zabbix en un clúster. Una gran manera de asegurarse de que su monitorización se mantiene en el aire en todo momento (o al menos tanto como sea posible).

Ahora tengo que ser honesto, no podemos hacer nada como el equilibrio de carga todavía. Pero eso está incluido en la hoja de ruta de Zabbix utilizando los proxies Zabbix en una versión posterior. Mantenga un ojo hacia fuera para cualquier actualización con respecto a que aquí:

https://www.zabbix.com/roadmap

### Preparación

Antes de empezar, ten en cuenta que la creación de una configuración de alta disponibilidad se considera un tema avanzado. Puede ser más difícil que otras recetas de este capítulo.

Para esta configuración, necesitaremos tres nuevas máquinas virtuales, ya que vamos a crear una configuración de Zabbix dividida, a diferencia de la configuración que creamos en la primera receta de este capítulo. Echemos un vistazo a cómo he nombrado a nuestras tres nuevas máquinas virtuales y cuáles serán sus direcciones IP:

* `lar-book-ha1` (`192.168.0.1`)
* `lar-book-ha2` (`192.168.0.2`)
* `lar-book-ha-db` (`192.168.0.10`)

Dos de estos servidores ejecutarán nuestro clúster de servidores Zabbix y un frontend Zabbix. El otro servidor es sólo para nuestra base de datos MySQL. Tenga en cuenta que las direcciones IP utilizadas en el ejemplo pueden ser diferentes para usted. Utilice las correctas para su entorno.

También necesitaremos una dirección IP virtual para nuestros nodos de cluster. Usaremos `192.168.0.5` en el ejemplo.

**Consejo**

En nuestra configuración, estamos utilizando sólo una base de datos MySQL Zabbix. Para asegurarse de que todas las partes de Zabbix están configuradas como altamente disponibles, podría valer la pena considerar la configuración de MySQL en una configuración maestro/maestro. Esto sería una gran combinación con la alta disponibilidad del servidor Zabbix.

Este libro de cocina NO utilizará SELinux o AppArmor, así que asegúrese de desactivarlas o añadir las políticas correctas antes de utilizar esta guía. Además, esta guía tampoco detalla cómo configurar su firewall, así que asegúrese de hacer esto de antemano también.

### Cómo hacerlo...

Para su comodidad, hemos dividido esta sección de Cómo hacerlo... en tres partes. Una es la configuración de la base de datos, la otra es la configuración del cluster de servidores Zabbix, y la última es cómo configurar el frontend Zabbix de forma redundante. La sección Cómo funciona... proporcionará una explicación sobre toda la configuración.

#### Configuración de la base de datos

Vamos a empezar con la configuración de nuestra base de datos Zabbix, lista para ser utilizada en una configuración de servidor Zabbix de alta disponibilidad:

1. Acceda a `lar-book-ha-db` e instale el repositorio MariaDB con el siguiente comando en sistemas basados en RedHat

   ```bash
   wget https://downloads.mariadb.com/MariaDB/mariadb_repo_setup
   chmod +x mariadb_repo_setup
   ./mariadb_repo_setup
   ```
2. A continuación, vamos a instalar la aplicación de servidor MariaDB con el siguiente comando.

   Para sistemas basados en RHEL:

   ```bash
   dnf install mariadb-server
   systemctl enable mariadb
   systemctl start mariadb
   ```

   Para sistemas Ubuntu:

   ```bash
   apt install mariadb-server
   systemctl enable mariadb
   systemctl start mariadb
   ```
3. Después de instalar MariaDB, asegúrese de asegurar su instalación con el siguiente comando:

   ```bash
   /usr/bin/mariadb_secure_installation
   ```
4. Asegúrate de responder a las preguntas con un sí (Y) y configura una contraseña raíz que sea segura. Es muy recomendable utilizar una bóveda de contraseñas para almacenarla.
5. Ahora vamos a crear nuestra base de datos Zabbix para que nuestros servidores Zabbix se conecten. Inicie sesión en MariaDB con el siguiente comando:

   ```bash
   mysql -u root -p
   ```
6. Introduzca la contraseña que estableció durante la instalación segura y cree la base de datos Zabbix con los siguientes comandos. No olvide cambiar la contraseña en los comandos segundo, tercero y cuarto:

   ```bash
   create database zabbix character set utf8mb4 collate utf8mb4_bin;
   create user zabbix@'192.168.0.1' identified by 'password';
   create user zabbix@'192.168.0.2' identified by 'password';
   create user zabbix@'192.168.0.5' identified by 'password';
   grant all privileges on zabbix.* to 'zabbix'@'192.168.0.1' identified by 'password';
   grant all privileges on zabbix.* to 'zabbix'@'192.168.0.2' identified by 'password';
   grant all privileges on zabbix.* to 'zabbix'@'192.168.0.5' identified by 'password';
   flush privileges;
   quit
   ```
7. Por último, necesitamos importar la configuración inicial de la base de datos Zabbix, pero para ello, necesitamos instalar el repositorio Zabbix.

   Para sistemas basados en RHEL:

   ```bash
   rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-6.0-1.el8.noarch.rpm
   dnf clean all
   ```
8. A continuación, tenemos que instalar el módulo SQL scripts Zabbix.

   Para los sistemas basados en RHEL:

   ```bash
   dnf install zabbix-sql-scripts
   ```

   Para sistemas Ubuntu:

   ```bash
   apt install zabbix-sql-scripts
   ```
9. A continuación, ejecutamos el siguiente comando, que puede tardar un poco, así que ten paciencia hasta que termine:

   ```bash
   zcat /usr/share/doc/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uroot -p zabbix
   ```

#### Configuración de los nodos del cluster del servidor Zabbix

La configuración de los nodos del cluster funciona de la misma manera que la configuración de cualquier nuevo servidor Zabbix. La única diferencia es que tendremos que especificar algunos parámetros nuevos de configuración.

Para sistemas Ubuntu, utilice el siguiente comando:

1. Empecemos añadiendo el repositorio Zabbix 6.0 a nuestros sistemas `lar-book-ha1` y `lar-book-ha2`:

   ```bash
   rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-6.0-1.el8.noarch.rpm
   dnf clean all
   ```

Para sistemas Ubuntu, utilice el siguiente comando:

```bash
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb
dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb
apt update
```

2. Ahora vamos a instalar la aplicación de servidor Zabbix con el siguiente comando.

Para sistemas basados en RHEL:

```bash
dnf install zabbix-server-mysql
```

Para sistemas Ubuntu:

```bash
apt install zabbix-server-mysql 
```

3. Ahora editaremos los archivos de configuración del servidor Zabbix, empezando por `lar-book-ha1`. Emita el siguiente comando:

   ```bash
   vim /etc/zabbix/zabbix_server.conf
   ```
4. A continuación, añada las siguientes líneas para permitir una conexión a la base de datos:

   ```bash
   DBHost=192.168.0.10
   DBPassword=password
   ```
5. Para habilitar la alta disponibilidad en este host, añada las siguientes líneas en el mismo archivo:

   ```bash
   HANodeName=lar-book-ha1
   ```
6. Para asegurarnos de que nuestro frontend Zabbix sabe dónde conectarse si hay un fallo de nodo, rellene lo siguiente:

   ```bash
   NodeAddress=192.168.0.1
   ```
7. Guarda el archivo y hagamos lo mismo para nuestro host `lar-book-ha2` editando su archivo:

   ```bash
   vim /etc/zabbix/zabbix_server.conf
   ```
8. A continuación, añada las siguientes líneas para permitir una conexión a la base de datos:

   ```bash
   DBHost=192.168.0.10
   DBPassword=password
   ```
9. Para habilitar la alta disponibilidad en este host, añada las siguientes líneas en el mismo archivo:

   ```bash
   HANodeName=lar-book-ha2
   ```
10. Para asegurarnos de que nuestro frontend Zabbix sabe dónde conectarse si hay un fallo de nodo, rellene lo siguiente:

    ```bash
    systemctl enable zabbix-server
    systemctl start zabbix-server
    ```

#### Configurando Apache con alta disponibilidad

Para asegurarnos de que nuestro frontend también está configurado de tal manera que si un servidor Zabbix tiene problemas falla, vamos a configurarlos con keepalived. Veamos cómo hacerlo.

1. Comencemos iniciando sesión en `lar-book-ha1` y `lar-book-ha2` e instalando keepalived.
   Para sistemas basados en RHEL:

   ```bash
   dnf install -y keepalived
   ```

   Para sistemas Ubuntu:

   ```bash
   apt install keepalived
   ```
2. Luego, en lar-book-ha1, edita la configuración keepalived con el siguiente comando:

   ```bash
    vim /etc/keepalived/keepalived.conf
   ```
3. Borre todo de este archivo y añada el siguiente texto al archivo:

   ```bash
   vrrp_track_process chk_apache_httpd {
      process httpd
      weight 10
      }

   vrrp_instance ZBX_1 {
      state MASTER
      interface ens192
      virtual_router_id 51
      priority 244
      advert_int 1

      authentication {
         auth_type PASS
         auth_pass password
      }

      track_process {
         chk_apache_httpd
      }

      virtual_ipaddress {
         192.168.0.5/24
         }
   }

   ```
4. No olvides actualizar la `contraseña `a algo seguro y editar la interfaz `ens192` a tu propio nombre/número de interfaz. Para Ubuntu, cambie `httpd` por `apache2`.
   Nota Importante

   En el archivo anterior, hemos especificado `virtual_router_id 51`. Asegúrese de que el ID de enrutador virtual 51 no se utiliza en cualquier lugar de la red todavía. Si lo es, simplemente cambie el ID del enrutador virtual a lo largo de esta receta.
5. En `lar-book-ha2`, edita el mismo archivo con el siguiente comando:

   ```bash
   vim /etc/keepalived/keepalived.conf
   ```
6. Borre todo del archivo con dG y esta vez añadiremos la siguiente información:

   ```bash
   vrrp_track_process chk_apache_httpd {
         process httpd
         weight 10
   }
   vrrp_instance ZBX_1 {
           state BACKUP
           interface ens192
           virtual_router_id 51
           priority 243
           advert_int 1
           authentication {
                   auth_type PASS
                   auth_pass password
           }
           track_process {
                   chk_apache_httpd
           }
           virtual_ipaddress {
                   192.168.0.5/24
           }
   }
   ```
7. Una vez más, no olvide actualizar la `password` a algo seguro y editar la interfaz `ens192` a su propio nombre/número de interfaz. Para Ubuntu, cambia `httpd` por `apache2`.
8. Ahora vamos a instalar el frontend Zabbix con el siguiente comando.

   Para sistemas basados en RHEL:

   ```bash
   dnf install httpd zabbix-web-mysql zabbix-apache-conf
   ```

   Para sistemas Ubuntu:

   ```bash
   apt install apache2 zabbix-frontend-php zabbix-apache-conf
   ```
9. Inicie el servidor web y keepalived para que su frontend Zabbix esté disponible con el siguiente comando:

   ```bash
   systemctl enable httpd keepalived
   systemctl start httpd keepalived
   ```
10. A continuación, estamos listos para configurar nuestro frontend Zabbix. Navega a tu dirección IP virtual (en el caso de la IP de ejemplo, `http://192.168.0.5/zabbix`) y verás la siguiente página:

    ![Figura 1.9 - Ventana de configuración inicial de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_009.jpg)

Figura 1.9 - Ventana de configuración inicial de Zabbix

11. Haga clic en Siguiente paso dos veces hasta que vea la siguiente página:
    ![Figura 1.10 - Ventana de configuración de la base de datos Zabbix para lar-book-ha1](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_010.jpg)

Figura 1.10 - Ventana de configuración de la base de datos Zabbix para lar-book-ha1

12. Asegúrese de rellenar el host de la base de datos con la dirección IP de nuestra base de datos Zabbix MariaDB (`192.168.0.10`). A continuación, rellene la contraseña de base de datos para nuestro usuario de base de datos zabbix.
13. Entonces, para el último paso, para nuestro primer nodo, configure el nombre del servidor Zabbix como `lar-book-ha1` y seleccione su zona horaria como se ve en la siguiente captura de pantalla.
    ![Figura 1.11 - Ventana de configuración del servidor Zabbix para lar-book-ha1](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_011.jpg)

Figura 1.11 - Ventana de configuración del servidor Zabbix para lar-book-ha1

14. A continuación, haga clic en **Next step** y **Finish**.
15. Ahora tenemos que hacer lo mismo con nuestro segundo frontend. Inicie sesión en `lar-book-ha1` y haga lo siguiente.

En sistemas basados en RHEL:

```bash
systemctl stop httpd
```

Para sistemas Ubuntu:

```bash
systemctl stop apache2
```

16. Cuando navegues a tu IP virtual (en el caso de la IP de ejemplo, `http://192.168.0.5/zabbix`), volverás a ver el mismo asistente de configuración.
17. Rellena de nuevo los datos de la base de datos:
    ![Figura 1.12 - Ventana de configuración de la base de datos Zabbix para lar-book-ha2](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_012.jpg)

    Figura 1.12 - Ventana de configuración de la base de datos Zabbix para lar-book-ha2
18. A continuación, asegúrese de configurar el nombre del servidor Zabbix como `lar-book-ha2` como se ve en la captura de pantalla.
    ![Figura 1.13 - Ventana de configuración del servidor Zabbix para lar-book-ha2](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_013.jpg)

    Figura 1.13 - Ventana de configuración del servidor Zabbix para lar-book-ha2
19. Ahora tenemos que volver a habilitar el frontend lar-book-ha1 haciendo lo siguiente.
    En sistemas basados en RHEL:

    ```bash
    systemctl start httpd
    ```

    Para sistemas Ubuntu:

    ```bash
    systemctl start apache2
    ```

Ese debería ser nuestro último paso. Ahora todo debería estar funcionando como se esperaba. Asegúrese de comprobar el archivo de registro del servidor Zabbix para ver si los nodos de HA están funcionando como se esperaba.

### Cómo funciona...

Ahora que lo hemos hecho, ¿cómo funciona realmente el servidor Zabbix en modo de alta disponibilidad? Empecemos por comprobar la página Informes | Información del sistema en nuestro frontend Zabbix.

![Figura 1.14 - Información del sistema del servidor Zabbix con información de HA](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_014.jpg)

Figura 1.14 - Información del sistema del servidor Zabbix con información de HA

Ahora podemos ver que tenemos nueva información disponible. Por ejemplo, el parámetro Cluster de alta disponibilidad. Este parámetro nos dice ahora si la alta disponibilidad está habilitada o no y cuál es el retardo de conmutación por error. En nuestro caso, es de 1 minuto, lo que significa que puede pasar hasta 1 minuto antes de que se inicie la conmutación por error.

Además, podemos ver cada nodo de nuestro cluster. Como Zabbix ahora soporta de uno a muchos nodos en un cluster, podemos ver cada uno de los que participan en nuestro cluster aquí mismo. Echemos un vistazo a la configuración que hemos construido:

![Figura 1.15 - Configuración de HA del servidor Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_01_015.jpg)

Figura 1.15 - Configuración de HA del servidor Zabbix

Como se puede ver en la configuración, hemos conectado nuestros dos nodos de servidor Zabbix lar-book-ha1 y lar-book-ha2 a nuestra única base de datos Zabbix lar-book-ha-db. Debido a que nuestra base de datos Zabbix es nuestra única fuente de verdad, se puede utilizar para mantener la configuración de nuestro clúster también. Al final, todo lo que hace Zabbix se mantiene siempre en la base de datos, desde la configuración del host a los datos de la historia a la información de alta disponibilidad. Es por eso que la construcción de un clúster Zabbix es tan simple como poner el `HANodeName`en el archivo de configuración del servidor Zabbix.

También incluimos el parámetro `NodeAddress` en el archivo de configuración. Este parámetro es utilizado por el frontend Zabbix para asegurarse de que nuestra información del sistema (widget) y el servidor Zabbix no están ejecutando el trabajo de notificación frontend. El parámetro `NodeAddress` le dirá al frontend a qué dirección IP conectarse para cada servidor respectivo una vez que se convierta en el servidor Zabbix activo.

Para llevar las cosas un poco más lejos, he añadido una simple configuración keepalived a esta instalación también. Keepalived es una forma de construir configuraciones simples VRRP fail over entre servidores Linux. En nuestro caso, hemos introducido el VIP como `192.168.0.5` y hemos añadido la monitorización del proceso `chk_apache_httpd` para determinar cuándo se produce la conmutación por error. Nuestra conmutación por error funciona de la siguiente manera:

```bash
lar-book-ha1 has priority 244
lar-book-ha2 has priority 243
```

Si **HTTPd** o **Apache 2** se está ejecutando en nuestro nodo, eso añade un peso de 10 a nuestra prioridad, lo que lleva a la prioridad total de `254` y `253`, respectivamente. Ahora imaginemos que `lar-book-ha1` ya no tiene el proceso del servidor web en ejecución. Eso significa que su prioridad cae a `244`, que es inferior a `253` en `lar-book-ha2`, que tiene el proceso de servidor web en ejecución.

Cualquier host que tenga la prioridad más alta es el host que tendrá el VIP `192.168.0.5`, lo que significa que ese host está ejecutando el frontend de Zabbix que será servido.

Combinando estas dos formas de configurar la alta disponibilidad, acabamos de crear redundancia para dos de las partes que componen nuestra configuración de Zabbix, asegurándonos de que podemos mantener las interrupciones al mínimo.

### Hay más...

Ahora te preguntarás, ¿qué pasa si quiero ir más allá en términos de configuración de alta disponibilidad? En primer lugar, la característica de alta disponibilidad de Zabbix está construida para ser simple y comprensible para toda la base de usuarios de Zabbix. Lo que significa que a partir de ahora, es posible que no vea la misma cantidad de características que obtendría con una implementación de terceros.

Sin embargo, la nueva característica de alta disponibilidad del servidor Zabbix ha demostrado ser una característica muy esperada que realmente añade algo a la mesa. Si desea ejecutar una configuración de alta disponibilidad como esta, la mejor manera de añadir un nivel más de complejidad a la alta disponibilidad es una configuración maestro/maestro MySQL. Configurar la base de datos Zabbix con alta disponibilidad, que es la principal fuente de verdad, se asegurará de que su configuración Zabbix realmente es fiable en tantas formas como sea posible. Para obtener más información acerca de la replicación MariaDB, consulte la documentación aquí: https://mariadb.com/kb/en/standard-replication/.

---
