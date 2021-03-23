# Manejo de rutas

Las rutas se encuentran en `routes/web.php` y son la forma de definir las páginas que tendrá la interfaz de la aplicación. Este es el archivo básico:

```php
<?php
use Illuminate\Support\Facades\Route;

Route::get('/bienvenida', function () {
    return view('welcome');
});
?>
```

Desde aquí se puede: 

- Definir páginas para _crear_, _editar_, _actualizar_ o _eliminar_ instancias de los modelos (un CRUD, vaya).
- Definir **zonas de usuario** (cliente, administrador, invitado...) y **redirigir** a donde haga falta.

En esencia, este archivo contiene funciones de la clase `Route` que:

- Devuelven una vista
- Derivan a un **controlador**, que crea automáticamente una serie de rutas.
- Redirigen a otro lado.

Hay varias formas de gestionar las rutas:

=== "Directa"
    Asignando directamente una vista o un resultado a una ruta:

    ```php
    <?php
    Route::get('/', function () {
        return view('inicio');
    });

    Route::get('/sobrenosotros', function () {
        return view('sobre_nosotros');
    });

    Route::get('/contacto', function () {
        return view('contacto');
    });

    Route::get('/post/{id}/{nombre}', function ($id, $nombre) {
        return "Este es el post nº ".$id." creado por ".$nombre;
    })->where('nombre', '[a-zA-Z]+');
    ?>
    ```
=== "Controlador"
    La lógica para mostrar vistas o realizar acciones se puede poner en un controlador y apuntar a él desde `web.php`:

    === "web.php"
        ```php
        <?php
        use App\Http\Controllers\FotoController;
        use App\Http\Controllers\PostController;
        use App\Http\Controllers\VideoController;

        // Una a una
        Route::get('/fotos/subir', [FotoController::class, 'create']);
        Route::get('/fotos/editar', [FotoController::class, 'edit']);

        // crea todas las rutas para el modelo Foto
        Route::resource('fotos', FotoController::class);

        // define rutas para varios modelos a la vez
        Route::resources([
            'posts' => PostController::class,
            'videos' => VideoController::class,
        ]);
        ?>
        ```
    === "FotoController.php"
        ```php
        <?php
        // ...
        public function create() {
            return view('subir_foto');
        }
        public function edit() {
            return view('editar_foto');
        }
        // ...
        ?>
        ```

## Sintaxis

=== "Vistas"
    ```php
    <?php
    // sintaxis
    Route::view('/ruta', 'vista', ['param' => 'valor']);
    // ejemplo
    Route::view('/bienvenida', 'bienvenida', ['nombre' => 'Jaime']);
    ?>
    ```
=== "Redirecciones"
    ```php
    <?php
    Route::redirect('/desde', '/hacia');

    /* Si le asignas un nombre a una ruta se puede usar después
     * para redirigir más fácilmente
     */
    Route::get('/usuario/perfil', function() { ... })->name('perfil');

    // genera una url
    $url = route('perfil');
    // genera una redirección
    return redirect()->route('perfil')
    ?>
    ```

    Si quieres redireccionar a una página en caso de que no se encuentre el modelo, en `web.php`:

    ```php
    <?php
    use App\Http\Controllers\FotoController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::resource('fotos', FotoController::class)
            ->missing(function (Request $request) {
                return Redirect::route('fotos.index');
            });
    ?>
    ```
=== "Verbos HTTP"
    ```php
    <?php
    use Illuminate\Support\Facades\Route;

    // Rutas para verbos HTTP
    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

    // Rutas para más de un verbo HTTP
    Route::match(['get', 'post'], '/', function() { ... });
    // Ruta para cualquier verbo HTTP
    Route::any('/', function() { ... });
    ?>
    ```

    !!! note "Protección CSRF"
        Todos los formularios HTML que apunten a una ruta `post`, `put`, `patch` o `delete` deben incluir un campo para el _token_ [CSRF](https://laravel.com/docs/8.x/csrf). Con **Blade** se añade así:

        ```blade
        <form method="post" action="/profile">
            @csrf
            ...
        </form>
        ```

    !!! note "Métodos diferentes de POST"
        Los formularios HTTP no soportan los métodos `PUT`, `PATCH` o `DELETE`. Para llamar a una ruta con estos métodos desde un formulario hay que añadir un campo oculto extra. En **Blade**:

        ```blade
        <form method="post" action="/profile">
            @method('PUT')
            @csrf
            ...
        </form>
        ```
=== "Parámetros"
    ```php
    <?php
    Route::get('/usuario/{id}', function($id) {
        return 'Usuario'.$id;
    });

    // parámetros opcionales
    Route::get('/usuario/{nombre?}', function($nombre = 'Juan') {
        return $nombre;
    });

    Route::get('/usuario/{nombre?}', function($nombre = null) {
        return $nombre;
    });

    ?>
    ```

    Si necesitas manejar alguna clase como argumento para las rutas, añádelo al comienzo; a esto se le llama **inyección de dependencias**:

    ```php
    <?php
    use Illuminate\Http\Request;

    Route::get('/usuarios', function(Request $request){ ... });
    ?>
    ```
=== "Restricciones"
    ```php
    <?php
    Route::get('/usuario/{nombre}', function($nombre) {
        // ...
    })->where('nombre', '[A-Za-z]+');

    Route::get('/usuario/{id}', function($id) {
        // ...
    })->where('id', '[0-9]+');

    // varias a la vez
    Route::get('/usuario/{id}', function($id, $nombre) {
        // ...
    })->where([
        'id' => '[0-9]+',
        'nombre' => '[A-Za-z]+'
    ]);
    ?>
    ```

## Con Middleware

Se puede asignar un _middleware_ cuando se invoca la ruta del controlador, por ejemplo, para **controlar las sesiones**:

=== "En la ruta"
    ```php
    <?php
    Route::get('/perfil', [UserController::class, 'show'])->middleware('auth');
    ?>
    ```
=== "En el controlador"
    ```php
    <?php
    class UserController extends Controller
    {
        /**
         * Crea una nueva instancia del controlador
         *
         * @return void
         */
        public function __construct()
        {
            // Opción 1 (requiere una clase middleware)
            $this->middleware('auth');
            $this->middleware('log')->only('index');
            $this->middleware('subscribed')->except('store');

            // Opción 2
            $this->middleware(function($request, $next) {
                return $next($request);
            });
        }
    }
    ?>
    ```

