# Audio event detection

## Исходная задача

Вам необходимо использовать доступные инструменты и библиотеки для обработки аудио и распознавания звуковых шаблонов.

1. Доступ к аудиодорожкам, расположенным на Яндекс Диске https://disk.yandex.ru/d/N8Xto6pu8nf62A

2. Используя инструменты и библиотеки, проведите анализ аудиодорожек и выявите звуковые события.

3. Для обработки аудио и распознавания звуковых шаблонов, вы можете использовать любой доступный продукт или библиотеку, включая ресурсы Google.
Можете использовать эти репозитории https://github.com/fschmid56/efficientat, (https://github.com/fschmid56/efficientat) https://github.com/microsoft/unilm/tree/master/beats или любые на свой выбор

4. Модель  нужно докеризовать/написать k8s чарт

5. Создайте простой отчет, включающий:
• Описание методологии, инструментов, репозиториев,  которые вы использовали для анализа и распознавания звуковых событий.
• Список обнаруженных звуковых событий и соответствующие временные метки (время начала и окончания событий).
• Любые обнаруженные аномалии или трудности в распознавании.

6. Оформите ваш код и инструкции таким образом, чтобы другой специалист мог воспроизвести вашу работу на другом наборе данных.

7. Как сервис, который вы написали, можно разбить на отдельные микросервисы, чтобы была возможность отдельно улучшать каждую из составляющих?

8. Покажите примеры на гитхабе, где вы разрабатывали собственные библиотеки. Если все разработки под нда, то распишите как вы организуете модули.
<br>

## Общий план

1. Изучить исходные данные, определить их свойства и характерные особенности. Выбрать пути решения задачи детекции аудиособытий. Оценка срока - 1 день.

2. Проверка выбранных путей решения на нескольких файлах. Кодирование прототипов и финального решения. Оценка срока - 2 дня.

3. Тестирование, подбор параметров, определение возможных путей улучшения работы алгоритма. Составление документации. Оценка срока - 1 день.

P.S. Работа выполнена по плану.

## Методология и инструменты

- Для предобработки аудио использовались библиотеки Librosa, Pydub, Scipy.signal и Pywt для конвертации файлов (MP3 в WAV и далее в тензоры), фильтрации шума и нормализации аудиодорожек.

- Обработка звуковых событий путем определения временных границ каждого звукового события на основе порога вероятности, предоставленной моделью <a href="https://github.com/tensorflow/models/tree/master/research/audioset/yamnet">YAMNet</a>.
  
- Фильтрация шума: использовались преэмфазный шумоподавитель, адаптивная фильтрация и вейвлет-шумоподавление для улучшения качества аудиосигнала перед классификацией.

- Для детекции звуковых событий использовалась предобученная модель <a href="https://github.com/tensorflow/models/tree/master/research/audioset/yamnet">YAMNet</a>, которая предназначена для классификации звуковых событий бытового характера в аудиодорожках.

- Визуализация: использовалась библиотека Matplotlib для создания мел-спектрограмм.

## Обнаруженные особенности во входных данных
- Низкий уровень сигнала.

- Значительная зашумленность. При этом шум довольно простого характера типа белого/розового.

- Редкие события на фоне продолжительных пауз и/или фоновых событий (например, работы телевизора).

- Наложение нескольких событий одно на другое. При этом наложенные события могут как значительно отличаться, так и быть близкими по уровню сигнала.

## Организация кода и воспроизводимость
Код организован таким образом, чтобы обеспечить легкую воспроизводимость работы на другом наборе данных:

- Параметризация: Все ключевые параметры (пороги, пути к файлам, настройки фильтрации) легко изменить для работы с другими данными.

- Модульность: Код разделен на функции для предобработки, обработки и визуализации, что облегчает его адаптацию и масштабирование.

- Документирование: Каждая функция снабжена комментариями, объясняющими её назначение и работу.

- Сохранение результатов: Результаты анализа сохраняются в CSV-файлы для каждого аудиофайла, что упрощает последующий анализ.

## Разбиение на микросервисы
Сервис можно разделить на следующие микросервисы:

- Микросервис загрузки данных: Отвечает за загрузку и хранение аудиоданных.

- Микросервис предобработки: Отвечает за конвертацию аудиофайлов, фильтрацию шума и нормализацию.

Можно развивать в сторону подбора фильтров и их параметров.
Выполнить отдельной подзадачей постороение отдельной нейросети, решаюшей задачу очистки от шумов адаптированно к нашему набору аудиоданных.

- Микросервис анализа аудио: Использует предобученную на большом AudioSet модель <a href="https://github.com/tensorflow/models/tree/master/research/audioset/yamnet">YAMNet</a> для определения и классификации звуковых событий бытового характера.

Можно развивать в сторону создания надстройки к ней своей нейросети и дообучения ее на своем размеченном датасете.

- Микросервис визуализации: Генерирует мел-спектрограммы и другие визуализации для анализа аудио.

- Микросервис хранения и отчетности: Управляет сохранением результатов в формате CSV и обеспечивает доступ к отчетам.

## Текущие упрощения и пути развития
- В коде реализована детекция одного звукового события на каждый временной шаг (0,48 с) с наибольшей вероятностью по мнению модели <a href="https://github.com/tensorflow/models/tree/master/research/audioset/yamnet">YAMNet</a>.

Очевидно, что в каждый момент времени могут существовать несколько событий и они будут обнаружены с разной вероятностью. Код можно доработать в сторону мультидетекции.
**Upd.** Сделал версию с потоковой обработкой с микрофона, с мультидетекцией одновременных событий, по кастомному списку распознаваемых событий. Обнаруженные события группируются в одно событие, если они имеют одинаковый классификатор, пока не возникнет пауза дольше пороговой длины. У события выводится усредненная по группе вероятность обнаружения. По завершению обработки потока, результаты сохраняются в csv файл.

- В проекте задействован функционал только предобученной сети <a href="https://github.com/tensorflow/models/tree/master/research/audioset/yamnet">YAMNet</a>.

Очевидно, что события в предлагаемых файлах могут не сответствовать классификации этой или любой другой предобученно сети.
Поэтому для повышения точности и достоверности работы детектора требуется дообучение этой сети надстроенной нейросетью с использованием собственного размеченного датасета.

### Итого сделал

- <a href="https://colab.research.google.com/drive/1KoObbZqzG2zsbhrAzazxc5fO-UtfnnoQ?usp=sharing">Решение по проекту в Google Colab</a>

- Для возможности запуска проекта на других машинах собрал код, входные файлы, русифицированные метки модели, окружение и зависисмости в Docker контейнер. Docker-образ можно взять в моем репозитории:

docker pull antonshalin/audio_event_detection:latest

Для простого вывода в консоль результатов обнаружения звуковых событий сделайте чистый запуск:

docker run antonshalin/audio_event_detection

Если вы хотите сохранить результаты работы приложения на хост-машине, используйте Docker volumes для монтирования соответствующих каталогов:

docker run -v /my/host/directory:/usr/src/app/data antonshalin/audio_event_detection

, здесь /my/host/directory - путь к каталогу на хост-машине, а /usr/src/app/data - путь внутри контейнера, куда этот каталог будет примонтирован.

- Upd. Сделал вариант детекции не по сохраненным файлам, а <a href="https://github.com/antonshalin76/audio_event_detection/blob/main/Audio_stream_event_detection.ipynb">потоковую детекцию событий с микрофона</a>. Работает на локальной машине, не в colab облаке, как прошлый вариант с файлами. <a href="https://github.com/antonshalin76/audio_event_detection/blob/main/yamnet_class_map_translated.csv">Файл классификации</a> положить в ту же папку, дле разместите рабочий ноутбук. Результаты будут сохраняться туда же. Запуск детекции происходит сразу по выполнению ячейки в Jupyter Notebook, прекращение по нажатию кнопки оостановки. Результат оставил в том же виде: накапливающийся перечень событий с метками реального времени и сохранение накопленного перечня в csv файл после остановки процесса детекции.
- Upd. Версия потоковой обработки с микрофона, с мультидетекцией одновременных событий, по кастомному списку распознаваемых событий.
 
## Контактная информация
- Email: anton.shalin@gmail.com
- Tel: +79025666002
- Telegram: https://t.me/anton_shalin
