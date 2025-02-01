# Segfault in LibSQL PHP/Laravel

## Description

When running a Laravel application using LibSQL as the database it causes a 502 Error when loading through the FPM or CLI SAPI.

Errors in the Nginx logs indicate that the upstream prematurely closed and PHP FPM's logs show a SIGSEGV:

```
2025/02/01 14:10:22 [error] 10649#646202: *1 upstream prematurely closed connection while reading response header from upstream, client: 127.0.0.1, server: libsql-laravel-segfault.test, request: "GET / HTTP/2.0", upstream: "fastcgi://unix:/Users/username/Library/Application Support/Herd/herd83.sock:", host: "libsql-laravel-segfault.test"
```

```
[01-Feb-2025 14:09:09] NOTICE: [pool herd] 'user' directive is ignored when FPM is not running as root
[01-Feb-2025 14:09:09] NOTICE: [pool herd] 'group' directive is ignored when FPM is not running as root
[01-Feb-2025 14:09:09] NOTICE: fpm is running, pid 10633
[01-Feb-2025 14:09:09] NOTICE: ready to handle connections
[01-Feb-2025 14:10:22] WARNING: [pool herd] child 10636 exited on signal 11 (SIGSEGV) after 73.094806 seconds from start
[01-Feb-2025 14:10:22] NOTICE: [pool herd] child 12159 started
```

## Reproduction steps

Prerequisites:

- Laravel Herd (on macOS)
- Laravel CLI
- FFI enabled in the php.ini

1. Check out the repository
2. Run `composer install`
3. Copy `.env.example` to `.env` if that wasn't done for you
4. Run `herd init` and `herd link`
5. Visit http://libsql-laravel-segfault.test

### CLI

Serve the application using `php artisan serve` and open http://localhost:8000

### Creating the repo

The steps I took to get a reproducable example is as follows:

1. Run `laravel new`
    a. Starter kit: No starter kit
    b. Testing framework: Pest
    c. Database: SQLite
    d. Run default migrations: No
    e. Install Composer dependencies: No
2. Run `composer require turso/libsql-laravel`
3. In `composer.json`, manually adjust the following:
    a. Set `minimum-stability` to `dev`
    b. Add `"turso/libsql": "dev-master#3733f98d1131d7649b780f0e46e6dc8dae5cdcac as 0.2.5"` to fix an issue with transactions
    c. Add `"ext-ffi": "*"`
    d. Set `"php": "^8.3"`
4. Run `composer update`
5. Set `DB_CONNECTION=libsql` in `.env`
6. Add a `libsql` connection in `config/database.php`, use `sqlite` as a template, adjust the default `DB_DATABASE` value
7. Run `php artisan migrate` (this will succeed)
