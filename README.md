# Filament Demo PoC — mass-assignment

## Purpose
Minimal reproduction for a security report: demonstrates that `Model::unguard()` allows `email_verified_at` mass-assignment (model primitive).

**Important:** This repository is for local reproduction only. Do **not** use this code in production.

## Included
- `app/Providers/AppServiceProvider.php` — contains `Model::unguard()` (vulnerable line).
- `.env.example` — configured for SQLite and local environment.
- `README.md` (this file).

## Vulnerable snippet
Include this exact snippet (or point to the file/lines) so reviewers immediately see the issue:

```php
// app/Providers/AppServiceProvider.php (excerpt)
use Illuminate\Database\Eloquent\Model;

public function boot()
{
    // vulnerable in production — allows mass-assignment of any attribute
    Model::unguard();

    if (app()->environment('production')) {
        URL::forceScheme('https');
    }
}
```

## Quick setup (tested on Linux)

> These commands assume PHP, Composer and sqlite3 are installed on your machine.

1. **Clone and install dependencies**
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   composer install
   ```
2. **Environment**

```bash
cp .env.example .env
# Edit .env if needed. Ensure:
# APP_ENV=local
# DB_CONNECTION=sqlite
```
```bash
php artisan key:generate
```

3. **Create SQLite database and migrate**

```bash
touch database/database.sqlite
php artisan migrate --seed
```

4. **Run dev server**

```bash
php artisan serve --host=127.0.0.1 --port=8000
```

5. **Demonstrate the mass-assignment primitive (tinker)**

```bash
php artisan tinker
```

## In the Tinker REPL paste:

```php
$attacker = \App\Models\User::factory()->create([
  'name' => 'tinkerpoc',
  'email' => 'tinkerpoc@example.test',
  'password' => bcrypt('tinkerpocPass!'),
  'email_verified_at' => '1337-01-01 00:00:00',
]);
```
Then from your shell:

```bash
sqlite3 database/database.sqlite "select id,email,email_verified_at from users where email='tinkerpoc@example.test';"
```
If `email_verified_at` contains the injected timestamp, the model-level mass-assignment primitive is demonstrated.

## Notes for maintainers / reviewers
* This repo intentionally preserves Model::unguard() to reproduce the issue. The fix is to remove Model::unguard() (or restrict it to local/testing) and add $fillable / $guarded on models.
* The Livewire registration component in the full demo may or may not forward arbitrary fields from the client; this repo demonstrates the model primitive rather than a specific HTTP exploit path.

## Safety & disclosure
* Do not run this against third-party servers. Only run locally on a VM you control.
* This repo is intended to be minimal and public for the sake of reproduction; do not include secrets.
