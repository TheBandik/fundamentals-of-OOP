# ORM

## Определение

ORM (англ. object-relational mapping, рус. объектно-реляционное отображение) — технология программирования, которая связывает базы данных с концепциями объектно-ориентированных языков программирования, создавая «виртуальную объектную базу данных». Существуют как проприетарные, так и свободные реализации этой технологии.

Задачи:

+ Обеспечение абстракции работы с базой данных
+ Предоставление интерфейса для CRUD операций над данными

## Особоенности использования

+ Много шаблонного кода (теряется много времени, допуск ошибок)
+ Оптимизация SQL запросов (binding переменных, изменение схемы данных требует изменения кода)
+ Сложности сопровождения проекта (миграция схемы данных, SQL инъекции, смена СУБД)

## SQLAlchemy

SQLAlchemy — это библиотека на языке Python для работы с реляционными СУБД с применением технологии ORM. Служит для синхронизации объектов Python и записей реляционной базы данных. SQLAlchemy позволяет описывать структуры баз данных и способы взаимодействия с ними на языке Python без использования SQL.

![lec7-1](lec7-1.png)

Использование SQLAlchemy для автоматической генерации SQL-кода имеет несколько преимуществ по сравнению с ручным написанием SQL:

+ Безопасность. Параметры запросов экранируются, что делает атаки типа внедрение SQL-кода маловероятными
+ Производительность. Повышается вероятность повторного использования запроса к серверу базы данных, что может позволить ему в некоторых случаях применить повторно план выполнения запроса
+ Переносимость. SQLAlchemy, при должном подходе, позволяет писать код на Python, совместимый с несколькими back-end СУБД. Несмотря на стандартизацию языка SQL, между базами данных имеются различия в его реализации, абстрагироваться от которых и помогает SQLAlchemy

### Создание схемы базы данных

#### Создание моделей

Модель — это класс Python, соответствующий таблице в базе данных, а его свойства — это колонки.

Чтобы класс был валидной моделью, нужно соответствовать следующим требованиям:

+ Наследоваться от декларативного базового класса с помощью вызова функции declarative_base()
+ Объявить имя таблицы с помощью атрибута __tablename__
+ Объявить как минимум одну колонку, которая должна быть частью первичного ключа

Следующий код показывает, как создать модель Post, которая используется для сохранения постов в блоге:

```python
from datetime import datetime

from sqlalchemy import Integer, String, Column, DateTime
from sqlalchemy.ext.declarative import declarative_base


# Базовый класс
Base = declarative_base()

class Post(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    title = Column(String(100), nullable=False)
    slug = Column(String(100), nullable=False)
    content = Column(String(50), nullable=False)
    published = Column(String(200), nullable=False, unique=True)    
    created_on = Column(DateTime(), default=datetime.now)
    updated_on = Column(DateTime(), default=datetime.now, onupdate=datetime.now)

```

#### Добавление ключей и ограничений

При использовании ORM ключи и ограничения добавляются с помощью атрибута __table_args__

```python
from datetime import datetime

from sqlalchemy import Integer, String, Column, DateTime, PrimaryKeyConstraint, UniqueConstraint, ForeignKeyConstraint
from sqlalchemy.ext.declarative import declarative_base


# Базовый класс
Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer)
    username = Column(String(100), nullable=False)
    email = Column(String(100), nullable=False)    
    password = Column(String(200), nullable=False)
    
    __table_args__ = (
        PrimaryKeyConstraint('id', name='user_pk'),
        UniqueConstraint('username'),
        UniqueConstraint('email')
    )


class Post(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    title = Column(String(100), nullable=False)
    slug = Column(String(100), nullable=False)
    content = Column(String(50), nullable=False)
    published = Column(String(200), nullable=False, default=False)
    user_id = Column(Integer, nullable=False)
    created_on = Column(DateTime(), default=datetime.now)
    updated_on = Column(DateTime(), default=datetime.now, onupdate=datetime.now)
    
    __table_args__ = (
        ForeignKeyConstraint(['user_id'], ['users.id'])
    )
```

#### Отношения

##### Один ко многим

Отношение один-ко-многим создается за счет передачи внешнего ключа в дочерний класс. Например:

```python
from sqlalchemy import Integer, String, Column, SmallInteger, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship


# Базовый класс
Base = declarative_base()
class Author(Base):
    __tablename__ = 'authors'
    id = Column(Integer, primary_key=True)
    first_name = Column(String(100), nullable=False)
    last_name = Column(String(100), nullable=False)
    books = relationship("Book")

class Book(Base):
    __tablename__ = 'books'
    id = Column(Integer, primary_key=True)
    title = Column(String(100), nullable=False)
    copyright = Column(SmallInteger, nullable=False)
    author_id = Column(Integer, ForeignKey('authors.id'))
```

Имея объект `a` класса `Author`, можно получить доступ к его книгам можно через `a.books`. А если нужно получить автора книги через объект `Book`?

Для этого можно определить отдельное отношение `relationship` в модели `Author`:

```python
from sqlalchemy import Integer, String, Column, SmallInteger, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship


# Базовый класс
Base = declarative_base()
class Author(Base):
    __tablename__ = 'authors'
    id = Column(Integer, primary_key=True)
    first_name = Column(String(100), nullable=False)
    last_name = Column(String(100), nullable=False)
    books = relationship("Book")

class Book(Base):
    __tablename__ = 'books'
    id = Column(Integer, primary_key=True)
    title = Column(String(100), nullable=False)
    copyright = Column(SmallInteger, nullable=False)
    author_id = Column(Integer, ForeignKey('authors.id'))
    author = relationship("Author")
```
Теперь через объект `b` класса `Book` можно получить автора `b.author`.

##### Один ко одному

Установка отношения один-к-одному в SQLAlchemy почти не отличается от одного-ко-многим. Единственное отличие в том, что нужно передать дополнительный аргумент `uselist=False` в функцию `relationship`. Например:

```python
from sqlalchemy import Integer, String, Column, ForeignKey, Date
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship


# Базовый класс
Base = declarative_base()

class Person(Base):
    __tablename__ = 'persons'
    id = Column(Integer(), primary_key=True)
    name = Column(String(255), nullable=False)
    designation = Column(String(255), nullable=False)
    doj = Column(Date(), nullable=False)
    dl = relationship('DriverLicense', uselist=False)


class DriverLicense(Base):
    __tablename__ = 'driverlicense'
    id = Column(Integer(), primary_key=True)
    license_number = Column(String(255), nullable=False)
    renewed_on = Column(Date(), nullable=False)
    expiry_date = Column(Date(), nullable=False)
    person_id = Column(Integer(), ForeignKey('persons.id'))
```

##### Многие ко многим

Для отношения многие-ко-многим нужна отдельная таблица. Она создается как экземпляр класса `Table` и затем соединяется с моделью с помощью аргумента `secondary` функции `relationship`.

```python
from sqlalchemy import Integer, String, Column, ForeignKey, SmallInteger, Table
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship


# Базовый класс
Base = declarative_base()

author_book = Table('author_book', Base.metadata, 
    Column('author_id', Integer(), ForeignKey("authors.id")),
    Column('book_id', Integer(), ForeignKey("books.id"))
)

class Author(Base):
    __tablename__ = 'authors'
    id = Column(Integer, primary_key=True)
    first_name = Column(String(100), nullable=False)
    last_name = Column(String(100), nullable=False)


class Book(Base):
    __tablename__ = 'books'
    id = Column(Integer, primary_key=True)
    title = Column(String(100), nullable=False)
    copyright = Column(SmallInteger, nullable=False)
    author_id = Column(Integer, ForeignKey('authors.id'))
    author = relationship("Author", secondary=author_book, backref="books")
```

Может потребоваться хранить дополнительную информацию в промежуточной таблице. Для этого нужно определить эту таблицу как класс модели:

```python
from sqlalchemy import Integer, String, Column, ForeignKey, SmallInteger, Table
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship


# Базовый класс
Base = declarative_base()

class Author_Book(Base):
    __tablename__ = 'author_book'
    id = Column(Integer, primary_key=True)
    author_id =  Column(Integer(), ForeignKey("authors.id"))
    book_id =  Column(Integer(), ForeignKey("books.id"))
    extra_data = Column(String(100))

class Author(Base):
    __tablename__ = 'authors'
    id = Column(Integer, primary_key=True)
    first_name = Column(String(100), nullable=False)
    last_name = Column(String(100), nullable=False)
    books = relationship("Author_Book", backref='author')

class Book(Base):
    __tablename__ = 'books'
    id = Column(Integer, primary_key=True)
    title = Column(String(100), nullable=False)
    copyright = Column(SmallInteger, nullable=False)    
    authors = relationship("Author_Book", backref="book")
```

#### Создание таблиц

```python
from sqlalchemy import create_engine, Integer, String, Column, DateTime, ForeignKey, Numeric, SmallInteger

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

from datetime import datetime

from settings import login, password

engine = create_engine(f'mysql+pymysql://{login}:{password}@192.168.50.102:3306/shop')

Base = declarative_base()

class Customer(Base):
    __tablename__ = 'customers'
    id = Column(Integer(), primary_key=True)
    first_name = Column(String(100), nullable=False)
    last_name = Column(String(100), nullable=False)
    username = Column(String(50), nullable=False)
    email = Column(String(200), nullable=False)
    created_on = Column(DateTime(), default=datetime.now)
    updated_on = Column(DateTime(), default=datetime.now, onupdate=datetime.now)
    orders = relationship("Order", backref='customer')
    

class Item(Base):
    __tablename__ = 'items'
    id = Column(Integer(), primary_key=True)
    name = Column(String(200), nullable=False)
    cost_price =  Column(Numeric(10, 2), nullable=False)
    selling_price = Column(Numeric(10, 2),  nullable=False)
    quantity = Column(Integer())


class Order(Base):
    __tablename__ = 'orders'
    id = Column(Integer(), primary_key=True)
    customer_id = Column(Integer(), ForeignKey('customers.id'))
    date_placed = Column(DateTime(), default=datetime.now)
    line_items = relationship("OrderLine", backref='order')
    

class OrderLine(Base):
    __tablename__ = 'order_lines'
    id =  Column(Integer(), primary_key=True)
    order_id = Column(Integer(), ForeignKey('orders.id'))
    item_id = Column(Integer(), ForeignKey('items.id'))
    quantity = Column(SmallInteger())
    item = relationship("Item")

Base.metadata.create_all(engine)
```

#### Подключение к БД

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import Session

engine = create_engine(f'mysql+pymysql://{login}:{password}@192.168.50.102:3306/shop')
session = Session(bind=engine)
```

#### INSERT

Для создания новой записи с помощью SQLAlchemy ORM нужно выполнить следующие шаги:

+ Создать объект
+ Добавить его в сессию
+ Сохранить сессию

```python
c1 = Customer(
    first_name = 'Alex',
    last_name = 'Morgan',
    username = 'almon',
    email = 'almon@mail.com'
)

session.add(c1)
session.commit()
```

```python
i1 = Item(name = 'Chair', cost_price = 9.21, selling_price = 10.81, quantity = 5)
i2 = Item(name = 'Pen', cost_price = 3.45, selling_price = 4.51, quantity = 3)
i3 = Item(name = 'Headphone', cost_price = 15.52, selling_price = 16.81, quantity = 50)
i4 = Item(name = 'Travel Bag', cost_price = 20.1, selling_price = 24.21, quantity = 50)
i5 = Item(name = 'Keyboard', cost_price = 20.1, selling_price = 22.11, quantity = 50)
i6 = Item(name = 'Monitor', cost_price = 200.14, selling_price = 212.89, quantity = 50)
i7 = Item(name = 'Watch', cost_price = 100.58, selling_price = 104.41, quantity = 50)
i8 = Item(name = 'Water Bottle', cost_price = 20.89, selling_price = 25, quantity = 50)
```

```python
o1 = Order(customer = c1)
o2 = Order(customer = c1)

line_item1 = OrderLine(order = o1, item = i1, quantity =  3)
line_item2 = OrderLine(order = o1, item = i2, quantity =  2)
line_item3 = OrderLine(order = o2, item = i1, quantity =  1)
line_item3 = OrderLine(order = o2, item = i2, quantity =  4)
```

#### SELECT

Чтобы сделать запрос в базу данных используется метод `query` объекта `session`. Он возвращает объект типа `sqlalchemy.orm.query.Query`, который называется просто `Query`. Он представляет собой инструкцию `SELECT`, которая будет использована для запроса в базу данных.

##### all

В базовой форме метод `query` принимает в качестве аргументов один или несколько классов модели или колонок. Следующий код вернет все записи из таблицы `customers`.

```python
session.query(Customer).all()
```

##### count

```python
session.query(Item).count()
```

##### filter

```python
session.query(Customer).filter(Customer.first_name == 'Vadim').all()
```

#### UPDATE

Для обновления объекта просто установите новое значение атрибуту, добавьте объект в сессию и сохраните ее.

```python
i = session.query(Item).get(8)
i.selling_price = 25.91
session.add(i)
session.commit()
```

#### DELETE

Для удаления объекта используйте метод `delete` объекта сессии. Он принимает объект и отмечает его как удаленный для следующего коммита.

```python
i = session.query(Item).filter(Item.name == 'Monitor').one()
session.delete(i)
session.commit()
```

## SQL инъекции

SQL инъекции - один из распространенных способов взлома сайтов и программ, работающих с базами данных, основанный на внедрении в запрос произвольного SQL кода.

```sql
-- https://example.com?id=1

select * from news where id = '$id'

-- https://example.com?id=1'

select * from news where id = '1''
```

```sql
-- Исходный запрос
select * from users where login = '$login' and pass = '$pass'

-- Передаем login = admin' --
select * from users where login = 'admin' --' and pass = '$pass'
```
