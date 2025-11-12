# Лабораторная работа 4.1. Сравнение подходов хранения больших данных

**Вариант 14:** Биотехнологическая компания: анализ геномных последовательностей (NGS), поиск паттернов в данных клинических исследований, моделирование лекарственного взаимодействия.  
Источники: данные секвенирования, данные клинических испытаний.


**Цель работы:** Cравнить производительность и эффективность различных подходов к хранению и обработке больших данных на примере реляционной базы данных PostgreSQL и документо-ориентированной базы данных MongoDB.

**Оборудование и программное обеспечение:**
 - Компьютер с операционной системой Ubuntu (или любой другой ОС с поддержкой Docker).
 - Docker и Docker Compose.
 - Python 3.x.
 - Jupyter Notebook или JupyterLab.
 - Библиотеки Python: psycopg2-binary, pymongo, pandas, matplotlib, sqlalchemy.
 - Конфигурация: https://github.com/BosenkoTM/BigDataAnalitic/tree/main/lw/2025/lw_4_1

## Теоретическая часть
В современном мире объемы данных растут экспоненциально, что приводит к необходимости использования эффективных методов их хранения и обработки. Существует два основных подхода к хранению больших данных:
1. Реляционные базы данных (например, PostgreSQL). Основаны на реляционной модели данных, где информация хранится в строго структурированных таблицах с заранее определенной схемой. Связи между таблицами устанавливаются с помощью внешних ключей. PostgreSQL является мощной объектно-реляционной СУБД с открытым исходным кодом, поддерживающей ACID-транзакции, сложные запросы и обладающей высокой степенью расширяемости.
2. NoSQL базы данных (например, MongoDB). Представляют собой широкий класс систем управления базами данных, которые отличаются от традиционных реляционных СУБД. MongoDB — это документоориентированная СУБД, которая хранит данные в гибких, JSONподобных документах. Схема данных не является фиксированной, что позволяет легко хранить иерархические и полуструктурированные данные. NoSQL-системы часто отдают предпочтение масштабируемости и производительности в ущерб строгой согласованности данных (в сравнении с реляционными СУБД).

Каждый из этих подходов имеет свои преимущества и недостатки, которые мы исследуем в ходе выполнения лабораторной работы.

## Практическая часть
### Шаг 1. Подготовка окружения с помощью Docker
Для упрощения развертывания и управления базами данных мы будем использовать Docker Compose. Этот подход позволяет запустить PostgreSQL, pgAdmin, MongoDB и Mongo Express в изолированных контейнерах одной командой.
**1.1. Создайте файл docker-compose.yml следующего содержания:** 
```python
version: '3.8'
services:
 mongodb:
 image: mongo:latest
 container_name: mongodb
 environment:
 MONGO_INITDB_ROOT_USERNAME: mongouser
 MONGO_INITDB_ROOT_PASSWORD: mongopass
 ports:
 - "27017:27017"
 volumes:
 - mongo_data:/data/db
 networks:
 - db_network
 mongo-express:
 image: mongo-express:latest
 container_name: mongo-express
 restart: always
 environment:
 ME_CONFIG_MONGODB_ADMINUSERNAME: mongouser
 ME_CONFIG_MONGODB_ADMINPASSWORD: mongopass
 ME_CONFIG_MONGODB_SERVER: mongodb
 ports:
 - "8081:8081"
 depends_on:
 - mongodb
 networks:
 - db_network
 postgresql:
 image: postgres:latest
 container_name: postgresql
 environment:
 POSTGRES_USER: pguser
 POSTGRES_PASSWORD: pgpass
 POSTGRES_DB: studpg # Сразу создаем базу данных studpg
 ports:
 - "5432:5432"
 volumes:
 - pg_data:/var/lib/postgresql/data
 networks:
 - db_network
 pgadmin:
 image: dpage/pgadmin4:latest
 container_name: pgadmin
 environment:
 PGADMIN_DEFAULT_EMAIL: admin@example.com
 PGADMIN_DEFAULT_PASSWORD: admin
 ports:
 - "5050:80"
 depends_on:
 - postgresql
 networks:
 - db_network
networks:
 db_network:
 driver: bridge
volumes:
 mongo_da
 ```
**1.2. В терминале, находясь в директории с файлом docker-compose.yml,выполните команду:**
```terminal
sudo docker compose up -d
```
![](files/Снимок%20экрана%202025-11-11%20221434.png)
**1.3. Установите необходимые библиотеки Python:**
```terminal
pip install psycopg2-binary pymongo pandas matplotlib sqlalchemy
```
### Шаг 2: Настройка баз данных
После запуска контейнеров у вас будет доступ к следующим сервисам:
- PostgreSQL: localhost:5432
- pgAdmin (веб-интерфейс для PostgreSQL): http://localhost:5050
- MongoDB: localhost:27017
- Mongo Express (веб-интерфейс для MongoDB): http://localhost:8081

**Для всех вариантов:**

1. PostgreSQL:
- База данных studpg уже создана согласно docker-compose.yml.
- Подключитесь к pgAdmin (Email: admin@example.com,Пароль: admin).
**Добавьте новый сервер:**
- Host name/address: postgresql
- Port: 5432
- Maintenance database: studpg
- Username: pguser
- Password: pgpass
**Откройте Query Tool для базы studpg и выполните SQL-запрос для создания пользователя student:**
```SQL
CREATE USER student WITH PASSWORD 'Stud2024!!!';
GRANT ALL PRIVILEGES ON DATABASE studpg TO student;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON
TABLES TO student;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON
SEQUENCES TO student;
```
![](files/Снимок%20экрана%202025-11-11%20222207.png)
В дальнейшем для подключения из Python используйте пользователя student.

**2. MongoDB:**
Создайте базу данных studmongo. Это можно сделать автоматически при первом подключении и вставке данных из вашего Python-скрипта.
![](files/Снимок%20экрана%202025-11-11%20223935.png)
### Шаг 3: Выполнение заданий
Выберите вариант из таблицы ниже и выполните три задания: для
PostgreSQL, для MongoDB и сравнительный анализ в Jupyter Notebook.
**Общий подход к выполнению:**

**1. Сгенерируйте данные. Напишите скрипт на Python для генерации данных в соответствии с вашим вариантом. Объем данных должен составлять не менее 10 000 записей, если иное не указано в задании.**

**2. Работа с PostgreSQL:**
- Определите реляционную схему (CREATE TABLE).
- Напишите скрипт для вставки сгенерированных данных
- Реализуйте запрос для вашего задания.
- Измерьте время выполнения запроса.

**3. Работа с MongoDB:**
- Определите структуру документа.
- Напишите скрипт для вставки сгенерированных данных.
- Реализуйте запрос (чаще всего, с помощью aggregation framework).
- Измерьте время выполнения запроса.

**4. Анализ в Jupyter Notebook:**
- Объедините все шаги в одном .ipynb файле.
- Представьте результаты измерений в виде таблицы Pandas.
- Постройте график (например, столбчатую диаграмму) для визуального сравнения производительности.
- Напишите выводы по результатам сравнения для вашего конкретного случая.

# Варианты заданий
Сформируйте данные объемом не менее 10 000 записей для обычных данных и 100 000 для "больших данных"

## Вариант 14

### Задание для Postgres
**Цель задания:** Удаление данных. Создатьтаблицу archive (100 000 записей). Измерить время выполнения операции DELETE для удаления 20% записей по условию.

**Скриншот выполнения**:
![](social_network_architecture.png)

### Задание для MongoDB

**Цель задания:** Удаление данных. Создать коллекцию archive (100 000 записей). Измерить время выполнения delete_many для удаления 20% записей по условию.

**Скриншот выполнения:**
![](social_network_architecture.png)
### Анализ в Jupyter Notebook

**Цель задания:** Сравнить производительность массового удаления данных. Проанализировать, как СУБД справляются с освобождением места и поддержкой производительности после удаления.

**Скриншот выполнения:**
![](social_network_architecture.png)
