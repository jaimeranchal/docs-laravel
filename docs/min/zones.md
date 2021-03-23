# Zonas de usuarios

1. Modificar o crear un _middleware_ para gestionar usuarios
2. Crear un controlador para usuarios de un tipo (admin p.e.)
3. Crear una ruta que apunte a la función `index()` del controlador

=== "Paso 1"
    - Limpiar las rutas
    - modificiar o crear el _middleware_

    === "web.php"
        ```php
        <?php
        Route::get('/', function() {
            return view('welcome');
        })
        ?>
        ```
    === "Middleware\esAdmin.php"
        ```php
        <?php
        use Illuminate\Support\Facades\Auth;

        public function handle($request, Closure $next){
            // recojemos el usuario autentificado
            $user = Auth::user();

            // el que no sea administrador, que vaya a home
            if(!$user->esAdmin()){
                return redirect('/');
            } 
            
            // si es administrador, reenvía a donde toque
            return $next($request);
            
        }
        ?>
        ```
=== "Paso 2"
    - Creo un controlador para los usuarios administradores, que se encargue de gestionarlos:

    ```sh
    php artisan make:controller AdminController
    ```

    ```php
    <?php
    class AdminController extends Controller
    {
        // cargar el middleware esAdmin, desde el constructor
        public function __construct() {
            $this->middleware('esAdmin');
        }

        // crear una función que diga lo que debe hacer con los 
        // usuario logueados como admin
        public function index() {
            return view('admin');
        }
    }
    ?>
    ```
=== "Paso 3"
    - Crear una ruta que apunte a la función `index()` del controlador `AdminController`:

    === "web.php"
    ```php
    <?php
    Route::get('/admin', 'AdminController@index');
    ?>
    ```
