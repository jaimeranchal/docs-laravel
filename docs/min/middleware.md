# Middleware

Un middleware es un puente que existe entre la petición que un usuario hace a una ruta y el controlador. Permiten restringir el acceso a tus rutas según el rol de cada usuario. Se llaman así porque actúan antes de que el controlador haga algo. El middleware **comprueba que se cumple una condición**, y si no, redirige al usuario o ejecuta alguna acción.

## Notas rápidas (clase de Nacho 27 de abril)

Tres páginas solo accesibles para tres tipos de usuarios:

- Usuario normal
- Administrador
- Superadministrador

### Pasos

1. Crear un campo _tipo_ o _rol_ (user, admin, superadmin...) en la tabla **User**
    - modificar **migración**
2. Añadir el campo al registro:
    - Campo _tipo_ en la **vista** _Registro_ (`views/auth/register.blade.php`)
    - Modificar el **controlador**, método `store()`
3. Crear **middleware** para cada _tipo_:
    ```sh
    php artisan make:middleware UserMiddleware
    ```

    ```php
    <?php
    public function handle(Request $request, Closure $next)
    {
        /*
         * Si ES un usuario pero NO ES del tipo deseado
        */
        if ($request->user() && $request->user()->type != 'member') {
            return view('unauthorized');
        }

        return $next($request);
    }
    ```

    - _Opcional_: darle un **alias** al middleware en `kernel.php`, dentro del array `routeMiddleware`
    - _Opcional_: **agrupar** rutas y aplicarles a todas un middleware
        ```php
        <?php
        // sin declarar un alias en kernel.php
        use Http\App\Middleware\AdminMiddleware;

        Route::middleware([AdminMiddleware::class])->group( function() {
            
            /*
             * - Métodos: match(['get', 'post'])
             * - Patrón de la uri: /adminOnlyPage/
             * - destino de la ruta: controlador, vista...
            */
            Route::match(['get', 'post'], '/adminOnlyPage/', [HomeController::class, 'admin'])    

        })

        // Con alias
        Route::middleware(['admin', 'superadmin'])->group( function() {

            // ...

        })
        ```
        
