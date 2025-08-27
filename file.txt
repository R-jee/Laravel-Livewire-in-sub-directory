APP_URL="https://www.host.com/universities"   # ← no trailing slash
APP_DIR="universities"

# (optional) keep if you like, helps the <script> tag
LIVEWIRE_PATH="universities/livewire"
LIVEWIRE_ASSET_URL="/universities/livewire/livewire.min.js"



Option A — Livewire v3 (most likely)

routes/web.php

use Livewire\Mechanisms\HandleRequests\HandleRequests;

Route::prefix('universities')->middleware('web')->group(function () {
    // v3 uses the HandleRequests controller with a `handle` method
    Route::post('livewire/update', [HandleRequests::class, 'handle'])->name('livewire.update');
});


Also make sure your config/livewire.php has the path under your subfolder:

return [
    'path'      => env('LIVEWIRE_PATH', 'universities/livewire'),
    'app_url'   => env('APP_URL'),
    'asset_url' => env('LIVEWIRE_ASSET_URL'),
];

Option B — Livewire v2

routes/web.php

use Livewire\Controllers\HttpConnectionHandler;

Route::prefix('universities')->middleware('web')->group(function () {
    // v2 uses an invokable controller
    Route::post('livewire/message/{name}', HttpConnectionHandler::class)->name('livewire.message');
});


(If you’re on v2, the front-end JS will call /livewire/message/... instead of /livewire/update.)

Make sure URL generation matches your subfolder

.env

APP_URL="https://www.uniranks.com/universities"   # no trailing slash
LIVEWIRE_PATH="universities/livewire"
LIVEWIRE_ASSET_URL="/universities/livewire/livewire.min.js"


AppServiceProvider

use Illuminate\Support\Facades\URL;

public function boot(): void
{
    URL::forceRootUrl(config('app.url'));
    URL::forceScheme('https');
}

Clear & verify
php artisan optimize:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan cache:clear

# sanity check
php artisan route:list | grep -i livewire


You should now see either:

POST universities/livewire/update (v3), or

POST universities/livewire/message/{name} (v2).

If your browser was still trying https://www.uniranks.com/livewire/update, hard-refresh and purge any CDN cache so the new base URL and routes are used.
