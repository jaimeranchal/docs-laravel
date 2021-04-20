# Componentes

Una forma de crear interfaces es descomponerlas en partes o _componentes_. Estos componentes suelen ser elementos **reutilizables**, con una misma estructura y aspecto pero diferentes valores o **propiedades**.

Bootstrap permite crear componentes del mismo modo que otros elementos, con **artisan**:

=== "Creación"
    === "consola"
        ```sh
        php artisan make:component Formulario
        # Añade dos archivos:
        # - app/view/components/Formulario.php
        # - resources/views/components/Formulario.blade.php

        php artisan make:components Formularios/Input
        # crea un componente 'input' dentro de la carpeta 'Formularios'
        ```
    === "app/.../input.php"
        ```php
        <?php
        namespace App\View\Components;
        use Illuminate\View\Component;

        class Input extends Component {

            /* Propiedades */
            public $tipo; // tipo del input
            public $valor; // valor del input

            /**
             * Crea la instancia del componente
             *
             * @param  string  $tipo
             * @param  string  $valor
             * @return void
             */
            public function __construct($tipo, $valor)
            {
                $this->tipo = $tipo;
                $this->message = $valor;
            }

            /**
             * Devuelve la vista que representa el componente
             *
             * @return \Illuminate\View\View|\Closure|string
             */
            public function render()
            {
                return view('components.input');
            }
        }
        ```
    === "resources/.../input.blade.php"
            ```blade
            <input type="{{ $tipo }}" value="{{ $valor }}"/>
        ```

    !!! note "camelCase y kebab-case"
        Las propiedes de un componente se indican con _camelCase_, pero en las plantillas blade se debe usar _kebab-case_

=== "Mostrar"
    Dentro de una **plantilla blade**:
    ```blade
    <x-formulario/>
    <x-formularios.input/>

    <!-- pasandole datos -->
    <x-formularios.input type="text" :value="$valor" />
    ```

## Métodos

Por hacer...

## Atributos adicionales

Todos los atributos que no estén definidos como propiedades de la clase se guardan en la variable `$attributes`. Si hemos definido un **valor por defecto** para un atributo, una _clase css_, por ejemplo, y queremos añadir más, usamos `merge` para _sumarlo_ a los ya existentes.

Por ejemplo, hemos definido dos clases css para un `div` y en la vista queremos añadirle un poco de margen:

=== "Componente"
    === "Clase"
        ```php
        <?php
        class Alert extends Component {
            public $type;
            public $message;

            public function render() 
            {
                return view('components.alert');
            }
        }
        ```
    === "Plantilla"
        ```blade
        <div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
            {{ $message }}
        </div>
        
        ```
=== "Vista"
    ```blade
    <x-alert type="error" :message="$message" class="mb-4" />
    ```
=== "Html final"
    ```html
    <div class="alert alert-error mb-4">
        <!-- contenido del componente -->
    </div>
    ```

## Slots

Si queremos añadir contenido más complejo, que no sea adecuado insertar como propiedad, usamos `slots`:

=== "Plantilla"
    ```blade
    <x-alert>
        <x-slot name="title">
            Título 1 (slot con nombre = variable)
        </x-slot>

        Texto del mensaje de alerta (slot anónimo)
    </x-alert>
    ```
=== "alert.blade.php"
    ```blade
    <div {{$attributes->merge(['class' => "bg-$color-100 border-1-4 border-$color-500"])}} role="alert">
        <p class="font-bold">{{$title}}</p>
        <p>{{$slot}}</p>
    </div>
    ```
=== "Alert.php"
    ```php
    <?php
    class Alert extends Component
    {
        public $color

        public function __construct($color = "verde")
        {
            $this->color = $color;
        }

        public function render()
        {
            return view('components.alert');
        }
    }
    ```

