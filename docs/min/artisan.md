# Comandos básicos de Artisan

```sh
# Artisan
# ===========================================================================
php artisan list # ver todos los comandos disponibles
php artisan serve # "sirve" la aplicación en el puerto 8000
php artisan make:$(tipo) # tipo puede ser: controller, view, migration etc

# Controladores
php artisan make:controller Controlador # crea un controlador simple (sin métodos)
php artisan make:controller Controlador --resource # controlador con métodos HTTP
php artisan make:controller Controlador --resource  --model=Photo # versión con enlazado modelo-rutas

# Vistas
php artisan make:view Agenda # crea una vista

# Migraciones (= manejo de bases de datos)
php artisan make:migration create_nombre_table --create="$(nombre_tabla_en_bd)"
php artisan make:migration add_campo_tabla --table="$(nombre_tabla_modificar)"
php artisan migrate # realiza las acciones definidas en las migrations
php artisan migrate:rollback # deshace el último cambio
php artisan migrate:reset # elimina todo menos la tabla migrations

php artisan migrate:status # información sobre qué migraciones se han ejecutado o no
```
