# Operaciones básicas con datos

La lógica para _leer_, _insertar_, _editar_ o _eliminar_ los datos de la base de datos está en el **controlador** asociado al **modelo** sobre el que queremos trabajar.

Si tenemos un modelo de _vuelos_, y el controlador correspondiente:

```php
<?php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Vuelo;
use Illuminate\Http\Request;

class VueloController extends Controller
{
    // funciones...
}
?>
```

=== "Crear"
    Se hace con la **función `store()`**. Tenemos dos opciones:

    - con `save()`
    - con `create()`; requiere que los atributos sean _sobreescribibles_

    === "save()"
        ```php
        <?php
        use App\Models\Vuelo;

        // guarda un vuelo
        public function store(Request $request)
        {
            // Validar los datos que entran del formulario ($request)

            $vuelo = new Vuelo;

            $vuelo->numero = $request->numero;
            $vuelo->destino = $request->destino;

            $vuelo->save();
        }
        ?>
        ```
    === "create()"
        ```php
        <?php
        // permite que se escriban los campos
        protected $fillable = [
            'numero',
            'destino',
        ];

        // guarda un vuelo
        public function store(Request $request)
        {
            // Validar los datos que entran del formulario ($request)

            $vuelo = Vuelo::create([
                'numero' => $request->name,
                'destino' => $request->destino,
            ]);

        }
        ?>
        ```
=== "Leer"
    Se pueden recuperar registros únicos o varios de golpe:

    - un sólo registro: `find()`, `first()`, `last()`
    - varios registros: `all()`, `get()`

    === "Resultado múltiple"
        - Con `all()`: todos los datos.
        - Con `get()`: solo los que cumplan ciertas condiciones

        ```php
        <?php
        use App\Models\Vuelo;

        // Muestra una lista de recursos
        public function index()
        {
            // Recupera todos los vuelos
            $vuelos = Vuelo::all();

            // Recupera los que cumplan ciertas condiciones
            $vuelos = Vuelo::where('activo', 1)
                    ->orderBy('nombre')
                    ->take(10)
                    ->get();

        }
        ```

    === "Resultado único"
        ```php
        <?php
        <?php
        use App\Models\Vuelo;

        // Muestra los datos de un recurso concreto
        public function show(){
            $vuelo = Vuelo::find(3); // por su id
            $vuelo = Vuelo::first(); // el primero

            // El primero con una condición
            $vuelo = Vuelo::where('numero', 'FR 900')->first(); 
            $vuelo = Vuelo::firstWhere('numero', 'FR 900'); // alternativa

            // hacer algo diferente si NO ENCUENTRA el resultado
            $vuelo = Vuelo::where('numero', 'ES 450')->firstOr(function() { ... });

            // Si quieres lanzar una excepción en caso de no encontrar nada
            $vuelo = Vuelo::findOrFail($id);
        }

        // o también
        Route::get('/api/vuelos/{numero}', function($numero){
            $vuelo = Vuelo::where('numero', $numero)->firstOrFail();
        })

        ```
    !!! note "Colecciones"
        Cuando ejecutas una consulta, Laravel devuelve objetos tipo **colección**. Como implementan _Iterable_ se pueden usar como si fueran **arrays**, y además incluyen algunos [métodos extra](https://laravel.com/docs/8.x/eloquent-collections#available-methods); por ejemplo:

        ```php
        <?php
        $usuarios = User::all();

        // búsqueda por clave primaria
        $usuario = $usuarios->find(1);
        // solo los modelos que tengan ciertas claves primarias
        $usuarios = $usuarios->only([1,2,3]);
        // devuelve todas las claves primarias
        $usuarios->modelKeys();
        ?>
        ```

=== "Leer (campos)"
    También se puede recuperar únicamente el valor de algunos campos:

    ```php
    <?php
    // Destino del vuelo nº FR900
    $vuelo = Vuelo::where('numero', 'FR 900')->value('destino');

    // Todos los destinos
    $destinos = Vuelo::pluck('destino');
    ```

    Laravel incluye filtros similares a las funciones de SQL:

    === "Agregación"
        ```php
        <?php
        /* Funciones de agregación */
        $total_vuelos = Vuelo::count();
        $maximo_pasajeros = Vuelo::max('pasajeros');
        $media_pasajeros = Vuelo::avg('pasajeros');

        /* Determinar si un registro existe */
        if (Vuelo::where('numero', 'ES 455')->exists()) { ... }
        // ... o no existe
        if (Vuelo::where('numero', 'ES 455')->doesntExist()) { ... }
        ?>
        ```
    === "Where"
        - Tres argumentos: _campo_, _operador_ y _valor_; admite cualquier operador sql (`= < > <= >= <> like`...)
        - Dos argumentos si el operador es '='

        ```php
        <?php
        $usuarios = User::where('peso', '=', 100)
                    ->where('edad', '>', 20)
                    ->get();

        // se pueden pasar varias condiciones con un array
        $usuarios = User::where([
            ['peso', '=', 100],
            ['edad', '>', 20],
        ])->get();

        // condiciones alternativas (OR)
        $usuarios = User::where('peso', '>', 100)
            ->orWhere(function($query) {
                $query->where('altura', '<', 190)
                    ->where('peso', '>', 80);
            })->get();

            // Si es una alternativa compleja (x OR (y + z))
            // el segundo grupo de condiciones se agrupa
            // con una función anónima
        ?>
        ```
    === "Rangos"
        ```php
        <?php
        // vuelos entre 20 y 70 pasajeros
        $vuelos_menores = Vuelo::whereBetween('pasajeros', [20, 70])->get();
        // vuelos con menos de 20 o más de 70 pasajeros
        $vuelos_mayores = Vuelo::whereNotBetween('pasajeros', [20, 70])->get();

        // usuarios cuya id sea exactamente 1,2 o 3
        $usuarios = User::whereIn('id', [1,2,3])->get();
        // usuarios cuya id NO sea 1,2 o 3
        $usuarios = User::whereNotIn('id', [1,2,3])->get();
        ?>
        ```
    === "Nulos"
        ```php
        <?php
        // Ver si un campo es nulo
        $usuarios = User::whereNull('updated_at')->get();
        // Ver si un campo NO es nulo
        $usuarios = User::whereNotNull('updated_at')->get();
        ?>
        ```
    === "Fecha/Hora"
        ```php
        <?php
        // Fecha igual a...
        $vuelos = Vuelo::whereDate('partida', '2016-12-31')->get();

        // usando solo mes, dia o año
        $vuelos = Vuelo::whereMonth('partida', '12')->get();
        $vuelos = Vuelo::whereDay('partida', '31')->get();
        $vuelos = Vuelo::whereYear('partida', '2016')->get();

        // Hora
        $vuelos_retrasados = Vuelo::whereTime('salida', '>', '11:30:25')->get();
        ?>
        ```
    === "Select"
        ```php
        <?php
        // seleccionar solo algunos campos, asignandole un alias
        $vuelos = Vuelo::select('destino', 'pasajeros as ocupacion')->get();

        // solo resultados diferentes
        $vuelos = Vuelo::distinct()->get();

        // añadir una columna a una selección previa
        $destinos = Vuelo::select('destino')->distinct()->get();
        $vuelos = $destinos->addSelect('numero')->get();
        ?>
        ```
=== "Releer"
    ```php
    <?php
    // Refrescar datos cuando ya hemos creado una instancia
    $vueloRefrescado = $vuelo->fresh(); // no afecta a la instancia 'vuelo'

    // 'resetea' los cambios realizado a una instancia volviendo a leer desde la base de datos
    $vuelo->numero = 'FR 456';
    $vuelo->refresh();
    $vuelo->numero; // "FR 900"
    ```
=== "Actualizar"
    ```php
    <?php
    use App\Models\Vuelo;

    // Actualiza un vuelo
    public function update(Request $request, Vuelo $vuelo)
    {
        // actualiza un único vuelo
        $vuelo_data = $request->validate([
            'nombre'=>'required|max:100',
            'destino'=>'required|max:255',
            'precio'=>'required|numeric'
        ]);

        $vuelo->update($vuelo_data);

        // actualizar varias a la vez
        Vuelo::where('activo', 1)
            ->where('destino', 'Sevilla')
            ->update(['retrasado' => 1]);
    }
    ?>
    ```

    **Upserts**

    A veces te interesa **crear** un modelo si al actualizar no existe previamente (_update_ + _insert_):

    ```php
    <?php
    // Un único upsert
    $vuelo = Vuelo::updateOrCreate(
        // actualiza (o crea) un vuelo con estas condiciones
        ['salida' => 'Granada', 'destino' => 'Barcelona'],
        // actualiza (o crea) estos campos
        ['precio' => 78, 'descuento' => 1]
    );

    // varios a la vez
    Vuelo::upsert(
        // valores a insertar / actualizar
        [
            ['salida' => 'Granada', 'destino' => 'Barcelona', 'precio' => 90],
            ['salida' => 'Malaga', 'destino' => 'London', 'precio' => 150],
            ['salida' => 'Malaga', 'destino' => 'Atenas', 'precio' => 375],
        ],
        // columnas que identifican los registros (DEBEN ser UNIQUE o PRIMARY)
        ['salida', 'destino'],
        // columna cuyo valor quiero actualizar
        ['precio']);
    ?>
    ```

    **Clonar modelos**

    Útil si tienes varias instancias de modelos que comparten casi todos los atributos:

    ```php
    <?php
    use App\Models\Direccion;

    $envio = Direccion::create([
        'tipo' => 'envio',
        'linea_1' => '123 Calle Ejemplo',
        'ciudad' => 'Jaimevilla',
        'provincia' => 'CO',
        'cp' => '14001',
    ]);

    $factura = $envio->replicate()->fill([
        'tipo' => 'factura'
    ]);

    $factura->save();
    ?>
    ```
=== "Delete"
    ```php
    <?php
    use App\Models\Vuelo;

    // elimina un vuelo
    public function destroy() {
        $vuelo = Vuelo::find(1);

        $vuelo->delete();

        // Si quieres eliminar los datos asociados en la base de datos
        // también resetea las id's autoincrementadas que existieran
        Vuelo::truncate();

        // borrar sin crear una instancia primero
        Vuelo::destroy(1);
        Vuelo::destroy(1,2,3);
        Vuelo::destroy([1,2,3]);
    }
    ?>
    ```

    **"Borrado suave" (soft deleting)**

    Esto no elimina los datos de la base de datos, sino que los _esconde_ y guarda la fecha de "borrado" en el campo _deleted_at_:

    === "app/Models/Vuelo.php"
        ```php
        <?php
        namespace App\Models;

        use Illuminate\Database\Eloquent\Model;
        use Illuminate\Database\Eloquent\SoftDeletes;

        class Vuelo extends Model
        {
            // sin esto no funciona
            use SoftDeletes;
        }
        ?>
        ```
    === "database/migrations/CreateVuelosTable.php"
        ```php
        <?php
        use Illuminate\Support\Facades\Schema;
        use Illuminate\Database\Schema\Blueprint;
        use Illuminate\Database\Migrations\Migration;

        class CreateVuelosTable extends Migration
        {
            /**
             * Run the migrations.
             */
            public function up()
            {
                Schema::create('vuelos', function (Blueprint $table) {
                    // ...
                    $table->softDeletes(); // crea la columna `deleted_at`
                    $table->timestamps();
                });
            }

            /**
             * Reverse the migrations.
             */
            public function down()
            {
                // Eliminar el campo `deleted_at` si fuera necesario
                Schema::table('vuelos', function(Blueprint $table) {
                    $table->dropSoftDeletes();
                });

                Schema::dropIfExists('vuelos');
            }
        }
        ?>
        ```

    Para listar los registros borrados de esta forma:

    ```php
    <?php
    $vuelos = Vuelo::withTrashed()->where('destino', 'Sevilla')->get();

    // solo los que han sido borrados
    $vuelos = Vuelo::onlyTrashed()->where('destino', 'Sevilla')->get();
    ?>
    ```


    Si quieres restaurar un registro borrado de esta forma:

    ```php
    <?php
    if ($vuelo->trashed()){
        $vuelo->restore();
    }

    // alternativa con condición
    Vuelo::withTrashed()->where('airline_id', 2)->restore();
    ?>
    ```

    Para borrarlo del todo

    ```php
    <?php
    $vuelo->forceDelete();
    ?>
    ```

