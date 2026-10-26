# Lab_7


public function up(): void
{
    Schema::create('books', function (Blueprint $table) {
        $table->id();

        $table->foreignId('user_id')
              ->constrained('users')
              ->cascadeOnDelete();

        $table->string('title', 150);
        $table->string('author', 100);
        $table->unsignedInteger('pages');

        $table->timestamps();
    });
}



<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    protected $fillable = [
        'title',
        'author',
        'pages',
        'user_id',
    ];
}




<?php

namespace App\Http\Controllers;

use App\Models\Book;
use Illuminate\Http\Request;

class BookController extends Controller
{
    // Display only the logged-in user's books
    public function index()
    {
        $books = Book::where('user_id', auth()->id())
            ->latest()
            ->get();

        return view('dashboard', compact('books'));
    }

    // Register a new book
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:150',
            'author' => 'required|string|max:100',
            'pages' => 'required|integer|min:1|max:10000',
        ]);

        $validated['user_id'] = auth()->id();

        Book::create($validated);

        return redirect()
            ->route('dashboard')
            ->with('success', 'Book registered successfully!');
    }
}




Route::middleware('auth')->group(function () {

    Route::get('/dashboard', [BookController::class, 'index'])
        ->name('dashboard');

    Route::post('/books', [BookController::class, 'store'])
        ->name('books.store');

});






<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <title>My Books Dashboard</title>

    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
          rel="stylesheet">
</head>
<body class="bg-light">

<div class="container py-5">

    <div class="d-flex justify-content-between align-items-center mb-4">
        <h2>My Books Dashboard</h2>

        <form action="{{ route('logout') }}" method="POST">
            @csrf
            <button class="btn btn-outline-danger" type="submit">
                Logout
            </button>
        </form>
    </div>

    @if (session('success'))
        <div class="alert alert-success">
            {{ session('success') }}
        </div>
    @endif

    <div class="card mb-4">
        <div class="card-header">Register a New Book</div>

        <div class="card-body">
            <form action="{{ route('books.store') }}" method="POST">
                @csrf

                <div class="mb-3">
                    <label for="title" class="form-label">Title</label>
                    <input id="title" type="text" name="title"
                           maxlength="150"
                           value="{{ old('title') }}"
                           class="form-control @error('title') is-invalid @enderror">

                    @error('title')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                <div class="mb-3">
                    <label for="author" class="form-label">Author</label>
                    <input id="author" type="text" name="author"
                           maxlength="100"
                           value="{{ old('author') }}"
                           class="form-control @error('author') is-invalid @enderror">

                    @error('author')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                <div class="mb-3">
                    <label for="pages" class="form-label">Pages</label>
                    <input id="pages" type="number" name="pages"
                           min="1" max="10000"
                           value="{{ old('pages') }}"
                           class="form-control @error('pages') is-invalid @enderror">

                    @error('pages')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>

                <button type="submit" class="btn btn-primary">
                    Register Book
                </button>
            </form>
        </div>
    </div>

    <div class="card">
        <div class="card-header">My Books</div>

        <div class="card-body">
            @if ($books->isEmpty())
                <p class="text-muted mb-0">
                    No books registered yet.
                </p>
            @else
                <div class="table-responsive">
                    <table class="table table-striped table-bordered">
                        <thead class="table-dark">
                            <tr>
                                <th>No.</th>
                                <th>Title</th>
                                <th>Author</th>
                                <th>Pages</th>
                            </tr>
                        </thead>

                        <tbody>
                            @foreach ($books as $book)
                                <tr>
                                    <td>{{ $loop->iteration }}</td>
                                    <td>{{ $book->title }}</td>
                                    <td>{{ $book->author }}</td>
                                    <td>{{ $book->pages }}</td>
                                </tr>
                            @endforeach
                        </tbody>
                    </table>
                </div>
            @endif
        </div>
    </div>

</div>

</body>
</html>




<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\AuthController;
use App\Http\Controllers\BookController;

// Redirect home page to login
Route::redirect('/', '/login');

// Show registration form
Route::get('/register', function () {
    return view('auth.register');
})->name('register');

// Process registration
Route::post('/register', [AuthController::class, 'register'])
    ->name('register.store');

// Show login form
Route::get('/login', function () {
    return view('auth.login');
})->name('login');

// Process login
Route::post('/login', [AuthController::class, 'login'])
    ->name('login.store');

// Dashboard: authenticated users only
Route::get('/dashboard', [BookController::class, 'index'])
    ->middleware('auth')
    ->name('dashboard');

// Register a book: authenticated users only
Route::post('/books', [BookController::class, 'store'])
    ->middleware('auth')
    ->name('books.store');

// Logout: authenticated users only
Route::post('/logout', [AuthController::class, 'logout'])
    ->middleware('auth')
    ->name('logout');






<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\AuthController;
use App\Http\Controllers\BookController;

// Redirect home page to login
Route::redirect('/', '/login');

// Show registration page
Route::get('/register', function () {
    return view('auth.register');
})->name('register');

// Process registration
Route::post('/register', [AuthController::class, 'register'])
    ->name('register.store');

// Show login page
Route::get('/login', function () {
    return view('auth.login');
})->name('login');

// Process login
Route::post('/login', [AuthController::class, 'login'])
    ->name('login.store');

// Protected routes: authenticated users only
Route::middleware('auth')->group(function () {

    // Display dashboard and user's books
    Route::get('/dashboard', [BookController::class, 'index'])
        ->name('dashboard');

    // Save a new book
    Route::post('/books', [BookController::class, 'store'])
        ->name('books.store');

    // Logout
    Route::post('/logout', [AuthController::class, 'logout'])
        ->name('logout');
});




<?php

namespace App\Http\Controllers;

use App\Models\Book;
use Illuminate\Http\Request;

class BookController extends Controller
{
    public function index()
    {
        $books = Book::where('user_id', auth()->id())
            ->latest()
            ->get();

        return view('dashboard', compact('books'));
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:150',
            'author' => 'required|string|max:100',
            'pages' => 'required|integer|min:1|max:10000',
        ]);

        $validated['user_id'] = auth()->id();

        Book::create($validated);

        return redirect()
            ->route('dashboard')
            ->with('success', 'Book registered successfully!');
    }
}






