## Проект: BookShelf — система управления библиотекой

Пользователь может управлять книгами и жанрами. Администратор видит всех пользователей и все книги.

---

### Структура базы данных

**Таблица `users`** — уже существует, нужно добавить поле:
- `role` — enum: `admin`, `user`, default `user`

**Таблица `genres`**:
- `id`
- `name` — string
- `user_id` — внешний ключ на `users`
- `timestamps`

**Таблица `books`**:
- `id`
- `title` — string
- `author` — string
- `description` — text, nullable
- `status` — enum: `want_to_read`, `reading`, `finished`, default `want_to_read`
- `rating` — unsignedTinyInteger, nullable (1-5)
- `finished_at` — date, nullable
- `user_id` — внешний ключ на `users`
- `genre_id` — внешний ключ на `genres`, nullOnDelete
- `timestamps`

---

### Задание 1 — Миграции

Форкните шаблон [ Laravel 13 ](https://github.com/31ISR/laravel-template) с названием как у этой лабораторной и ваша фамилия

```bash
# Выполните команды для создания миграций:
# 1. Миграция для добавления поля role в таблицу users
# 2. Миграция для таблицы genres
# 3. Миграция для таблицы books
# Затем примените миграции
```

```php
// Заполните migration для genres самостоятельно.
// Не забудьте про внешний ключ и cascadeOnDelete.
Schema::create('genres', function (Blueprint $table) {
    // TODO: ваш код здесь
});

// Проверьте: enum поля, nullable поля, два внешних ключа.
// Подсказка: для finished_at используйте ->date()->nullable()
Schema::create('books', function (Blueprint $table) {
    // TODO: ваш код здесь
});
```

---

### Задание 2 — Модели

```bash
# Создайте модели командой artisan.
# Подсказка: модели уже можно было создать вместе с миграциями флагом -m
```

```php
// app/Models/User.php
// Добавьте в существующую модель:

protected $fillable = [
    // TODO: перечислите все заполняемые поля включая role
];

// TODO: добавьте метод isAdmin(): bool

// TODO: добавьте связь books() — один пользователь имеет много книг

// TODO: добавьте связь genres() — один пользователь имеет много жанров
```

```php
// app/Models/Genre.php
// Подсказка: структура аналогична модели Category из прошлого проекта

protected $fillable = [
    // TODO: перечислите поля
];

// TODO: связь books() — один жанр имеет много книг
```

```php
// app/Models/Book.php

protected $fillable = [
    // TODO: перечислите все поля
];

protected $casts = [
    // TODO: finished_at должен каститься в дату
    // TODO: rating кастите в integer
];

// TODO: связь user() — книга принадлежит пользователю

// TODO: связь genre() — книга принадлежит жанру
```

---

### Задание 3 — Базовый контроллер

```php
// app/Http/Controllers/Controller.php
// Подсказка: вспомните какой трейт нужен для $this->authorize()

abstract class Controller
{
    // TODO: подключите нужный трейт

    // TODO: добавьте метод currentUser(): User
}
```

---

### Задание 4 — Policy

```bash
# Создайте BookPolicy для модели Book
```

```php
// app/Policies/BookPolicy.php

public function update(User $user, Book $book): bool
{
    // TODO: только владелец может редактировать книгу
}

public function delete(User $user, Book $book): bool
{
    // TODO: владелец ИЛИ администратор может удалить
}
```

```php
// app/Providers/AppServiceProvider.php
// TODO: зарегистрируйте BookPolicy для модели Book
```

---

### Задание 5 — Контроллеры авторизации

```bash
# Создайте три контроллера:
# Auth/LoginController
# Auth/RegisterController
# Auth/LogoutController
```

```php
// Auth/RegisterController.php

public function show()
{
    // TODO: верните view auth.register
}

public function store(Request $request): RedirectResponse
{
    // TODO: валидация — name, email (unique), password (min:8, confirmed)
    // TODO: создайте пользователя, не забудьте bcrypt для пароля
    // TODO: залогиньте пользователя сразу после регистрации
    // TODO: редирект на dashboard
}
```

```php
// Auth/LoginController.php

public function show()
{
    // TODO: верните view auth.login
}

public function store(Request $request): RedirectResponse
{
    // TODO: валидация email и password
    // TODO: попытка входа через Auth::attempt()
    // TODO: при неудаче — back() с ошибкой на поле email
    // TODO: при успехе — regenerate сессию, редирект на dashboard
}
```

```php
// Auth/LogoutController.php

public function __invoke(Request $request): RedirectResponse
{
    // TODO: разлогиньте пользователя
    // TODO: инвалидируйте сессию
    // TODO: regenerateToken
    // TODO: редирект на login
}
```

---

### Задание 6 — DashboardController

```php
// app/Http/Controllers/DashboardController.php

public function index(): View
{
    $user = $this->currentUser();

    // TODO: соберите статистику через $user->books():
    // total — всего книг
    // want_to_read — хочу прочитать
    // reading — читаю сейчас
    // finished — прочитано

    // TODO: получите последние 5 книг с жанром (eager loading)

    // TODO: верните view dashboard с данными
}
```

---

### Задание 7 — GenreController

```bash
# Создайте GenreController
```

```php
// app/Http/Controllers/GenreController.php
// Подсказка: структура полностью аналогична CategoryController из прошлого проекта
// Не забудьте про:
// — withCount('books') в index()
// — приватный метод authorizeGenre() вместо Policy
// — валидацию поля name (required, string, max:255)

public function index(): View { /* TODO */ }
public function create(): View { /* TODO */ }
public function store(Request $request): RedirectResponse { /* TODO */ }
public function edit(Genre $genre): View { /* TODO */ }
public function update(Request $request, Genre $genre): RedirectResponse { /* TODO */ }
public function destroy(Genre $genre): RedirectResponse { /* TODO */ }

private function authorizeGenre(Genre $genre): void
{
    // TODO: abort(403) если жанр не принадлежит текущему пользователю
}
```

---

### Задание 9 — BookController

```bash
# Создайте BookController с флагом --resource
```

```php
// app/Http/Controllers/BookController.php

public function index(): View
{
    // TODO: получите книги текущего пользователя
    // с eager loading жанра
    // отсортированные от новых к старым
    // пагинация по 10
}

public function create(): View
{
    // TODO: получите жанры текущего пользователя
    // верните view tasks.create с жанрами
}

public function store(Request $request): RedirectResponse
{
    // TODO: валидация:
    // title — required, string, max:255
    // author — required, string, max:255
    // description — nullable, string
    // status — required, in: want_to_read, reading, finished
    // rating — nullable, integer, min:1, max:5
    // finished_at — nullable, date
    // genre_id — nullable, exists:genres,id

    // TODO: создайте книгу через связь currentUser()->books()
    // редирект с flash-сообщением
}

public function edit(Book $book): View
{
    // TODO: авторизация через Policy
    // TODO: получите жанры пользователя
    // верните view books.edit
}

public function update(Request $request, Book $book): RedirectResponse
{
    // TODO: авторизация через Policy
    // TODO: валидация (аналогично store, но без after_or_equal для дат)
    // TODO: обновите книгу
}

public function destroy(Book $book): RedirectResponse
{
    // TODO: авторизация через Policy
    // TODO: удалите книгу
    // редирект с flash-сообщением
}
```

---

### Задание 10 — Маршруты

```php
// routes/web.php
// TODO: группа для гостей (middleware guest) — login, register, logout

// TODO: группа для авторизованных (middleware auth):
// — dashboard
// — resource для books
// — resource для genres (без show)

// TODO: группа для админов (middleware auth + admin, prefix admin, name admin.):
// — GET /users — список всех пользователей
```

---

### Задание 11 — Seeder

```php
// database/seeders/DatabaseSeeder.php
// TODO: создайте администратора:
// name: Admin, email: admin@bookshelf.com, password: password, role: admin

// TODO: создайте обычного пользователя:
// name: User, email: user@bookshelf.com, password: password

// TODO: для обычного пользователя создайте:
// — 2 жанра (например Фантастика и Детектив)
// — 3 книги с разными статусами
```

---

### Blade-шаблоны

`resources/views/layouts/app.blade.php`
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BookShelf — @yield('title', 'Моя библиотека')</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
    <style>
        .status-badge { font-size: 0.75rem; }
        .rating-stars { color: #f59e0b; letter-spacing: 2px; }
    </style>
</head>
<body>
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container">
        <a class="navbar-brand" href="{{ route('dashboard') }}">📚 BookShelf</a>
        @auth
        <div class="d-flex align-items-center gap-3">
            <a href="{{ route('books.index') }}" class="text-white text-decoration-none">Книги</a>
            <a href="{{ route('genres.index') }}" class="text-white text-decoration-none">Жанры</a>
            @if(auth()->user()->isAdmin())
                <a href="{{ route('admin.users') }}" class="text-warning text-decoration-none">
                    <span class="badge bg-warning text-dark">Admin</span>
                </a>
            @endif
            <span class="text-white-50">{{ auth()->user()->name }}</span>
            <form action="{{ route('logout') }}" method="POST">
                @csrf
                <button class="btn btn-sm btn-outline-light">Выйти</button>
            </form>
        </div>
        @endauth
    </div>
</nav>

<main class="container py-4">
    @if(session('success'))
        <div class="alert alert-success alert-dismissible fade show">
            {{ session('success') }}
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>
    @endif

    @if($errors->any())
        <div class="alert alert-danger">
            <ul class="mb-0">
                @foreach($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    @yield('content')
</main>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

`resources/views/auth/login.blade.php`
```html
@extends('layouts.app')
@section('title', 'Вход')
@section('content')
<div class="row justify-content-center">
    <div class="col-md-5">
        <div class="card shadow-sm">
            <div class="card-body p-4">
                <h4 class="mb-4">Вход в систему</h4>
                <form action="{{ route('login') }}" method="POST">
                    @csrf
                    <div class="mb-3">
                        <label class="form-label">Email</label>
                        <input type="email" name="email"
                               class="form-control @error('email') is-invalid @enderror"
                               value="{{ old('email') }}" autofocus>
                        @error('email')
                            <div class="invalid-feedback">{{ $message }}</div>
                        @enderror
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Пароль</label>
                        <input type="password" name="password"
                               class="form-control @error('password') is-invalid @enderror">
                        @error('password')
                            <div class="invalid-feedback">{{ $message }}</div>
                        @enderror
                    </div>
                    <div class="mb-3 form-check">
                        <input type="checkbox" name="remember" class="form-check-input" id="remember">
                        <label class="form-check-label" for="remember">Запомнить меня</label>
                    </div>
                    <button type="submit" class="btn btn-primary w-100">Войти</button>
                </form>
                <hr>
                <p class="text-center mb-0">
                    Нет аккаунта? <a href="{{ route('register') }}">Зарегистрироваться</a>
                </p>
            </div>
        </div>
    </div>
</div>
@endsection
```

---

`resources/views/auth/register.blade.php`
```html
@extends('layouts.app')
@section('title', 'Регистрация')
@section('content')
<div class="row justify-content-center">
    <div class="col-md-5">
        <div class="card shadow-sm">
            <div class="card-body p-4">
                <h4 class="mb-4">Регистрация</h4>
                <form action="{{ route('register') }}" method="POST">
                    @csrf
                    <div class="mb-3">
                        <label class="form-label">Имя</label>
                        <input type="text" name="name"
                               class="form-control @error('name') is-invalid @enderror"
                               value="{{ old('name') }}" autofocus>
                        @error('name')
                            <div class="invalid-feedback">{{ $message }}</div>
                        @enderror
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Email</label>
                        <input type="email" name="email"
                               class="form-control @error('email') is-invalid @enderror"
                               value="{{ old('email') }}">
                        @error('email')
                            <div class="invalid-feedback">{{ $message }}</div>
                        @enderror
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Пароль</label>
                        <input type="password" name="password"
                               class="form-control @error('password') is-invalid @enderror">
                        @error('password')
                            <div class="invalid-feedback">{{ $message }}</div>
                        @enderror
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Повторите пароль</label>
                        <input type="password" name="password_confirmation" class="form-control">
                    </div>
                    <button type="submit" class="btn btn-primary w-100">Зарегистрироваться</button>
                </form>
                <hr>
                <p class="text-center mb-0">
                    Уже есть аккаунт? <a href="{{ route('login') }}">Войти</a>
                </p>
            </div>
        </div>
    </div>
</div>
@endsection
```

---

`resources/views/dashboard.blade.php`
```html
@extends('layouts.app')
@section('title', 'Главная')
@section('content')
<h1 class="mb-4">Добро пожаловать, {{ auth()->user()->name }}!</h1>

<div class="row g-3 mb-4">
    <div class="col-md-3">
        <div class="card text-center border-0 bg-secondary bg-opacity-10">
            <div class="card-body">
                <h2 class="mb-0">{{ $stats['total'] }}</h2>
                <small class="text-muted">Всего книг</small>
            </div>
        </div>
    </div>
    <div class="col-md-3">
        <div class="card text-center border-0 bg-primary bg-opacity-10">
            <div class="card-body">
                <h2 class="mb-0">{{ $stats['want_to_read'] }}</h2>
                <small class="text-muted">Хочу прочитать</small>
            </div>
        </div>
    </div>
    <div class="col-md-3">
        <div class="card text-center border-0 bg-warning bg-opacity-10">
            <div class="card-body">
                <h2 class="mb-0">{{ $stats['reading'] }}</h2>
                <small class="text-muted">Читаю сейчас</small>
            </div>
        </div>
    </div>
    <div class="col-md-3">
        <div class="card text-center border-0 bg-success bg-opacity-10">
            <div class="card-body">
                <h2 class="mb-0">{{ $stats['finished'] }}</h2>
                <small class="text-muted">Прочитано</small>
            </div>
        </div>
    </div>
</div>

<h5 class="mb-3">Последние добавленные</h5>
@forelse($recentBooks as $book)
    <div class="card mb-2">
        <div class="card-body d-flex justify-content-between align-items-center py-2">
            <div>
                <strong>{{ $book->title }}</strong>
                <small class="text-muted ms-2">{{ $book->author }}</small>
            </div>
            <div class="d-flex gap-2 align-items-center">
                @if($book->rating)
                    <span class="rating-stars">{{ str_repeat('★', $book->rating) }}{{ str_repeat('☆', 5 - $book->rating) }}</span>
                @endif
                @if($book->genre)
                    <span class="badge bg-secondary">{{ $book->genre->name }}</span>
                @endif
                <span class="badge status-badge bg-{{ match($book->status) {
                    'want_to_read' => 'primary',
                    'reading'      => 'warning',
                    'finished'     => 'success',
                } }}">
                    {{ match($book->status) {
                        'want_to_read' => 'Хочу прочитать',
                        'reading'      => 'Читаю',
                        'finished'     => 'Прочитано',
                    } }}
                </span>
            </div>
        </div>
    </div>
@empty
    <p class="text-muted">Книг пока нет. <a href="{{ route('books.create') }}">Добавить первую →</a></p>
@endforelse
@endsection
```

---

`resources/views/books/index.blade.php`
```html
@extends('layouts.app')
@section('title', 'Мои книги')
@section('content')
<div class="d-flex justify-content-between align-items-center mb-4">
    <h1>Мои книги</h1>
    <a href="{{ route('books.create') }}" class="btn btn-primary">+ Добавить книгу</a>
</div>

<div class="row g-3">
@forelse($books as $book)
    <div class="col-12">
        <div class="card">
            <div class="card-body d-flex justify-content-between align-items-center">
                <div>
                    <h5 class="mb-1">{{ $book->title }}</h5>
                    <div class="d-flex gap-2 align-items-center flex-wrap">
                        <small class="text-muted">{{ $book->author }}</small>
                        @if($book->rating)
                            <span class="rating-stars small">{{ str_repeat('★', $book->rating) }}{{ str_repeat('☆', 5 - $book->rating) }}</span>
                        @endif
                        <span class="badge status-badge bg-{{ match($book->status) {
                            'want_to_read' => 'primary',
                            'reading'      => 'warning',
                            'finished'     => 'success',
                        } }}">
                            {{ match($book->status) {
                                'want_to_read' => 'Хочу прочитать',
                                'reading'      => 'Читаю',
                                'finished'     => 'Прочитано',
                            } }}
                        </span>
                        @if($book->genre)
                            <span class="badge bg-secondary">{{ $book->genre->name }}</span>
                        @endif
                        @if($book->finished_at)
                            <small class="text-muted">Прочитано: {{ $book->finished_at->format('d.m.Y') }}</small>
                        @endif
                    </div>
                </div>
                <div class="d-flex gap-2">
                    <a href="{{ route('books.edit', $book) }}" class="btn btn-sm btn-outline-secondary">Редактировать</a>
                    <form action="{{ route('books.destroy', $book) }}" method="POST"
                          onsubmit="return confirm('Удалить книгу?')">
                        @csrf @method('DELETE')
                        <button class="btn btn-sm btn-outline-danger">Удалить</button>
                    </form>
                </div>
            </div>
        </div>
    </div>
@empty
    <p class="text-muted">Книг пока нет. <a href="{{ route('books.create') }}">Добавить первую →</a></p>
@endforelse
</div>

<div class="mt-4">{{ $books->links() }}</div>
@endsection
```

---

`resources/views/books/create.blade.php`
```html
@extends('layouts.app')
@section('title', 'Добавить книгу')
@section('content')
<div class="row justify-content-center">
    <div class="col-md-7">
        <div class="card shadow-sm">
            <div class="card-body p-4">
                <h4 class="mb-4">Добавить книгу</h4>
                <form action="{{ route('books.store') }}" method="POST">
                    @csrf
                    <div class="mb-3">
                        <label class="form-label">Название</label>
                        <input type="text" name="title"
                               class="form-control @error('title') is-invalid @enderror"
                               value="{{ old('title') }}" autofocus>
                        @error('title') <div class="invalid-feedback">{{ $message }}</div> @enderror
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Автор</label>
                        <input type="text" name="author"
                               class="form-control @error('author') is-invalid @enderror"
                               value="{{ old('author') }}">
                        @error('author') <div class="invalid-feedback">{{ $message }}</div> @enderror
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Описание</label>
                        <textarea name="description" rows="3"
                                  class="form-control @error('description') is-invalid @enderror">{{ old('description') }}</textarea>
                        @error('description') <div class="invalid-feedback">{{ $message }}</div> @enderror
                    </div>
                    <div class="row g-3 mb-3">
                        <div class="col-md-4">
                            <label class="form-label">Статус</label>
                            <select name="status" class="form-select">
                                <option value="want_to_read" {{ old('status') === 'want_to_read' ? 'selected' : '' }}>Хочу прочитать</option>
                                <option value="reading"      {{ old('status') === 'reading'      ? 'selected' : '' }}>Читаю</option>
                                <option value="finished"     {{ old('status') === 'finished'     ? 'selected' : '' }}>Прочитано</option>
                            </select>
                        </div>
                        <div class="col-md-4">
                            <label class="form-label">Оценка</label>
                            <select name="rating" class="form-select">
                                <option value="">— Без оценки —</option>
                                @for($i = 1; $i <= 5; $i++)
                                    <option value="{{ $i }}" {{ old('rating') == $i ? 'selected' : '' }}>
                                        {{ str_repeat('★', $i) }} ({{ $i }})
                                    </option>
                                @endfor
                            </select>
                        </div>
                        <div class="col-md-4">
                            <label class="form-label">Дата прочтения</label>
                            <input type="date" name="finished_at"
                                   class="form-control @error('finished_at') is-invalid @enderror"
                                   value="{{ old('finished_at') }}">
                            @error('finished_at') <div class="invalid-feedback">{{ $message }}</div> @enderror
                        </div>
                    </div>
                    <div class="mb-4">
                        <label class="form-label">Жанр</label>
                        <select name="genre_id" class="form-select">
                            <option value="">— Без жанра —</option>
                            @foreach($genres as $genre)
                                <option value="{{ $genre->id }}" {{ old('genre_id') == $genre->id ? 'selected' : '' }}>
                                    {{ $genre->name }}
                                </option>
                            @endforeach
                        </select>
                    </div>
                    <div class="d-flex gap-2">
                        <button type="submit" class="btn btn-primary">Добавить</button>
                        <a href="{{ route('books.index') }}" class="btn btn-outline-secondary">Отмена</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
@endsection
```

---

`resources/views/books/edit.blade.php`
```html
@extends('layouts.app')
@section('title', 'Редактировать книгу')
@section('content')
<div class="row justify-content-center">
    <div class="col-md-7">
        <div class="card shadow-sm">
            <div class="card-body p-4">
                <h4 class="mb-4">Редактировать книгу</h4>
                <form action="{{ route('books.update', $book) }}" method="POST">
                    @csrf
                    @method('PUT')
                    <div class="mb-3">
                        <label class="form-label">Название</label>
                        <input type="text" name="title"
                               class="form-control @error('title') is-invalid @enderror"
                               value="{{ old('title', $book->title) }}" autofocus>
                        @error('title') <div class="invalid-feedback">{{ $message }}</div> @enderror
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Автор</label>
                        <input type="text" name="author"
                               class="form-control @error('author') is-invalid @enderror"
                               value="{{ old('author', $book->author) }}">
                        @error('author') <div class="invalid-feedback">{{ $message }}</div> @enderror
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Описание</label>
                        <textarea name="description" rows="3"
                                  class="form-control @error('description') is-invalid @enderror">{{ old('description', $book->description) }}</textarea>
                        @error('description') <div class="invalid-feedback">{{ $message }}</div> @enderror
                    </div>
                    <div class="row g-3 mb-3">
                        <div class="col-md-4">
                            <label class="form-label">Статус</label>
                            <select name="status" class="form-select">
                                <option value="want_to_read" {{ old('status', $book->status) === 'want_to_read' ? 'selected' : '' }}>Хочу прочитать</option>
                                <option value="reading"      {{ old('status', $book->status) === 'reading'      ? 'selected' : '' }}>Читаю</option>
                                <option value="finished"     {{ old('status', $book->status) === 'finished'     ? 'selected' : '' }}>Прочитано</option>
                            </select>
                        </div>
                        <div class="col-md-4">
                            <label class="form-label">Оценка</label>
                            <select name="rating" class="form-select">
                                <option value="">— Без оценки —</option>
                                @for($i = 1; $i <= 5; $i++)
                                    <option value="{{ $i }}" {{ old('rating', $book->rating) == $i ? 'selected' : '' }}>
                                        {{ str_repeat('★', $i) }} ({{ $i }})
                                    </option>
                                @endfor
                            </select>
                        </div>
                        <div class="col-md-4">
                            <label class="form-label">Дата прочтения</label>
                            <input type="date" name="finished_at"
                                   class="form-control @error('finished_at') is-invalid @enderror"
                                   value="{{ old('finished_at', $book->finished_at?->format('Y-m-d')) }}">
                            @error('finished_at') <div class="invalid-feedback">{{ $message }}</div> @enderror
                        </div>
                    </div>
                    <div class="mb-4">
                        <label class="form-label">Жанр</label>
                        <select name="genre_id" class="form-select">
                            <option value="">— Без жанра —</option>
                            @foreach($genres as $genre)
                                <option value="{{ $genre->id }}" {{ old('genre_id', $book->genre_id) == $genre->id ? 'selected' : '' }}>
                                    {{ $genre->name }}
                                </option>
                            @endforeach
                        </select>
                    </div>
                    <div class="d-flex gap-2">
                        <button type="submit" class="btn btn-primary">Сохранить</button>
                        <a href="{{ route('books.index') }}" class="btn btn-outline-secondary">Отмена</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
@endsection
```

---

`resources/views/genres/index.blade.php`
```html
@extends('layouts.app')
@section('title', 'Жанры')
@section('content')
<div class="d-flex justify-content-between align-items-center mb-4">
    <h1>Жанры</h1>
    <a href="{{ route('genres.create') }}" class="btn btn-primary">+ Новый жанр</a>
</div>

@forelse($genres as $genre)
    <div class="card mb-2">
        <div class="card-body d-flex justify-content-between align-items-center py-2">
            <div class="d-flex align-items-center gap-3">
                <span>{{ $genre->name }}</span>
                <small class="text-muted">{{ $genre->books_count }} книг</small>
            </div>
            <div class="d-flex gap-2">
                <a href="{{ route('genres.edit', $genre) }}" class="btn btn-sm btn-outline-secondary">Редактировать</a>
                <form action="{{ route('genres.destroy', $genre) }}" method="POST"
                      onsubmit="return confirm('Удалить жанр?')">
                    @csrf @method('DELETE')
                    <button class="btn btn-sm btn-outline-danger">Удалить</button>
                </form>
            </div>
        </div>
    </div>
@empty
    <p class="text-muted">Жанров пока нет.</p>
@endforelse
@endsection
```

---

`resources/views/genres/create.blade.php`
```html
@extends('layouts.app')
@section('title', 'Новый жанр')
@section('content')
<div class="row justify-content-center">
    <div class="col-md-5">
        <div class="card shadow-sm">
            <div class="card-body p-4">
                <h4 class="mb-4">Новый жанр</h4>
                <form action="{{ route('genres.store') }}" method="POST">
                    @csrf
                    <div class="mb-4">
                        <label class="form-label">Название</label>
                        <input type="text" name="name"
                               class="form-control @error('name') is-invalid @enderror"
                               value="{{ old('name') }}" autofocus>
                        @error('name') <div class="invalid-feedback">{{ $message }}</div> @enderror
                    </div>
                    <div class="d-flex gap-2">
                        <button type="submit" class="btn btn-primary">Создать</button>
                        <a href="{{ route('genres.index') }}" class="btn btn-outline-secondary">Отмена</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
@endsection
```

---

`resources/views/genres/edit.blade.php`
```html
@extends('layouts.app')
@section('title', 'Редактировать жанр')
@section('content')
<div class="row justify-content-center">
    <div class="col-md-5">
        <div class="card shadow-sm">
            <div class="card-body p-4">
                <h4 class="mb-4">Редактировать жанр</h4>
                <form action="{{ route('genres.update', $genre) }}" method="POST">
                    @csrf
                    @method('PUT')
                    <div class="mb-4">
                        <label class="form-label">Название</label>
                        <input type="text" name="name"
                               class="form-control @error('name') is-invalid @enderror"
                               value="{{ old('name', $genre->name) }}" autofocus>
                        @error('name') <div class="invalid-feedback">{{ $message }}</div> @enderror
                    </div>
                    <div class="d-flex gap-2">
                        <button type="submit" class="btn btn-primary">Сохранить</button>
                        <a href="{{ route('genres.index') }}" class="btn btn-outline-secondary">Отмена</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
@endsection
```
