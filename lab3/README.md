# Лабораторная работа 3

## Ход работы

### Часть 1. Поднимаем Postgres

1. Создаем директорию для работы и все нужные файлы, после чего билдим контейнеры <br><br>
![Screenshot](images/Screenshot_0.png)

2. Проверяем, что контейнеры были запущены <br><br>
![Screenshot](images/Screenshot_1.png)

3. Проверяем, что `pg-master` стал лидером, а `pg-slave` — репликой <br><br>
![Screenshot](images/Screenshot_3.png)

### Часть 2. Проверяем репликацию

1. Подключаемся к базам данных через pgAdmin по localhost и их портам <br><br>
![Screenshot](images/Screenshot_2.png)

2. Cоздаем таблицу `my_first_replication` с тестовыми столбцами и данными <br><br>
![Screenshot](images/Screenshot_4.png)

3. Видим, что в `pg-slave` создалась таблица `my_first_replication`, которая содержит такие же данные <br><br>
![Screenshot](images/Screenshot_5.png)

4. Также пробуем что-то изменить в таблице у `pg-slave` и закономерно получаем ошибку, так как данные закрыты для редактирования <br><br>
![Screenshot](images/Screenshot_6.png)

### Часть 3. Делаем высокую доступность

1. Создаем все необходимые файлы по заданию, после чего перезапускаем проект <br><br>
![Screenshot](images/Screenshot_12.png)

2. Успешно подключаемся к `haproxy` <br><br>
![Screenshot](images/Screenshot_7.png)

### Часть 4. Задание

1. После остановки `pg-master` с помощью `docker stop pg-master` пробуем изменить данные в `haproxy` (меняем `age`) <br><br>
![Screenshot](images/Screenshot_8.png)

2. В `pg-slave` данные также изменились даже с отключенным `pg-master` <br><br>
![Screenshot](images/Screenshot_9.png)

3. Проанализировав логи, увидим, что после отключения `pg-master` с помощью `patroni` перераспредилились роли, и `pg-slave` стал лидером <br><br>
![Screenshot](images/Screenshot_10.png)

4. Также у `pg-slave` пропало ограничение на запись, что логично, потому что он теперь — лидер (меняем `age`) <br><br>
![Screenshot](images/Screenshot_11.png)

## Ответы на вопросы

##### Порты 8008 и 5432 вынесены в разные директивы, expose и ports. По сути, если записать 8008 в ports, то он тоже станет exposed. В чем разница?
- Директива `expose` делает порты доступными только внутри сети `Docker Compose` для взаимодействия между контейнерами, а директива `ports` открывает порты на хост-машине, обеспечивая доступ извне

##### При обычном перезапуске композ-проекта, будет ли сбилден заново образ? А если предварительно отредактировать файлы postgresX.yml? А если содержимое самого Dockerfile? Почему?
- В `Docker Compose` образы не пересобираются автоматически, и для применения изменений из `Dockerfile` требуется использовать флаг `--build`, тогда как изменения в конфигурации применяются сразу