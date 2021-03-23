# Vistas

Las vistas son la parte que el usuario ve, la **interfaz** y se construyen con [Blade](https://laravel.com/docs/8.x/blade)

## Mostrar datos

Una vista puede obtener datos como **parámetros** desde el método que la invoca, y también puede mostrar datos de **funciones PHP**:

=== "web.php"
    ```php
    <?php
    Route::get('/', function(){
        return view('bienvenida', ['nombre' => 'Jaime']);
    })
    ?>
    ```
=== "bienvenida.blade.php"
    ```blade
    {{-- Esto es un comentario --}}
    ¡Hola {{ nombre }}, la hora UNIX ahora mismo es {{ time() }}!
    ```

## Directivas

=== "Condiciones"
    ```blade
    @if (count($registros) === 1)
        Tengo un registro!
    @elseif (count($registros) > 1)
        Tengo varios!
    @else
        Todavía no tengo nada...
    @endif
    ```

    ```blade
    @unless (Auth::check())
        No has iniciado sesión
    @endunless
    ```
=== "Isset/empty"
    ```blade
    @isset($registros)
        // $registros está definido y no es nullo
    @endisset

    @empty($registros)
        // $registros está vacío...
    @endempty
    ```
=== "Auth"
    ```blade
    @auth('admin')
        // el usuario ha iniciado sesión como administrador...
    @endauth

    @guest('admin')
        // el usuario NO es administrador...
    @endguest
    ```
=== "Switch"
    ```blade
    @switch($i)
        @case(1)
            Primer caso...
            @break
        @case(2)
            Segundo caso...
            @break
        @default
            Caso por defecto...
    @endswitch
    ```
=== "Bucles"
    ```blade
    @for ($i = 0; $i < 10; $i++)
        El valor actual es {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>Este es el usuario {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>Sin usuarios</p>
    @endforelse

    @while (true)
        <p>Bucle infinito.</p>
    @endwhile
    ```

    Dentro de un bucle, Blade genera una variable `$loop` con [información útil](https://laravel.com/docs/8.x/blade#the-loop-variable).

## Layouts

La forma más sencilla de construir una interfaz consistente es definir una plantilla base, de la que hereden todas las demás, y a la que ir añadiendo componentes según sea necesario.

Las directivas que hay que conocer son:

- `@section`: define un bloque de contenido
- `@yield('plantilla')`: muestra los contenidos de la plantilla
- `@extends('plantilla')`

=== "base"
    ```blade
    <html>
        <head>
            <title>Aplicación - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                Barra lateral maestra
            @show

            <div class="container">
                @yield('contenido')
            </div>
        </body>
    </html>
    ```
=== "contenido"
    ```blade
    @extends('base')

    @section('title', 'Título de la página')

    @section('sidebar')
        @parent

        <p>Esto se añadirá al contenido de la barra lateral.</p>
    @endsection

    @section('contenido')
        <p>Este es el contenido del cuerpo.</p>
    @endsection
    ```

## Formularios

=== "Campos especiales"
    - [protección CSRF](https://laravel.com/docs/8.x/csrf)
    - Método (cuando no sea `post`)

    ```blade
    <form method="POST" action="/perfil">
        @csrf
        @method('PUT')
        ...
    </form>
    ```

=== "Errores"
    ```blade
    <label for="title">Título del post</label>
    <input id="title" type="text" class="@error('title') is-invalid @enderror">

    @error('title')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

    <label for="email">Email address</label>
    <input id="email" type="email" class="@error('email', 'login') is-invalid @enderror">

    @error('email', 'login')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror
    ```

## Componentes

Es una nueva cosa de Laravel 8. Mientras dejo un resumen aquí, ver la [documentación oficial](https://laravel.com/docs/8.x/blade#components)
