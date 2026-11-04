Веб-приложение на Flask + компьютерное зрение на YOLOv8: загружаете фото торгового зала / витрины — система находит телевизоры, рисует bounding box'ы, считает количество и сохраняет историю. По истории строятся отчёты PDF и Excel.
Возможности
Загрузка фото (PNG, JPG, JPEG, WEBP, BMP, до 16 МБ)
Детекция телевизоров моделью YOLOv8n (класс COCO tv, id 62)
Подпись каждого бокса: TV + уверенность (зелёная рамка)
Сообщение на изображении, если телевизоры не найдены
История всех обработок (SQLite history.db)
Страница истории /history + JSON API /history/data
Статистика /stats: всего запросов, всего TV, средняя уверенность
Отчёты: PDF (/report/pdf) и Excel (/report/excel)

Структура проекта:
tv-detection/
├── app.py              # Flask-приложение, YOLOv8, маршруты, отчёты
├── requirements.txt    # зависимости Python
├── yolov8n.pt          # веса предобученной модели YOLOv8n
├── history.db          # SQLite-база истории (создаётся автоматически)
├── templates/
│   ├── index.html      # главная страница (загрузка фото)
│   └── history.html    # страница истории
├── static/             # обработанные изображения *_result.*
├── uploads/            # оригиналы загруженных фото
├── test_tv.jpg, test_tv2.jpg, test_real.jpg, ...  # тестовые изображения
└── README.md           # этот файл

Требования
Python 3.10+
pip
~500 МБ свободного места (torch CPU + ultralytics + веса модели)

Установка
cd /home/user/tv-detection
pip install -r requirements.txt
Веса yolov8n.pt уже лежат в папке проекта. Если файла нет — при первом запуске ultralytics скачает его автоматически.

Запуск:
Вариант 1 (основной для практики)
cd /home/user/tv-detection
python3 app.py
Сервер поднимется на http://127.0.0.1:5000 (внутри sandbox слушает 0.0.0.0:5000).
Вариант 2 (через Flask CLI, порт 6539)
cd /home/user/tv-detection
python3 -m flask --app app run --host 0.0.0.0 --port 6539
Открыть в браузере: http://127.0.0.1:6539 (или адрес sandbox-стола с этим портом).
Как пользоваться
Откройте главную страницу /.
Загрузите фото зала с телевизорами (кнопка загрузки).
Дождитесь результата: количество TV, средняя уверенность, размеры кадра, картинка с боксами.
Откройте /history — таблица всех проверок.

Скачайте отчёт:
PDF: кнопка / маршрут /report/pdf
Excel: кнопка / маршрут /report/excel

Для проверки можно использовать тестовые файлы из папки проекта:
test_tv.jpg, test_tv2.jpg, test_tv_custom.jpg, test_real.jpg, test_bus.jpg, test_electronics.jpg.

Пример запроса:
curl -F "image=@test_tv.jpg" http://127.0.0.1:5000/process

Пример ответа:
{
  "request_id": "a1b2c3d4",
  "tv_count": 3,
  "confidence_avg": 0.871,
  "image_width": 1280,
  "image_height": 720,
  "result_image": "/static/a1b2c3d4_result.jpg",
  "boxes": [{"x1": 100, "y1": 50, "x2": 400, "y2": 300, "confidence": 0.91}]
}

Как это работает:
Фото сохраняется в uploads/<id>_original.<ext>.
cv2.imread → model(img) (YOLOv8n).
Фильтр только класса 62 (tv), остальные классы COCO игнорируются.
Отрисовка зелёных рамок + подписей через OpenCV.
Результат — static/<id>_result.<ext>.
Запись в SQLite history.db, таблица requests: request_id, filename, timestamp, tv_count, confidence_avg, image_width, image_height, result_image.

Частые проблемы:
Файл не загружен / неверный формат — проверьте расширение: разрешены png, jpg, jpeg, webp, bmp.
Файл больше 16 МБ — сожмите изображение, лимит MAX_CONTENT_LENGTH.
Нет данных для отчёта — сначала обработайте хотя бы одно фото.
Модель долго грузится при старте — нормально: yolov8n.pt + torch CPU грузятся 10–30 сек.
Порт занят — смените порт: python3 app.py правится в конце app.py (port=5000), для Flask CLI — флагом --port.
### Автор работы
Побережный Димитрий Сергеевич УБВТ2404
Все материалы находятся в архиве .zip для увеличения скорости и простоты скачивания.
