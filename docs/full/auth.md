# Login, roles y manejo de usuarios en Laravel

Laravel trae por defecto una carpeta con los campos básicos de usuario. En Laravel 8, si instalamos el paquete `laravel/ui` podemos crear rápidamente un mecanismo de login y registro de usuarios:

```sh
# Instala el paquete
composer require laravel/ui
# crea el sistema básico de autentificación
php artisan ui:auth
```

!!! info "Fuentes"
    Lo siguiente es un resumen de la [documentación oficial de Laravel 8.x](https://laravel.com/docs/8.x/authentication)

Formas de iniciar sesión y registrar usuarios.

## Introducción

Las herramientas de inicio de sesión en Laravel se componen de _defensas_ (**Guards**) y _proveedores_ (**Providers**):

- Las _defensas_ definen cómo se autentica el usuario para cada petición
  Un ejemplo sería la defensa `session` que mantiene información del estado del usuario mediante _session storage_ y _cookies_.
- Los _proveedores_ definen cómo se recupera información de usuario desde la base de datos (o donde guardemos la información). Por defecto, Laravel permite hacer esto con [Eloquent](https://laravel.com/docs/8.x/eloquent) y el constructor de queries. 

El **archivo de configuración** está en `config/auth.php`

### Kits de inicio

- Instala una de los kits de inicio en una instalación nueva.
    ```sh
    # 1. crea el proyecto
    composer create-project laravel/laravel nombre
    # alt: curl -s https://laravel.build/example-app | bash
    cd example-app
    # 2. crea las tablas básicas
    php artisan migrate
    # 3. Instala Breeze
    composer require laravel/breeze --dev
    php artisan breeze:install
    # 3.2. instala y compila las librerías necesarias (css y js)
    npm install
    npm run dev
    # 4. Crea tablas y rutas adicionales
    php artisan migrate
    ```
- Todas las rutas de Breeze están definidas en `routes/auth.php`
- Navega hasta `/register` y el kit se encargará de todo.

!!! note "No confundir"
    Los kits de inicio solo proporcionan una **interfaz** y un mecanismo básico y funcional para crear usuarios e iniciar sesión. **Todo lo demás** (rutas, extra, reglas de contraseñas, campos de usuario, permisos y roles) debe **configurarse aparte**.

### Consideraciones sobre la base de datos

Laravel incluye por defecto un **modelo de usuario** en `App\Models\User`. Si quieres usar un controlador diferente a Eloquent, puedes usar el proveedor de autentificación `database`. Sólo ten en cuenta:

- al crear un esquema para la base de datos para el modelo `App\Models\User`, el campo _password_ debe tener **al menos 60 caracteres** (la tabla creada por defecto ya lo hace).
- la tabla `users` debe contener un campo _nullable_, de tipo _string_, llamado `remember_token` con una extensión de 100 caracteres.
  Se usa para guardar un _token_ para usuarios que han marcado la opción "recuérdame" al iniciar sesión.

### Vistazo general al ecosistema

Vamos a repasar cómo funciona la autentificación en una aplicación web:

1. Un usuario proporciona un _nombre de usuario_ y una _contraseña_ con un formulario.
2. Si las credenciales son correctas la aplicación guarda información sobre el usuario en una [sesión](https://laravel.com/docs/8.x/session).
    - El navegador (lado del cliente) guarda la **id de sesión** con una _cookie_ para que las siguientes peticiones a la aplicación asocien el usuario correctamente con la sesión.
    - La aplicación (lado del servidor) usa la id de sesión de la cookie para recuperar información del usuario guardada en la sesión.

Cuando se trabaja con APIs, no se suelen usar cookies sino _tokens_. Un _token_ de API es una cadena que se envía a la API en cada petición y que se contrasta con una lista de _tokens_ autorizados para considerar que la petición procede de un usuario autentificado.

### Servicios de autentificación preinstalados en Laravel

Se accede a ellos a través de las _Facades_ `Auth` y `Session`; proporcionan:

- autentificación mediante cookies (para peticiones desde un navegador)
- métodos para verificar las credenciales

Además, de forma automática:

- guardan los datos de autentificación en la sesión
- generan una cookie con la id de sesión.

=== "Aplicaciones"
    Si estás desarrollando una aplicación web, considera usar una de estas tres opciones:

    - [Breeze](https://laravel.com/docs/8.x/starter-kits#laravel-breeze)
    - [JetStream](https://laravel.com/docs/8.x/starter-kits#laravel-jetstream)
    - [Fortify](https://laravel.com/docs/8.x/fortify)

=== "API"
    Para APIS, Laravel proporciona dos paquetes adicionales para manejar tokens y peticiones de autentificación:

    - [Passport](https://laravel.com/docs/8.x/passport)
    - [Sanctum](https://laravel.com/docs/8.x/sanctum)

## Inicio rápido

=== "Instalar un kit de inicio"
    - [Breeze](https://laravel.com/docs/8.x/starter-kits#laravel-breeze): conjunto de plantillas Blace usando el estilo de css [Tailwind](https://tailwindcss.com/).
    - [JetStream](https://laravel.com/docs/8.x/starter-kits#laravel-jetstream): más robusto, usa [Livewire](https://laravel-livewire.com/) o [Inertia.js y Vue](https://inertiajs.com/). Permite autentificación en dos pasos, equipos, manejo de perfil, manejo de sesiones del navegador etc.

=== "Recuperar usuario"
    ```php
    <?php
    use Illuminate\Support\Facades\Auth;

    // Recupera el usuario que ahora mismo está logueado
    $user = Auth::user();

    // Recupera la id del usuario que ahora mismo está logueado
    $id = Auth::id();

    // Comprueba que el usuario está autentificado
    /* NOTA: eso se suele hacer con MIDDLEWARE */

    if (Auth::check()){
        // El usuario se ha loqueado en...
    }
    ?>
    ```
    
    También puedes acceder mediante una instancia de `Illuminate\Http\Request`:

    ```php
    <?php
    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class VueloController extends Controller
    {
        /**
         * Actualiza la información de vuelo de un vuelo existente
         *
         * @param  \Illuminate\Http\Request  $request
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request)
        {
            // $request->user()
        }
    }
    ?>
    ```

=== "Proteger rutas"
    Para controlar que solo usuarios identificados puedan acceder a ciertos recursos usamos [Middleware](https://laravel.com/docs/8.x/middleware). Un _middleware_ es una forma de inspeccionar y filtrar peticiones HTTP entrantes.

    Laravel ya incluye uno, `auth`, que apunta a la clase `Illuminate\Auth\Middleware\Authenticate`. Para usarlo, solo hay que **añadirlo a las rutas**:
    
    ```php
    <?php
    Route::get('/vuelos', function(){
        // Solo usuarios identificados pueden acceder aquí
    })->middleware('auth');
    ?>
    ```

    Para **redirigir a un usuario** no identificado hacia otra página hay que actualizar la función `redirectTo` de `app\Http\Middleware\Authenticate.php`:

    ```php
    <?php
    /**
     * Get the path the user should be redirected to.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return string
     */
    protected function redirectTo($request)
    {
        // por defecto reenvía a la pantalla de login, pero puede ser la que quieras
        return route('login');
    }
    ?>
    ```

    Cuando añades el middleware `auth` a una ruta, puede especificar que _defensa_ (guard) se va a usar para identificar al usuario. Debe corresponderse con una de las claves almacenadas en el array `guards` del archivo `auth.php`:

    ```php
    <?php
    Route::get('/flights', function () {
        // Solo usuarios identificados pueden acceder aquí
    })->middleware('auth:admin');
    ?>
    ```
    
=== "Límite de intentos"
    Por defecto, con Breeze o JetStream, si el usuario falla al loguearse varias veces seguidas, tiene que esperar _un minuto_ antes de volver a intentarlo. Este límite se asigna de forma única al nombre de usuario, su email y la dirección IP.

    Para limitar los intentos de acceso a otras rutas, ver la [documentación](https://laravel.com/docs/8.x/routing#rate-limiting)

## Autentificación manual

Si no quieres usar los sistemas incluídos en los kits de inicio, se puede crear un sistema de autentificación manualmente. Sólo hay que tener en cuenta algunas cosas.

Partiendo de una clase controlador para el login llamada `LoginController`:

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * Handle an authentication attempt.
     *
     * @param  \Illuminate\Http\Request $request
     * @return \Illuminate\Http\Response
     */
    public function authenticate(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();

            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ]);
    }
}
?>
```

Vemos por pasos lo que ocurre

=== "1.Importar"
    Importar la _facade_ `Auth` al principio de la clase.

    ```php
    <?php
    use Illuminate\Support\Facades\Auth;
    ?>
    ```

=== "2.Intentos"
    El método `attempt()` sirve para manejar los intentos de inicio de sesión del formulario de login.

    ```php
    <?php
    $credentials = $request->only('email', 'password');

    if (Auth::attempt($credentials)) {
        $request->session()->regenerate();

        return redirect()->intended('dashboard');
    }
    ?>
    ```

    - Este método acepta como parámetros un array asociativo con las columnas que se van a usar para buscar al usuario en la base de datos.
    - Si lo encuentra, compara el hash de la contraseña guardada con la introducida (no hace falta convertirla en hash).
    - Si coinciden, devuelve `true`.
=== "3.Sesión"
    Si el usuario se autentifica con éxito, hay que **regenerar la sesión** del usuario para prevenir [sesiones fijas](https://en.wikipedia.org/wiki/Session_fixation).

    ```php
    <?php
    $request->session()->regenerate();
    ?>
    ```
=== "4.Redirigir"
    La función `intended()` usada para redirigir al usuario a la página en cuestión acepta una dirección de _reserva_ en caso de que la página no esté disponible (`dashboard`, en este caso):

    ```php
    <?php
    return redirect()->intended('dashboard');
    ?>
    ```
=== "5.Condiciones extra"
    Se pueden incluir más condiciones a la _query_ que busca usuarios en la base de datos. Por ejemplo, para incluir que el usuario esté "activo":

    ```php
    <?php
    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // Authentication was successful...
    }
    ?>
    ```

    También se pueden usar _defensas_ (guards) para autentificar al usuario, y de esa forma mantener modelos de autentificación separadas:

    ```php
    <?php
    if (Auth::guard('admin')->attempt($credentials)) { ... }
    ?>
    ```

### Recordar usuarios

Para habilitar la función de "recuérdame" al iniciar sesión, hay que pasarle un valor booleano como **segundo argumento** al método `attempt()`:

```php
<?php
if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // No te olvidaremos, Tony...
}
?>
```

Cuando este valor es `true`, Laraval mantendrá al usuario logueado _indefinidamente_ o hasta que cierre sesión. La tabla de usuarios debe incluir una columna `remember_token`, donde se guardará el _token_; la tabla `users` definida por defecto ya la incluye

#### Otras formas de autentificación

=== "Una instancia de usuario"
    ```php
    <?php
    use Illuminate\Support\Facades\Auth;

    Auth::login($user);
    Auth::login($user, $remember = true);
    Auth::guard('admin')->login($user);
    ?>
    ```
=== "Por su id"
    ```php
    <?php
    Auth::loginUsingId(1);
    Auth::loginUsingId(1, $remember = true);
    ?>
    ```
=== "Una única vez"
    ```php
    <?php
    if (Auth::once($credentials)) { ... }
    ?>
    ```

### Cerrar sesión

Para cerrar manualmente la sesión, hay que usar el método `logout()`:

```php
<?php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * Log the user out of the application.
 *
 * @param  \Illuminate\Http\Request $request
 * @return \Illuminate\Http\Response
 */
public function logout(Request $request)
{
    Auth::logout();

    // Invalida la sesión de usuario
    $request->session()->invalidate();

    // regnera el código CSRF (seguridad)
    $request->session()->regenerateToken();

    return redirect('/');
}
?>
```

Para **invalidar sesión en otros dispositivos** hay que hacer un par más de cosillas:

=== "1"
    Comprobar que el middleware `AuthenticateSession` está presente y **descomentado** en el grupo `web` de la clase `App\Http\Kernel`:

    ```php
    <?php
    'web' => [
        // ...
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        // ...
    ],
    ?>
    ```
=== "2"
    Usar el método `logoutOtherDevices`:

    ```php
    <?php
    Auth::logoutOtherDevices($currentPassword);
    ?>
    ```

## Confirmar la contraseña

Si necesitas que el usuario confirme su contraseña en alguna página, Laravel incluye un _middleware_ para hacerlo. Es necesario definir **dos rutas**:

- una vista para pedir al usuario que confirme su contraseña
- una vista que confirme que la contraseña es válida y redirija al usuario a su destino

### Configuración
Normalmente, la contraseña de un usuario se mantiene válida durante _tres horas_. Ese tiempo se puede cambiar con la opción `password_timeout` en el archivo de configuración `config/auth.php`.

### Rutas

=== "Formulario de confirmación"
    ```php
    <?php
    Route::get('/confirme-password', function(){
        return view('auth.confirm-password');
    })->middleware('auth')->name('password.confirm');
    ?>
    ```
=== "Confirmar la contraseña"
    ```php
    <?php
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Facades\Redirect;

    Route::post('/confirm-password', function (Request $request) {
        // 1. Si hay errores, vuelve atrás mostrando error
        if (! Hash::check($request->password, $request->user()->password)) {
            return back()->withErrors([
                'password' => ['The provided password does not match our records.']
            ]);
        }

        // 2. Guarda la hora de confirmación en la sesión
        $request->session()->passwordConfirmed();

        // 3. Redirige al usuario a donde tenía que ir
        return redirect()->intended();
    })->middleware(['auth', 'throttle:6,1'])->name('password.confirm');
    ?>
    ```

### Proteger las rutas

Todas las rutas cuya acción necesite confirmar la contraseña deben tener asignado el _middleware_ `password.confirm`. El _middleware_ guarda la ruta a donde tenía que ir para redirigir al usuario correctamente después de confirmar su contraseña:

```php
<?php
Route::get('/settings', function(){
    //...
})->middleware(['password.confirm']);

Route::post('/settings', function(){
    //...
})->middleware(['password.confirm']);
?>
```

## Añadir _Guards_ personalizadas

Puedes definir tus propias defensas (guards) usando el método `extend` de la Facade `Auth`, dentro de un **proveedor** de servicio. El proveedor incluido por defecto en Laravel es `AuthServiceProvider`:

=== "AuthServiceProvider"
    ```php
    <?php
    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Auth;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function ($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }
    ?>
    ```
=== "auth.php"
    ```php
    <?php
    //...
    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],
    //...
    ?>
    ```

### Closure Request Guards

La forma más simple de implementar un sistema de autentificación HTTP basado en peticiones es usando el método `Auth::viaRequest`. Te permite definir un proceso de autentificación usando un único _closure_.

!!! info "Clausura o _closure_"
    Una clausura o closure es una función anónima que captura el scope actual, y proporciona acceso a ese scope cuando se invoca el closure ([Diego Lázaro](https://diego.com.es/funciones-anonimas-y-clausuras-en-php#:~:text=Una%20funci%C3%B3n%20an%C3%B3nima%20no%20es,cuando%20se%20invoca%20el%20closure.&text=Cuando%20se%20emplea%20use,%20se%20hereda%20una%20variable%20del%20%C3%A1mbito%20padre.)).

1. Llama al método `Auth::viaRequest` dentro del método `boot` de tu `AuthServiceProvider`.
    - El método `viaRequest` acepta un controlador de autentificación como primer argumento (cadena).
    - El segundo argumento es un _closure_ que recibe la petición HTTP entrante y devuelve una instancia de usuario o `null`, si falla

=== "AuthServiceProvider"
    ```php
    <?php
    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::viaRequest('custom-token', function (Request $request) {
            return User::where('token', $request->token)->first();
        });
    }
    ?>
    ```
=== "AuthServiceProvider"
    ```php
    <?php
    //...
    'guards' => [
        'api' => [
            'driver' => 'custom-token',
        ],
    ],
    //...
    ?>
    ```
