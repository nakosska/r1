<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

echo "<h1>🚀 УСТАНОВКА БАЗЫ ДАННЫХ</h1>";

$host = 'localhost';
$user = 'root';
$pass = '';
$dbname = 'project_manager';

try {
    // Подключаемся к MySQL
    $pdo = new PDO("mysql:host=$host", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    
    echo "<p style='color:green;'>✅ Подключение к MySQL успешно</p>";
    
    // Удаляем старую БД если есть
    $pdo->exec("DROP DATABASE IF EXISTS $dbname");
    echo "<p>🗑️ Старая БД удалена</p>";
    
    // Создаем новую БД
    $pdo->exec("CREATE DATABASE $dbname CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci");
    echo "<p style='color:green;'>✅ База данных '$dbname' создана</p>";
    
    // Выбираем БД
    $pdo->exec("USE $dbname");
    
    // 1. Таблица users
    $pdo->exec("
        CREATE TABLE users (
            id INT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(100) NOT NULL,
            email VARCHAR(100) UNIQUE NOT NULL,
            password VARCHAR(255) NOT NULL,
            role ENUM('Прораб', 'Менеджер', 'Директор', 'Бухгалтер') NOT NULL,
            phone VARCHAR(20),
            is_active TINYINT DEFAULT 1,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        ) ENGINE=InnoDB
    ");
    echo "<p>✅ Таблица 'users' создана</p>";
    
    // 2. Таблица projects
    $pdo->exec("
        CREATE TABLE projects (
            id INT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(200) NOT NULL,
            address VARCHAR(300),
            client VARCHAR(200),
            description TEXT,
            status VARCHAR(20) DEFAULT 'planning',
            progress INT DEFAULT 0,
            budget DECIMAL(15,2) DEFAULT 0,
            spent DECIMAL(15,2) DEFAULT 0,
            deadline DATE,
            start_date DATE,
            tasks_total INT DEFAULT 0,
            tasks_completed INT DEFAULT 0,
            workers INT DEFAULT 0,
            manager_id INT,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (manager_id) REFERENCES users(id) ON DELETE SET NULL
        ) ENGINE=InnoDB
    ");
    echo "<p>✅ Таблица 'projects' создана</p>";
    
    // 3. Таблица tasks
    $pdo->exec("
        CREATE TABLE tasks (
            id INT AUTO_INCREMENT PRIMARY KEY,
            title VARCHAR(200) NOT NULL,
            description TEXT,
            project_id INT NOT NULL,
            assignee_id INT,
            created_by INT,
            deadline DATE,
            status VARCHAR(20) DEFAULT 'todo',
            priority VARCHAR(10) DEFAULT 'medium',
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
            FOREIGN KEY (assignee_id) REFERENCES users(id) ON DELETE SET NULL,
            FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE SET NULL
        ) ENGINE=InnoDB
    ");
    echo "<p>✅ Таблица 'tasks' создана</p>";
    
    // 4. Таблица task_comments
    $pdo->exec("
        CREATE TABLE task_comments (
            id INT AUTO_INCREMENT PRIMARY KEY,
            task_id INT NOT NULL,
            user_id INT NOT NULL,
            comment TEXT NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (task_id) REFERENCES tasks(id) ON DELETE CASCADE,
            FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
        ) ENGINE=InnoDB
    ");
    echo "<p>✅ Таблица 'task_comments' создана</p>";
    
    // 5. Таблица task_photos
    $pdo->exec("
        CREATE TABLE task_photos (
            id INT AUTO_INCREMENT PRIMARY KEY,
            task_id INT NOT NULL,
            user_id INT NOT NULL,
            photo_path VARCHAR(500) NOT NULL,
            description TEXT,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (task_id) REFERENCES tasks(id) ON DELETE CASCADE,
            FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
        ) ENGINE=InnoDB
    ");
    echo "<p>✅ Таблица 'task_photos' создана</p>";
    
    // 6. Таблица acts
    $pdo->exec("
        CREATE TABLE acts (
            id INT AUTO_INCREMENT PRIMARY KEY,
            number VARCHAR(50) NOT NULL,
            date DATE NOT NULL,
            project_id INT NOT NULL,
            sum DECIMAL(15,2) NOT NULL,
            status VARCHAR(20) DEFAULT 'waiting',
            type VARCHAR(100),
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE
        ) ENGINE=InnoDB
    ");
    echo "<p>✅ Таблица 'acts' создана</p>";
    
    // 7. Таблица notifications
    $pdo->exec("
        CREATE TABLE notifications (
            id INT AUTO_INCREMENT PRIMARY KEY,
            user_id INT NOT NULL,
            type VARCHAR(50),
            title VARCHAR(200) NOT NULL,
            message TEXT,
            link VARCHAR(500),
            is_read TINYINT DEFAULT 0,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
        ) ENGINE=InnoDB
    ");
    echo "<p>✅ Таблица 'notifications' создана</p>";
    
    // Добавляем тестовых пользователей
    $password = password_hash('123', PASSWORD_DEFAULT);
    
    $stmt = $pdo->prepare("INSERT INTO users (name, email, password, role, phone) VALUES (?, ?, ?, ?, ?)");
    
    $users = [
        ['Иван Петров', 'prorab@test.ru', $password, 'Прораб', '+79991234567'],
        ['Анна Смирнова', 'manager@test.ru', $password, 'Менеджер', '+79995678901'],
        ['Алексей Морозов', 'director@test.ru', $password, 'Директор', '+79996789012'],
        ['Елена Соколова', 'buh@test.ru', $password, 'Бухгалтер', '+79997890123']
    ];
    
    foreach ($users as $u) {
        $stmt->execute($u);
    }
    echo "<p style='color:green;'>✅ 4 тестовых пользователя добавлены</p>";
    
    // Добавляем тестовые проекты
    $pdo->exec("
        INSERT INTO projects (name, address, client, status, progress, budget, spent, deadline, start_date, manager_id) VALUES
        ('ЖК Восход', 'ул. Ленина, 15', 'ООО СтройИнвест', 'active', 75, 1500000, 1125000, '2025-06-01', '2025-01-15', 2),
        ('Коттедж Берег', 'ул. Береговая, 7', 'Иванов А.А.', 'active', 30, 850000, 255000, '2025-07-01', '2025-02-01', 2),
        ('Ремонт офиса', 'БЦ Плаза', 'ООО ТехноСервис', 'pause', 60, 2100000, 1260000, '2025-05-10', '2025-01-20', 2)
    ");
    echo "<p style='color:green;'>✅ 3 тестовых проекта добавлены</p>";
    
    // Добавляем тестовые задачи
    $pdo->exec("
        INSERT INTO tasks (title, description, project_id, assignee_id, created_by, deadline, status, priority) VALUES
        ('Сделать замеры', 'Провести замеры помещений', 1, 1, 2, '2025-04-20', 'todo', 'high'),
        ('Завезти материалы', 'Доставка стройматериалов', 2, 1, 2, '2025-04-18', 'inprogress', 'high'),
        ('Сдать акт', 'Подготовить акт работ', 3, 1, 2, '2025-04-25', 'todo', 'medium'),
        ('Проверить стяжку', 'Проверка качества', 1, 1, 2, '2025-04-19', 'review', 'medium'),
        ('Обновить чертежи', 'Внести изменения', 2, 1, 2, '2025-04-22', 'todo', 'low')
    ");
    echo "<p style='color:green;'>✅ 5 тестовых задач добавлено</p>";
    
    // Обновляем счетчики задач
    $pdo->exec("UPDATE projects SET tasks_total = 2 WHERE id = 1");
    $pdo->exec("UPDATE projects SET tasks_total = 2 WHERE id = 2");
    $pdo->exec("UPDATE projects SET tasks_total = 1 WHERE id = 3");
    
    echo "<hr>";
    echo "<h2 style='color:green;'>✅✅✅ БАЗА ДАННЫХ УСПЕШНО УСТАНОВЛЕНА!</h2>";
    
    // Проверяем пользователей
    $stmt = $pdo->query("SELECT id, name, email, role FROM users");
    echo "<h3>Пользователи в БД:</h3>";
    echo "<table border='1' cellpadding='5'>";
    echo "<tr><th>ID</th><th>Имя</th><th>Email</th><th>Роль</th></tr>";
    while ($u = $stmt->fetch()) {
        echo "<tr><td>{$u['id']}</td><td>{$u['name']}</td><td>{$u['email']}</td><td>{$u['role']}</td></tr>";
    }
    echo "</table>";
    
    echo "<p style='margin-top: 30px;'><a href='login.php' style='background: #f97316; color: white; padding: 15px 30px; text-decoration: none; border-radius: 8px; font-size: 18px;'>👉 ПЕРЕЙТИ НА ВХОД</a></p>";
    echo "<p><strong>Логин:</strong> manager@test.ru<br><strong>Пароль:</strong> 123</p>";
    
} catch (PDOException $e) {
    echo "<h2 style='color:red;'>❌ ОШИБКА</h2>";
    echo "<p><b>Текст ошибки:</b> " . $e->getMessage() . "</p>";
    echo "<p><b>Код ошибки:</b> " . $e->getCode() . "</p>";
}
?>
