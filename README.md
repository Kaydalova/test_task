<div align="center">
 <img src="test_task_app/static/img/male_sourcer.png" alt="Logo"> 
<h3 >Мониторинг доступности ресурсов</h3>
Приложение помогает каталогизировать и структурировать информацию по различным веб-ресурсам, а также производить мониторинг их доступности.
</div>

### Используемые технологии
- :snake: Python 3.8.10
- :incoming_envelope: Flask 2.3.2
- :rose: Jinja2 3.1.2
- :package: PostgreSQL 14.7
- :memo: Alembic 1.11.1

### Описание проекта
Проет имеет REST API и веб интерфейсы, снабжен логированием.

#### REST API:
- Эндпоинт '/api/add_source' принимает ссылку, раскладывает ее на протокол, домен, доменную зону и путь.
Если в ссылке присутствуют параметры - преобразует их в словарь.
Возвращает ответ в формате json с разложенными данными и статусом обработки.

Пример запроса:
```
{
    "url": "https://belleyou.ru/catalog/nizhnee_bele/topy/"
}
```

Пример ответа:
```
{
    "status": "Новый ресурс создан",
    "url": {
        "domain": "belleyou",
        "domain_zone": "ru",
        "id": "355a4eb2-1486-412f-ab5e-847764c0f91b",
        "params": null,
        "path": "catalog/nizhnee_bele/topy/",
        "protocol": "https"
    }
}
```

- Эндпоинт '/api/add_sources_zip' принимает zip архив (key - 'file')
с csv файлом с перечнем ссылок и обрабатывает каждую ссылку по образцу предыдущего запроса.
Возвращает ответ в формате json с общим статусом обработки файла.

Пример ответа:
```
{
    "all_urls": 22,
    "error_urls": 1,
    "urls_for_saving": 21
}
```

- Эндпоинт '/api/add_screenshot' принимает uuid ресурса (key - 'source_id') и картинку(key - 'screenshot').
Скриншот сохраняется в БД в таблице ресурсов. 

Пример ответа:
```
{
    "success": "Скриншот сохранен."
}
```

- Эндпоинт '/api/all_sources' принимает GET запрос и выводит все сохраненные ссылки из БД с последним статус-кодом ответа ресурса для каждой ссылки.
Можно сделать выборку по доменной зоне, id, uuid, доступности, добавить пагинацию к json-ответу с целью снижения нагрузки на сервер. 

Пример запроса:
```
http://127.0.0.1:5000/api/all_sources?page=1&per_page=3&is_awailable=True&domain_zone=com
```

Пример ответа:
```
{
    "links": [
        {
            "domain_zone": "com",
            "full_link": "https://habr.com/ru/articles/201386/",
            "id": "6921a8eb-0893-4953-9beb-4ae5b4881693",
            "is_awailable": true,
            "status_code": "200"
        },
        {
            "domain_zone": "com",
            "full_link": "https://www.bogotobogo.com/python/python_function_with_generator_send_method_yield_keyword_iterator_next.php",
            "id": "01f2e73d-bcbf-42f5-b8d0-6c31c2913a5d",
            "is_awailable": true,
            "status_code": "200"
        },
        {
            "domain_zone": "com",
            "full_link": "https://timeweb.com/ru/community/articles/chto-takoe-cron",
            "id": "42dc854f-6429-4232-a6c1-a31648f8a64a",
            "is_awailable": true,
            "status_code": "200"
        }
    ],
    "page": "1",
    "per_page": "3",
    "total": 12
}
```

- Эндпоинт '/api/logs' принимает GET запрос и возвращает последние 50 строчек лог-файла.

Пример ответа:
```
[2023-07-17 11:29:22,726] INFO in api_views: Новый запрос к /api/add_source_zip.
[2023-07-17 11:29:22,743] INFO in api_views: Запущен процесс записи объектов из файла.
[2023-07-17 11:33:56,713] INFO in api_views: Новый запрос к /api/add_screenshot
[2023-07-17 11:33:56,985] INFO in api_views: Скриншот сохранен.
```

#### Веб-интерфейс:
- Страница 1 (http://127.0.0.1:5000/add_source) содержит формы для добавления в приложение новых веб-ресурсов.
Формы добавляют веб-ресурсы как поштучно, так и загрузкой файла. 

- Страница 2 (http://127.0.0.1:5000/) содержит таблицу, отображающую все ссылки из базы данных с разбивкой на страницы.
Также страница содержит элементы управления - поиск по доменному имени, возможность фильтрования по доменной зоне и статусу доступности,
 и возможность удаления конкретного элемента из таблицы и базы данных соответственно. 

- Страница 3 (http://127.0.0.1:5000/logs) отображает строки из лог-файла.
По клику на кнопку "скачать журнал" скачивается файл с отображаемыми строчками лога в формате pdf. 

- Страница 4 (http://127.0.0.1:5000/news) Лента новостей. На эту страницу выводится информация об изменениях:
изменился код ответа сайта, ресурс был добавлен в базу, ресурс был удален из базы и т.д.

- Страница ресурса. Выводятся все данные по ресурсу из БД, картинка (если есть), лента с новостями по ресурсу.
Попасть на страницу каждого ресурса можно через http://127.0.0.1:5000/
- кликнув на полную ссылку интересующего ресурса.


### Как запустить проект:
1. Клонировать репозиторий и перейти в него в командной строке:

```
git clone git@github.com:Kaydalova/test_task.git
```

```
cd test_task_app
```

2. Запустить контейнеры с проектом и базой:
```
sudo docker-compose build
sudo docker-compose up
```

3. После этого нужно открыть соседний терминал, зайти в контейнер с проектом и выполнить миграции:
```
sudo docker exec -it test_task_app_1 bash
export FLASK_APP=test_task_app
flask db init
flask db migrate
flask db upgrade
```
4. После выполнения миграций сайт будет доступен по адресу http://127.0.0.1:5000

5. Чтобы запустить мониторинг нужно перейти по адресу http://127.0.0.1:5000/start_monitoring



TODO:
Страница ресурса добавить полное описание heavy_check_mark
Очистка фильтра на странице все ресурсы heavy_check_mark
Страница логов: фильтр по дате, пагинация heavy_check_mark
Журнал: фильтр по дате, пагинация
Нормальный запуск в докере heavy_check_mark
Запуск мониторинга перед запуском Flask приложение
Перенести все конфигурационные данные в yaml heavy_check_mark
Авторизация и регистрация с токенами
Безопасная обработка архива
Логгирование с буферизацией

