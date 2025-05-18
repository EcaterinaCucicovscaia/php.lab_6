Лабораторная работа №6

Взаимодействие с базой данных. Авторизация и аутентификация

Описание лабораторной работы
Целью данной работы является разработка веб-приложения для платформы проведения городских мероприятий. Приложение позволяет гражданам узнавать о мероприятиях, регистрироваться на них, а также реализует систему ролей и авторизации, обеспечивая разграничение доступа и управление мероприятиями через административную панель.

Структура проекта
PHP + MySQL (работа через XAMPP)

База данных: event_platform

Таблицы:

users — пользователи с ролями

roles — роли пользователей (user, manager)

events — мероприятия

event_records — записи пользователей на мероприятия

(Дополнительно) capabilities и roles_capabilities для расширенной системы прав

Инструкции по запуску проекта
Установите XAMPP (или другой локальный сервер с PHP и MySQL).

Импортируйте базу данных из файла event_platform.sql (созданного по схеме).

Скопируйте проект в папку htdocs (или аналогичную) вашего XAMPP.

Настройте параметры подключения к базе данных в файле config.php.

Запустите Apache и MySQL через панель управления XAMPP.

Откройте браузер и перейдите по адресу http://localhost/название_проекта.
Создание базы данных
CREATE DATABASE IF NOT EXISTS event_platform
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

USE event_platform;

Структура базы данных
Таблица roles
Поле	Тип	Описание
id	INT	Идентификатор роли
name	VARCHAR(50)	Название роли (user, manager)

CREATE TABLE roles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
);



Таблица users
Поле	Тип	Описание
id	INT	Идентификатор пользователя
username	VARCHAR(50)	Логин
password	VARCHAR(255)	Хэшированный пароль
email	VARCHAR(100)	Почта
role_id	INT	Внешний ключ на roles(id)

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    role_id INT NOT NULL,
    FOREIGN KEY (role_id) REFERENCES roles(id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);


Таблица events
Поле	Тип	Описание
id	INT	Идентификатор мероприятия
name	VARCHAR(100)	Название мероприятия
price	DECIMAL(8,2)	Цена мероприятия
number_seats	INT	Количество мест
date	DATETIME	Дата и время проведения

CREATE TABLE events (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(8,2) NOT NULL DEFAULT 0.00,
    number_seats INT NOT NULL,
    date DATETIME NOT NULL
);


Таблица event_records
Поле	Тип	Описание
id	INT	Идентификатор записи
user_id	INT	Внешний ключ на users(id)
event_id	INT	Внешний ключ на events(id)

CREATE TABLE event_records (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    event_id INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    FOREIGN KEY (event_id) REFERENCES events(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    UNIQUE KEY unique_user_event (user_id, event_id)
);


(Дополнительно) Таблицы для расширенной системы прав
capabilities — список возможностей (например, can_view_panel, can_edit_event)

CREATE TABLE capabilities (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);


roles_capabilities — связь ролей с возможностями

Основные страницы и функционал
1. Страница текущих мероприятий (events.php)
Вывод списка всех доступных мероприятий с краткой информацией.

Кнопка записи для авторизованных пользователей.

2. Страница записи на мероприятие (event_register.php)
Отображение детальной информации о мероприятии.

Форма записи (если есть свободные места).

Проверка и сохранение записи в event_records.

3. Страница регистрации (register.php)
Форма для создания нового пользователя.

Валидация данных.

Хранение паролей с хешированием (например, password_hash).

4. Страница авторизации (login.php)
Форма логина с проверкой введённых данных.

Создание сессии для авторизованного пользователя.

Перенаправление на главную или админ-панель в зависимости от роли.

5. Административная панель (admin_panel.php)
Доступна только пользователю с ролью manager.

Добавление и редактирование мероприятий.

Просмотр списка пользователей, записанных на конкретное мероприятие.

Примеры использования и фрагменты кода
Подключение к базе данных (config.php)

<?php
$host = 'localhost';
$db   = 'event_platform';
$user = 'root';
$pass = '';
$charset = 'utf8mb4';

$dsn = "mysql:host=$host;dbname=$db;charset=$charset";
$options = [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
];

try {
    $pdo = new PDO($dsn, $user, $pass, $options);
} catch (PDOException $e) {
    die('Ошибка подключения к базе: ' . $e->getMessage());
}
?>
Пример запроса на вывод мероприятий

$stmt = $pdo->query("SELECT * FROM events WHERE date >= NOW() ORDER BY date ASC");
$events = $stmt->fetchAll();
foreach ($events as $event) {
    echo "<h3>{$event['name']}</h3>";
    echo "<p>Дата: {$event['date']}</p>";
    echo "<p>Цена: {$event['price']} мдл.</p>";
}
Проверка роли при авторизации

session_start();
// после успешной аутентификации
$_SESSION['user_id'] = $user['id'];
$_SESSION['role'] = $user['role_name']; // например, 'user' или 'manager'

// проверка доступа
if ($_SESSION['role'] !== 'manager') {
    header('Location: access_denied.php');
    exit;
}
Ответы на контрольные вопросы
Как реализована авторизация и аутентификация?
Авторизация реализована через форму логина с проверкой логина и хеша пароля. При успешном входе создаётся сессия с данными пользователя, включая роль. Роль используется для разграничения доступа.

Как реализована связь между таблицами?
Используются внешние ключи: users.role_id ссылается на roles.id, event_records.user_id на users.id, а event_records.event_id на events.id.

Как реализован доступ к административной панели?
Доступ разрешён только пользователям с ролью manager, проверка роли происходит на уровне сессии.

Какие дополнительные возможности предусмотрены для ролей?
Возможна расширяемая система прав через таблицы capabilities и roles_capabilities, позволяющая задавать различные права для каждой роли.

Список использованных источников
Официальная документация PHP: https://www.php.net/manual/ru/

Документация MySQL: https://dev.mysql.com/doc/

Руководства по PDO: https://www.php.net/manual/ru/book.pdo.php

Учебные материалы по авторизации и сессиям в PHP

