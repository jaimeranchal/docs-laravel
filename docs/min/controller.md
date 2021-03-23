# Controladores

Un controlador sirve para gestionar la lógica relacionada con algo en particular; muchas veces, con un **modelo**. Por ejemplo, una clase `UserController` manejaría las rutas relacionadas con _crear_, _mostrar_, _actualizar_ o _eliminar_ usuarios.

Los controladores se pueden crear con artisan y se guardan en `app/Http/Controllers`.

=== "Artisan"
    ```sh
    # pelao y mondao
    php artisan make:controller EjemploController
    # con métodos para un CRUD
    php artisan make:controller EjemploController --resource
    # vincula el controlador con un modelo ya existente
    php artisan make:controller EjemploController --resource  --model=Ejemplo

    # Combo: crea modelo Y controlador
    php artisan make:model Ejemplo -c
    ```
=== "Modelo"
    ```php
    <?php
    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Database\Eloquent\Model;

    class Ejemplo extends Model
    {
        use HasFactory;
        // campos rellenables
        protected $fillable = [
            'nombre',
            'descripción',
        ];
    }
    ?>
    ```
=== "Controlador"
    ```php
    <?php
    namespace App\Http\Controllers;

    use App\Models\Ejemplo;
    use Illuminate\Http\Request;

    class EjemploController extends Controller{
        public function index() { ... }

        public function create() { ... }    // formulario para insertar
        public function edit() { ... }      // formulario para editar

        public function store() { ... }     // insertar
        public function show() { ... }      // mostrar
        public function update() { ... }    // actualizar
        public function destroy() { ... }   // borrar
    }
    ?>
    ```

    Esto genera las siguientes rutas:

    | Verbo     | URI                        | Acción  | nombre de ruta   |
    |-----------|----------------------------|---------|------------------|
    | GET       | `/ejemplos`                | index   | ejemplos.index   |
    | GET       | `/ejemplos/create`         | create  | ejemplos.create  |
    | POST      | `/ejemplos`                | store   | ejemplos.store   |
    | GET       | `/ejemplos/{ejemplo}`      | show    | ejemplos.show    |
    | GET       | `/ejemplos/{ejemplo}/edit` | edit    | ejemplos.edit    |
    | PUT/PATCH | `/ejemplos/{ejemplo}`      | update  | ejemplos.update  |
    | DELETE    | `/ejemplos/{ejemplo}`      | destroy | ejemplos.destroy |

    **Recursos anidados**

    Si por ejemplo tienes una foto con muchos comentarios asociados, se pueden _anidar_ los controladores, para que se creen rutas estilo `/fotos/{foto}/comentarios/{comentario}`:

    ```php
    <?php
    use App\Http\Controllers\FotoCommentController;

    Route::resource('fotos.comentarios', FotoCommentController::class);
    ?>
    ```

=== "web.php" 
    ```php
    <?php
    use App\Http\Controllers\EjemploController;

    Route::resource('/ejemplos', EjemploController::class);
    ?>
    ```

