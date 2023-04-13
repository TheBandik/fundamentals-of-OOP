# Работа с MySQL в Python с помощью библиотеки pymysql

## Установка библиотеки

```
pip install pymysql
```

## Подключение к базе данных

```python
db = pymysql.connect(
    host='host',
    port='port',
    user='user',
    password='password',
    db='db',
    charset='charset'
)

cursor = db.cursor()
```

## Работа с базой данных

Для отправки `SQL` запроса в базу данных используется:

```python
cursor.execute('select * from games')
```

Для получения результатов выполнения запроса используется:

```python
# Получение одной строки
data = cursor.fetchone()
# Получение всех строк
data = cursor.fetchall()
```

Для получения информации о столбацах запроса используется:

```python
metadata = cursor.description
```
После запросов, меняющих данные в базе, необходимо указывать сохранение с помощью:

```python
db.commit()
```

После окончания работы, курсор и подключение нужно закрыть:

```python
cursor.close()
db.close()
```

## Примеры

```python
import pymysql


class DataBase():
    def connection(self):
        self.db = pymysql.connect(
            host='192.168.1.102',
            port=3306,
            user='student',
            password='Hn523cv-',
            db='shop',
        )

        self.cursor = self.db.cursor()
    
    def create_tables(self):

        self.connection()

        self.cursor.execute(
            'CREATE TABLE if not exists providers(' +
            'id int auto_increment primary key,' +
            'name varchar(255))'
        )

        self.cursor.execute(
            'create table if not exists products (' +
            'id int auto_increment primary key,' +
            'name varchar(255),' +
            'provider_id int,' +
            'FOREIGN KEY (provider_id) REFERENCES providers (id))'
        )
    
    def add_provider(self, provider):
        self.connection()

        self.cursor.execute(f"insert providers (name) values ('{provider}')")

        self.db.commit()
        self.db.close()
    
    def get_providers(self, id):
        self.connection()

        self.cursor.execute(f"select * from providers where id = {id}")

        data = self.cursor.fetchall()

        print(data)

        self.db.close()

    def add_product(self, name, provider):
        self.connection()

        self.cursor.execute(f"select id from providers where name = '{provider}'")
        provider_id = self.cursor.fetchone()[0]
        
        if provider_id:
            self.cursor.execute(f"select * from products where name = '{name}' and provider_id = {provider_id}")
            data = self.cursor.fetchone()
            if data is None:
                self.cursor.execute(f"insert products (name, provider_id) values ('{name}', {provider_id})")
                self.db.commit()
        
        self.db.close()


db = DataBase()
db.create_tables()
# db.add_provider('Шевченко')
# db.get_providers(2) 
db.add_product('Арбузы', 'Файзуллаев')
```

```python
import pymysql

import faker

db = pymysql.connect(
    host='95.154.68.63',
    port=3306,
    user='student',
    password='Hn523cv-',
    db='energies'
)

cursor = db.cursor()

def create_tables():
    cursor.execute(
        'create table if not exists persons(' +
        'id int auto_increment primary key,' +
        'name varchar(255))'
    )

    cursor.execute(
        'create table if not exists energy_drinks(' +
        'id int auto_increment primary key,' +
        'name varchar(255))'
    )

    cursor.execute(
        'create table if not exists person_to_energy_drink(' +
        'id int auto_increment primary key,' +
        'person_id int,' +
        'energy_drink_id int,' +
        'FOREIGN KEY (person_id) REFERENCES persons (id),' + 
        'FOREIGN KEY (energy_drink_id) REFERENCES energy_drinks (id))'
    )

def get_persons():
    cursor.execute('select * from persons')
    data = cursor.fetchall()
    print(data)

def get_energies():
    cursor.execute('select * from energy_drinks')
    data = cursor.fetchall()
    print(data)

def add_person(name):
    cursor.execute(f"insert into persons (name) values ('{name}')")
    db.commit()

def add_energy(name):
    cursor.execute(f"insert into energy_drinks (name) values ('{name}')")
    db.commit()

def delete_energy():
    cursor.execute(f'delete from energy_drinks')
    db.commit()

create_tables()

fake = faker.Faker(['ru', 'ar', 'uk'])

# for i in range(100):
#     name = fake.name()
#     add_person(name)

for i in range(100):
    name = fake.company()
    add_energy(name)

# delete_energy()
get_energies()
```
