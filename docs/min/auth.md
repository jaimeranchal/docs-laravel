# Inicio de sesión y tipos de usuarios

Lo primero es instalar un [kit de inicio](), por ejemplo, _Breeze_. Esto instalará un conjunto de vistas, rutas y controladores:

!!! note "Ojo laravelitas viejunos"
    _Breeze_ usa [componentes](https://laravel.com/docs/8.x/blade#components), un nuevo invento de Laravel 8.x que sustituye la forma anterior de crear vistas mediante las directivas `@yield`, `@section` etc.

=== "Composer"
    ```sh
    composer require laravel/breeze --dev
    php artisan breeze:install
    # Instala y compila las librerías necesarias (css y js)
    npm install
    npm run dev
    # Crea tablas y rutas adicionales
    php artisan migrate
    ```
=== "routes" 
    ```sh
    # añadido
    auth.php
    ```
=== "resouces/views" 
    ```sh
    # base
    welcome.php

    # añadido
    auth/
        confirm-password.blade.php
        forgot-password.blade.php
        login.blade.php
        register.blade.php
        reset-password.blade.php
        verify-email.blade.php
    components/
        application-logo.blade.php
        auth-card.blade.php
        auth-session-status.blade.php
        auth-validation-errors.blade.php
        button.blade.php
        dropdown-link.blade.php
        dropdown.blade.php
        input.blade.php
        label.blade.php
        nav-link.blade.php
        responsive-nav-link.blade.php
    layouts/
        app.blade.php
        guest.blade.php
        navigation.blade.php
    dashboard.blade.php
    ```

    Añade una ruta para un tablón genérico (_dashboard_) en `web.php`:

    ```php
    <?php
    Route::get('/dashboard', function () {
        return view('dashboard');
    })->middleware(['auth'])->name('dashboard');
    ?>
    ```

=== "Http/Controllers" 
    ```sh
    # base
    Controller.php

    # añadido
    Auth/
        AuthenticatedSessionController.php
        ConfirmablePasswordController.php
        EmailVerificationNotificatableController.php
        EmailVerificationPromptController.php
        NewPasswordController.php
        PasswordResetLinkController.php
        RegisteredUserController.php
        VerifyEmailController.php
    ```

## Usando Bootstrap

Vale, no te va _Breeze_ y todo el rollo de los **componentes**, pero eres un máster de Bootstrap. Puedes instalar una interfaz de autentificación con Bootstrap súper fácil con **composer** y **npm**; ventaja, usa el antiguo sistema de plantillas heredadas ;)

=== "Consola"
```sh
# Después de crear un proyecto vacío

# 1. añadimos dependencias
composer require laravel/ui 
# 2. crea las vistas, controllers etc
php artisan ui bootstrap --auth 
# 3. instala las librerías css y js de Bootstrap
npm install && npm run dev 
```

Listo, la vista principal por defecto después de hacer login es `home.blade.php` (no `dashboard`, como en _Breeze_). Hala, a tunear.

## Roles de usuario

Los pasos serían:

1. Crear modelos y migraciones con los valores necesarios
2. Modificar formulario de registro
3. Crear _middleware_ y añadirlos al kernel
4. Modificar las rutas


=== "Paso 1"
    Crear modelo de roles y la migración:

    ```sh
    php artisan make:model Role -m
    ```

    Tunear las migraciones para reflejar la relación entre **User** y **Role**:

    === "CreateUserTable.php"
        ```php
        <?php
        public function up()
        {
            Schema::create('users', function(Blueprint $table){
                //...
                $table->integer('role_id');
            })
        }
        ?>
        ```

    === "CreateRolesTable.php"
        ```php
        <?php
        public function up()
        {
            Schema::create('roles', function(Blueprint $table){
                //...
                $table->string('nombre_rol');
            })
        }
        ?>
        ```

    Refrescar la base de datos

    ```sh
    php artisan migrate:refresh
    ```

=== "Paso 2"
    - Relación entre **User** y **Role** en los modelos (_clave foránea_)
    - Cambiar la lógica del formulario de registro para que inserte un valor para el rol
    - Insertar valores para los roles

    === "User.php"
        ```php
        <?php
        // app/Models/User.php, al final del todo
        public function role() {
            return $this->belongsTo('App\Models\Role');
        }

        public function esAdmin() {
            if($this->role->nombre_rol == 'admin'){
                return true;
            }

            return false;
        }
        ?>
        ```
    === "RegisteredUserController.php"
        ```php
        <?php
        // app/Http/Controllers/RegisteredUserController.php
        public function store(Request $request)
        {
            // $request->validate([
            // ...

            Auth::login($user = User::create([
                //...
                'role_id' => 1, // cliente por defecto, p.e.
                //...
            ]));

            //...
        }
        ?>
        ```
=== "Paso 3"
    Crear un _middleware_ para gestionar los roles:
    ```sh
    php artisan make:middleware Role
    php artisan make:middleware EsAdmin
    ```

    Añadirlos a `kernel.php`

    ```php
    <?php
    protected $routeMiddleware = [
        // ...
        'role' => \App\Http\Middleware\RoleMiddleware::class,
        'esAdmin' => \App\Http\Middleware\EsAdminMiddleware::class,
    ];
    ?>
    ```

    Editar `RoleMiddleware`:

    ```php
    <?php
    public function handle($request, Closure $next)
    {
        // redirige a donde quieras según el rol
        return redirect('/');
    }
    ?>
    ```
=== "Paso 4"
    Cambiar rutas:

    ```php
    <?php
    // web.php
    Route::get('/', function() {
        // llamamos al middleware para controlar si es admin
        if($user->esAdmin()){
            //...
        } else {
            //...
        }
        return view('welcome');
    }
    ?>
    ```
