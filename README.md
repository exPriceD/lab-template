# Лабораторная работа по созданию веб-приложения с использованием Flask

## Цель работы
Научиться создавать веб-приложение, используя фреймворк Flask на языке программирования Python. Реализовать взаимодействие между фронтендом и бекендом для отображения данных из API.
Для решения лабораторной работы Вам понадобится:
-	Язык программирования Python + среда разработки (желательно PyCharm или VS Code).
-	Основы работы с Python.
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

- ``discount_value`` - целочисленное поле\
- ``image`` - строка с максимальной длиной 512 символов
  
## Полезные материалы
-	https://flask.palletsprojects.com/en/latest/
-	https://learn.javascript.ru/fetch
-	https://learn.javascript.ru/dom-nodes
-	https://habr.com/ru/companies/macloud/articles/557422/
