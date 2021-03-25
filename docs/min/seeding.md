# Seeding

Laravel permite rellenar la base de datos con datos de prueba, usando clases llamadas _seeders_ (del inglés _seed_, "sembrar").

- Estas se guardan en `database/seeders`.
- Por defecto, Laravel incluye una clase `DatabaseSeeder` con un método `run()`. En este método codificamos qué queremos introducir y cómo.

=== "Artisan"
    ```sh
    # Ejecuta todos los seeders en DatabaseSeeder
    php artisan db:seed
    # Ejecuta los seeds que quieras
    php artisan db:seed --class=UserSeeder

    # Combo: migración + seeders (cuidado que borra todo lo previo)
    php artisan migrate:fresh --seed
    ```
=== "DatabaseSeeder"
    Desde aquí se ejecutan los seeders que digamos para introducir información en la base de datos. Se puede hacer de varias maneras:

    1. Usando la clase `DB` e insertando directamente los datos
    2. Llamando a la **factoría** de un Modelo o varios
    3. Llamando a otros seeders definidos en archivos separados con el método `call()`.
    === "DB::table()->insert"
        ```php
        <?php
        class DatabaseSeeder extends Seeder
        {
            /**
             * Run the database seeders.
             */
            public function run()
            {
                DB::table('users')->insert([
                    'name' => Str::random(10),
                    'email' => Str::random(10).'@gmail.com',
                    'password' => Hash::make('password'),
                ]);
            }
        }
        ?>
        ```
    === "Model::factory()"
        ```php
        <?php
        class DatabaseSeeder extends Seeder
        {
            use App\Models\User;
            /**
             * Run the database seeders.
             */
            public function run()
            {
                public function run()
                {
                    User::factory()
                            ->count(50)
                            ->hasPosts(1)
                            ->create();
                }
            }
        }
        ?>
        ```
    === "call(Seeder::class)"
        ```php
        <?php
        class DatabaseSeeder extends Seeder
        {
            /**
             * Run the database seeders.
             */
            public function run()
            {
                $this->call([
                    UserSeeder::class,
                    PostSeeder::class,
                    CommentSeeder::class,
                ]);
            }
        }
        ?>
        ```

## _Seeders_ y Factorías

Si hay que insertar datos de varios modelos, lo suyo es crear _seeders_ separados e invocar los que necesitemos desde `DatabaseSeeder.php`. Cada _seeder_ utiliza una **clase factoría** para crear tantas instancias del modelo como quieras, con los **parámetros** que quieras.

Para crear 50 usuarios:

1. Creamos un _seeder_ para el modelo **User**
2. Modificamos los parámetros del _seeder_ como necesitemos.
3. Si queremos cambiar los parámetros de _creación_ del modelo (cómo se generan los nombres, email, contraseñas...) hay que modificar la **factoría** correspondiente a ese modelo.

=== "Artisan"
    ```sh
    # Crea un seeder en blanco para el modelo User
    php artisan make:seeder UserSeeder
    ```
=== "UserSeeder.php"
    ```php
    <?php
    use App\Models\User;

    /**
     * Run the database seeders.
     */
    public function run()
    {
        // crea 50 usuarios con 1 Post cada uno
        User::factory()
                ->count(50)
                ->hasPosts(1)
                ->create();
    }
    ?>
    ```
=== "UserFactory.php"
    ```php
    <?php
    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Str;

    class UserFactory extends Factory
    {
        /* El nombre del modelo al que se corresponde la factoría */
        protected $model = User::class;

        /* Define el "estado" por defecto del modelo */
        public function definition()
        {
            return [
                'name' => $this->faker->name,
                'email' => $this->faker->unique()->safeEmail,
                'email_verified_at' => now(),
                'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 
                // 'password' => Hash::make('tu_contraseña'),
                'remember_token' => Str::random(10),
            ];
        }

        /* Indica que el email del modelo no debe verificarse */
        public function unverified()
        {
            return $this->state(function (array $attributes) {
                return [
                    'email_verified_at' => null,
                ];
            });
        }
    }
    ?>
    ```

### Crear y personalizar Factorías

- Como todo, se crean con `artisan`
- Las parámetros generales se ajustan en el método `definition()`:
    - Laravel incluye la librería php [Faker](https://github.com/FakerPHP/Faker) que permite generar **datos aleatorios** (nombres, números, calles, emails...).
- Si queremos hacer algún ajuste particular sólo a algunos modelos, que no sea aleatorio (p.e., crear 5 usuarios cliente y solo 2 administradores) hay que manipular el **estado** de la factoría:
    - Esto se hace con el método `state()`.

=== "Artisan"
    ```sh
    # Crea una factoría 
    php artisan make:factory PostFactory
    # Crea una factoría con los atributos de un modelo
    php artisan make:factory PostFactory --model=Post
    ```
=== "Parámetros"
    La función `definition()` devuelve una array con los atributos y valores del Modelo:

    ```php
    <?php
    class PostFactory extends Factory
    {
        /* El nombre del modelo al que se corresponde la factoría */
        protected $model = Post::class;

        /* Define el "estado" por defecto del modelo */
        public function definition()
        {
            // array asociativo; aplica las funciones que necesites en el valor
            return [
                'titulo' => $this->faker->name,
                'fecha_post' => now(),
                'contenido' => Str::random(100),
                // ...
            ];
        }
    }
    ?>
    ```
=== "Estados"
    Si queremos que algunas intancias tengan atributos diferentes, creamos una **función** con el **nombre del atributo**. Dentro de esa función se usa el método `state()` para cambiar el valor del campo en cuestión:

    ```php
    <?php
    class UserFactory extends Factory
    {
        // Indica que el usuario es administrador
        public function admin()
        {
            return $this->state(function (array $attributes){
                return [
                    'role' => 1,
                ];
            });
        }

        // Indica que el usuario es cliente
        public function cliente()
        {
            return $this->state(function (array $attributes){
                return [
                    'role' => 2,
                ];
            });
        }
    }
    ?>
    ```
=== "Instanciar"
    Dentro de un _seeder_ podemos invocar el método `factory()` para crear instancias del modelo:

    ```php
    <?php
    // Un usuario
    $user = User::factory()->make();

    // 3 usuarios
    $users = User::factory()->count(3)->make();

    // 2 usuarios administradores
    $users = User::factory()->count(2)->admin()->make();
    ?>
    ```

### Cómo usar Faker

!!! info "Documentación"
    Siempre es bueno consultar la [documentación oficial](https://fakerphp.github.io)

Lo primero es cambiar la **localización** (el idioma) en el que se van a generar los datos. Esto se hace en `config/app.php`:

```php
<?php
/*
|--------------------------------------------------------------------------
| Faker Locale
|--------------------------------------------------------------------------
|
| This locale will be used by the Faker PHP library when generating fake
| data for your database seeds. For example, this will be used to get
| localized telephone numbers, street address information and more.
|
*/
'faker_locale' => 'es_ES',
?>
```

Los métodos para crear diferentes tipos de datos se llaman **formateadores**; aparte, hay tres **modificadores** generales que se pueden aplicar a todos:

=== "Formateadores"
    === "Persona"
        ```php
        <?php
        name($genero = null|'male' | 'female')      // nombre completo
        firstName($genero = null|'male' | 'female') // nombre
        firstNameMale()                             // nombre masculino
        firstNameFemale()                           // nombre femenino
        lastName()                                  // Apellido
        dni()                                       // dni
        ?>
        ```
    === "Dirección"
        ```php
        <?php
        address()        // dirección completa
        streetPrefix()   // tipo de vía
        streetName()     // calle
        buildingNumber() // número
        state()          // provincia
        city()           // ciudad
        postcode()       // Código Postal
        country()        // país
        ?>
        ```
    === "Tfno"
        ```php
        <?php
        phoneNumber()    // número de teléfono
        // adaptado a España
        tollFreeNumber() // fijo
        mobileNumber()   // móvil
        ?>
        ```
    === "Números"
        ```php
        <?php
        randomDigit()                        // aleatorio entre 0 y 10
        randomDigitNot($numero)              // aleatorio entre 0 y 10 menos $numero
        randomNumber($desde, $numeroDigitos) // aleatorio entre $desde y con un máximo de $numeroDigitos
        randomFloat($decimales)              // float aleatorio con x decimales
        numberBetween($min, $max)            // número entre min y max
        ?>
        ```
    === "Texto"
        ```php
        <?php
        randomElements($array, $numero) // un $numero de elementos aleatorios de un $array
        numerify('usuario-####')        // genera una cadena con tantos dígitos aleatorios como # haya
        lexify('id-????')               // cadena con tantas letras aleatorias como ? haya
        regexify($regex)                // genera una cadena aleatoria basada en una expresión regular
        ```
        ```php
        <?php
        // caracteres
        randomLetter() // letra aleatoria

        /* Sacado del Lorem Ipsum */

        // palabras
        word();                  // una palabra
        words($x)                // array con x palabras
        words($x, true)          // cadena con x palabras
                                 // frases
        sentence()               // una frase
        sentence($palabras)      // una frase con x palabras
        sentences($frases)       // array de x frases; 3 si se deja en blanco
        sentences($frases, true) // cadena con x frases
                                 // texto
        paragraph($frases)       // párrafo con x frases
        paragraphs($frases)      // Array de párrafos con x frases
        text($palabras);         // cadena de texto; por defecto 200 palabras
        ?>
        ```
    === "Fecha y Hora"
        ```php
        <?php
        unixTime()                            // fecha unix entre 0 y "ahora"
        dateTime()                            // fecha entre 1 de enero de 1970 y ahora
        dateTimeBetween('-1 week', '+1 week') // fecha entre x e y; por defecto -30 años y ahora
        date($formato = 'Y-m-d')              // fecha con formato entre 1-1-1970 y ahora
        time($formato = 'H:i:s')              // hora con formato
        dayOfMonth()                          // día del mes
        dayOfWeek()                           // día de la semana
        month()                               // número de mes
        monthName()                           // nombre del mes
        year()                                // año
        ?>
        ```
        
    === "Internet"
        ```php
        <?php
        url()                                  // una dirección web
        user_name()                            // nombre de usuario
        email($dominio)                        // CUIDADO (puede generar correos reales)
        company_email()                        // rollo raquel@morales.net así con apellidos
        safe_email()                           // una dirección email que no puede ser real
        image_url($ancho = none, $alto = none)
        password($min, $max)                   // contraseña entre x e y caracteres; por defecto 6 y 20
        ?>
        ```
=== "Modificadores"
    Son tres:

    - `unique()`
    - `optional()`
    - `valid()`
