# Laboratorio #2: CRUD rápido en Laravel

> Acercamiento a un CRUD rápido con Laravel utilizando migraciones, un generador de código (`ibex/crud-generator`), rutas tipo *resource* y autenticación con Laravel Breeze.

| | |
|---|---|
| **Estudiante** | Gabriel Chifundo |
| **Cédula** | 3-760-1894 |
| **Curso** | Desarrollo Web |
| **Universidad** | Universidad Tecnológica de Panamá |
| **Profesora** | Ing. Irina Fong |
| **Fecha** | 23/10/2026 |

---

## Tabla de contenido

1. [Objetivo](#1-objetivo)
2. [Requisitos previos](#2-requisitos-previos)
3. [Creación del proyecto](#3-creación-del-proyecto)
4. [Configuración del entorno (`.env`)](#4-configuración-del-entorno-env)
5. [Error de longitud de cadena (`Schema`)](#5-error-de-longitud-de-cadena-schema)
6. [Modelo y migración de `Product`](#6-modelo-y-migración-de-product)
7. [Generación del CRUD](#7-generación-del-crud)
8. [Rutas](#8-rutas)
9. [Autoload de Composer](#9-autoload-de-composer)
10. [Frontend y autenticación (Laravel Breeze)](#10-frontend-y-autenticación-laravel-breeze)
11. [Errores comunes y soluciones](#11-errores-comunes-y-soluciones)
12. [Conceptos adicionales: `$fillable` y `$guarded`](#12-conceptos-adicionales-fillable-y-guarded)
13. [Resultado final](#13-resultado-final)
14. [Referencias](#14-referencias)

---

## 1. Objetivo

Desarrollar un CRUD (*Create, Read, Update, Delete*) completo para la entidad **Product** en Laravel, aprendiendo a:

- Crear un proyecto Laravel y configurar la conexión a la base de datos.
- Crear modelos y migraciones con Artisan.
- Generar automáticamente controlador, vistas y rutas con un paquete generador de CRUD.
- Proteger el sistema con login y registro usando Laravel Breeze.
- Resolver los errores típicos que aparecen durante el proceso.

## 2. Requisitos previos

| Herramienta | Uso | Estado |
|---|---|---|
| WAMP (Apache + MySQL + PHP) | Servidor local | Completado |
| Composer | Gestor de dependencias de PHP | Completado |
| Laravel Installer | Crear el proyecto con `laravel new` | Completado |
| Node.js y npm | Compilar assets con Vite | Completado |
| phpMyAdmin | Administrar la base de datos | Completado |
| VS Code | Edición del código | Completado |

> Versión de Laravel utilizada: `13.37.0`

## 3. Creación del proyecto

Desde la carpeta `www` de WAMP:

```bash
cd C:\wamp64\www
laravel new crud_rapido
```

Esperar a que termine de cargar la instalación. Al finalizar, Laravel indica que la aplicación está lista.

Para levantar el servidor de desarrollo:

```bash
php artisan serve
```

La aplicación queda disponible en `http://127.0.0.1:8000`. Esta es la pantalla de bienvenida de **Laravel 13** (versión `v13.37.0`), con los accesos de *Log in* y *Register*:

![Pantalla de bienvenida de Laravel 13](assets/capturas/01-bienvenida-laravel.png)

### Estructura del proyecto

Estructura de carpetas generada por Laravel en el editor:

![Estructura del proyecto](assets/capturas/09-estructura-proyecto.png)

## 4. Configuración del entorno (`.env`)

Antes de ejecutar las migraciones, verificar que el archivo `.env` tenga las credenciales correctas de la base de datos:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=crud_rapido
DB_USERNAME=root
DB_PASSWORD=
```

La base de datos `crud_rapido` debe existir previamente en MySQL (se crea desde phpMyAdmin).

## 5. Error de longitud de cadena (`Schema`)

Al correr las migraciones en algunas versiones de MySQL puede aparecer un error por la longitud máxima de las columnas string/índices. Se resuelve en `app/Providers/AppServiceProvider.php`:

```php
use Illuminate\Support\Facades\Schema; // IMPORTANTE: añadir esta línea

public function boot(): void
{
    Schema::defaultStringLength(191);
}
```

> `Schema` es la clase que permite crear, modificar y eliminar tablas en la base de datos. Forma parte del sistema de migraciones de Laravel.

## 6. Modelo y migración de `Product`

### 6.1 Crear el modelo con su migración

```bash
php artisan make:model Product -m
```

- Crea el modelo `Product` en `app/Models/`.
- La opción `-m` genera además la migración de la tabla `products` en `database/migrations/`.

### 6.2 Definir los campos de la tabla

En el archivo de migración `create_products_table.php`:

```php
public function up(): void
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('description');
        $table->double('price', 8, 2); // 8 dígitos en total, 2 decimales
        $table->integer('stock');
        $table->timestamps();
    });
}
```

### 6.3 Ejecutar la migración

```bash
php artisan migrate
```

### 6.4 Comandos útiles de migración

| Comando | Descripción |
|---|---|
| `php artisan migrate` | Ejecuta las migraciones pendientes |
| `php artisan migrate:fresh` | Elimina **todas** las tablas y ejecuta las migraciones desde cero (estado limpio, destructivo) |
| `php artisan migrate:rollback` | Revierte solo el último lote de migraciones (más seguro y menos destructivo) |
| `php artisan config:clear` | Limpia la caché de configuración. Laravel guarda la configuración de `config/` en `bootstrap/cache/config.php`; este comando obliga a usar los valores actualizados (útil tras cambiar el `.env`) |

## 7. Generación del CRUD

Flujo en tres pasos con el paquete `ibex/crud-generator`, que genera automáticamente **controladores, vistas, modelos, migraciones y rutas**.

### 7.1 Instalar el paquete

```bash
composer require ibex/crud-generator --dev
```

### 7.2 Publicar los archivos del paquete

```bash
php artisan vendor:publish --tag=crud
```

Publica los archivos registrados con el tag `crud` (configuración, vistas, plantillas, etc.) desde `vendor/` hacia tu aplicación para poder usarlos y modificarlos.

### 7.3 Ejecutar el generador

```bash
php artisan make:crud products
```

`products` es el nombre de la tabla sobre la que se genera el CRUD.

## 8. Rutas

En `routes/web.php`:

```php
use App\Http\Controllers\ProductController;

Route::resource('products', ProductController::class);
```

Esta única línea define automáticamente todas las rutas del CRUD:

| Método | URI | Nombre | Acción |
|---|---|---|---|
| GET | `/products` | `products.index` | Listar |
| GET | `/products/create` | `products.create` | Formulario de creación |
| POST | `/products` | `products.store` | Guardar |
| GET | `/products/{product}` | `products.show` | Ver detalle |
| GET | `/products/{product}/edit` | `products.edit` | Formulario de edición |
| PUT/PATCH | `/products/{product}` | `products.update` | Actualizar |
| DELETE | `/products/{product}` | `products.destroy` | Eliminar |

Para verificar las rutas registradas:

```bash
php artisan route:list
```

## 9. Autoload de Composer

Si Laravel no reconoce el controlador o las clases recién generadas:

```bash
composer dump-autoload
```

Actualiza el mapa de clases PHP cuando se crean, mueven o eliminan archivos. Es necesario al añadir nuevas clases, cambiar el `autoload` de `composer.json` o mover archivos. Con `composer dump-autoload -o` se optimiza el cargador (recomendado para producción).

## 10. Frontend y autenticación (Laravel Breeze)

Para que las vistas tengan estilos y el sistema cuente con **login y registro**, se utilizó **Laravel Breeze** con Blade. Es el sustituto moderno y oficial de `laravel/ui`: funciona de forma nativa con Vite y Axios, publica las vistas en `resources/views/auth/` (Blade puro, fácil de modificar) y usa Tailwind CSS.

```bash
composer require laravel/breeze --dev
php artisan breeze:install
```

Opciones del instalador interactivo (para mantenerlo simple):

- **Breeze stack:** `blade`
- **Dark mode support:** `no`
- **Testing framework:** `Pest` o `PHPUnit` (indistinto para el laboratorio)

Luego compilar los assets:

```bash
npm install
npm run build
```

> Alternativa vista en clase: `composer require laravel/ui` y `php artisan ui bootstrap --auth`. Breeze evita los problemas de compilación y de paquetes heredados que suelen aparecer con `laravel/ui`.

### Inicio de sesión

Pantalla de login generada por Breeze:

![Pantalla de login](assets/capturas/02-login.png)

### Dashboard

Una vez autenticado, el usuario llega al dashboard, personalizado con un resumen del inventario: total de productos, stock total y valor del inventario, además del acceso a la lista de productos.

![Dashboard](assets/capturas/03-dashboard.png)

## 11. Errores comunes y soluciones

| Error | Causa | Solución |
|---|---|---|
| `Specified key was too long` al migrar | Longitud de cadena por defecto en MySQL | `Schema::defaultStringLength(191);` en `AppServiceProvider` (sección 5) |
| `Unable to locate file in Vite manifest: resources/sass/app.scss` | Se intenta cargar `app.scss` con Vite pero no existe o no fue compilado | Verificar que exista `resources/sass/app.scss`, que `vite.config.js` esté bien configurado y ejecutar `npm run build` |
| Clase `ProductController` no encontrada | Autoload desactualizado | `composer dump-autoload` |
| Cambios en `.env` que no se reflejan | Configuración en caché | `php artisan config:clear` |
| Vistas sin estilos | Assets sin compilar o `laravel/ui` sin instalar | Sección 10 |

## 12. Conceptos adicionales: `$fillable` y `$guarded`

Ambos son atributos de los modelos Eloquent que controlan la **asignación masiva** (*mass assignment*): crear o actualizar un modelo pasando un array completo de datos (por ejemplo `$request->all()`) mediante `create()`, `update()` o `fill()`.

### `$fillable` (lista blanca)

Define **qué campos sí se pueden** asignar masivamente. Todo lo demás se ignora.

```php
class Product extends Model
{
    protected $fillable = ['description', 'price', 'stock'];
}
```

Evita que datos sensibles o no deseados (como `id`) sean modificados accidental o maliciosamente.

### `$guarded` (lista negra)

Funciona a la inversa: define **qué campos no** se pueden asignar masivamente. Todo lo demás sí.

```php
class Product extends Model
{
    protected $guarded = ['id'];
}
```

### Diferencia principal

| | `$fillable` | `$guarded` |
|---|---|---|
| Enfoque | Establece explícitamente los atributos **permitidos** | Establece explícitamente los atributos **no permitidos** |
| Comodín `*` | `['*']` permite todos (⚠️ riesgo de seguridad) | `['*']` bloquea todos |

Es importante usar uno de los dos para protegerse de vulnerabilidades de asignación masiva.

## 13. Resultado final

Con el servidor en marcha (`php artisan serve`), el CRUD queda disponible en `http://127.0.0.1:8000/products`.

### Listado de productos (`index`)

Tabla con todos los productos y las acciones **Show**, **Edit** y **Delete** por fila, más el botón **Add new**.

![Listado de productos](assets/capturas/04-crud-index.png)

### Crear producto (`create`)

Formulario con los campos descripción, precio y stock.

![Formulario de creación](assets/capturas/05-crud-create.png)

### Detalle de un producto (`show`)

![Detalle del producto](assets/capturas/06-crud-show.png)

### Editar producto (`edit`)

El formulario carga los datos actuales del producto para modificarlos.

![Formulario de edición](assets/capturas/07-crud-edit.png)

### Eliminar producto (`destroy`)

Antes de borrar, el sistema pide confirmación al usuario.

![Confirmación de eliminación](assets/capturas/08-crud-delete-confirmacion.png)

## 14. Referencias

- Video de referencia: <https://www.youtube.com/watch?v=j5baJsM_Adc&t=111s>
- Documentación oficial de Laravel: <https://laravel.com/docs>
- Laravel Breeze: <https://laravel.com/docs/starter-kits#laravel-breeze>
- Guía del Laboratorio #2 – Ing. Irina Fong
