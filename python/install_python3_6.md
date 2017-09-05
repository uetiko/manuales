Como instalar python 3.6.x en centos
====================================

En este manual mostrare como se puede instalar python 2.7.13 y 3.6.2 (las últimas versiones de python) sobre centOS, esto compilando directo desde el código fuente.

Si usas centOS 6, puedes instalar python 2.7.x y python 3.6.x, pero si usas CentOS 7, solo debes instalar 3.6.x, y antes de seguir, hagamos una aclaración sobre esto.

> **Nota**: En el caso de CentOS 7, solo se debe compilar python 3.6.x, debido a que centos se entrega con python como una parte critica del sistema base, la verson de python
> no se actualiza constantemente mas allá de parches de seguridad. Esto significa que los usuaros de centos 6 tendran instalado python 2.6.6 liberado en agisto del 2010, y
> los usuarios de centos 7, python 2.7.5, lanzado en mayo del 2013.
> El problema en centos 7, que contiene 2.7.5, terminara teniendo (Si decides compilar 2.7.13) el sistema, dos binarios de 2.7 diferentes entre si, con sus propios directorios 
> de paquetes y si, esto causara problemas dificiles de diagnosticar.

Ahora, si ustedes son de los que les gusta dar respuestas y no excusas, hay una anotación mas que quiero comentar y que se deben considerar.

- Unicode
> La relación entre python y el soporte para este formato de string ha sido larga y complicada (Como la relación con mi ex esposa), por que a menos que tengas razones muy 
> especificas y obscuras, debes configurar 2.7 para habilitar el soporte de UTF-32, el cual aumenta el uso de ma memoria, pero mejora la compatibilidad. Pero en python 3.3.x
> Pues lo han mandado a la goma y ha sido rescrito y las cadenas ahora se guardan usando la codificación mas eficiente.
- Librerias compartidas
> Debe compilar python como una libreria compartida, dado a que la mayoria de diestros GNU/Linux modernas traen python compilado de esta forma, y si no te da la gana, te dire
> que modulos como mod_wsgi o blender no funcionaran si no compilas de esta forma. Descuida, el argumento para esto es `LDFLAGS="-Wl,-rpath /usr/local/lib"`
- Usa `make altinstall` para evitarte dolores de cabaza.
> Es fundamental que utilice make altinstall cuando instale su versión personalizada de Python.

## Instalando dependencias

Necesitaras algunas herramientas extras para compilar python, no son necesarias, pero creeme, sin ellas, tu interprete te dará algunos dolores de cabeza.

Perdonen si no uso sudo y paso a ser direcatemente root, aclarado esto, sigamos.

```sh
yum update
# aquí vienen compiladores como gcc y otras harramientas
yum groupinstall -y "development tools"
# estas son algunas librerias que se requeriran durante el compilado de python
yum install -y zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel expat-devel
# Si tienes una version minima de centos, requerias de wget
yum install -y wget
```

## Bajando, compilando e instalando python

Python 2.7.13

```sh
cd /usr/local/src
wget http://python.org/ftp/python/2.7.13/Python-2.7.13.tar.xz
tar xf Python-2.7.13.tar.xz
cd Python-2.7.13
./configure --prefix=/usr/local --enable-unicode=ucs4 --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
make && make altinstall
```

Python 3.6.2

```sh 
cd /usr/local/src
wget http://python.org/ftp/python/3.6.2/Python-3.6.2.tar.xz
tar xf Python-3.6.2.tar.xz
cd Python-3.6.2
./configure --prefix=/usr/local --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
make && make altinstall
```

Después de ejecutar los comandos anteriores, su intérprete Python recién instalado estará disponible como `/usr/local/bin/python2.7` o `/usr/local/bin/python3.6`. La versión del sistema de Python 2.6.6 continuará disponible como `/usr/bin/python`, `/usr/bin/python2` y `/usr/bin/python2.6`.

También puede, si desea, eliminar símbolos de la biblioteca compartida para reducir la huella de memoria.

## Instalando pip, actualizando setuptools y wheel

Recuerda que cada interprete de python necesita que instales si propio pip, asi que la forma mas rápida y simple es por medio del script get-pip-py

```sh
# Baja el script de get-pip
wget https://bootstrap.pypa.io/get-pip.py
# Instala pip en cada interprete
python2.7 get-pip.py
python3.6 get-pip.py
 
 # podras usar pip de la siguiete forma
 pip2.7 install [packagename]
 pip2.7 install --upgrade [packagename]
 pip2.7 uninstall [packagename]et the script:
```

Recuerda que pip quedara como pip2.7 y pip3.6 respectivamente.

Con esto, ya tendrás python en un diestro CentOS 6/7

Con amor, Uetiko.
