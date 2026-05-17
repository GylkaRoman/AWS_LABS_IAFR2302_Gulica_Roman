# Лабораторная работа 6.
## IAFR2302 Gulica Roman  
**Тема:** Балансирование нагрузки в облаке и авто-масштабирование

## Цель работы
Закрепить навыки работы с AWS EC2, Elastic Load Balancer, Auto Scaling и CloudWatch, создав отказоустойчивую и автоматически масштабируемую архитектуру.

## Этапы работы
1. Создание VPC и подсетей
2. Создание и настройка виртуальной машины
3. Создание AMI
4. Создание Launch Template
5. Создание Target Group
6. Создание Application Load Balancer
7. Создание Auto Scaling Group
8. Тестирование Application Load Balancer
9. Тестирование Auto Scaling
10. Завершение работы и очистка ресурсов

## 1. Подготовка среды
1. Для выполнения лабораторных работ использовался **аккаунт** из **aws academy**.<br>
2. Как основная среда использовался аккаунт из **Lab6Scale & Load Balance your Architecture**<br>
![ACC](screenshots/ACC.png)<br>

3. Так как в sandbox ошибка при создании Auto Scaling Group на 2 этапе поэтому и использовался аккаунт из Lab 6.<br>
4. Также были выполнены все пункты из лабораторной работы в мудле но названия и некоторые моменты были использованны по требованию пунктов из aws academy Lab_6.<br>

## 2. Создание и настройка виртуальной машины
1. Запускаю виртуальную машину:<br>

2. Указываю:<br>
- AMI: `Amazon Linux 2`
- Тип: `t3.micro`
- Выбераю созданную `VPC` и `подсеть`.
- публичный `IP-адрес` `Enable auto-assign public IP`.<br>

![EC2_1](screenshots/EC2_1.png)<br>
![EC2_2](screenshots/EC2_2.png)<br>
![EC2_3](screenshots/EC2_3.png)<br>
![EC2_4](screenshots/EC2_4.png)<br>

3. В настройках безопасности создаю новую группу безопасности с правилами:<br>
Входящие правила:<br>
- SSH (порт 22) - источник: `мой IP`
- HTTP (порт 80) - источник: `0.0.0.0/0`<br>

Исходящие правила:<br>
- Все трафики - источник: `0.0.0.0/0`<br>

![EC2_5](screenshots/EC2_5.png)<br>

4. В `Advanced Details` => `Detailed CloudWatch monitoring`: `Enable`.<br>
![EC2_6](screenshots/EC2_6.png)<br>
![EC2_7](screenshots/EC2_7.png)<br>

## 3. Создание AMI
1. `EC2` => `Instance` => `Actions` => `Image and templates` => `Create image`.<br>
- name: `WebServerAmi`.<br>
![IAM_1](screenshots/IAM_1.png)<br>
![IAM_2](screenshots/IAM_2.png)<br>

**Контрольный вопрос**
Что такое image и чем он отличается от snapshot? Какие есть варианты использования AMI?<br>

AMI это шаблон виртуальной машины EC2, из которого запускаются новые инстансы.<br>
Snapshot это резервная копия одного диска (EBS volume).<br>

Варианты использования AMI<br>
- Миграция между регионами и аккаунтами
- Auto Scaling
- CI/CD и DevOps
- Резервное копирование серверов
- Быстрое создание одинаковых серверов<br>

## 4. Создание Launch Template

1. `EC2` => `Launch Templates` => `Create launch template`.<br>
2. Указываю параметры:<br>
- Launch template name: `Labconfig`
- Auto Scaling guidance: `галочка`
- Amazon Machine Image: `Web Server AMI`
- Instance type: `t2.micro`
- Key pair name: `vockey`
- Security groups: `Web Security Group`
- Detailed CloudWatch monitoring setting. `галочка`<br>
![LAUNCH_1](screenshots/LAUNCH_1.png)<br>
![LAUNCH_2](screenshots/LAUNCH_2.png)<br>
![LAUNCH_3](screenshots/LAUNCH_3.png)<br>
![LAUNCH_4](screenshots/LAUNCH_4.png)<br>
![LAUNCH_5](screenshots/LAUNCH_5.png)<br>


**Контрольный вопрос**
Что такое Launch Template и зачем он нужен? Чем он отличается от Launch Configuration?<br>
Launch Template это шаблон, который описывает, как запускать EC2 (AMI, тип инстанса, сеть, security group, user-data и т.д.).<br>
Он нужен для автоматического и одинакового запуска серверов.<br>

Отличие от Launch Configuration:<br>
- Launch Template это современный и рекомендуемый
- Launch Configuration это устаревший, без версий и с ограничениями
- поддерживает версии
- поддерживает все новые возможности EC2<br>

## 5. Создание Target Group
1. `EC2` => `Target Groups` => `Create target group`.<br>

2. Указываю параметры:<br>
Название: `LabGroup`
Тип: `Instances`
Протокол: `HTTP`
Порт: `80`
VPC: `выбераю созданную VPC`<br>

3. Нажимаю `Next` => `Next` => `Create target group`.<br>
![TARGET_1](screenshots/TARGET_1.png)<br>
![TARGET_2](screenshots/TARGET_2.png)<br>

**Контрольный вопрос**
Зачем необходим и какую роль выполняет Target Group?<br>
Target Group это группа ресурсов, на которые Load Balancer отправляет трафик.<br>

Зачем нужен Target Group<br>
- определяет куда именно передавать запросы
- выполняет health check (проверку работоспособности)
- автоматически исключает неработающие инстансы
- позволяет балансировать нагрузку<br>

Роль Target Group<br>
- Load Balancer принимает запрос
- Target Group решает, какие инстансы живы
- трафик идёт только на здоровые цели<br>

## 6. Создание Application Load Balancer
1. `EC2` => `Load Balancers` => `Create Load Balancer` => `Application Load Balancer`.

2. Указываю параметры:<br>
- Название: `LabELB`
- Scheme: `Internet-facing`.
- Subnets: `выбераю созданные 2 публичные подсети`.
- Security Groups: `Web Security Group`.
- Listener: `протокол HTTP, порт 80`.
- Default action: `Target Group project-target-group`.<br>

3. Нажимаю `Create load balancer`.<br>
![BALANCER_1](screenshots/BALANCER_1.png)<br>
![BALANCER_2](screenshots/BALANCER_2.png)<br>
![BALANCER_3](screenshots/BALANCER_3.png)<br>
![BALANCER_4](screenshots/BALANCER_4.png)<br>

**Контрольный вопрос**
В чем разница между Internet-facing и Internal?<br>
Internet-facing Load Balancer<br>
- Доступен из интернета
- Имеет публичный IP
- Используется для сайтов, API, публичных сервисов<br>

Internal Load Balancer<br>
- Доступен только внутри VPC
- Не имеет публичного IP
- Используется для внутренних сервисов (backend, базы, микросервисы)<br>

**Контрольный вопрос**
Что такое Default action и какие есть типы Default action?<br>

Что такое Default action<br>
Default action это действие, которое Load Balancer выполняет, если запрос не подошёл ни под одно правило.<br>

Типы Default action<br>
Forward - пересылает запрос в Target Group<br>
Redirect - перенаправляет пользователя на другой URL (например HTTP => HTTPS)<br>
Fixed response - возвращает готовый ответ (например ошибка 404 или сообщение)<br>

## 7. Создание Auto Scaling Group
`Actions menu` => `Create Auto Scaling group`<br>

1. (launch template or configuration):
- Auto Scaling group name: `Lab Auto Scaling Group`
- Launch template: `LabConfig`.<br>
![AUTO_1](screenshots/AUTO_1.png)<br>

2. (instance launch options):
- VPC: `Lab VPC`
- Availability Zones and subnets: `Private Subnet 1 and Private Subnet 2`.<br>
![AUTO_1](screenshots/AUTO_2.png)<br>

3. (Configure advanced options):
- `Attach to an existing load balancer`
- Existing load balancer target groups: `LabGroup`.
- Additional settings pane: `Enable group metrics collection within CloudWatch`<br>
![AUTO_1](screenshots/AUTO_3.png)<br>
![AUTO_1](screenshots/AUTO_4.png)<br>

4. (Configure group size and scaling policies - optional): 
- Desired capacity: `2`
- Minimum capacity: `2`
- Maximum capacity: `6`<br>
![AUTO_1](screenshots/AUTO_5.png)<br>

Under Scaling policies: Target tracking scaling policy and configure:
- Scaling policy name: `LabScalingPolicy`
- Metric type: `Average CPU Utilization`
- Target value: `60`<br>
![AUTO_1](screenshots/AUTO_6.png)<br>

5. (Add notifications - optional):
- `Next`<br>
![AUTO_1](screenshots/AUTO_7.png)<br>

6. (Add tags - optional):
`Add tag`:
- Key: `Name`
- Value: `Lab Instance`<br>
![AUTO_1](screenshots/AUTO_8.png)<br>

6. (Review):<br>
Нажимаю `Create Auto Scaling group`<br>
![AUTO_1](screenshots/AUTO_9.png)<br>

**Контрольный вопрос**
Почему для Auto Scaling Group выбираются приватные подсети?<br>
- Инстансам не нужен прямой доступ из интернета
- Доступ к ним идёт через Load Balancer
- Это безопаснее (нет публичных IP)
- Соответствует best practice AWS<br>

**Контрольный вопрос**
Зачем нужна настройка: Availability Zone distribution?<br>
- Равномерно распределяет инстансы по разным AZ
- Повышает отказоустойчивость<br>

**Контрольный вопрос**
Что такое Instance warm-up period и зачем он нужен?<br>
Instance warm-up period это время, которое даётся новому EC2-инстансу, чтобы полностью запуститься и стать готовым к работе.<br>

Нужно, чтобы Auto Scaling не реагировал на сырые метрики и не масштабировал группу раньше времени.<br>

## 8. Тестирование Application Load Balancer
1. `EC2` => `Load Balancers`, выбераю созданный `Load Balancer` и копирую его `DNS-имя`.<br>
2. Вставляю `DNS-имя` в браузер и вижу страницу веб-сервера.<br>
3. Обновляю страницу несколько раз и смотрю на `IP-адреса` в ответах.<br>
![SITE_1](screenshots/SITE_1.png)<br>

**Контрольный вопрос**
Какие IP-адреса вы видите и почему?<br>
При обновлении страницы в ответах меняются IP-адреса.<br>

Обращаюсь не к EC2, а к Load Balancer который распределяет запросы между разными EC2-инстансами и каждый ответ приходит от другого инстанса, у которого свой private IP.<br>

## 9. Тестирование Auto Scaling
1. Сейчас запущено толко `2 инстанса`<br>
![TEST_1](screenshots/TEST_1.png)<br>

2. Перехожу в `CloudWatch` => `Alarms`<br>
На экране появилось `2 alarms`<br>
![TEST_2](screenshots/TEST_2.png)<br>

3. Возвращаюсь на вкладку браузера с `веб-приложением`.<br>
4. Выбераю Load Test рядом с логотипом `AWS`.<br>
![TEST_6](screenshots/TEST_6.png)<br>

5. Возвращаюсь на вкладку браузера с консолью `CloudWatch`.<br>
Через 5 минут статус тревоги `AlarmLow` измениться на `OK`, а статус тревоги `AlarmHigh` на `in allarm`.<br>
![TEST_3](screenshots/TEST_3.png)<br>

На графике `AlarmHigh`, видно увеличение процента использования `ЦП`.<br>
Как только он пересечет линию 60% более чем на 3 минуты, запустится автоматическое масштабирование для добавления дополнительных экземпляров.<br>
![TEST_4](screenshots/TEST_4.png)<br>

`AlarmHigh` в состояние `in allarm`.<br>
6. Теперь могу увидеть дополнительные запущенные экземпляры.<br>
Новые экземпляры были созданы автоматическим масштабированием в ответ на тревогу `CloudWatch`.<br>
![TEST_5](screenshots/TEST_5.png)<br>

**Контрольный вопрос**
Какую роль в этом процессе сыграл Auto Scaling?<br>
- Auto Scaling обеспечил наличие нескольких EC2-инстансов для обработки запросов.
- автоматически создал нужное количество инстансов
- поддерживает их число (добавляет/удаляет при необходимости)
- обеспечивает отказоустойчивость вместе с Load Balancer<br>

## 10 Удаление ресурсов
Все ресурсы удаляться автоматически так как это аккаунт действует только 2 часа.

## Источники
https://elearning.usm.md/<br>
https://docs.aws.amazon.com/<br>
Презентация. Введение в облачные вычисления<br>
Презентация. Архитектура облачных систем и основы виртуализации<br>
Презентация. Введение в Amazon Web Services (AWS). Глобальная архитектура AWS<br>
Презентация. Введение в вычислительные сервисы. Amazon AWS Elastic Compute Cloud<br>
Презентация. Виртуальные сети в облаке. Amazon VPC Fișier<br>
Презентация. Балансировка нагрузки и автоматическое масштабирование. AWS ELB, EC2 AutoScaling<br>

## Вывод
Научился:<br>
1. Создавать AMI
2. Создавать Launch Template
3. Создавать Target Group
4. Создавать Application Load Balancer
5. Создавать Auto Scaling Group
6. Тестировать Application Load Balancer
7. Тестировать Auto Scaling