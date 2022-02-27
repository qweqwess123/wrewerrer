# Добро пожаловать в документацию testblog API

Данная документация описывает структуры данных (называемыми моделями) приложения и способы взаимодействия с ними через **HTTP(S) Restful API**


> Версия: 0.0.1
> URL: https://testblog-app.herokuapp.com/api/

# Об аутентификации и csrf защите
Аутентификация в API происходит на уровне сессий.  Сессии обслуживаются с помощью cookie параметра **sessionid**. В случае ошибок аутентификации сервер возвращает **403 Forbidden**. Для авторизации используется метод /accounts/login 

# Модели
В этом списке будет выведено описание моделей и поля, которые они хранят
## Модель - Account
Пользователь блога


> **id: Integer(primary_key=True)**

> **email: String(max_length=60, unique=True, null=False, blank=False)**
> 
> Используется в качестве логина при аутентификации

> **username: String(max_length=60, unique=True, null=False, blank=False)**
> 
> Отображается пользователям приложения

> **date_joined: DateTime(auto_now_add=True)**

> **last_login: DateTime(auto_now=True)**

> **is_admin: Bool(default=False)**
> 
> Пользователи, являющимися администраторами имеют доступ к панели администратора  
 
> **is_staff: Bool(default=False)** 
> 
> Пользователи, являющимися частью персонала имеют право модерировать блог

## Модель - Post

Публикация в блоге. Их может создавать только модерация

> **id: Integer(primary_key=True)**

> **author: ForeignKey([Account](#модель---account), null=False, blank=False)**

> **categories: Array([PostCategory](#модель---postcategory))**
> 
> Массив, содержащий вторичные ключи категорий публикаций, "теги"

> **title: String(null=False, blank=False, max_length=200)**

> **content: String(null=False, blank=False)**

> **date: DateTime(auto_now_add=True, null=False, blank=False)**
> 
> Дата публикации поста

> **is_active: Bool(null=False, default=True)**
> 
> Если пост активен, то его могут просматривать все пользователи. Иначе - только автор или модерация



## Модель - PostCategory
Тег публикации. Одна публикация может содержать много тегов. Создает только модерация

> **id: Integer(primary_key=True)**

> **name: Slug(max_length=40, null=False, blank=False, unique=True)**
## Модель - PostComment
Комментарий к публикации. Создавать их могут все авторизованные пользователи.

> **id: Integer(primary_key=True)**

> **name: Slug(max_length=40, null=False, blank=False, unique=True)**

> **post: ForeignKey([Post](#модель--post), null=False, blank=False)**

> **author: ForeignKey([Account](#модель---account), null=False, blank=False)**
## Модель - PostReaction

Реакция к публикации. Все авторизованные пользователи могут поставить одну реакцию к посту - "лайк", или "дизлайк"

> **id: Integer(primary_key=True)**

> **post: ForeignKey([Post](#модель--post), null=False, blank=False)**

> **author: ForeignKey([Account](#модель---account), null=False, blank=False)**

> **reaction: String(max_length=1, choices=["+", "-"], null=False, blank=False)**
> Строка принимает два значения из списка - "+" или "-"
> 
# Валидация данных, передаваемых клиентом
Данные, которые передает клиент проходят валидацию. В случае ошибки сервер возвращает **400 Bad Request** и передает в теле ответа поля, не прошедшие валидацию и конкретные ошибки валидации.
<details> 
  <summary>Пример</summary>
  
    HTTP 400 Bad Request
    Allow: GET, POST, HEAD, OPTIONS
    Content-Type: application/json
    Vary: Accept

    {
        "email": [
            "Enter a valid email address."
        ],
        "password": [
            "This password is too short. It must contain at least 8 characters.",
            "This password is too common."
        ]
    }
</details>

# API Методы
В этом списке будут описаны все API методы

## GET  /api/accounts
Возвращает список аккаунтов c полями модели Account
Если запрос совершает не персонал, то поля  **email**, **is_active**  исключаются, а неактивные пользователи `(is_active=False)` не отображаются


> Фильтруемые поля: **email, username, is_admin, is_staff, is_active**
<details> 
  <summary>Пример</summary>
  
    GET  /api/accounts?limit=2
  
    HTTP 200 OK
    Allow:GET, POST, HEAD, OPTIONS
    Content-Type: application/json
    
    [
        {
            "id": 5,
            "username": "hihiuser44",
            "date_joined": "2022-02-27T21:32:04.409254+03:00",
            "last_login": "2022-02-27T21:32:04.416124+03:00",
            "is_admin": false,
            "is_staff": false
        },
        {
            "id": 4,
            "username": "usercat12",
            "date_joined": "2022-02-27T20:50:56.499264+03:00",
            "last_login": "2022-02-27T20:50:56.512828+03:00",
            "is_admin": false,
            "is_staff": false
        } 
    ]
</details>

## POST: /accounts/
Создание аккаунта
> Модель: **Account**

> Поля: **email**, **username**, **password**

> Дополнительные поля доступные для персонала: **is_staff, is_active**

### Ответ: 201 Created

Возвращает некоторые поля модели созданного пользователя и авторизует клиента на сайте передавая Cookie сессии, если он не был авторизован под другим аккаунтом

> Поля: **id, email,, username, is_active**
## GET /accounts/{id}
Возвращает информацию об аккаунте. Если запрос совершает не персонал, то неактивные аккаунты не включаются в выборку. Если аккаунт не найдет, то возвращается ошибка **404 Not Found**

> Модель: **Account**

> Поля: все, кроме **email** и **is_active**

> Дополнительные поля доступные для персонала или владельца аккаунта: **is_staff, is_active**
>
 ## GET /accounts/me
 Тоже, что и  **GET /accounts/{id}**, но вместо id используется пользователь текущей сессии. Если пользователь не авторизован возвращает **403 Forbidden**

## PATCH /accounts/{id} или PATCH /accounts/me
Обновление информации об аккаунте. Передаются и возвращаются те же поля, что и при **POST /accounts/**

## GET /accounts/login
Авторизует пользователя. Принимает ключи **email** и **password**. При удачной валидации возвращает те же поля, что и при **GET /accounts/me**, а при неудачной - **403 Forbidden**

## GET /accounts/logout
Деавторизует пользователя, возвращая **204 No Content**. Если пользователь не был авторизован возвращает **403 Forbidden**
