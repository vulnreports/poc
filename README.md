# Filament Demo PoC — mass-assignment

## Purpose
Minimal reproduction for a security report: demonstrates that `Model::unguard()` allows `email_verified_at` mass-assignment (model primitive).

**Important:** This is a minimal demo for reproducing the issue *only*. Do **not** use this code in production.

## Included
- `app/Providers/AppServiceProvider.php` — contains `Model::unguard()` (vulnerable line).
- `.env.example` — configured for SQLite and local environment.
- `README.md` (this file).

## Quick setup (tested on Linux)
1. ### Clone:
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   composer install
   cp .env.example .env - Ensure `APP_ENV=local` and `DB_CONNECTION=sqlite` in `.env`
   php artisan key:generate
   touch database/database.sqlite
   php artisan migrate --seed
   php artisan serve --host=127.0.0.1 --port=8000
   ----------------------------------------------
   php artisan tinker
   ----------------------------------------------
   $attacker = \App\Models\User::factory()->create([
    'name' => 'tinkerpoc',
    'email' => 'tinkerpoc@example.test', 
    'password' => bcrypt('tinkerpocPass!'),
    'email_verified_at' => '1337-01-01 00:00:00',
   ]);
   ```
   > `sqlite3 database/database.sqlite "select id,email,email_verified_at from users where email='tinkerpoc@example.test';"`

If `email_verified_at is persisted`, model mass-assignment primitive is demonstrated.




