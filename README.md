# Rollen in Laravel

Dit is een eenvoudig project om te demonstreren hoe je rollen kunt gebruiken in Laravel. We zullen een eenvoudig results maken met twee rollen: teacher en student.

## Inhoudsopgave

-   [Rollen in Laravel](#rollen-in-laravel)
    -   [Inhoudsopgave](#inhoudsopgave)
    -   [Aan de slag](#aan-de-slag)
        -   [ResultsController](#resultscontroller)
        -   [Route](#route)
        -   [Index Methode](#index-methode)
        -   [Results View](#results-view)
        -   [Navigatie aanpassen](#navigatie-aanpassen)
        -   [Route Groep](#route-groep)
            -   [Prefix](#prefix)
            -   [Name](#name)
        -   [Verwijderen standaard Dashboard](#verwijderen-standaard-dashboard)
        -   [Route verwijderen](#route-verwijderen)
    -   [Standaard Redirect](#standaard-redirect)
    -   [Role Model](#role-model)
    -   [Registreer gebruiker](#registreer-gebruiker)
        -   [Teacher Controller](#teacher-controller)
        -   [Teacher methode](#teacher-methode)
        -   [Teacher View](#teacher-view)
        -   [Teacher Route](#teacher-route)
    -   [Layouts aanpassen](#layouts-aanpassen)
        -   [Student Layout](#student-layout)
        -   [Student View Aanpassen](#student-view-aanpassen)
            -   [Student Navigatie](#student-navigatie)
    -   [Teacher Layout](#teacher-layout)
        -   [Teacher View Aanpassen](#teacher-view-aanpassen)
            -   [Teacher Navigatie](#teacher-navigatie)
    -   [Redirect op basis van rol](#redirect-op-basis-van-rol)

## Aan de slag

1. Fork deze repository
2. Clone de repository naar je lokale machine
3. Voer `composer install` uit
4. Maak een `.env`-bestand
5. Voer `php artisan key:generate` uit
6. Voer `php artisan migrate --seed` uit
7. Voer `php artisan serve` uit
8. Voer `npm install` uit
9. Voer `npm run dev` uit
10. Bezoek `http://localhost:8000`

### ResultsController

We moeten een `ResultsController` maken. Deze controller plaatsen we in de map `Student`, die automatisch wordt aangemaakt wanneer we de controller maken.

```bash
php artisan make:controller Student/ResultsController
```

### Route

We moeten een route maken voor een resultatenoverzicht. We zullen de route in het bestand `routes/web.php` plaatsen.

```php
use App\Http\Controllers\Student\ResultsController;
//...

 Route::get('student/results', [ResultsController::class, 'index']);
```

We geven ook een naam aan de route door name-methode te gebruiken. We passen dus de code in `web.php` aan:

```php
use App\Http\Controllers\Student\ResultsController;
//...

// we voegen name('student.results') toe aan de bestaande route
Route::get('student/results', [ResultsController::class, 'index'])->name('student.results');
```

Maar we willen het resultatenoverzicht (privacy!) alleen toegankelijk maken voor geregistreerde gebruikers. We kunnen dit doen door de middleware `auth` te gebruiken. We passen dus de code opnieuw aan. Middleware `auth` zorgt ervoor dat de gebruiker is ingelogd moet zijn om de pagina te kunnen bezoeken.

```php
use App\Http\Controllers\Student\ResultsController;

//...

// we voegen middleware('auth') toe aan de bestaande route
Route::get('student/results', [ResultsController::class, 'index'])->middleware('auth')->name('student.results');
```

### Index Methode

We maken een methode `index` in de controller. Deze methode retourneert een view.

```php
public function index()
{
    return view('student.results');
}
```

### Results View

We moeten een view maken voor het results. Deze view plaatsen we in de map `student`, die automatisch wordt aangemaakt wanneer we de view maken.

```bash
php artisan make:view student/results
```

We zetten onderstaande code erin

```html
<x-app-layout>
    <x-slot name="header">
        <h2
            class="text-xl font-semibold leading-tight text-gray-800 dark:text-gray-200"
        >
            {{ __('Resultaten overzicht') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="mx-auto max-w-7xl sm:px-6 lg:px-8">
            <div
                class="overflow-hidden bg-white shadow-sm dark:bg-gray-800 sm:rounded-lg"
            >
                <div class="p-6 text-gray-900 dark:text-gray-100">
                    {{ __('Resultaten overzicht komt hier te staan') }}
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

### Navigatie aanpassen

**layout/navigation.blade.php**

````html
<x-nav-link
    :href="route('student.results')"
    :active="request()->routeIs('student.results')"
>
    {{ __('Resultaten overzicht') }}
</x-nav-link>
```
````

### Route Groep

We kunnen ook een route groep maken voor alle studentenroutes. We passen dus de code in `web.php` aan:

```php
use App\Http\Controllers\Student\ResultsController;

//...

Route::middleware(['auth'])->group(function () { //hier begint de groep
    Route::get('student/results', [ResultsController::class, 'index'])->name('student.results');

    // andere studentenroutes zoals student/profile, student/courses, student/grades, etc.
    Route::get('student/courses', [CourseController::class, 'index'])->name('student.courses');

});//hier eindigt de groep
```

In de bovenstaande routes herhalen we de middleware `auth` niet. We hebben de middleware `auth` toegevoegd aan de route groep. Dit betekent dat alle routes in de groep alleen toegankelijk zijn voor geregistreerde gebruikers.

#### Prefix

Dan kunnen we nog een zogenaamde prefix toevoegen aan de route groep

```php
use App\Http\Controllers\Student\ResultsController;

//...

Route::middleware(['auth'])
    ->prefix('student')//deze prefix is toegevoegd zodat alle urls beginnen met student/
    ->group(function () { //hier begint de groep
    Route::get('results', [ResultsController::class, 'index'])->name('student.results');

    // andere studentenroutes zoals profile, courses, grades, etc.
    Route::get('courses', [CourseController::class, 'index'])->name('student.courses');

});//hier eindigt de groep
```

#### Name

We kunnen ook de `namen` van van de routes toevoegen aan de groep

```php
use App\Http\Controllers\Student\ResultsController;

//...

Route::middleware(['auth'])
    ->prefix('student')//deze prefix is toegevoegd zodat alle urls beginnen met student/
    ->name('student.')//deze naam is toegevoegd zodat alle routes in de groep de naam student. krijgen
    ->group(function () { //hier begint de groep
    Route::get('results', [ResultsController::class, 'index'])->name('results');

    // andere studentenroutes zoals profile, courses, grades, etc.
    // Route::get('courses', [CourseController::class, 'index'])->name('courses');

});//hier eindigt de groep
```

### Verwijderen standaard Dashboard

We verwijderen de standaard dashboard en daarvoor moeten we eerst de navigatie aanpassen:

> Regels 5 t/m 19 verwijderen:

```html
<div class="flex">
    <!-- deze code verwijderen -->
</div>
```

> Regels 55 t/m 59 verwijderen:

```html
<div class="pt-2 pb-3 space-y-1">
    <x-responsive-nav-link
        :href="route('dashboard')"
        :active="request()->routeIs('dashboard')"
    >
        {{ __('Dashboard') }}
    </x-responsive-nav-link>
</div>
```

### Route verwijderen

We `__verwijderen__` de standaard dashboard route:

```php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard');
```

## Standaard Redirect

We kunnen de standaard redirect aanpassen. In het bestand `app/Http/Controllers/Auth/AuthenticatedSessionController.php` passen we de store methode aan:

```php
class AuthenticatedSessionController extends Controller
{
    // ...

    public function store(LoginRequest $request): RedirectResponse
    {
        // ...

        $request->authenticate();

        $request->session()->regenerate();

        // return redirect()->intended(route('dashboard', absolute: false));
        return redirect()->intended(route('student.results', absolute: false));
    }
}
```

We passen ook de redirect aan bij de registratie:

```php
class RegisteredUserController extends Controller
{
    // ...

    public function store(Request $request): RedirectResponse
    {
        // ...

        // return redirect(route('dashboard', absolute: false));
        return redirect()->intended(route('student.results', absolute: false));
    }
}
```

Nu kunnen we ook de standaard dashboard view verwijderen: `resources/views/dashboard.blade.php`

## Role Model

We gaan nu rollen toevoegen. We maken een nieuw model `Role` aan en de migratie:

```bash
php artisan make:model Role -m
```

En daarna voegen we een kolom toe aan de tabel `users`:

```bash
php artisan make:migration "add role id to users table"
```

We passen het Model Role aan:

```php
class Role extends Model
{
    protected $fillable = [
        'name',
    ];
}
```

Maar ook het Model User:

```php
class User extends Authenticatable
{
    use HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'role_id',  //deze voegen we toe
    ];

    // ...
}
```

We passen de migratie aan:

```php
//database/migrations/xxx_create_roles_table.php:
public function up()
{
    Schema::create('roles', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->timestamps();
    });
}

//database/migrations/xxx_add_role_id_to_users_table.php:
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->foreignId('role_id')->constrained()->onDelete('cascade');
    });
}
```

Dan moeten we nog seeders maken om de rollen toe te voegen. We doen dit in de DatabaseSeeder:

```php
class DatabaseSeeder extends Seeder
{
    public function run()
    {
        Role::create(['name' => 'student']);
        Role::create(['name' => 'teacher']);

        User::factory()->create([
            'name' => 'Student Student',
            'email' => 'student@novacollege.nl',
            'role_id' => 1,
        ]);
    }
}
```

En dan bouwen de dataase opnieuw op:

```bash
php artisan migrate:fresh --seed
```

## Registreer gebruiker

We registreren een gebruiker via de route `register`. We dienen ook een rol op te geven bij het aanmaken van een gebruiker.
We dienen de volgende html code toe te voegen aan de view `resources/views/auth/register.blade.php`:

```html
<!-- Role -->
<div class="mt-4">
    <x-input-label for="role_id" :value="__('Register as:')" />

    <label>
        <input type="radio" name="role_id" value="1" checked /> Student
    </label>
    <label class="ml-2">
        <input type="radio" name="role_id" value="2" /> Teacher
    </label>

    <x-input-error :messages="$errors->get('role_id')" class="mt-2" />
</div>
```

En we passen de `RegisteredUserController` aan:

```php
class RegisteredUserController extends Controller
{
    // ...

    public function store(Request $request): RedirectResponse
    {
        $request->validate([
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'string', 'lowercase', 'email', 'max:255', 'unique:'.User::class],
            'password' => ['required', 'confirmed', Rules\Password::defaults()],

            //hier wordt de rol verplicht
            'role_id' => ['required', 'in:1,2'],
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),

            //hier wordt de rol opgeslagen
            'role_id' => $request->role_id,
        ]);

        event(new Registered($user));

        Auth::login($user);

        return redirect(route('student.results', absolute: false));
    }
}
```

Opmerking: voor de validatie gebruiken we de in validatieregel `in` plaats van `exists`, omdat we ons alleen kunnen registreren met deze twee rollen. We willen dat gebruikers zich niet registreren met een admin-rol.

Na het registreren met de rol van docent zien we in de database dat de juiste rol is ingesteld.

Maar nu komt de docent nog steeds in het studenten gedeelte terecht. Dit gaan we fixen.

### Teacher Controller

We maken een nieuwe controller voor de docenten:

```bash
php artisan make:controller Teacher/ResultsController
```

### Teacher methode

We voegen een methode toe aan de `TeacherResultsController`:

```php
public function index()
{
    return view('teacher.results');
}
```

### Teacher View

We maken een nieuwe view voor de docenten:

```bash
php artisan make:view teacher/results
```

We passen de view aan:

```html
<x-app-layout>
    <x-slot name="header">
        <h2
            class="text-xl font-semibold leading-tight text-gray-800 dark:text-gray-200"
        >
            {{ __('Overzicht voor Docent') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="mx-auto max-w-7xl sm:px-6 lg:px-8">
            <div
                class="overflow-hidden bg-white shadow-sm dark:bg-gray-800 sm:rounded-lg"
            >
                <div class="p-6 text-gray-900 dark:text-gray-100">
                    {{ __("Overzicht voor Docent") }}
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

### Teacher Route

We voegen een nieuwe route toe voor de docenten. Maar we hebben al een ResultsController. We geven deze nieuwe ResultsController een alias: TeacherResultsController.

```php
use App\Http\Controllers\Teacher\ResultsController as TeacherResultsController;

//..
    Route::prefix('teacher')
        ->name('teacher.')
        ->group(function () {
            Route::get('results', [TeacherResultsController::class, 'index'])
                ->name('results');
    });
```

Als je nu gaat naar `/teacher/results` dan zie je de view voor de docenten.

## Layouts aanpassen

We kunnen de layouts aanpassen voor de docenten en studenten. Zodat elke rol zijn eigen layout heeft.

Laravel maakt gebruikt van `Components`. We maken voor iedere rol een `Layout Component` zodat we de juiste layout kunnen tonen.

### Student Layout

```bash
php artisan make:component StudentLayout
```

We passen de `StudentLayout` aan:

```php
use Illuminate\View\View;
use Illuminate\View\Component;

class StudentLayout extends Component
{
    /**
     * Get the view / contents that represent the component.
     */
    public function render(): View|Closure|string
    {
        return view('layouts.student');
    }
}
```

En we **verplaatsen** én **hernoemen** de aangemaakte layout in de components map: `resources/views/components/student-layout.blade.php` naar `resources/views/layouts/student.blade.php`

Als we dat gedaan hebben passen we de view van de layout inhoudelijk aan:

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <meta name="csrf-token" content="{{ csrf_token() }}" />

        <title>{{ config('app.name', 'Laravel') }}</title>

        <!-- Fonts -->
        <link rel="preconnect" href="https://fonts.bunny.net" />
        <link
            href="https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap"
            rel="stylesheet"
        />

        <!-- Scripts -->
        @vite(['resources/css/app.css', 'resources/js/app.js'])
    </head>
    <body class="font-sans antialiased">
        <div class="min-h-screen bg-gray-100">
            @include('layouts.navigation.student')

            <!-- Page Content -->
            <main>{{ $slot }}</main>
        </div>
    </body>
</html>
```

### Student View Aanpassen

Daarna passen we de student results view aan (is momenteel identiek aan de app layout):

```html
<x-student-layout>
    <x-slot name="header">
        <h2
            class="text-xl font-semibold leading-tight text-gray-800 dark:text-gray-200"
        >
            {{ __('Resultaten overzicht') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="mx-auto max-w-7xl sm:px-6 lg:px-8">
            <div
                class="overflow-hidden bg-white shadow-sm dark:bg-gray-800 sm:rounded-lg"
            >
                <div class="p-6 text-gray-900 dark:text-gray-100">
                    {{ __('Resultaten overzicht komt hier te staan') }}
                </div>
            </div>
        </div>
    </div>
</x-student-layout>
```

#### Student Navigatie

De navigatie moet ook nog aangepast worden. We maken een nieuwe navigatie voor de studenten:

```bash
php artisan make:view layouts/navigation/student
```

En passen de view aan door de content van de navigatie uit `layouts/navigation.blade.php` te verplaatsen naar de nieuwe view. En een link toe te voegen naar de student results:

```html
<x-nav-link
    :href="route('student.results')"
    :active="request()->routeIs('student.results')"
>
    {{ __('Resultaten overzicht') }}
</x-nav-link>
```

## Teacher Layout

We maken ook voor de docenten een layout component:

```bash
php artisan make:component TeacherLayout
```

We passen de `TeacherLayout` aan in de map `View/Components`:

```php
use Illuminate\View\View;
use Illuminate\View\Component;

class TeacherLayout extends Component
{
    /**
     * Get the view / contents that represent the component.
     */
    public function render(): View|Closure|string
    {
        return view('layouts.teacher');
    }
}
```

En we **verplaatsen** én **hernoemen** de aangemaakte layout in de components map: `resources/views/components/teacher-layout.blade.php` naar `resources/views/layouts/teacher.blade.php`

Als we dat gedaan hebben passen we de view van de layout inhoudelijk aan:

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <meta name="csrf-token" content="{{ csrf_token() }}" />

        <title>{{ config('app.name', 'Laravel') }}</title>

        <!-- Fonts -->
        <link rel="preconnect" href="https://fonts.bunny.net" />
        <link
            href="https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap"
            rel="stylesheet"
        />

        <!-- Scripts -->
        @vite(['resources/css/app.css', 'resources/js/app.js'])
    </head>
    <body class="font-sans antialiased">
        <div class="min-h-screen bg-gray-100">
            @include('layouts.navigation.teacher')

            <!-- Page Content -->
            <main>{{ $slot }}</main>
        </div>
    </body>
</html>
```

### Teacher View Aanpassen

Daarna passen we de teacher results view aan door de `<x-app-layout>` te vervangen door `<x-teacher-layout>`:

```html
<x-teacher-layout>
    <x-slot name="header">
        <h2
            class="text-xl font-semibold leading-tight text-gray-800 dark:text-gray-200"
        >
            {{ __('Resultaten overzicht') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="mx-auto max-w-7xl sm:px-6 lg:px-8">
            <div
                class="overflow-hidden bg-white shadow-sm dark:bg-gray-800 sm:rounded-lg"
            >
                <div class="p-6 text-gray-900 dark:text-gray-100">
                    {{ __('Resultaten overzicht komt hier te staan') }}
                </div>
            </div>
        </div>
    </div>
</x-teacher-layout>
```

#### Teacher Navigatie

Voor de docenten maken we een nieuwe navigatie:

```bash
php artisan make:view layouts/navigation/teacher
```

En passen de view aan door de content van de navigatie uit `layouts/navigation.blade.php` te verplaatsen naar de nieuwe view. En een link toe te voegen naar de teacher results:

```html
<x-nav-link
    :href="route('teacher.results')"
    :active="request()->routeIs('teacher.results')"
>
    {{ __('Docenten Resultaten overzicht') }}
</x-nav-link>
```

## Redirect op basis van rol

Na het inloggen en registreren willen we de gebruiker doorsturen naar de juiste pagina. We passen de `AuthenticatedSessionController` aan door een methode toe te voegen die de juiste route teruggeeft:

```php
    public function getRedirectRouteName(): string
    {
        return match ((int) $this->role_id) {
            1 => 'student.timetable',
            2 => 'teacher.timetable',
        };
    }
```

In de AuthenticatedSessionController passen we de store methode aan:

```php
    public function store(LoginRequest $request): RedirectResponse
    {
        $request->authenticate();

        $request->session()->regenerate();

        // return redirect()->intended(route('student.timetable', absolute: false));
        return redirect()->intended(route(auth()->user()->getRedirectRouteName(), absolute: false));
    }
```

En in de RegisteredUserController passen we de store methode aan:

```php
    public function store(Request $request): RedirectResponse
    {
        // ...

        // return redirect(route('student.timetable', absolute: false));
        return redirect(route(auth()->user()->getRedirectRouteName(), absolute: false));
    }
```

Na het inloggen of registreren wordt de gebruiker nu doorgestuurd naar de juiste pagina op basis van de rol.

Maar als iemand de url kent dan kan hij nog steeds naar de andere pagina gaan. We kunnen dit voorkomen door een middleware toe te voegen.

```bash
php artisan make:middleware RoleMiddleware
```

We passen de middleware aan:

```php
    public function handle(Request $request, Closure $next, int $roleId): Response
    {
        abort_if(auth()->user()->role_id !== $roleId, Response::HTTP_FORBIDDEN);

        return $next($request);
    }
```

We registereren de Middleware in het bestand bootstrap/app.php:

```php
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'role' => \App\Http\Middleware\RoleMiddleware::class,
        ]);
    })
```
