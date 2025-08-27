# 📌 Laravel + Livewire in Subfolder (`/universities`)

When running your Laravel app in a subdirectory like `/universities`, you need to properly configure `APP_URL`, Livewire paths, and route prefixes.  

---

## 1. `.env` Configuration

```dotenv
APP_URL="https://www.uniranks.com/universities"   # ← no trailing slash
APP_DIR="universities"

# Livewire path & asset config
LIVEWIRE_PATH="universities/livewire"
LIVEWIRE_ASSET_URL="/universities/livewire/livewire.min.js"
```

⚠️ **Important:** Never include a trailing slash in `APP_URL`. Trailing slashes cause weird joins.

---

## 2. Force URL Generation to Base

Update `App\Providers\AppServiceProvider.php`:

```php
use Illuminate\Support\Facades\URL;

public function boot(): void
{
    // Always respect APP_URL (important for subfolder like /universities)
    URL::forceRootUrl(config('app.url'));

    // Instead of hard-coding HTTPS, detect scheme
    if (request()->isSecure()) {
        URL::forceScheme('https');
    } else {
        URL::forceScheme('http');
    }
}
```

### 🔒 Proxy/CDN Note
If behind Cloudflare or a load balancer, configure **TrustProxies** to detect `X-Forwarded-Proto`:

```php
namespace App\Http\Middleware;

use Fideloper\Proxy\TrustProxies as Middleware;
use Illuminate\Http\Request;

class TrustProxies extends Middleware
{
    protected $proxies = '*'; // or specify your proxy IPs
    protected $headers = Request::HEADER_X_FORWARDED_ALL;
}
```

---

## 3. Livewire Configuration

Publish/edit `config/livewire.php`:

```php
return [
    // Routes live under /universities/livewire/*
    'path'      => env('LIVEWIRE_PATH', 'universities/livewire'),

    // Use base URL for subfolder setup
    'app_url'   => env('APP_URL'),

    // Script tag asset
    'asset_url' => env('LIVEWIRE_ASSET_URL', null),

    // ...keep the rest default
];
```

---

## 4. Route Setup

### ✅ Option A — Livewire v3 (likely)

```php
// routes/web.php
use Livewire\Mechanisms\HandleRequests\HandleRequests;

Route::prefix('universities')->middleware('web')->group(function () {
    Route::post('livewire/update', [HandleRequests::class, 'handle'])
        ->name('livewire.update');
});
```

- Frontend calls `/universities/livewire/update`

---

### ✅ Option B — Livewire v2

```php
// routes/web.php
use Livewire\Controllers\HttpConnectionHandler;

Route::prefix('universities')->middleware('web')->group(function () {
    Route::post('livewire/message/{name}', HttpConnectionHandler::class)
        ->name('livewire.message');
});
```

- Frontend calls `/universities/livewire/message/{name}`

---

## 5. Clear Caches

```bash
php artisan optimize:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan cache:clear
```

---

## 6. Sanity Check

```bash
php artisan route:list | grep -i livewire
```

You should see one of:

- **v3:** `POST universities/livewire/update`
- **v2:** `POST universities/livewire/message/{name}`

---

## 7. Final Notes

- If browser still requests `https://www.uniranks.com/livewire/update` instead of `/universities/livewire/update`,  
  → Hard refresh browser, purge CDN (Cloudflare) cache.  
- With `request()->isSecure()`, Laravel auto-detects HTTP vs HTTPS in dev vs prod.  

---

✅ This ensures Livewire works properly inside `/universities` without broken asset paths.  
