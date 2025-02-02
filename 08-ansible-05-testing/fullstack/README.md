# Сценарий проверки всего стека **Clickhouse** + **Vector** + **Lighthuse**

## Зависимости

- Сторонняя роль установки и настройки [Clickhouse](https://github.com/NamorNinayzuk/ansible-clickhouse) 
- Собственная роль установки и настройки [Vector](https://github.com/NamorNinayzuk/vector-role)
- Собственная роль установки и настройки [Lighthouse](https://github.com/NamorNinayzuk/lighthouse-role)

> Также в файле зависимостей для информации указана коллекция, содержащая собственные роли - коллекции автоматически не устанавливаются
## Описания контейнеров

Для тестирования используется драйвер **Docker**.

Роли поддерживают **CentOS 7**, **CentOS 8**, **Ubuntu**, но согласно поставленной задаче используем только **CentOS 7**

Создаются три контейнера:
- `clickhouse` с открытыми портами **9000** и **8123** для возможности ручной проверки через GUI
- `vector`
- `lighthouse` с открытым портом **8080** для возможности ручного контроля

## Предназначение файлов

- `cleanup.yml` - **playbook Molecule** для этапа **Cleanup** - предназначен для чистки сервисов от следов проверок, а именно обнуляется таблица в **Clickhouse** и удаляются контролируемые **Vector** файлы
- `converge` - **playbook Molecule** для этапа **Converge** - подготовка инфраструктуры сервисов: устанавливает скрипт эмулятор SystemD во все контейнеры и применяет все роли к соответствующим контейнерам
- `molecule` - конфигурационный файл **Molecule**
- `requirements` - файл зависимостей **Molecule** для **Ansible-galaxy**
- `verify` - основной **playbook Molecule** для этапа **verify** - верифицирует установку **Vector** и **Lighthouse**, создаёт контролируемые **Vector** файлы и наполняет их содержимым
- `verify_clickhouse` - **playbook** запрашивает данные из **Clickhouse** и сравнивает их с эталоном. Реализован отдельно с поддержкой нескольких итераций, так как из-за особенностей реализации **Vector** не все данные сразу отправляются в **Clickhouse**.

## Примечания

Для успешного функционирования всего стека ролям сервисов **Vector** и **Lighthouse** нужно передать IP адрес сервиса **Clickhouse** внутри сети контейнеров **Docker**.
По умолчанию **Molecule** создаёт контейнеры в сети в драйвером **bridge**. Определение IP адреса контейнера **Clickhouse** и передача его другим осуществляется в **play** с именем `Detect Clickhouse IP`.
Для ручного контроля используемые порты проброшены в основную систему и доступны по локальному IP адресу `127.0.0.1`
