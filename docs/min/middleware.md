# Middleware

!!! info "Fuente"
    - [Programacionymas.com](https://programacionymas.com/blog/restringir-acceso-solo-administradores-laravel-usando-middlewares)

Un middleware es un puente que existe entre la petición que un usuario hace a una ruta y el controlador. Permiten restringir el acceso a tus rutas según el rol de cada usuario.

Se llaman así porque actúan antes de que el controlador haga algo. El middleware **comprueba que se cumple una condición**, y si no, redirige al usuario o ejecuta alguna acción.

------

Algunos middlewares están ya definidos por defecto, pero también se pueden crear

```sh
php artisan make:middleware AdminMiddleware
```

El último término `AdminMiddleware` es el nombre que usaremos en nuestro ejemplo.

Este nombre puede variar según el propósito.

Por ejemplo, si nuestra página es para el público en general pero tenemos una sección de apuestas, podemos crear un `AgeMiddleware`, y **restringir el acceso**, para que solo puedan acceder personas mayores de edad.

En fin.

Una vez que hemos ejecutado el comando, se creará un archivo en la siguiente ruta: `app\Http\Middleware\AdminMiddleware.php`.

Y tendrá el siguiente contenido:

```php
<?php

namespace App\Http\Middleware;

use Closure;

class AdminMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        return $next($request);
    }
}
```

## Definiendo nuestro middleware

En el código generado para nuestro `AdminMiddleware`, es importante comprender que esta línea:

```php
<?php
return $next($request);
```

se expresa como "continúa tu camino".

Si nuestro middlware no realiza ninguna validación en el método `handle` y directamente devuelve esa respuesta, entonces es como si no existiera porque no impone ninguna restricción.

La ejecución seguirá su camino.

Eso significa que luego de pasar el `AdminMiddleware` la ejecución pasará por los siguientes middlewares que hagan falta, y por último llegará al controlador.

Entonces, es aquí donde tenemos que añadir nuestra condición.

Quedamos en que queremos validar si un usuario es administrador, y dejarlo pasar, solo si es administrador. ¿Recuerdas?

Entonces vamos a añadir una columna en nuestra tabla de usuarios, llamada `is_admin`. Esta nos permitirá saber si un usuario es administrador o no.

```php
<?php
$table->boolean('is_admin')->default(false);
```

De tal forma que nuestro método `handle` quede de esta manera:

```php
<?php
public function handle($request, Closure $next)
{
    if (auth()->check() && auth()->user()->is_admin)
        return $next($request);

    return redirect('/');
}
```

Este último código se lee como:

> Si el usuario autenticado es administrador, déjalo pasar, y si no, redirígelo a la ruta principal.

## Registrando nuestro middleware

Para facilitar el uso de nuestro middleware es importante registrarlo bajo un nombre.

Vamos a `app\Http\Kernel.php` y en el siguiente arreglo vamos a agregar nuestro middleware:

```php
<?php
protected $routeMiddleware = [
    'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```

Es decir, añadimos un elemento más:

```php
<?php
protected $routeMiddleware = [
    'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
    'admin' => \App\Http\Middleware\AdminMiddleware::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```

## Aplicando nuestro middleware

¡Ya tenemos nuestro middleware registrado!

Solo falta aplicarlo.

Podemos usar nuestro middlware:

- directamente sobre una ruta en específico,
- sobre un conjunto de rutas,
- o sobre un controlador (afectando a todas las peticiones que este controlador está destinado a resolver).

> ¿Cómo es que deberíamos aplicar nuestros middlewares? ¿Cuál es la forma correcta?

Todas las formas son correctas.

Usar una u otra va a depender del proyecto que estés desarrollando.

Por ejemplo, si el panel para administradores solo difiere un poco del panel de usuarios, es probable que solo quieras aplicar el middleware a un controlador.

Sin embargo, si el administrador puede gestionar muchas entidades, a diferencia de un usuario regular, sería conveniente crear un grupo de rutas y aplicar el middleware a todas ellas.

------

Permíteme explicarlo a través de **un ejemplo**.

Digamos que estoy desarrollando ahora mismo una aplicación donde los usuarios pueden conectarse y ver una serie de tutoriales, organizados por categorías.

El usuario administrador debe ser el único que pueda editar la información de las series.

Entonces, vamos a crear un grupo de rutas y aplicar el middleware `admin` sobre ellas:

```php
<?php
Route::group(['middleware' => 'admin'], function () {
    Route::get('/admin/series', 'Admin\SeriesController@index');
    Route::get('/admin/series/{id}', 'Admin\SeriesController@edit');
});
```

El código anterior funciona muy bien.

Y hay 2 cosas que debemos tener en cuenta:

- Ambas rutas se encuentran bajo el prefijo `/admin`.
- El controlador `SeriesController` se encuentra bajo el namespace `Admin`.

Podemos simplificar nuestro grupo de rutas de la siguiente manera:

```php
<?php
Route::group([
    'middleware' => 'admin',
    'prefix' => 'admin',
    'namespace' => 'Admin'
], function () {
    Route::get('/series', 'SeriesController@index');
    Route::get('/series/{id}', 'SeriesController@edit');
});
```

De tal forma que, podemos continuar añadiendo rutas a este grupo, y todas ellas:

- Estarán validadas por el middlware `admin`.
- Tendrán como prefijo `/admin`.
- Sus controladores respectivos se encontrarán ubicados en `App\Http\Controllers\Admin`.
