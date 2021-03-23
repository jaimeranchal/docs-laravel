# Crear modelos de datos

Con [Eloquent]() definir el modelo de datos se divide en varios pasos:

1. Crear un **modelo** con una _clase_ que defina los atributos y métodos de cada entidad, y cómo acceder a ellos.
2. Crear una **migración** con los _campos_ de la base de datos.
3. Implementar el archivo de migración para crear las tablas y sus relaciones en la base de datos con **artisan**

=== "Artisan"
    ```sh
    # crea una clase para el modelo
    php artisan make:model Vuelo

    # clase para el modelo + el archivo de migración 
    php artisan make:model Vuelo -m
    php artisan make:model Vuelo --migration

    # clase + controlador
    php artisan make:model Vuelo -c
    php artisan make:model Vuelo --controller

    # doble combo: clase + controlador + migración
    php artisan make:model Vuelo -mc
    ```

=== "Editar modelo"
    ```php
    <?php
    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Vuelo extends Model {
        /* Atributos */
        // --------------------------------------------------
        // La tabla se suele asociar directamente. Si no:
        protected $table = 'mis_vuelos';
        // Se asume una clave primaria 'id'. Si no:
        protected $primaryKey = 'vuelo_id';
        // Valores por defecto
        protected $attributes = [
            'retrasado' => false,
        ];
        
        /* Métodos */
        // --------------------------------------------------

    }
    ?>
    ```

=== "Editar migración"
    ```php
    <?php
    use Illuminate\Database\Migrations\Migration;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    class CreateVuelosTable extends Migration {
        public function up() {
            Schema::create('vuelos', function (Blueprint $table) {
                // columnas por defecto
                $table->id();
                $table->timestamps();
                // añadimos las columnas y el tipo
                $table->string('numero');
                $table->string('destino');
                $table->float('precio');
            });
        }
        public function down() {
            Schema::dropIfExists('vuelos');
        }
    }
        
    ?>
    ```
=== "Migrar"
    ```sh
    # aplica los cambios en la base de datos
    php artisan migrate
    ```

!!! info "Configuración previa"
    Los datos necesarios para que Laravel pueda interactuar con la base de datos se configuran en el archivo `.env`:

    ```sh
    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=laravel
    DB_USERNAME=root
    DB_PASSWORD=
    ```

