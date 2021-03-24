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

## Relaciones entre modelos

Lo que viene a ser **claves foráneas** y **multiplicidad**. Vamos a ver un ejemplo con cuatro _modelos_: "profesor", "alumno", "equipo" y "examen":

- Un profesor tiene:
    - varios alumnos (1:N)
    - varios exámenes (1:N)
- Un alumno tiene:
    - varios profesores (relación 1:N)
    - un sólo equipo (1:1)
    - varios exámenes (relación 1:N)
- Cada equipo pertenece solo a un alumno (1:1)
- Examen (N:M)
    - Un alumno tiene varios exámenes
    - El mismo examen lo pueden tener varios alumnos

Para representar todo esto, en las **migraciones** añadimos el _id_ de la tabla relacionada como una **clave foránea**, y en los **modelos** codificamos funciones que recuperan otros modelos relacionados:

=== "Migraciones"
    La norma es añadir la _id_ de la tabla a la que **pertenezca** un modelo; es decir, si en un modelo usamos `belongsTo(Alumno::class)` tendremos que añadir `alumno_id` como campo de tipo _entero_.

    === "CreateProfesorTable"
        ```php
        <?php
        class CreateProfesorTable extends Migration 
        {
            public function up() 
            {
                Schema::create('profesores'), function (Blueprint $table) {
                    // campos predefinidos...

                    // Como es el profesor al que "apuntan" alumnos y exámenes
                    // no añadimos nada
                }
            }
        }
        ?>
        ```
    === "CreateAlumnoTable"
        ```php
        <?php
        class CreateAlumnoTable extends Migration 
        {
            public function up() 
            {
                Schema::create('alumnos'), function (Blueprint $table) {
                    // campos predefinidos...

                    $table->integer('profesor_id');
                }
            }
        }
        ?>
        ```
    === "CreateEquipoTable"
        ```php
        <?php
        class CreateEquipoTable extends Migration 
        {
            public function up() 
            {
                Schema::create('equipos'), function (Blueprint $table) {
                    // campos predefinidos...

                    $table->integer('alumno_id');
                }
            }
        }
        ?>
        ```
    === "CreateExamenTable"
        ```php
        <?php
        class CreateExamenTable extends Migration 
        {
            public function up() 
            {
                Schema::create('examenes'), function (Blueprint $table) {
                    // campos predefinidos...

                    $table->integer('profesor_id');
                }
            }
        }
        ?>
        ```
=== "Modelos"
    === "Profesor.php"
        ```php
        <?php
        namespace App\Models;

        use Illuminate\Database\Eloquent\Factories\HasFactory;
        use Illuminate\Database\Eloquent\Model;
        use Illuminate\Database\Eloquent\SoftDeletes;

        class Profesor extends Model
        {
            // 1:N
            public function alumnos() {
                return $this->hasMany(Alumno::class);
            }

            // 1:N
            public function examenes() {
                return $this->hasMany(Examen::class);
            }
        }    
        ?>
        ```
    === "Alumno.php"
        ```php
        <?php
        namespace App\Models;

        use Illuminate\Database\Eloquent\Factories\HasFactory;
        use Illuminate\Database\Eloquent\Model;
        use Illuminate\Database\Eloquent\SoftDeletes;

        class Alumno extends Model
        {
            // 1:1
            public function equipo() {
                return $this->hasOne(Equipo::class);
            }

            // N:1
            public function profesor() {
                return $this->belongsTo(Profesor::class);
            }

            // N:M

            // Nota: withPivot se usa para devolver atributos extra de la tabla pivot
            public function examenes() {
                return $this->belongsToMany(Examen::class)
                    ->withPivot('nota', 'fecha');
            }
        }    
        ?>
        ```
    === "Equipo.php"
        ```php
        <?php
        namespace App\Models;

        use Illuminate\Database\Eloquent\Factories\HasFactory;
        use Illuminate\Database\Eloquent\Model;
        use Illuminate\Database\Eloquent\SoftDeletes;

        class Equipo extends Model
        {

            // 1:1
            public function alumno() {
                return $this->belongsTo(Alumno::class);
            }

        }    
        ?>
        ```
    === "Examen.php"
        ```php
        <?php
        namespace App\Models;

        use Illuminate\Database\Eloquent\Factories\HasFactory;
        use Illuminate\Database\Eloquent\Model;
        use Illuminate\Database\Eloquent\SoftDeletes;

        class Examen extends Model
        {
            // N:1
            public function profesor() {
                return $this->belongsTo(Profesor::class);
            }
            // N:M
            public function alumnos() {
                return $this->belongsToMany(Alumno::class);
            }
        }    
        ?>
        ```

## Relaciones Muchos a Muchos

Si queremos representar que un alumno puede tener muchos exámenes y a la vez, que un examen puede corresponder a varios alumnos, necesitamos una tabla intermedia que las relacione a través de sus **claves primarias**:

!!! note "Importante"
    El nombre de la tabla intermedia debe ser el nombre de las tablas relacionadas, ordenadas por orden alfabético, separadas por un guión bajo: `tabla1_tabla2`.

=== "Artisan"
    ```sh
    # creamos la tabla con una migración
    php artisan make:migration create_alumno_examen_table --create=alumno_examen
    ```
=== "Migración"
    ```php
    <?php
    // create_alumno_examen_table

    class CreateAlumnoExamenTable extends Migration 
    {
        public function up() 
        {
            Schema::create('alumno_examen'), function (Blueprint $table) {
                // campos predefinidos...

                // claves primarias de las tablas relacionadas
                $table->integer('alumno_id');
                $table->integer('examen_id');
                // campos extra (fecha, nota, etc)...
            }
        }
    }
    ?>
    ```

## Acceder a los datos

=== "Relaciones 1:1"
    ```php
    <?php
    use App\Models\Alumno
    use App\Models\Equipo

    // recuperar el equipo de un alumno
    $equipo = Alumno::find(1)->equipo;
    // mostrar el alumno que tiene asignado un equipo
    $alumno = Equipo::find(1)->alumno;
    ?>
    ```
=== "Relaciones 1:N"
    ```php
    <?php
    use App\Models\Alumno
    use App\Models\Profesor

    // Mostrar todos los alumnos de un profesor
    $alumnos = Profesor::find(1)->alumnos;

    foreach ($alumnos as $alumno) {
        // mostrar datos p.e
    }

    // Mostrar el nombre del profesor de un alumno
    $nombre_profesor = Alumno::find(1)->profesor->nombre;
    ?>
    ```
=== "Relaciones N:M"
    ```php
    <?php
    // buscar la nota del examen que hizo un alumno concreto
    use App\Models\Alumno;

    $alumno = Alumno::find(1);

    foreach ($alumno->examenes as $examen) {
        echo $examen->pivot->nota;
    }
    ?>
    ```
