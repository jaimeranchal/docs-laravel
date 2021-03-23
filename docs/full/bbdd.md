# Manejando bases de datos con Laravel

## Migrations

- crear, actualizar y borrar bases de datos

## Consultas SQL Nativas

Utilizando la **interfaz DB** y sus cuatro _métodos estáticos_:

- `DB::insert()`
- `DB::update()`
- `DB::select()`
- `DB::delete()`

## Laravel

Nomenclatura:

- Modelo = nombre de la tabla en singular empezando con Mayúscula
    - propiedades
    - métodos

```sh
# crea una clase con el modelo
php artisan make:model Nombre
# crea una clase con el modelo la migración correspondiente
php artisan make:model Nombre -m
php artisan make:model Nombre --migration
```

