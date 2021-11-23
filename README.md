## Codeigniter 4 : Configuración sin Spark

El objetivo de este repositorio es enseñarle como puede configurar **Codeigniter 4**
de manera local, es decir, sin utilizar micro-framework **Spark**. El procedimiento a
mostrar fue realizado en **GNU Linux: Fedora 35** con **PHP 7.4 (Remi Repository)** y 
**Apache Server**.


### Requerimiento

 - Apache Server
 - PHP 7.2+
 - Descargar la última versión de Codeigniter
 - Tener acceso administrativo en su GNU Linux.

### ¿Comó instalar Apache Server y PHP 7.x en Fedora?

Para instalar **Apache Server** y **PHP 7.x** puede seguir el siguiente tutorio [Video](https://youtu.be/vaiDWsTYD3o) 
o las instrucciones de la [Wiki](https://github.com/arteaprogramar/Linux-Installations/wiki/Fedora-%7C-Apache-Server-y-PHP-7.x).

### ¿Donde descargar la última versión de Codeigniter4?

[Para descargar la última versión de Codeigniter](https://codeigniter.com/download)

### Configuración

Instrucciones

- Iniciar como administrador y comenzar Apache Server

```sh
    $ su
    $ systemctl start httpd
    
    ##  Obtener estado del servicio
    $ systemctl status httpd
    
    ● httpd.service - The Apache HTTP Server
         Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
        Drop-In: /usr/lib/systemd/system/httpd.service.d
                 └─php74-php-fpm.conf
         Active: active (running) since Mon 2021-11-22 19:43:20 CST; 1h 15min ago
           Docs: man:httpd.service(8)
       Main PID: 16225 (httpd)
         Status: "Total requests: 116; Idle/Busy workers 100/0;Requests/sec: 0.0255; Bytes served/sec:  52 B/sec"
          Tasks: 230 (limit: 14133)
         Memory: 19.6M
            CPU: 5.477s
         CGroup: /system.slice/httpd.service
                 ├─16225 /usr/sbin/httpd -DFOREGROUND
                 ├─16231 /usr/sbin/httpd -DFOREGROUND
                 ├─16233 /usr/sbin/httpd -DFOREGROUND
                 ├─16234 /usr/sbin/httpd -DFOREGROUND
                 ├─16235 /usr/sbin/httpd -DFOREGROUND
                 └─16481 /usr/sbin/httpd -DFOREGROUND
    
    nov 22 19:43:20 fedora systemd[1]: Starting The Apache HTTP Server...
    nov 22 19:43:20 fedora httpd[16225]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::410b:540c:5648:72c4%wlp3s0. Set >
    nov 22 19:43:20 fedora httpd[16225]: Server configured, listening on: port 80
    nov 22 19:43:20 fedora systemd[1]: Started The Apache HTTP Server.
```

- Descomprimir el archivo comprimido de Codeigniter y guardar en la siguiente ruta `/var/www/html/`

```sh
    $ mv codeigniter4-CodeIgniter4-8c7f701/ /var/www/html/Codeigniter4  
```

- Obtener el Status de SELinux (Importante en distribuciones RedHat Family)

```sh
    $ sestatus
    
    SELinux status:                 enabled
    SELinuxfs mount:                /sys/fs/selinux
    SELinux root directory:         /etc/selinux
    Loaded policy name:             targeted
    Current mode:                   enforcing
    Mode from config file:          enforcing
    Policy MLS status:              enabled
    Policy deny_unknown status:     allowed
    Memory protection checking:     actual (secure)
    Max kernel policy version:      33
```

- Si `SELinux status: enabled`  realizar la siguiente configuración para poder ejecutar `Codeigniter4`

```sh
  # Dar todos los permisos a todos los usuarios
  $ chmod 777 -R Codeigniter4/

  # Cambiar el contexto del SELinux a la carpeta de Codeigniter4
  # Objetivo: SELinux debe permitir acceder al contenido del sitio web.
  #   Los archivos son accesibles (solo lectura) y los script ejecutados
  #   https://docs.fedoraproject.org/en-US/Fedora/24/html/SELinux_Users_and_Administrators_Guide/sect-Managing_Confined_Services-The_Apache_HTTP_Server-Types.html
  $ chcon -R -t httpd_sys_content_t /var/www/html/Codeigniter4/
  
  # Cambiar el contexto del SELinux a la carpeta de writable de Codeigniter
  # Objetivo: SELinux permitira leer y escribir archivos
  $ chcon -R -t httpd_sys_rw_content_t Codeigniter4/writable/
```

- Modificar estructura del proyecto de Codeigniter

```sh
    # Estructura original (Antes)
    /app
    /public
          /index.php
          /.htaccess
          /favicon.ico
          /robots.txt
    /system
    /test (Puede eliminar (Opcional))
    /writable
    
    
    # Estructura modificada (Después de sacar index.php y htaccess a raiz)
    /app
    /public
          /favicon.ico
          /robots.txt
    /system
    /test (Puede eliminar (Opcional))
    /writable
    /index.php
    /.htaccess
```

- Modificar el archivo `index.php` que quedo en raiz de la siguiente manera

```php 
    
    <?php

    // Path to the front controller (this file)
    define('FCPATH', __DIR__ . DIRECTORY_SEPARATOR);
    
    /*
     *---------------------------------------------------------------
     * BOOTSTRAP THE APPLICATION
     *---------------------------------------------------------------
     * This process sets up the path constants, loads and registers
     * our autoloader, along with Composer's, loads our constants
     * and fires up an environment-specific bootstrapping.
     */
    
    // Ensure the current directory is pointing to the front controller's directory
    chdir(__DIR__);
    
    // Load our paths config file
    // This is the line that might need to be changed, depending on your folder structure.
    $pathsConfig = FCPATH . 'app/Config/Paths.php';
    // ^^^ Change this if you move your application folder
    require realpath($pathsConfig) ?: $pathsConfig;
    
    $paths = new Config\Paths();
    
    // Location of the framework bootstrap file.
    $bootstrap = rtrim($paths->systemDirectory, '\\/ ') . DIRECTORY_SEPARATOR . 'bootstrap.php';
    $app       = require realpath($bootstrap) ?: $bootstrap;
    
    /*
     *---------------------------------------------------------------
     * LAUNCH THE APPLICATION
     *---------------------------------------------------------------
     * Now that everything is setup, it's time to actually fire
     * up the engines and make this app do its thang.
     */
    $app->run();

```

- Modificar archivo `env` que se encuentra en raíz

```sh
    # Cambiar nombre del archivo env a .env y descomentar la siguiente linea
    app.baseURL = 'http://127.0.0.1/Codeigniter4'
    # La URL será según el nombre del proyecto
    
```

- Ha terminado de configurar Codeigniter 4.