# CakePHP 2 para PHP 8: versión de la UEJN

Fork de [kamilwylegala/cakephp2-php8](https://github.com/kamilwylegala/cakephp2-php8), que a su vez
es un fork de CakePHP 2.10.24 con los arreglos mínimos para PHP 8. Lo usa
[AfiGestión](https://github.com/uejn/afigestion) para salir de PHP 7.1, que no tiene parches
(uejn/afigestion#428, uejn/afigestion#429).

**Por qué un fork propio y no el de kamilwylegala directo:** ése no tiene tags ni releases y se
instala como `dev-master`, así que cualquier commit nuevo entraría a producción sin que nadie lo
revise. Acá cada versión que usa AfiGestión es un tag.

- **Exige PHP ≥ 8.0.** No anda en 7.x.
- El nombre del paquete sigue siendo `cakephp/cakephp`, para que lo que depende de CakePHP (Croogo,
  los plugins) lo encuentre sin cambios.
- El CI corre el job de upstream (PHP 8.0/8.1) y el nuestro, `uejn`: PHP 8.2 con MySQL 8, lo que
  corre en producción.

## Versiones

Los tags son `2.10.24.N` (`2.10.24.1`, `2.10.24.2`, …): composer los toma como más nuevos que el
2.10.24 original. Cada tag tiene su entrada en "Cambios de la UEJN", abajo.

En el `composer.json` de AfiGestión:

```json
{
	"repositories": [
		{ "type": "vcs", "url": "https://github.com/uejn/cakephp" }
	],
	"require": {
		"cakephp/cakephp": "2.10.24.1"
	}
}
```

La versión va fija, sin `~` ni `^`: se sube a mano, con un PR en AfiGestión.

## Traer los arreglos de kamilwylegala

Una vez por mes, o cuando haya un arreglo de seguridad:

```bash
git remote add upstream https://github.com/kamilwylegala/cakephp2-php8.git   # la primera vez
git fetch upstream
git checkout -b uejn/upstream-AAAA-MM master
git merge upstream/master
```

PR contra `master` de este repo; con el CI en verde, merge y tag nuevo.

## Cambios de la UEJN

### 2.10.24.3 (22/10/2026)

- `CakeTestCase::getMock($clase, null)` no reemplaza ningún método, como en PHPUnit 3.7 y como
  dice su docblock. Antes `null` se trataba igual que `array()` y los reemplazaba todos (todos
  devolvían `null`). Lo usan 6 tests de Croogo.

### 2.10.24.2 (22/10/2026)

- `DboSource::commit()`/`rollback()` no tiran `There is no active transaction` cuando un DDL
  (`CREATE`/`ALTER`/`DROP TABLE`) ya cerró la transacción en MySQL. Desde PHP 8, PDO se entera
  de ese cierre implícito; antes no. `commit()` devuelve `true` (lo hecho ya quedó grabado) y
  `rollback()`, `false` (no queda nada que deshacer). Cualquier otro error de PDO se sigue
  tirando. Lo necesita `cakedc/migrations`, que corre las migraciones dentro de una
  transacción, y con él Croogo (uejn/afigestion#430).

### 2.10.24.1 (21/10/2026)

- Base: `kamilwylegala/cakephp2-php8` al commit `b024bc1` (18/09/2026, "Fix readFile bypass for
  https urls in Xml::build()").
- CI: job `uejn` con PHP 8.2 y MySQL 8.0.
- `Security::encrypt()`/`decrypt()` usan OpenSSL cuando mcrypt no está instalado, en vez de morir
  con `Undefined constant "MCRYPT_RIJNDAEL_128"`. El formato es el mismo, así que lo cifrado con
  mcrypt se sigue leyendo. Antes había que prender `Security.useOpenSsl` a mano; con mcrypt
  instalado, sin esa clave, se sigue usando mcrypt como hasta ahora.
- Tests para MySQL 8: aceptan `utf8mb3` (el nombre que usa MySQL desde 8.0.30 para `utf8`) y que
  `int(11)` se describa sin ancho (desde 8.0.19). Sólo cambian los tests, no el driver.

---

# Fork of CakePHP 2 with support for PHP8

~~For original README content please check original repository: https://github.com/cakephp/cakephp/tree/2.x~~

Unfortunately branch 2.x in original repository was taken down.

## Why I created this fork? 🤔

CakePHP 2 stopped getting updates in the end of 2019 (AFAIR). Unfortunately in my case it's too expensive to migrate to newer versions of CakePHP. I started migrating to Symfony framework, but I still use ORM from CakePHP (and actually I like it). So in order to keep up with the newest PHP versions I decided to create fork of the framework.

## Why you should NOT use? ⛔

- Intention of this fork is to support PHP 8.*. Fork is not going to receive new features. Instead, fork is going to get minimal set of patches to comply with newer versions of PHP.
- If for example you're still on 5.6 or 7.0, you should **not** use this fork. Original `cakephp/cakephp` works perfectly fine on all PHP 7.* versions. You should migrate to newer versions of PHP and keep using original code. Once your application is battle tested on production I suggest migrating to PHP 8.

## When you could use this fork? ✅

Only prerequisite is to have your application already on PHP 7.4. Upgrade project to PHP 8.0 and replace CakePHP with this fork.

### Migration

Here are steps I took to migrate my project through all versions to PHP 8.1, maybe it can inspire you:

1. Decouple your tests from `CakeTestCase` and other utilities that are coupled to old PHPUnit version.
2. Once decoupled you can upgrade PHPUnit to the newest version accordingly to your PHP version.
3. Start upgrading gradually to newer versions of PHP. CakePHP 2 works perfectly fine on 7.0 - 7.4.
4. Once you're on 7.4 you can switch to 8 and this fork.

## Before using this fork ⚠️

- ~~Tests of CakePHP framework aren't refactored yet to support PHP 8. Main issue is old version of PHPUnit that is tightly coupled to framework's tests. Issue for fixing this situation is here: https://github.com/kamilwylegala/cakephp2-php8/issues/7~~ Framework tests are migrated to PHPUnit 9.*. Github actions are running tests on PHP 8.0, 8.1.
- ~~Due to lack of tests ☝️~~ - **you also need to rely** on tests in your application after integrating with this fork.
- If after integration you spot any issues related to framework please let me know by creating an issue or pull request with fix.

### Breaking changes

- In order to get rid of `strftime()` deprecation notices, it's required to switch to `IntlDateFormatter` class. This class is available in `intl` extension. Fork doesn't require it explicitly but to be able to use its functions Symfony ICU Polyfill is installed. To provide `strftime` behavior compatibility, `PHP81_BC\strftime` is used. `PHP81_BC` doesn't fully cover strftime, your code should work but there is a chance you'll get slightly different results. Discussed (here)[https://github.com/kamilwylegala/cakephp2-php8/pull/64] and (here)[https://github.com/kamilwylegala/cakephp2-php8/issues/65].

## Installation

This repository **is not** available in packagist, therefore your project's `composer.json` must be changed to point to custom repository.

Example configuration:
```
{
	"require": {
		"cakephp/cakephp": "dev-master as 2.10.24",
	},
	"repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/kamilwylegala/cakephp2-php8"
        }
    ]
}
```

It means that composer will look at `master` branch of repository configured under `repositories` to resolve update of `cakephp/cakephp` package.

## Changelog

### 2025-02-04

- Fixes for PHP 8.4: `session_set_save_handler` accepts object, removed `E_STRICT` reference.
- Removed github action with php code sniffer. It's quite painful to work with. Need to migrate to something newer, that will affect code base as little as possible.

### 2024-11-16

- Inflector fix: str_place with null.

### 2024-09-21

- Added wrapper for PDOException to avoid creating dynamic property `queryString`.

### 2024-07-24

- Csrf vulnerabity fix back ported from Cake PHP 3
- Explicit type hint definition of nullable parameters.

### 2024-06-05

- Removed usage of `strftime`, replaced with Intl extension.

### 2024-05-24

- Fix deprecation error in Model: `Automatic conversion of false to array is deprecated`

### 2024-05-22

- Fix deprecation error in I18n: `Automatic conversion of false to array is deprecated`

### 2024-02-02

- `str_len` deprecation warning fix in CakeResponse (passing null instead of `string`)

### 2024-01-19

- `strotime()` and `preg_split()` in CakeResponse deprecation warning fixes (passing null)

### 2024-01-11

- `preg_replace` deprecation warning fixes (passing null instead of `string`)

### 2023-12-22

- `preg_quote()` passing null fix

### 2023-12-19

- Muted dynamic property creation warnings in Controller.php
- Fix passing a null input to h function (PR #56)
- Fix Hash class callback callable pattern deprecated (PR #58)

### 2023-11-13

- Silence dynamic property creation warning in Model.php

### 2023-11-02

- Fixed: unitialized property in Debugger.php

### 2023-10-20

- Fallback to empty string from `env()` in basics.php and request handler.

### 2023-10-19

- Removed usage of deprecated `redis->getKeys()` in favor of `redis->keys()`.
- Added docker-compose setup to run tests locally.

### 2023-09-18

- Fix for `ShellDispatcher` where `null` was passed to `strpos` function.

### 2023-08-18

- Fixed PHP8 deprecation notices. Related mostly to passing null as a `$haystack` value.

### 2023-06-02

- Fixed PHP 8.2 deprecation notices in CakeEvent: `Creation of dynamic property ... is deprecated.`

### 2023-02-19

- Fixed PHP 8.1 MySQL test suite.

### 2023-02-11

- Fixed PostgreSQL test suite.

### 2023-01-30

- `PaginatorHelper` fix.

### 2023-01-22

- Fixed views cache when relative time is specified.

### 2023-01-11

- Fixed test suite to run under PHPUnit 9.5 and PHP8. Big kudos to @tenkoma :clap:

### 2022-10-20

- `MailTransport` fix.

### 2022-10-08

- Support for `full_path` when uploading a file, PHP 8.1 only.

### 2022-09-27

- Fixed multiple `CREATE UNIQUE INDEX` statements from schema shell that did not work on PostgreSQL.

### 2022-03-08

- Fixed passing `params["pass"]` argument to `invokeArgs` when resolving controller action - `array_values` used to avoid problems with named parameters.

### 2022-03-03

- Removed `String` class.

### 2022-03-02

- Fixed `ConsoleErrorHandler::handleError` to respect error suppression.

### 2022-01-31

- Fixed `Folder->read`, `array_values` is used to remove keys to prevent usign named arguments in `call_user_func_array`

### 2022-01-16

- Fix Shell `ReflectionMethod::__construct` default null argument in hasMethod

### 2022-01-15

- Readme file update - more explicit content.

### 2022-01-04

- Fixed more deprecation notices
    - `strtoupper` + `converting false to array` in Mysql.php
    - `preg_match` where `$subject = null` in CakeRoute.php
    - `strtoupper` in DboSource.php
    - Check history for details ☝️


### 2021-12-20

- Fixed deprecation notices in PHP 8.1 for production code implementations:
    - `ArrayAccess`
    - `Countable`
    - `IteratorAggregate`
- PHP 8.0 requirement in composer.json
- **Warning:** Tests are not updated, Cake's tests rely on old version of PHPUnit so running them may show a lot of deprecations notices. Added issue to cover it: #7

### 2021-02-24

- Fixed ErrorHandler accordingly to PHP8 migration guide. Otherwise, error handler is logging too much and doesn't respect configured `error_reporting`.
