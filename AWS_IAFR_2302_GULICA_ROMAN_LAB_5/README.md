# Лабораторная работа 5.
## IAFR2302 Gulica Roman  
**Тема:** Облачные базы данных. Amazon RDS, DynamoDB

## Цель работы
Целью работы является ознакомиться с сервисами Amazon RDS (Relational Database Service) и Amazon DynamoDB, а также научиться:
- Создавать и настраивать экземпляры реляционных баз данных в облаке AWS с использованием Amazon RDS.
- Понимать концепцию Read Replicas и применять их для повышения производительности и отказоустойчивости баз данных.
- Подключаться к базе данных Amazon RDS с виртуальной машины EC2 и выполнять базовые операции с данными (создание, чтение, обновление, удаление записей - CRUD).

## Этапы работы
1. Подготовка среды (VPC/подсети/SG)
2. Развертывание Amazon RDS
3. Создание виртуальной машины для подключения к базе данных
4. Подключение к базе данных и выполнение базовых операций
5. Создание Read Replica
6. Подключение приложения к базе данных
7. 6.a Развертывание CRUD приложения


## Шаг 1. Подготовка среды
1. Для выполнения лабораторных работ использовался **аккаунт** из **aws academy**.<br>
2. Как основная среда использовался аккаунт из **Sandbox Environment**<br>
![Sandbox Environment](screenshots/task0.png)<br>

## Подготовка среды (VPC/подсети/SG)
1. Создаю `VPC` `project-vpc`<br>
с `двумя публичными` и `двумя приватными подсетями` в разных зонах доступности `AZ`.<br>
![VPC_1](screenshots/VPC_1.png)<br>
![VPC_2](screenshots/VPC_2.png)<br>

2. Создаю группу безопасности `web-security-group`:<br>
- Входящий: HTTP (порт 80) от `любого источника`<br>
- Входящий: SSH (порт 22) от `моего IP-адреса`<br>
![VPC_3](screenshots/VPC_3.png)<br>

3. Создаю группу безопасности `db-mysql-security-group` для моей базы данных:<br>
- Входящий: `MySQL/Aurora` `порт 3306` от `web-security-group`<br>
![VPC_4](screenshots/VPC_4.png)<br>

4. Изменяю `web-security-group`, добавив правило для `исходящего трафика`<br>
- Исходящий: `MySQL/Aurora` `порт 3306` к `db-mysql-security-group`<br>
![VPC_5](screenshots/VPC_5.png)<br>

## Шаг 2. Развертывание Amazon RDS
**Контрольный вопрос**
Что такое Subnet Group? И зачем необходимо создавать Subnet Group для базы данных?<br>
Subnet Group это набор подсетей внутри VPC, в которых разрешено размещать базу данных (например, Amazon RDS).<br>

Зачем нужна Subnet Group для базы данных?<br>
Без Subnet Group AWS не знает, в каких именно подсетях можно размещать RDS.<br>

1. Перехожу в консоль Amazon Aurora and RDS.<br>
2. Создаю Subnet Group для моей базы данных.<br>

3. Указываю:<br>
- Название: `project-rds-subnet-group`
- Выбераю созданный ранее `VPC`
- Добавляю 2 `приватные` `подсети` из 2 разных AZ.<br>
![RDS_02](screenshots/RDS_02.png)<br>
![RDS_01](screenshots/RDS_01.png)<br>

4. Создаю экземпляр базы данных `Amazon RDS Databases` => `Create database`.<br>
- Choose a database creation method: `Standard Create`.<br>

5. Указываю параметры базы данных:<br>
- Engine type: `MySQL`
- Version: `MySQL 8.4.7`
- Templater: `Free tier`
- Availability and durability: `Single-AZ DB instance deployment`
- DB instance identifier: `project-rds-mysql-prod`
- Master username: `admin`
- Master password: `password`
- DB instance class: `Burstable classes, db.t3.micro`<br>
![RDS_1](screenshots/RDS_1.png)<br>
![RDS_2](screenshots/RDS_2.png)<br>
![RDS_3](screenshots/RDS_3.png)<br>
![RDS_4](screenshots/RDS_4.png)<br>
![RDS_6](screenshots/RDS_6.png)<br>

Storage:<br>
- Storage type: `General Purpose SSD (gp3)`
- Allocated storage: `20 GB`
- Enable storage autoscaling: `Checked`
- Maximum storage threshold: `100 GB` <br>
![RDS_5](screenshots/RDS_5.png)<br>
![RDS_7](screenshots/RDS_7.png)<br>

Connectivity:<br>
- `Don’t connect to an EC2 compute resource`
- VPC: `Выбераю созданный ранее VPC`
- DB subnet group: `project-rds-subnet-group`
- Public access: `No`
- Existing VPC security groups: `db-mysql-security-group`
- Availability zone: `No preference` <br>
![RDS_8](screenshots/RDS_8.png)<br>

Additional configuration<br>
- Initial database name: `project_db`
- Backup Enable automated backup: `галочка`
- Backup Enable encryption: `снимаю галочку`
- Maintanance (Enable auto minor version upgrade): `снимаю галочку`<br>

6. Нажимаю `Create database` <br>
![RDS_9](screenshots/RDS_9.png)<br>
![RDS_10](screenshots/RDS_10.png)<br>

7. Жду завершения создания базы данных статус: `Available`.<br>
8. Копирую `Endpoint` базы данных.<br>
![RDS_11](screenshots/RDS_11.png)<br>

## 3. Создание виртуальной машины для подключения к базе данных
1. Создаю виртуальную машину EC2 в публичной подсети <br>
Использую группу безопасности web-security-group<br>
При инициализации виртуальной машины устанавливаю MySQL клиент<br>
```
#!/bin/bash
dnf update -y
dnf install -y mariadb105 # Установка MariaDB/MySQL клиента (подходит и для MySQL и для MariaDB)
```
<br>

![EC2_1](screenshots/EC2_1.png)<br>
![EC2_2](screenshots/EC2_2.png)<br>
![EC2_3](screenshots/EC2_3.png)<br>
![EC2_4](screenshots/EC2_4.png)<br>
![EC2_5](screenshots/EC2_5.png)<br>
![EC2_6](screenshots/EC2_6.png)<br>

## 4. Подключение к базе данных и выполнение базовых операций
1. Подключаюсь к моей виртуальной машине EC2 по SSH.<br>
2. Подключаюсь к базе данных RDS с помощью MySQL клиента:<br>
3. Ввожу пароль администратора базы данных<br>
![BD_1](screenshots/BD_1.png)<br>

4. Создаю две таблицы c связами.<br>
5. Вставляю несколько записей в каждую таблицу<br>
![BD_2](screenshots/BD_2.png)<br>
![BD_3](screenshots/BD_3.png)<br>

6. Выполняю несколько запросов на выборку данных, включая JOIN между таблицами.<br>
![BD_4](screenshots/BD_4.png)<br>
![BD_5](screenshots/BD_5.png)<br>

## 5. Создание Read Replica
1. Выбераю базу данных RDS в консоли AWS.<br>
2. Нажимаю на `Actions` => `Create read replica`.<br>

3. Указываю параметры для `Read Replica`:<br>
- DB instance identifier: `project-rds-mysql-read-replica`
- Instance class: `db.t3.micro`
- Storage type: `General Purpose SSD (gp3)`<br>

`Monitoring`<br>
- Enable Enhanced monitoring: `снимаю галочку`
- Public access: `No`
- VPC security groups: `db-mysql-security-group`<br>

4. Жду, пока реплика перейдёт в `Available`.<br>

5. Подключаюсь к `Read Replica` с моей виртуальной машины `EC2`<br>
6. Выполняю запросы на чтение данных `SELECT` из таблиц, созданных на основном экземпляре базы данных.<br>
![REPLICA_1](screenshots/REPLICA_1.png)<br>

**Контрольный вопрос**
Какие данные вы видите? Объясните почему. <br>

Вижу:<br>
- Таблицу `categories` и таблицу `todos` со всеми записями, созданными на основном экземпляре<br>

Почему так?<br>
- Потому что `Read Replica` получает данные с основного `RDS` через механизм `репликации`:<br>
- Все изменения выполняются только на основном экземпляре<br>
- `Read Replica` асинхронно копирует эти данные<br>

1. Пробую выполнить запрос на запись `INSERT/UPDATE` на реплике.<br>
![REPLICA_6](screenshots/REPLICA_6.png)<br>

**Контрольный вопрос**
Получилось ли выполнить запись на `Read Replica`? Почему?<br>
`ERROR 1290 (HY000): The MySQL server is running with the --read-only option`<br>

Запись не выполнилась потому что:<br>
- `Read Replica` работает в режиме `read-only`<br>
- `MySQL` на уровне сервера запрещает любые операции записи<br>

7. Перехожу на основной экземпляр базы данных и добавляю новую запись в одну из таблиц.<br>
![REPLICA_2](screenshots/REPLICA_2.png)<br>

8. Возвращаюсь к подключению к `Read Replica` и выполняю запрос на чтение.<br>
![REPLICA_4](screenshots/REPLICA_4.png)<br>

**Контрольный вопрос**
Отобразилась ли новая запись на реплике? Объясните почему.<br>
Новая запись отобразилась<br>

Почему она появилась?<br>
- Read Replica автоматически получила изменения<br>
- Данные синхронизировались<br>

**Контрольный вопрос**
Объясните, зачем нужны Read Replicas и в каких сценариях их использование будет полезным.<br>

`Read Replica` это копия основной базы данных, которая автоматически синхронизируется с ней и предназначена только для чтения.<br>

Зачем нужны `Read Replicas`<br>
- Географическое распределение
- Аналитика и отчёты
- Повышение отказоустойчивости
- Повышение производительности приложения
- Масштабирование чтения<br>

В каких сценариях `Read Replicas` особенно полезны<br>
- Высоконагруженные системы
- Веб-приложения с большим количеством SELECT
- Интернет-магазины
- Системы блогов, новостные сайты<br>

## Шаг 6. Подключение приложения к базе данных
### Шаг 6a. Развертывание CRUD приложения
Разрабатываю и разворачиваю простое веб-приложение на моей виртуальной машине Amazon EC2,<br>
Которое подключаю к базе данных Amazon RDS <br>
И выполняю базовые операции с данными (создание, чтение, обновление, удаление записей - CRUD).<br>


CRUD с Master + Read Replica в AWS RDS.<br>
1. Конфигурации баз данных<br>
![PY_2](screenshots/PY_2.png)<br>

MASTER_DB:<br>
- основной экземпляр RDS
- разрешает запись (INSERT / UPDATE / DELETE)<br>

REPLICA_DB<br>
- Read Replica
- используется только для чтения (SELECT)
- данные автоматически приходят с Master<br>

2. Функция подключения к БД<br>
3. Чтение данных через Read Replica<br>
![PY_3](screenshots/PY_3.png)<br>

get_connection(db_config)<br>
- принимает конфигурацию (Master или Replica)
- возвращает соединение с MySQL
- используется во всех CRUD-операциях<br>

read_todos()<br>
- подключение идёт к Read Replica
- выполняется только SELECT
- SELECT * FROM todos<br>

4. Создание задачи INSERT => Master
![PY_4](screenshots/PY_4.png)<br>

5. Обновление задачи UPDATE => Master<br>
6. Удаление задачи DELETE => Master<br>
![PY_5](screenshots/PY_5.png)<br>

update_todo(id, title, status):<br>
- изменение данных
- только основной экземпляр<br>

7. Запуск программы
![PY_6](screenshots/PY_6.png)<br>

Последовательность:<br>
- Читаем список задач с реплики
- Создаём новую задачу в Master
- Обновляем задачу в Master
- Удаляем задачу в Master
- Снова читаем с Read Replica<br>

В EC2<br>
![PY_1](screenshots/PY_1.png)<br>
Python и pip установил<br>
PyMySQL установился<br>
Запускаю файл но не успеваю увидеть так истекло 3 часа использования аккаунта.<br>

Поэтому примерный вывод:<br>
```
Список задач с реплики:
{'id': 1, 'title': 'Сделать лабораторную', 'status': 'done', 'category_id': 1}
{'id': 2, 'title': 'Подготовиться к экзамену', 'status': 'in progress', 'category_id': 1}
{'id': 3, 'title': 'Закоммитить проект', 'status': 'todo', 'category_id': 2}
{'id': 4, 'title': 'Отправить резюме', 'status': 'todo', 'category_id': 2}
{'id': 5, 'title': 'Купить продукты', 'status': 'done', 'category_id': 3}

Создаём новую задачу в мастере:
Todo создано!

Обновляем задачу с id=1:
Todo обновлено!

Удаляем задачу с id=2:
Todo удалено!

Список задач после изменений (реплика может обновиться с задержкой):
{'id': 1, 'title': 'Обновлённая задача', 'status': 'done', 'category_id': 1}
{'id': 3, 'title': 'Закоммитить проект', 'status': 'todo', 'category_id': 2}
{'id': 4, 'title': 'Отправить резюме', 'status': 'todo', 'category_id': 2}
{'id': 5, 'title': 'Купить продукты', 'status': 'done', 'category_id': 3}
{'id': 6, 'title': 'Новая задача', 'status': 'todo', 'category_id': 1}
```

<br>

## Источники
https://elearning.usm.md/<br>
https://docs.aws.amazon.com/<br>
Презентация. Введение в облачные вычисления<br>
Презентация. Архитектура облачных систем и основы виртуализации<br>
Презентация. Введение в Amazon Web Services (AWS). Глобальная архитектура AWS<br>
Презентация. Введение в вычислительные сервисы. Amazon AWS Elastic Compute Cloud<br>
Презентация. Виртуальные сети в облаке. Amazon VPC Fișier<br>
Презентация. Базы данных в облаке. Amazon RDS, DynamoDB<br>

## Вывод
Научился:<br>
1. Развертывать Amazon RDS
2. Подключаться к базе данных через виртуальные сервера
3. Выполнение базовых операций в базе данных через виртуальные сервера
4. Создвать Read Replica
5. Подключать приложения к базе данных<br>