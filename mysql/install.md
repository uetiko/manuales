Instalando mysql desde el código fuente
=======================================

## Temario

- Creando grupos y usuarios
- Istalando dependencias (sobre un diestro basado en redhat)
- Compilando mysql
- Configuracion de mysql
- Configuración con alternative
- configuración de supervisor para mysql


## Creando grupos y usuarios

Como una buena practica, se debe crear un [usuario y grupo] para cada servicio que instalemos en el sistema, en este caso
usaremos usuarios de sistema para poder correr los procesos/demonios.

Recordemos que `-r` es equivalente a `--sistem`. No es necesario crear un directorio home en este caso, pero si gustas, puedes crearle uno.

```sh
groupadd -r mysql
useradd --system --gid mysql -s /bin/zsh mysql
```

## Istalando dependencias (sobre un diestro basado en redhat)

Antes de poder continuar, deberas instalar algunas dependencias para que puedas compilar mysql sin problemas, estan son:

Dependencias

:   [bison]
    > Bison es un generador sintáctico de propósito general que convierte una gramática libre de contexto anotada en un LR determinista o un analizador LR generalizado (GLR) que emplea tablas de analizador LALR.

:   [cmake]
    > CMake es una familia de herramientas de código abierto y multiplataforma diseñada para construir, probar y empaquetar software.

:   [gcc-c++]
    > La colección de compiladores de GNU incluye front ends para C, C ++, Objective-C, Fortran, Ada y Go, así como bibliotecas para estos lenguajes (libstdc ++, ...). GCC fue escrito originalmente como el compilador para el sistema operativo GNU. El sistema GNU fue desarrollado para ser 100% software libre, libre en el sentido de que respeta la libertad del usuario.

Para usuarios de fedora, aunque yum aun esta instalado, se recomuenda ya el uso de dnf.

```sh
#centOS
yum install cmake bison gcc-c++ -y
#fedora
dnf install cmake bison gcc-c++ -y
```


## Compilando mysql

Recierden siempre la maxima de Unix/Linux RTFM (Read The Fucking Manual)

```sh
cd /usr/local/src
curl -sL https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.19.tar.gz | tar xz
cd mysql-5.7.19/
less README && less INSTALL
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql/5.5.28 -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_bin
make
make install
```

## Configuración con alternative

Alternative es un gestor de enlaces sinbolicos para los comandos por default del sistema. Siempre puedes no usarlo y tirar `ln -s` si sabes como hacerlo bien.

```sh
alternatives --install /usr/local/mysql/default mysql /usr/local/mysql/5.5.28 50528
```

## Configuracion de mysql

Algo que debemos tener muy en cuenta, es la correcta configuración de nuesto servicio, algo que no nos preocupa cuando usamos paquetes precompilados, pero que debemos saber si vamos a instalar desde las fuentes algo.

```sh
mkdir -p /var/mysqld
chown mysql:mysql /var/mysqld
/usr/local/mysql/default/scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql/default --datadir=/var/mysqld/data
```

Para este momento nos hace falta algo importante en nuestro sistema, y es la configuración de mysql definido en el archivo `my.cnf`, que se encuentra en la carpeta del codigo fuente y debemos copiar a `/etc`

```sh
cp /usr/local/src/mysql-5.7.19/my.cnf /etc/my.cnf
```

## configuración de supervisor para mysql

Suponiendo que ta tenemos instalado supervisor y que lo veremos en otro manual aquí publicado, solo nos enfocaremos a crear el archivo ini para que supervisor se haga cargo del proceso.

Recuerda que el archivo ini para supervisor debe estar en la siguiente ruta `/etc/supervisord.d/`, le puedes poner mysqld.ini

```ini
[program:mysqld]
user=mysql
autostart=true
autorestart=true
stopsignal=QUIT
command=/usr/bin/pidproxy /var/mysqld/mysqld.pid /usr/local/mysql/default/bin/mysqld_safe --pid-file=/var/mysqld/mysqld.pid
```

Ya solo queda agregar la confiuración y disfrutar.

```
supervisorctl reread mysqld
supervisorctl add mysqld
```

Recuerden que es mysqld por que así le llame al proceso `[program:mysqld]`.

Con amor, Uetiko.

[usuario y grupo]: <https://wiki.archlinux.org/index.php/users_and_groups>
[bison]: <https://www.gnu.org/software/bison/>
[cmake]: <https://cmake.org>
[gcc-c++]: <https://gcc.gnu.org>
