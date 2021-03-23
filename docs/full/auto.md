# Autorización

!!! info "Fuentes"
    Lo siguiente es un resumen de la [documentación oficial de Laravel 8.x](https://laravel.com/docs/8.x/authorization)

Formas de asignar **roles** y **restringir el acceso** a zonas para tipos de usuarios.

## Introducción

Laravel proporciona dos formas básicas de restringir el acceso según el tipo o **rol** del usuario: **[gates](https://laravel.com/docs/8.x/authorization#gates)** (portales) y **[policies](https://laravel.com/docs/8.x/authorization#creating-policies)** (reglas).

El funcionamiento de uno y otro es similar a las **rutas** y **controladores**. 

- Los portales son más simples y se basan en _closures_,
- Las reglas engloban una lógica de programación relacionada con un modelo particular o un recurso.

Los portales son más apropiados para acciones no vinculadas a un modelo concreto (ver el panel de administración, por ejemplo). Cuando lo que buscamos es autorizar una acción para un determinado modelo o recurso, lo suyo es usar reglas.

## Portales (Gates)

### Escribir un Portal

Los portales se definen en el método `boot` de `App\Providers\AuthServiceProvider` usando la Facade `Gate`. Un portal siempre recibe recibe una instancia de usuario y luego argumentos adicionales, p.e. un modelo Eloquent que tenga que ver.

Vamos a definir un portal para controlar si un usuario puede actualizar el modelo `App\Models\Post`. Para hacerlo, el portal compara la `id` del usuario con el `user_id` del usuario que creó el Post.

=== "Función de clausura"
    ```php
    <?php
    use App\Models\Post;
    use App\Models\User;
    use Illuminate\Support\Facades\Gate;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id;
        });
    }
    ?>
    ```
=== "Array de llamada a clase"
    ```php
    <?php
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', [PostPolicy::class, 'update']);
    }
    ?>
    ```

### Autorizando acciones

Para autorizar una acción con portales, hay que usar los métodos `allows` o `denies` de la Facade `Gate`. Ojo que no necesitan recibir el usuario autentificado como parámetro; ya se encarga Laravel.

Eso suele hacerse en un controlador:

=== "PostController"
    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Gate;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \App\Models\Post  $post
         * @return \Illuminate\Http\Response
         */
        public function update(Request $request, Post $post)
        {
            if (! Gate::allows('update-post', $post)) {
                abort(403);
            }

            // Update the post...
        }
    }
    ?>
    ```
=== "Otros usuarios"
    ```php
    <?php
    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }
    ?>
    ```
=== "Varias acciones"
    ```php
    <?php
    if (Gate::any(['update-post', 'delete-post'], $post)) {
        // The user can update or delete the post...
    }

    if (Gate::none(['update-post', 'delete-post'], $post)) {
        // The user can't update or delete the post...
    }
    ?>
    ```

#### Arrojar excepciones (error 403)

```php
<?php
Gate::authorize('update-post', $post);

// la acción está permitida
// lanza un error 403 si el usuario no puede hacer esa acción
?>
```

Si quieres definir mejor el contexto para permitir o no una acción, los métodos `allosw`, `denies`, `check`, `any`, `none`, `authorize`, `can` y `cannot`, así como las directivas de Blase `@can`, `@cannot`, `@canany` pueden recibir un **array de opciones** como segundo argumento:

```php
<?php
use App\Models\Category;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::define('create-post', function (User $user, Category $category, $pinned) {
    if (! $user->canPublishToGroup($category->group)) {
        return false;
    } elseif ($pinned && ! $user->canPinPosts()) {
        return false;
    }

    return true;
});

if (Gate::check('create-post', [$category, $pinned])) {
    // The user can create the post...
}
?>
```

### Respuesta del Portal

Para devolver un mensaje de error hay que devolver una instancia de `Illuminate\Auth\Access\Response`:

```php
<?php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
                ? Response::allow()
                : Response::deny('You must be an administrator.');
});
?>
```

Aun así, el método `Gate::allows` solo devuelve `true` o `false`. Para devolver el mensaje completo del error hay que usar `Gate::inspect`:

```php
<?php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}
?>
```

Si se usa el método `Gate::authorize` el mensaje de error se envía automáticamente a la respuesta HTTP.

### Interceptando comprobaciones del Portal

Si quieres dar todos los permisos a un usuario en concreto puedes usar el método `before` para definir una _closure_ que se ejecute _antes_ que las comprobaciones de autorización. De forma similar, con efecto contrario se puede usar el método `after`:

```php
<?php
use Illuminate\Support\Facades\Gate;

// si before o after devuelve un valor no-nulo, ese será el resultado de la comprobación de autorización

Gate::before(function ($user, $ability) {
    if ($user->isAdministrator()) {
        return true;
    }
});

Gate::after(function ($user, $ability, $result, $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});
?>
```

## Crear Reglas (Policies)

Las reglas son **clases** que organizan la lógica de autorización para un **modelo** particular o un **recurso**. Si tu aplicación es un blog y tiene un modelo `App\Models\Post` tendrá su correspondiente `App\Policies\PostPolicy` para autorizar que los usuarios cree o actualicen sus posts.

### Generar Reglas

Se pueden generar con **artisan**, y se guardarán en `app/Policies`:

```sh
php artisan make:policy PostPolicy # vacía
php artisan make:policy PostPolicy --model=Post # con métodos CRUD para el modelo Post
```

### Registrar Reglas

Después de crearla, hay que registrarla; decirle a Laravel qué reglas usar cuando autorice acciones para un determinado modelo.

Esto se hace actualizando la propiedad `policies` de `App\Providers\AuthServiceProvider`:

```php
<?php

namespace App\Providers;

use App\Models\Post;
use App\Policies\PostPolicy;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Gate;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        //
    }
}
?>
```

!!! note "Autodescubrir reglas"
    Si se siguen las convenciones de nomenclatura de Laravel, éste descubrirá automáticamente las reglas establecidas para los modelos: `${modelo}Policy` (`UserPolicy`, `VueloPolicy` etc).

## Escribir Reglas

### Métodos para Reglas

=== "Método update"
    ```php
    <?php
    <?php

    namespace App\Policies;

    use App\Models\Post;
    use App\Models\User;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\Models\User  $user
         * @param  \App\Models\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }    
    ?>
    ```

### Respuestas

Si quieres fijar un valor de retorno que no sea booleano, por ejemplo un mensaje de error, puedes devolver una instancia de `Illuminate\Auth\Access\Response`:

```php
<?php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Determine if the given post can be updated by the user.
 *
 * @param  \App\Models\User  $user
 * @param  \App\Models\Post  $post
 * @return \Illuminate\Auth\Access\Response
 */
public function update(User $user, Post $post)
{
    return $user->id === $post->user_id
                ? Response::allow()
                : Response::deny('You do not own this post.');
}
?>
```

También se puede recuperar el mensaje de autorización con el método `Gate::inspect()`:

```php
<?php
use Illuminate\Support\Facades\Gate;

$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}
?>
```

### Métodos sin Modelo

Por ejemplo, para validar que un usuario pueda crear posts, lo único que recibe es una instancia de usuario:

```php
<?php
public function create(User $user){
    return $user->role == 'writer';
}
?>
```

### Usuarios invitados

Por defecto, los portales y las rutas devuelve `false` y la petición entrante no ha sido iniciada por un usuario identificado. Para evitarlo se puede proporcionar un argumento `optional`:

```php
<?php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Determine if the given post can be updated by the user.
     *
     * @param  \App\Models\User  $user
     * @param  \App\Models\Post  $post
     * @return bool
     */
    public function update(?User $user, Post $post)
    {
        return optional($user)->id === $post->user_id;
    }
}
?>
```

### Filtros para Reglas

Si quieres que un usuario pueda realizar todas las acciones de una Regla, hay que usar `before`:

```php
<?php
use App\Models\User;

/**
 * Perform pre-authorization checks.
 *
 * @param  \App\Models\User  $user
 * @param  string  $ability
 * @return void|bool
 */
public function before(User $user, $ability)
{
    if ($user->isAdministrator()) {
        return true;
    }
}
?>
```

- Si devuelve `false` se **deniegan** todas las acciones para ese usuario.
- Si devuelve `null` se deja la verificación al método de la Regla
- `$ability` debe ser un método de la clase con el mismo nombre


## Autorizando acciones mediante Reglas
### Mediante el modelo User

Con los métodos `can` y `cannot`, con dos parámetros:

- acción
- modelo

Suelen incluírse en un controlador:

```php
<?php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Update the given post.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Post $post)
    {
        if ($request->user()->cannot('update', $post)) {
            abort(403);
        }

        // Update the post...
    }

    /**
     * Crear un post (no necesita modelo, recibe la clase)
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        if ($request->user()->cannot('create', Post::class)) {
            abort(403);
        }

        // Create the post...
    }
}
?>
```

### Mediante los asistentes de Controlador

El el método `authorize()` mencionado antes:

```php
<?php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Update the given blog post.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Http\Response
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', $post);

        // The current user can update the blog post...
    }

    /**
     * Crea un nuevo post: no necesita modelo, recibe una clase
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }
}
?>
```

### Mediante Middleware

Laravel incluye un _middleware_ que puede autorizar acciones antes de que la petición alcance las rutas o los controladores. Por defecto, la clase `App\Http\Kernel` asigna la palabra clave `can` al _middleware_ `Illuminate\Auth\Middleware\Authorize`. Veamos un ejemplo para autorizar que un usuario actualice un post:


=== "web.php"
    ```php
    <?php
    use App\Models\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

    // si el método no necesita una instancia de modelo
    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Models\Post');
    ?>
    ```
    
### Mediante plantillas Blade

=== "can y cannot"
    ```blade.php
    @can('update', $post)
        <!-- The current user can update the post... -->
    @elsecan('create', App\Models\Post::class)
        <!-- The current user can create new posts... -->
    @else
        <!-- ... -->
    @endcan

    @cannot('update', $post)
        <!-- The current user cannot update the post... -->
    @elsecannot('create', App\Models\Post::class)
        <!-- The current user can now create new posts... -->
    @endcannot
    ```
=== "If y unless"
    Lo mismo pero con `if` y `unless`:

    ```blade.php
    @if (Auth::user()->can('update', $post))
        <!-- The current user can update the post... -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- The current user cannot update the post... -->
    @endunless
    ```
=== "canany"
    Autoriza un conjunto de acciones a la vez:

    ```blade.php
    @canany(['update', 'view', 'delete'], $post)
        <!-- The current user can update, view, or delete the post... -->
    @elsecanany(['create'], \App\Models\Post::class)
        <!-- The current user can create a post... -->
    @endcanany
    ```
=== "Acciones sin modelo"
    ```blade.php
    @can('create', App\Models\Post::class)
        <!-- The current user can create posts... -->
    @endcan

    @cannot('create', App\Models\Post::class)
        <!-- The current user can't create posts... -->
    @endcannot
    ```

### Proporcionar contexto adicional


```php
<?php
/**
 * Determine if the given post can be updated by the user.
 *
 * @param  \App\Models\User  $user
 * @param  \App\Models\  $post
 * @param  int  $category
 * @return bool
 */
public function update(User $user, Post $post, int $category)
{
    return $user->id === $post->user_id &&
           $user->canUpdateCategory($category);
}
?>
```

Para ver si puede actualizar un post en concreto:

```php
<?php
/**
 * Update the given blog post.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \App\Models\Post  $post
 * @return \Illuminate\Http\Response
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function update(Request $request, Post $post)
{
    $this->authorize('update', [$post, $request->category]);

    // The current user can update the blog post...
}
?>
```


