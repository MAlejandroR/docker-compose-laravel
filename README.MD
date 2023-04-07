# docker para desarrollo con laravel

Se facilita un docker-compose para desarrollar aplicacones con laravel
Se ha cogido una imagen de bitname. Es muy eficiente, pero tiene el inconveniente que no suelen dejar abiertos sus ficheros Dockerfile.
## Servicios incluídos
A continuación se comentan los servicios y la forma de utilizarlo
### mysql
```yaml
  mysql:
    image: mysql
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MYSQL_USER=manuel
      - MYSQL_PASSWORD=manuel
      - MYSQL_DATABASE=roles
    volumes:
      - "./mysql_data:/var/lib/mysql" 
```
Todo lo expuesto es conocido. Observa que se comparte el  directorio mysql_data para tener los datos persistentes (si no existe, docker lo creará la levantar los servicios)
No olvides de incluir esta carpeta en el fichero .gitignore para que no se suba a git.
Por supuesto actualiza los valores de las variables de entorno según consideres, estas credenciales se necesitan para acceder a mysql desde las aplicaciones.

### myapp
```yaml
myapp:
  image: docker.io/bitnami/laravel:10.0.2
    ports:
      - '8000-8005:8000-8005'
      - '5170-5175:5170-5175'
  environment:
    - DB_HOST=mysql
    - DB_PORT=3306
    - DB_USERNAME=manuel
    - DB_PASSWORD=manuel
    - DB_DATABASE=empresas
  volumes:
    - './app:/app'
  depends_on:
    - mysql
```
Se ha seleccionado esta imagen de *bitnami* por comodidad. Tiene el inconveniente que no dejan abierto el fichero Dockerfile a partir del cual generan la imagen, pero funciona bien.

Está en ella incluída php, npm, y todo lo necesario para realizar un desarrollo con laravel

Se comparte un rango de puertos en lugar de uno solo por si sin querer volvemos a abrir otra vez un puerto (si por algún error, tienes el perto 8000 abierto y vuelves a levantar el servidor desde docker, lo levatan en el 8001, ...). Lo mismo ocurre con vite.

La carpeta donde se creará el proyecto es ***app***
Cuando levanta el servicio, ya ha creado una aplicación y el puerto 8000 está abierto igual que el puerto 5170 para **vite**

### phpmyadmin
```yaml
  phpmyadmin:
    image: phpmyadmin
    ports:
      - 8800:80
    depends_on:
      - mysql
    environment:
      - PMA_ARBITRARY=1
      - PMA_HOST=mysql
```
No es que lo necesitemos, pero viene muy bien este servicio. Dejamos abierto el puerto 8800 para poder acceder a la base de datos a través de esta aplicación conocida , ***phpmyadmin***

## Funcionamiento
* Lanzamos el entorno ejectuando en bacground **-d** los servicios
* Ubicados en el directorio donde está el proyecto escribimos

```shell
docker compose up -d
```
Si en algún momento queremos parar todo y elmiminar los contenedores creados (bueno hacerlo antes de apagar el equipo, al acabar de desarrollar)
```shell
docker compose down
```
* Vemos que ya tenemos el proyecto abierto (podemos escribir en el navegador localhost:8000) y también vemos que la carpeta app tiene un proyecto de laravel.
* Igualmente está  instalado y levantado vite
* Problemas con permisos. Si el usuario que tienes no tiene el grupo 1000 (cuestiones de sistemas operativos), tendrás problemas con los permisos
* Cada vez que docker cree algo, luego tienes que dar permisos a ese fichero
* Tienes que hacer muchas veces este comando, donde usuario es tu usuario del sistema. Comentaremos esta cuestión en clase
```shell
sudo chown usuario:usuario ./app -R  
```
Por poner un ejemplo vamos a instalar *****breeze***** 
Si ahora queremos ejectuar comandos dentro de laravel, laravel está dentro del contenedor docker (concretamente el contenedor del servicio myapp) que es donde está nuestro proyecto.
Pero como tenemos la carpeta compartida podemos hacerlo en el directorio app, como lo hacemos habitualmente
Si queremos ejecutar una acción con composer en nuestro host desde la carpeta **app**, debemos tener instalado en el sistema composer. Si no lo tenemos, tendremos que ejectuar composer dentro del contenedor, escribien antes del comando **docker compose exec myapp **:
```shell
docker compose exec myapp composer require laravel/breeze 
```
Si tenemos composer, podemos ir a la carpeta **app** y escribir
```shell
 composer require laravel/breeze 
```
Ambas dos funcionarán perfectamente, quizá no está bien explicado aquí, pero si lo refexionáis bien, es lógico y sencillo, cualquier duda me comentáis