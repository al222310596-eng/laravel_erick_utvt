# CI Pipeline para Laravel

## Objetivo

Automatizar la instalación de dependencias y la ejecución de pruebas de la aplicación
Laravel mediante GitHub Actions, garantizando que cada cambio enviado al repositorio
sea validado automáticamente antes de integrarse a la rama principal.

---

## Tecnologías

- PHP 8.2
- Laravel
- Composer
- PHPUnit (a través de `php artisan test`)
- Node.js 22 y Vite (para la construcción de recursos frontend)
- Git y GitHub
- GitHub Actions

---

## Requisitos

- PHP compatible con el proyecto (8.2 en esta implementación)
- Composer
- Node.js 22 y npm
- Git
- Acceso al repositorio de GitHub

---

## Instalación local

1. Clonar el repositorio:

   ```bash
   git clone https://github.com/al222310596-eng/erick.git
   cd erick
   ```

2. Instalar dependencias de PHP con Composer:

   ```bash
   composer install
   ```

3. Copiar el archivo de entorno de ejemplo:

   ```bash
   cp .env.example .env
   ```

4. Generar la clave de la aplicación:

   ```bash
   php artisan key:generate
   ```

5. Configurar la base de datos requerida en el archivo `.env`
   (para pruebas se utiliza SQLite en memoria).

6. Ejecutar las migraciones si son necesarias:

   ```bash
   php artisan migrate
   ```

7. Instalar dependencias de frontend y compilar los assets:

   ```bash
   npm ci
   npm run build
   ```

8. Ejecutar las pruebas:

   ```bash
   php artisan test
   ```

---

## Integración continua

El workflow `.github/workflows/ci.yml` se ejecuta automáticamente al enviar cambios
(`push`) a la rama principal o al crear una solicitud de integración (`pull_request`)
dirigida a ella.

El pipeline realiza las siguientes etapas:

1. **Checkout del código** (`actions/checkout@v4`).
2. **Configuración de PHP 8.2** con las extensiones necesarias
   (`mbstring`, `pdo_sqlite`, `sqlite3`, `bcmath`, `fileinfo`).
3. **Validación de Composer** (`composer validate`).
4. **Instalación de dependencias PHP** (`composer install`).
5. **Configuración de Node.js 22** con caché de npm.
6. **Instalación de dependencias frontend** (`npm ci`).
7. **Construcción de recursos frontend** (`npm run build` con Vite).
8. **Preparación del entorno Laravel** (`.env`, `key:generate`, `config:clear`).
9. **Ejecución de pruebas automatizadas** (`php artisan test` con SQLite en memoria).

### Construcción de recursos frontend

Se agregó una etapa de build frontend utilizando `npm ci` y `npm run build`,
requerida porque el proyecto utiliza **Vite** para compilar assets. Esta etapa
permite detectar errores de compilación del frontend antes de ejecutar las pruebas
y garantiza que el pipeline reproduce el flujo real de despliegue. La salida de
Vite se genera en `public/build/` dentro del runner (no se versiona).

---

## Pruebas

Se implementó la prueba automatizada `HomePageTest::test_home_page_is_accessible`,
que verifica que la ruta pública `/` responda con un código HTTP de éxito (200).

Archivo: `tests/Feature/HomePageTest.php`

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class HomePageTest extends TestCase
{
    public function test_home_page_is_accessible(): void
    {
        $response = $this->get('/');
        $response->assertSuccessful();
    }
}
```

### Resultado esperado

La prueba debe pasar cuando la ruta `/` responde con HTTP 200.

### Resultado obtenido

1 Primera subida Éxito
2 Fallo controlado Falla 
3 Corrección Éxito 
4 build Éxito 

## Evidencias

A continuación se listan las ejecuciones del workflow en GitHub Actions. Cada
enlace corresponde a una ejecución real del pipeline.


 1 | Primera subida — pipeline básico |  Éxito | https://github.com/al222310596-eng/laravel_erick_utvt/actions/runs/38984336449  |
 2 | Fallo controlado (`assertStatus(404)`) |  Falla | https://github.com/al222310596-eng/laravel_erick_utvt/actions/runs/38984932502 |
 3 | Restauración tras fallo controlado |  Éxito | https://github.com/al222310596-eng/laravel_erick_utvt/actions/runs/38985337203 |
 4 | Pipeline completo con build frontend |  Éxito | https://github.com/al222310596-eng/laravel_erick_utvt/actions/runs/38992893819 |

### Ejecución exitosa (run #4)

- Todos los pasos del job `laravel-tests` se completaron sin errores.
- El paso `npm run build` compiló los assets con Vite
  (`vite v5.4.21 building for production... ✓ built in X.XXs`).
- El paso `php artisan test` mostró `PASS Tests\Feature\HomePageTest` con 1 test pasado.
- **Duración aproximada:** 22 segundos.

### Ejecución fallida controlada (run #2)

- Se modificó temporalmente la aserción de la prueba a `assertStatus(404)`.
- GitHub Actions marcó el job como fallido.
- Log del error:

  ```
  FAIL  Tests\Feature\HomePageTest
    ⨯ home page is accessible
  Failed asserting that the response status code 200 is identical to 404.
  Tests:    1 failed
  ```

- **Solución aplicada:** restaurar la aserción original `assertSuccessful()` y
  enviar un nuevo commit. El pipeline se ejecutó de nuevo y finalizó con éxito.

### Ejecución exitosa posterior (run #3)

- Tras restaurar la aserción, el pipeline volvió a ejecutarse automáticamente
  y finalizó correctamente.
- Confirma el ciclo completo: **commit → CI → detección → corrección → CI exitoso**.

---

## Conclusiones

- La integración continua permite detectar errores automáticamente antes de
  fusionar cambios a la rama principal, reduciendo el riesgo de integrar código
  defectuoso.
- El fallo controlado demostró que el pipeline no solo se ejecuta, sino que
  **detecta y comunica errores de forma clara** mediante los logs de GitHub Actions,
  indicando el archivo, la prueba y la causa exacta del fallo.
- La incorporación de la etapa de construcción frontend (`npm ci` + `npm run build`)
  demuestra que el pipeline cubre tanto la validación de PHP/Laravel como la
  compilación de assets con Vite, acercando el CI al flujo real de despliegue.
- Utilizar SQLite en memoria para las pruebas hace que el pipeline sea rápido,
  reproducible y sin dependencias externas, lo cual es ideal para entornos de CI.
- Los beneficios principales observados son: retroalimentación temprana a los
  desarrolladores, mayor confianza en el código integrado, menor cantidad de
  errores en producción y estandarización del proceso de validación.

---

## Estructura del proyecto

```
erick/
├── .github/
│   └── workflows/
│       └── ci.yml
├── app/
├── config/
├── database/
├── public/
├── resources/
├── routes/
├── tests/
│   ├── Feature/
│   │   └── HomePageTest.php
│   └── Unit/
├── .env.example
├── .gitignore
├── artisan
├── composer.json
├── composer.lock
├── package.json
├── package-lock.json
├── phpunit.xml
└── README.md
```
```
