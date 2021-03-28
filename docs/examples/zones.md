# Zonas de usuarios

Se puede hacer de muchas formas; en parte depende de lo que quieras:

1. Una portada, una serie de páginas para usuarios y un panel de administración.
2. Una serie de páginas para usuarios y otra serie distinta para administradores.

## Caso 1: panel de administrador

Nuestra aplicación tiene una página (o unas pocas) para administradores: esa será la **zona de administración** y el resto para usuarios normales.

Vamos a hacerlo con _middleware_ para filtrar los intentos de acceso a una dirección, por ejemplo `/admin`; si el usuario _es_ administrador, le dejamos pasar, si no, página de error.

Los pasos:

1. Modificar el modelo _user_ para que incluya el Rol
2. Crear un _middleware_ con **Artisan** y el controlador necesario
3. Registrar el _middleware_ en el **kernel**
4. Crear una **vista** que sirva de entrada para cada tipo de usuario
5. Modificar las **reglas** para aceptar peticiones en el _middleware_
6. Aplicar el _middleware_ a las **rutas**

=== "Modelo"
    - Crear modelo `Role` y añadir campos a la migración correspondiente
    - Crear relación entre `Role` y `User`:
        - añadir campo `role_id` a la migración de `User`
        - añadir función `role` y `esAdmin` al modelo `User`

    === "Artisan"
        ```sh
        php artisan make:model Role -m
        ```
    === "Role"
        ```php
        <?php
        use Illuminate\Database\Migrations\Migration;
        use Illuminate\Database\Schema\Blueprint;
        use Illuminate\Support\Facades\Schema;

        class CreateRolesTable extends Migration
        {
            /**
             * Run the migrations.
             */
            public function up()
            {
                Schema::create('roles', function (Blueprint $table) {
                    $table->id();
                    $table->string('nombre');
                    $table->timestamps();
                });
            }

            // ...
        }
        ?>
        ```
    === "User"
        === "Migrations/CreateUsersTable"
            ```php
            <?php
            use Illuminate\Database\Migrations\Migration;
            use Illuminate\Database\Schema\Blueprint;
            use Illuminate\Support\Facades\Schema;

            class CreateUsersTable extends Migration
            {
                /**
                 * Run the migrations.
                 */
                public function up()
                {
                    Schema::create('users', function (Blueprint $table) {
                        // ...
                        $table->integer('role_id');
                        // ...
                    });
                }

                // ...
            }
            ?>
            ```
        === "Models/User"
            ```php
            <?php
            namespace App\Models;

            use Illuminate\Contracts\Auth\MustVerifyEmail;
            use Illuminate\Database\Eloquent\Factories\HasFactory;
            use Illuminate\Foundation\Auth\User as Authenticatable;
            use Illuminate\Notifications\Notifiable;

            class User extends Authenticatable
            {
                // ...

                /**
                 * Recupera el rol del usuario
                 */
                public function role() {
                    return $this->belongsTo('App\Models\Role');
                }

                /**
                 * Comprueba si el rol es administrador
                 * @return boolean
                 */
                public function esAdmin() {
                    if ($this->role->nombre == 'admin') {
                        return true;
                    }

                    return false;
                }
            }
            ?>
            ```
=== "Vista"
    - Crear tantas vistas como necesitemos para los roles de la apliación

    === "admin.blade.php"
    ```blade
    @extends('layouts.app')

    @section('content')
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Panel</div>

                    <div class="card-body">
                        @if (session('status'))
                            <div class="alert alert-success" role="alert">
                                {{ session('status') }}
                            </div>
                        @endif

                        Estás en el panel del ADMINISTRADOR
                    </div>
                </div>
            </div>
        </div>
    </div>
    ```
=== "Controlador"
    - Crear controlador y middleware con Artisan
    - Registrar el middleware en el kernel
    - Implementar la lógica en el middleware
    - Aplicar el middleware en el controlador
    - Definir las rutas

    === "Artisan"
        ```sh
        # Crear controladores
        php artisan make:controller AdminController

        # Crear middleware
        php artisan make:middleware AdminMiddleware
        ```
    === "Kernel.php"
        ```php
        <?php
        /**
         * The application's route middleware.
         *
         * These middleware may be assigned to groups or used individually.
         *
         * @var array
         */
        protected $routeMiddleware = [
            // ...
            'admin' => \App\Http\Middleware\AdminMiddleware::class,
            // ...
        ];   
        ?>
        ```
    === "AdminMiddleware.php"
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
                if (!$request->user()->esAdmin()) {
                    abort(401, 'Acción no permitida');

                    // si quieres redirigir a una página especial
                    // return redirect('/acceso_denegado');
                }
                return $next($request); # Sigue hacia la ruta que buscabas
            }
        }
        ```
    === "AdminController.php"
        ```php
        <?php
        namespace App\Http\Controllers;

        use Illuminate\Http\Request;

        class AdminController extends Controller
        {
            public function __construct()
            {
                $this->middleware('auth');
                $this->middleware('admin');
            }

            public function index()
            {
                return view('admin.home');
            }
        }
        ?>
        ```
    === "web.php"
        ```php
        <?php
        // Si el administrador accede a una página similar a las demás
        Route::get('/admin', 'AdminController@index');

        // Si el administrador maneja un grupo de páginas
        Route::group(['middleware' => 'admin'], function () {
            Route::get('/admin/series', 'Admin\SeriesController@index');
            Route::get('/admin/series/{id}', 'Admin\SeriesController@edit');
        });

        // Otra forma de agrupar bajo un mismo namespace
        // - Estar validadas por el middlware `admin`.
        // - Tiene como prefijo `/admin`.
        // - Sus controladores respectivos están en `App\Http\Controllers\Admin`.
        Route::group([
            'middleware' => 'admin',
            'prefix' => 'admin',
            'namespace' => 'Admin'
        ], function () {
            Route::get('/series', 'SeriesController@index');
            Route::get('/series/{id}', 'SeriesController@edit');
        });
        ```

!!! info "Fuente"
    - [Programacionymas.com](https://programacionymas.com/blog/restringir-acceso-solo-administradores-laravel-usando-middlewares)

## Caso 2: paneles cliente y administrador

Si tenemos una aplicación con cierto contenido general y además una **zona de clientes** separada del contenido general _y_ de la zona de administrador. 

!!! info "Fuente"
    - [medium.fabcoding](https://medium.com/fabcoding/laravel-redirect-users-according-to-roles-and-protect-routes-bde324fe1823)
