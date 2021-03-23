# Fundamentos de Composer

```sh
# crea directorio con los componentes básicos
composer create-project laravel/laravel nombre 
# alternativa: instalador de Laravel como dependencia global de Composer
    composer global require laravel/installer

    laravel new nombre_app
    laravel new nombre_app --git # crea automáticamente un repositorio
    laravel new nombre_app --github # crea automáticamente un repo en Github
    laravel new nombre_app --github="--public --team laravel" # admite opciones de Github cli
```

Para mejorar el acceso remoto a los archivos:

```sh
# Mejor usar github-cli con ssh:
gh config set git_protocol ssh --host github.com
```

