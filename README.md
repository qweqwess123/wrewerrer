# Добро пожаловать в документацию testblog API

Данная документация описывает структуры данных (называемыми моделями) приложения и способы взаимодействия с ними через **HTTP(S) Restful API**


> Версия: 0.0.1
> URL: https://testblog-app.herokuapp.com/api/

# Table of contents

- [Добро пожаловать в документацию testblog API](#----testblog-api)
- [Об аутентификации и csrf защите](#---csrf-)
- [Модели](#)
  - [Модель - Account](#---account)
  - [Модель - Post](#---post)
  - [Модель - PostCategory](#---postcategory)
  - [Модель - PostComment](#---postcomment)
  - [Модель - PostReaction](#---postreaction)
- [Валидация данных, передаваемых клиентом](#---)
- [API Методы](#api-)
  - [GET  /api/accounts](#get--apiaccounts)
  - [POST: /accounts/](#post-accounts)
  - [GET /accounts/{id}](#get-accountsid)
  - [GET /accounts/me](#get-accountsme)
  - [PATCH /accounts/{id} или PATCH /accounts/me](#patch-accountsid--patch-accountsme)
  - [GET /accounts/login](#get-accountslogin)
  - [GET /accounts/logout](#get-accountslogout)
  - [posts](#posts)
  - [comments](#comments)
  - [/reactions](#reactions)

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
<details>
 <summary>Пример</summary>
 
    PATCH /api/accounts/5

    {"username": "newusername12"}

    HTTP 200 OK
    Allow: GET, PATCH, HEAD, OPTIONS
    Content-Type: application/json
    Vary: Accept

    {
        "id": 5,
        "email": "hihiuser44@gmail.com",
        "username": "newusername12",
        "is_staff": false,
        "is_active": true
    }
 
</details>

## GET /accounts/login
Авторизует пользователя. Принимает ключи **email** и **password**. При удачной валидации возвращает те же поля, что и при **GET /accounts/me**, а при неудачной - **403 Forbidden**

## GET /accounts/logout
Деавторизует пользователя, возвращая **204 No Content**. Если пользователь не был авторизован возвращает **403 Forbidden**

## posts
Метод для работой с данными модели Post

>  Если is_active=False, то публикация считается неактивной

 Если пост не активен, то они скрываются для обычных пользователей и не выводятся в списке для непривилегированных пользователей
 
 #### Расширяемые поля:
>**author**: все поля модели **Account**, кроме **email** и **is_active**

>**categories**: все поля модели **PostCaregory**

### GET /posts
Получение списка публикаций.

> Поля для обычных пользователей: **id, author, title, content, date, categories**

> Поля доступные только привилегированным пользователям : **is_active**

> Поля доступные для фильтрации: **titiel, author__username, is_active, limit, offset**

<details>
 <summary>Пример</summary>
 
    GET /api/posts?expand=categories,author&fields=id,author.username,title,content,date,categories.name
 
    HTTP 200 OK
    Allow: GET, POST, PATCH, HEAD, OPTIONS
    Content-Type: application/json
    Vary: Accept

    [
        {
            "id": 2,
            "author": {
                "username": "q1kebuff"
            },
            "title": "Another post",
            "content": "Content!",
            "date": "2022-02-28T12:56:35.176484+03:00",
            "categories": [
                {
                    "name": "anime"
                },
                {
                    "name": "cinema"
                }
            ]
        },
        {
            "id": 1,
            "author": {
                "username": "q1kebuff"
            },
            "title": "This is post",
            "content": "With some content",
            "date": "2022-02-28T12:55:45.113817+03:00",
            "categories": [
                {
                    "name": "cinema"
                }
            ]
        }
    ]
 
</details>


### POST /posts
Создание публикации, доступно только привилегированным пользователям. 
Поле author заполняется автоматически на основе текущей сессии


> Модель: **Post**

> Обязательные поля: **title**, **content**

> Необязательные поля: **categories(default=[])**, **is_active(default=True)**

Непривилегированным или неавторизованным пользователям возвращается ошибка **403 Forbidden**
#### Ответ 200 OK
Публикация создана успешно. Возвращает все поля модели **Post**


### PATCH /posts
Редактирование поля **is_active** для всех постов, автором которого является текущий авторизованный пользователь

>Поля: **is_active**

Непривилегированным или неавторизованным пользователям возвращается ошибка **403 Forbidden**

#### Ответ: 200 OK
Обновление постов прошло успешно. 
Возвращает `{  "count":  <количество затронутых публикаций>  }`

### GET /posts/{id}
Возвращает информацию о посте по его id с теми же полями, что и  **GET /posts**

### PATCH /posts/{id}
Обновление данных о существующем посте. Принимает те же поля и возвращает те же данные, что и  **POST /posts**

## comments
Метод для работы с данными модели PostComments.

### GET /comments
Отображает все поля модели PostComment

> Поля, доступные для фильтрации: **post**, **author**

<details>
 <summary>Пример</summary>
 
    GET /api/comments?post=2&expand=author&fields=content,author.username
 
    HTTP 200 OK
    Allow: GET, POST, HEAD, OPTIONS
    Content-Type: application/json
    Vary: Accept

    [
        {
            "content": "nice!",
            "author": {
                "username": "q1kebuff"
            }
        },
        {
            "content": "fun",
            "author": {
                "username": "q1kebuff"
            }
        }
    ]
 
</details>



### POST /comments
Метод для создания комментария. Доступен только авторизованным пользователям

> Поля: **post**, **content**

Возвращает все поля новой сущности PostComment
### GET /comments/{id}
Отображает конкретный комментарий по его id. Если id не существует, то возвращается **404 Not Found**

### PATCH /comments/{id}
Обновляет существующий комментарий. Доступно только для привилегированных пользователей или авторов комментариев. 

> Поля: **content**

#### Ответ 200 OK
Возвращает все поля обновленной сущности
### DELETE /comments/{id}
Удаляет сущность. Доступно только для привилегированных пользователей или авторов комментариев. При успешном удалении возвращает **204 No Content**

## /reactions

Метод для работы с данными модели **PostReaction**
Комбинация полей **author** и **post** должна быть уникальной, то-есть пользователь не может создать 2 сущности, содержащих ссылку на одну публикацию

### GET /reactions
Отображает список реакций, содержащий все поля модели **PostReaction**

> Поля, доступные для фильтрации: **author**, **post**, **reaction**

<details>
 <summary>Пример</summary>
 
    GET /api/reactions?expand=author,post&fields=author.username,post.title,reaction
 
    HTTP 200 OK
    Allow: GET, POST, HEAD, OPTIONS
    Content-Type: application/json
    Vary: Accept

    [
        {
            "author": {
                "username": "q1kebuff"
            },
            "reaction": "+",
            "post": {
                "title": "Another post"
            }
        },
        {
            "author": {
                "username": "q1kebuff"
            },
            "reaction": "-",
            "post": {
                "title": "This is post"
            }
        }
    ]
 
</details>

### POST /reactions
Создает новую сущность модели. Поле author определяется автоматически.
Если пользователь не авторизован, то возвращается **403 Forbidden**

> Поля: **post**, **reaction**

#### Ответ 200 OK
> Поля: **id**, **author**, **post**, **reaction**

#### Ответ 400 Bad Request
Ошибка с поясняющим текстом `[  "Reaction to this post already exist"  ]` возвращается, если пара полей (**author**, **post**) является не уникальной

### GET /reactions/{id}
Получение реакции по id сущности. Если сущность найдена, возвращает те же поля, что и **GET /reactions**, иначе возвращает **404 Not Found**

### DELETE /reactions/{id}
Удаляет сущность. Доступно только для привилегированных пользователей и для авторов реакции. При успешном удалении возвращает **204 No Content**
