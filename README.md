# Лабораторная работа по созданию веб-приложения с использованием Flask

## Цель работы
Научиться создавать веб-приложение, используя фреймворк Flask на языке программирования Python. Реализовать взаимодействие между фронтендом и бекендом для отображения данных из API.
Для решения лабораторной работы Вам понадобится:
-	Язык программирования Python + среда разработки (желательно PyCharm или VS Code).
-	Основы языкапрограммирования Python.
-	Установленный Git на локальной машине.

## Настройка проекта
Клонируйте репозиторий с GitHub.

```git clone <URL>```

После клонирования Вы можете видеть несколько папок и файлы. Это основные файлы, которые мы будем использовать при разработке. 
-	**instance** – папка, в которой хранится наша база данных (про нее чуть позже)
-	**static** – папка, содержащая статические файлы для клиентской части приложения
-	**templates** – папка, в которой мы будем хранить html-разметку и шаблоны
-	**.env** – файл для хранения переменных окружения 
- **requirements.txt** - файл, содержащий список пакетов или библиотек, необходимых для работы над проектом.
  
Можно видеть, что клиентская часть приложения уже готова. Написана html разметка страницы, прописаны все стили и присутствуют изображения.

Теперь погрузимся в Python и фреймворк Flask. 

Flask - это легкий фреймворк для создания веб-приложений на языке программирования Python. Он разработан с акцентом на простоту и легкость использования, предоставляя необходимый минимум для создания веб-приложений.

Установим все требуемые пакеты с помощью команды 
```pip install -r requirements.txt```

И попробуем создать простое веб-приложение на Flask, для этого создадим файл с названием ``test.py`` и напишем в нем такой код:

```python
from flask import Flask
application = Flask(__name__)

@application.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug=True)
```
Запустим программу командой ```python test.py```:

После запуска в консоле отобразятся основные настройки нашего приложения и ссылка, по которой мы можем попасть на страницу сайта. По стандарту это ``http://127.0.0.1:5000``

Если перейти по этой ссылке, то мы можем увидеть текст ``Hello, World!``


**Разберемся как работает этот код.**

 - В самой первой строке мы импортировали класс Flask из модуля flask. 

 - Далее нам требуется создать само приложение, это мы делаем с помощью ``application = Flask(__name__)``

 - Теперь у нас есть приложение ``application`` и мы можем задать маршрут с помощью декоратора ``@application.route('/')``, где ``/`` наш маршрут. 

 - После декоратора мы создаем функцию (название может быть любым), которая возвращает строку "Hello, World!"

 - В самом конце мы делаем проверку, является ли файл основным, если да, то запускаем приложение в режиме debug ``app.run(debug=True)``. Режим debug автоматически перезагружает приложение при измениях в коде.

Теперь определимся со структорой нашего проекта.\
Мы бы могли написать весь код в одном файле, но тогда приверженность этому стилю может привести к ряду проблем, когда приложение становится более сложным и масштабируемым.

Мы предлагаем такую структуру:
```
├── instance
│     └── database.db
| 
├── static
│     ├── css
|         ├── reset.css
|         └── styles.css
│     ├── fonts
|         └── ...
│     ├── img
|         └── ...
│     └── js
|         └── script.js
| 
├── templates
|     └── index.html
| 
├── app.py - файл для запуска программы
├── config.py - конфигурация веб-приложения
├── models.py - модели бд
├── views.py - основная логика и эндпоинты
| 
├── .env
└── requirements.txt
```

Большая часть файлов у вас уже имеется, создайте остальные.

Приступим к конфигурации веб-приложения. Откройте файл ``config.py`` и импортируйте в него следующие пакеты:
```python
from flask import Flask
from dotenv import load_dotenv
from flask_sqlalchemy import SQLAlchemy
import os
```

- ``dotenv`` нужен для для загрузки переменных окружения из файла ``.env``
- ``flask_sqlalchemy`` потребуется нам при создании базы данных и дальнейшей работы с ней.

Теперь инициализируем приложение и укажем путь конфигу. 

```python
application = Flask(__name__)
application.config.from_object(__name__)
```

``__name__`` - указывает на название текущего файла, в нашем случае ``config.py``

Займемся настройкой самого конфига:
```python
application.secret_key = os.getenv("SECRET_KEY")
application.config['SQLALCHEMY_DATABASE_URI'] = os.getenv("SQLALCHEMY_DATABASE_URI")
application.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
application.config['JSON_AS_ASCII'] = False
```

Здесь:
- ``secret_key`` в Flask используется для подписывания кук (cookie) и других данных, таких как сессии. Это важное средство для обеспечения безопасности вашего веб-приложения.
- ``SQLALCHEMY_DATABASE_URI`` - путь к базе данных
- ``SQLALCHEMY_TRACK_MODIFICATIONS`` - отслеживание изменения в базе данных
- ``JSON_AS_ASCII`` - определяет, следует ли использовать кодировку ASCII для преобразования строк в формат JSON

И создадим экземпляр класса SQLAlchemy, свяжем его с нашим приложением:
```python
db = SQLAlchemy(application)
```

В результате файле ``config.py`` должен выглядеть так:
```python
from flask import Flask
from dotenv import load_dotenv
from flask_sqlalchemy import SQLAlchemy
import os

load_dotenv()

application = Flask(__name__)
application.config.from_object(__name__)

application.secret_key = os.getenv("SECRET_KEY")
application.config['SQLALCHEMY_DATABASE_URI'] = os.getenv("SQLALCHEMY_DATABASE_URI")
application.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
application.config['JSON_AS_ASCII'] = False

db = SQLAlchemy(application)
```

Однако, файл .env пуст, и приложение не сможет найти наш ``SECRET_KEY`` и ``SQLALCHEMY_DATABASE_URI``

Откроем файл ``.env`` и добавим эти переменные:

```python
SECRET_KEY="your_secret_key"
SQLALCHEMY_DATABASE_URI="sqlite:///database.db"
```
Секретный ключ может быть любым

На этом конфигурация проекта завершена, можем приступать к основной работе :)

## Создание моделей базы данных
В прошлой главе мы упоминали о базе данных и SQLAlchemy.
В нашем проекте мы будем использовать базу данных ``SQLite``. Это встраиваемая реляционная база данных, которая предоставляет легкую и простую в использовании систему управления базами данных. Она не требует отдельного сервера и хранит базу данных в одном файле на диске. Мы заранее создали базу данных и наполнили ее данными, которые вам понадобятся.

SQLAlchemy - это библиотека для работы с базами данных в языке программирования Python. Она предоставляет ORM (Object-Relational Mapping) — механизм, который позволяет вам взаимодействовать с базой данных, используя объекты Python вместо языка SQL.

Зайдем в файл ``models.py`` и импортируем переменную ``db`` из ``config.py``:

```python
from config import db
```

Теперь создадим модель ``Items`` с полями:
- id: int - порядковый номер записи в базе данных
- name: str - название товара
- price: int - цена товара
- discount: bool - присутсвует ли скидка
- discount_value: int - размер скидки в процентах, например, 10, 15, 20
- image: str - путь к изображению товара, например, static/img/ps5.png

Создадим класс Items и пропишем основыне поля:
```python
class Items(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(512))
    price = db.Column(db.Integer)
    discount = db.Column(db.Boolean)
    discount_value = ...
    image = ...

    def __repr__(self):
        return f'Product {self.id}'
```
Здесь:

``class Items(db.Model)``: Определяет класс Items, который является моделью данных для работы с базой данных.

``id = db.Column(db.Integer, primary_key=True)``: Определяет поле id как целочисленный тип данных и устанавливает его как первичный ключ (primary key) в базе данных.

``name = db.Column(db.String(512))``: Определяет поле name как строковый тип данных с максимальной длиной 512 символов.

Попробуйте дописать код определив поля discount_value и image. 

- ``discount_value`` - целочисленное поле
- ``image`` - строка с максимальной длиной 512 символов

Теперь мы можем использовать таблицу Items.

## Эндпоинты

Конфиг, модели настроили, теперь займемся созданием эндпоинтов

В контексте веб-разработки, термин "эндпоинт" (endpoint) обозначает конечную точку (URI, URL), доступную веб-службе или веб-приложению для взаимодействия с ней. Эндпоинт представляет собой конечную точку, к которой можно обратиться для выполнения определенной операции.

Каждый эндпоинт может предоставлять определенные функциональные возможности, например, получение данных, отправка данных, обновление данных или удаление данных. Обращение к эндпоинту выполняется посредством HTTP-запроса (например, GET, POST, PUT, DELETE).

Откроем файл views.py и создадим наш первый эндпоинт. Суть будет примерно той же, как и в файле ``test.py`` вы пробовали создать эндпоинт ``hello_world``

Импортируем требущиеся пакеты:
```python
from flask import render_template, Response
from config import application, db
from models import Items
import json
```
Первый эндпоинт будет отвечать за отображение главной страницы, которая находится в файле ``index.html``. Путь будет таким: ``/`` или же ```http://127.0.0.1:5000```

```python
@application.route('/')
def main():
    return render_template("index.html")
```
Здесь, как и в самом начале, мы использовали декоратор для указания маршрута, и объявили функцию ``main``. Однако теперь мы должны вернуть html-разметку, для этого мы можем использовать функцию ``render_template()``, в которой мы указали название файла разметки. 

Важно, чтобы файл находился именно в папке ``templates``, по стандарту Flask ищет шаблоны именно там. 

Если вы хотите изменить путь до папки с шаблонами, то вам нужно добавить строчку

``app.config['TEMPLATE_FOLDER'] = '/полный/путь/к/новой/папке/templates'``

в файле ``config.py``

В данный момент, если перейти на главную страницу сайта, то мы не увидим никаких товаров. Мы бы могли добавить товары прям в html код, однако, нам бы пришлось каждый раз добавлять новые товары вручную, что требует определенных знаний html и css.

Решением этой проблемы является хранение информации о товарах в базе данных и их автоматической подгрузкой. Тем самым, чтобы добавить новый товар на сайт, нам требуется только добавить новую запись в базу данных и изображение товара.

Определим маршрут ``/api/items`` с методом ``GET`` для получения списка товаров.

```python
@application.route('/api/items', methods=["GET"])
def get_items():
    ...
```

Данный маршрут принимает только GET запросы, для этого мы явно указали ``methods=["GET"]`` в автрибутах декоратора.

Пропишите основную логику:
```python
@application.route('/api/items', methods=["GET"])
def get_items():
    try:
        # PUT YOUR CODE HERE
    except Exception as E:
        print(E)
        response = {"status": 500, "text": "Unexpected error"}
        return Response(
            response=json.dumps(response, ensure_ascii=False),
            status=500,
            mimetype="application/json",
        )
```
Чтобы получить все записи из таблицы используйте ``items = Items.query.all()``. В ``items`` будет находится список из записей таблицы (одна запись = одна строка таблицы). Вы можете пройти по всем записям через цикл ``for item in items``, теперь вы можете обращаться к значениям полей через точку, например ``item.id``, что выведет id записи

В рузельтате у вас должен получиться словарь со данной структурой:
```json
{
  "status": 200,
  "items": [
    {
      "name": ...,
      "price": ...,
      "discount": ...,
      "discount_value": ...,
      "image": ...
    },
    {
      "name": ...,
      "price": ...,
      "discount": ...,
      "discount_value": ...,
      "image": ...
    },
    ...
  ]
}
```

После создания словаря, нужно вернуть ответ со статусом 200, если все хорошо, а также сам словарь в формате JSON. Для этого можно использовать ``Response``. Реализуйте это, пользуясь примером из блока try-except выше.

Теперь открем файл ``app.py``. В нашем проекте он будет предназначен для запуска программы, вставим в негой следующий код:

```python
from config import application, db
import views

if __name__ == '__main__':
    application.run(debug=True)
    with application.app_context():
        db.create_all()
```

Мы импортировали все маршруты из файла ``views.py``, а также само приложение и базу данных. По анологии с самым первым примером запускаем приложение через ``application.run(debug=True)``, но при этом мы добавили 2 строчки, которые отвечают за создание базы данных, если она отсуствует.

## Связь сервеной и клиентской части приложения

Страница есть, данные из бд получаем, а как же их вывести? Для этого мы будем использовть JavaScript!!! Мы бы могли использовать уже встроенный во Flask модуль ``Jinja2`` и передавать данные прям в html шаблон, однако сайты начинают использовать REST и RESTful API. 
REST API популярны из-за простоты, масштабируемости, универсальности, разделения клиента и сервера, поддержки HTTP, гибкости и безопасности. 

JavaScript (JS) - это высокоуровневый, интерпретируемый язык программирования, который применяется в первую очередь для создания веб-страниц с интерактивным поведением. Он является одним из основных технологических компонентов для разработки веб-приложений.

Создадим файл ``static/js/script.js`` и откроем его.

Вспомним, что мы создавали маршрут ``/api/items``, который вернет нам json со списком товаров после GET запроса. 

Как же нам получить этот самый json? Для этого мы будем использовать функцию ``fetch``,  она используется для отправки сетевых запросов.

Добавим в файл ``script.js`` данный код:
```javascript
window.onload = async function getItems() {
    const response = await fetch(URL);
    if (response.ok) {
        const json = await response.json();
        const items = json.items;
        displayItems(items);
    }
}
```
Вместо URL добавьте путь к эндпоинту, который возвращает json с товарами.

Попробуем разобраться, что он делает

- ``window.onload = `` - это событие, которое происходит, когда вся страница полностью загружена
- ``async function getItems() {...}`` - здесь мы объявили асинхронную функцию ``getItems()``
- ``const response = await fetch("http://127.0.0.1:5000/api/items")`` - Эта строка выполняет асинхронный запрос к указанному URL и сохраняет ответ в переменную ``response``
- ``if (response.ok) {...}`` - проверяем что статус полченного ответа находится в диапазоне от 200 до 299 (подробнее см. HTTP status code)
- ``const json = await response.json()`` - достаем json из ответа
- ``const items = json.items`` - получаем массив товаров
- ``displayItems(items)`` - вызываем функцию для отображения всех товаров на странице

Теперь поговорим про ``displayItems(items)``, что же это за функция? 

Мы получили массив ``items``, состоящий из словарей. Полдела сделано! Теперь эти данные нужно вывести на страницу, именно для этого мы напишем отдельную функцию. 

Ниже предыдущего кода, объявим функцию ``displayItems``, которая будет принимать аргумент ``items`` - список:

```javascript
function displayItems(items) {
  ...
}
```
Карточка в которой будет отображаться товар будет иметь такую структуру:
```html
<li class="card__sales">
   <div class="card__sale__amount">
      <p class="card__sale__text">скидка 15%</p>
   </div>
   <img src="static/img/favorite.svg" alt="favorite" class="favorite">
   <img src="static/img/docstation.png" alt="pink_gamepad" class="docstation">
   <a href="#!" class="card__name">Зарядная станция Sony для PlayStation VR2 Sen...</a>
   <div class="card__prices">
      <p class="card__old__price">11000 ₽</p>
      <p class="card__new__price">9350 ₽</p>
   </div>
</li>
```
Нам нужно с помощью JS создать ее.

```javascript
function displayItems(items) {
    const three_productsContainer = document.getElementById("three-products-container");
    const productsContainer = document.getElementById("products-container");
    items.forEach((product, index) => {
        console.log(index);
        const listItem = document.createElement("li");
        listItem.classList.add("card__sales");

        // Создаем блок для цен
        const pricesDiv = document.createElement("div");
        pricesDiv.classList.add("card__prices");

        // Создаем div для скидки
        if (product.discount) {
            const saleAmountDiv = document.createElement("div");
            saleAmountDiv.classList.add("card__sale__amount");

            const saleText = document.createElement("p");
            saleText.classList.add("card__sale__text");
            saleText.textContent = `скидка ${product.discount_value}%`;

            saleAmountDiv.appendChild(saleText);
            listItem.appendChild(saleAmountDiv);

            const oldPrice = document.createElement("p");
            oldPrice.classList.add("card__old__price");
            oldPrice.textContent = `${product.price} ₽`;

            const newPrice = document.createElement("p");
            newPrice.classList.add("card__new__price");
            newPrice.textContent = `${product.price * (1 - product.discount_value / 100)} ₽`;

            pricesDiv.appendChild(oldPrice);
            pricesDiv.appendChild(newPrice);
        } else {
            const price = document.createElement("p");
            price.classList.add("card__new__price");
            price.textContent = `${product.price} ₽`;

            pricesDiv.appendChild(price);
        }

        // Создаем изображение для "избранного"
        const favoriteImg = document.createElement("img");
        favoriteImg.src = "static/img/favorite.svg";
        favoriteImg.alt = "favorite";
        favoriteImg.classList.add("favorite");

        // Создаем изображение товара
        // PUT YOUR CODE HERE
        let route = product.image.split("/");
        let class_name = ((route.at(-1)).split(".")).at(0);
        productImg.classList.add(`${class_name}`);
        productImg.alt = `${class_name}`;

        // Создаем название товара
        const productName = processingProductName(product.name);

        // Добавляем созданные элементы в li
        listItem.appendChild(favoriteImg);
        listItem.appendChild(productImg);
        listItem.appendChild(productName);
        listItem.appendChild(pricesDiv);

        // Добавляем li в контейнер товаров
        if (index < 3){
            three_productsContainer.appendChild(listItem);
        } else {
            productsContainer.appendChild(listItem);
        }
    });
}

function processingProductName(name) {
    const maxChars = 45;
    const productNameLink = document.createElement("a");
    productNameLink.href = "#!";
    productNameLink.classList.add("card__name");
    productNameLink.textContent = name.length > maxChars ? name.substring(0, maxChars) + "..." : name;
    return productNameLink;
}
```

Ваша задача создать тег ``img`` с навзанием ``productImg`` и указать путь (``src``) до файла, для этого изучите, как создавались другие элементы в этом коде. 

После создания скрипта, нам нужно подключить его к HTML. Для этого зайдем в файл ``index.html`` и в теге ``head`` добавим строчку:

```html
<script type="text/javascript" src="{{url_for('static', filename='путь/к/файлу/script.js')}}"></script>
```

Здесь мы используем конструкцию url_for из Jinja2, чтобы указать путь до файла из проекта Flask.

## Финал

Теперь вы можете зайти на сайт и увидеть что все товары из базы данных отображаются на странице. Если вы хотите добавить товар, то для этого вам понадобится добавить запись в таблицу и загрузить изображения товара в папку ``static/img``. Для просмотра базы данных можно использовать ``DB Browser`` или другие приложения.  

## Полезные материалы
-	https://flask.palletsprojects.com/en/latest/
-	https://ru.hexlet.io/courses/http-api/lessons/crud/theory_unit
-	https://learn.javascript.ru/fetch
-	https://learn.javascript.ru/dom-nodes
-	https://habr.com/ru/companies/macloud/articles/557422/
