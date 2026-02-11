<!-- resources/views/front/layouts/frontlayout.blade.php или app.blade.php -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'Блог')</title>
    
    <!-- Подключаем Bootstrap (если ещё нет) -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    
    <style>
        .navbar-category {
            background-color: #f8f9fa;
            padding: 10px 0;
            border-bottom: 1px solid #dee2e6;
        }
        .category-item {
            margin-right: 20px;
            display: inline-block;
        }
        .category-link {
            text-decoration: none;
            color: #495057;
            font-weight: 500;
        }
        .category-link:hover {
            color: #007bff;
        }
        .post-card {
            margin-bottom: 30px;
            transition: transform 0.2s;
            border: 1px solid #ddd;
            border-radius: 8px;
            overflow: hidden;
        }
        .post-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }
        .post-image {
            height: 200px;
            object-fit: cover;
            width: 100%;
        }
        .post-content {
            padding: 20px;
        }
        .read-more-btn {
            margin-top: 10px;
            display: inline-block;
            background: #007bff;
            color: white;
            padding: 8px 16px;
            border-radius: 4px;
            text-decoration: none;
        }
        .read-more-btn:hover {
            background: #0056b3;
            color: white;
            text-decoration: none;
        }
        .post-meta {
            color: #6c757d;
            font-size: 0.9rem;
            margin-bottom: 10px;
        }
        .post-views {
            float: right;
        }
    </style>
</head>
<body>
    <!-- Подключаем header из partials -->
    @include('front.partials.header')
    
    <!-- Навигация с категориями (можно добавить в header или здесь) -->
    <nav class="navbar-category">
        <div class="container">
            <strong>Категории:</strong>
            @if(isset($categories) && $categories->count())
                @foreach($categories as $category)
                    <span class="category-item">
                        <a href="{{ route('posts.category', $category->slug) }}" 
                           class="category-link">
                            {{ $category->name }} 
                            <span class="badge bg-secondary">{{ $category->posts_count ?? 0 }}</span>
                        </a>
                    </span>
                @endforeach
            @else
                <span>Категорий пока нет</span>
            @endif
        </div>
    </nav>

    <main class="py-4">
        <div class="container">
            @yield('content')
        </div>
    </main>

    <!-- Подключаем footer из partials -->
    @include('front.partials.footer')
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
