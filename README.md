# Яндекс закрыл публичный API телепрограммы, поэтому граббер более не работоспособен.


## tv_grab_ru_yatv

Программа [tv.yandex.ru](https://tv.yandex.ru) в HTS Tvheadend.

Модуль-граббер EPG для стриминг сервера [HTS Tvheadend](https://tvheadend.org/) ([https://github.com/tvheadend/tvheadend](https://github.com/tvheadend/tvheadend)). Конвертирует JSON-данные телепрограммы с [tv.yandex.ru](https://tv.yandex.ru) в формат [XMLTV](http://xmltv.org). Умеет пересчитывать время передач под разные временные зоны. Написан на Python3 с использованием стандартной библиотеки.


### Установка

Для стандартной установки tvheadend:

    wget --no-check-certificate -O /usr/bin/tv_grab_ru_yatv https://raw.githubusercontent.com/gSpotx2f/tv_grab_ru_yatv/master/tv_grab_ru_yatv
    chmod +x /usr/bin/tv_grab_ru_yatv

Для Entware:

    wget --no-check-certificate -O /opt/bin/tv_grab_ru_yatv https://raw.githubusercontent.com/gSpotx2f/tv_grab_ru_yatv/master/tv_grab_ru_yatv
    chmod +x /opt/bin/tv_grab_ru_yatv
    opkg install python3 python3-setuptools python3-openssl

В web-интерфейсе tvheadend, на странице настройки грабберов включите граббер с названием `Russian tv.yandex.ru (tv_grab_ru_yatv)`


### Настройка

По умолчанию, без использования конфигурационного файла, граббер возвращает XMLTV, который содержит все доступные каналы. Более тонкая настройка позволяет запрашивать у api tv.yandex.ru только необходимые Вам каналы и тэги передач, что снижает как трафик, так и время обновления программы (особенно в случае медленного железа на девайсах вроде роутеров, NAS и пр.). Настройку каналов можно выполнить двумя способами:

1. Явно указать в конфиге в параметре `channels_ids` XMLTV-id нужных каналов (или в `channels_titles` можно задать названия, по которым граббер сам найдёт id) и, если необходимо, индивидуальное смещение времени канала (в часах) для пересчёта времени передач (0, 2, -3 и т.д.):

        "channels_ids": {
            "146": None,
            "82": None,
            "529": None,
            "765": None,
        },
        "channels_titles": {
            "CBS Reality": None,
            "Nickelodeon": None,
            "Fox": 2,
            "Hollywood HD": None,
        },

2. Использовать опцию, при включении которой граббер во время работы сам возьмёт названия каналов из конфигов tvheadend и будет запрашивать данные только для этих каналов:

        "channels_from_tvh": True,
        "tvh_channels_dir": "/home/hts/.hts/tvheadend/channel/config",   # при стандартном размещении конфигов tvheadend
        "tvh_channels_dir": "/opt/etc/tvheadend/channel/config",         # при использовании tvheadend из Entware

Параметр `tvh_channels_dir` определяет директорию с конфигами каналов tvheadend. Если в `channels_titles` указаны каналы, которые граббер найдёт в tvheadend, то для них будет применяться указанное смещение времени передач.

Если смещение времени передач одинаково для всех каналов, то можно использовать глобальный параметр смещения времени передач:

    "default_prg_time_offset": 0

Это смещение применяется к каналам (в `channels_ids` и `channels_titles`) со значением сдвига = None, а также ко всем остальным каналам, не указанным в конфиге.


### Использование

Основные параметры запуска стандартны для грабберов XMLTV:

    --config-file <путь>    # конфигурационный файл граббера
    --configure             # этот параметр запускает граббер в режиме генерации стандартного конфигурационного файла (вывод в STDOUT), который содержит в параметре channels_ids все доступные сейчас каналы. Использование этого конфига без изменений аналогично запуску без конфига (т.е. граббер возвращает XMLTV для всех каналов). Если одновременно указаны два стартовых параметра --configure --config-file <путь>, то конфиг записывается в указанный файл (или перезаписывает уже существующий!). Далее его можно править для дальнейшей настройки.
    --output <путь>         # файл для вывода XMLTV вместо STDOUT. Если указан, то программа вместо tvheadend отправится в этот файл
    --offset <days>         # кол-во дней, которое прибавляется к текущей дате, начиная с которой запрашивается программа (1 - программа с завтрашнего дня, 2 - с послезавтра и т.д.)
    --days <days>           # кол-во дней, на которое запрашивается программа. Начиная с сегодняшнего дня (с учётом --offset)
    --region <num>          # номер региона для tv.yandex.ru

В последних версиях tvheadend параметры для граббера можно задавать на странице настройки модулей EPG.


### Конфигурационный файл

    {
        "default_prg_time_offset": 0,   # смещение времени в часах для пересчёта дат в передачах по умолчанию. Применяется ко всем каналам, для которых смещение не указано в конфиге (= None). (0, 2, -3 и пр.)
        "user_region": None,            # регион программы для tv.yandex.ru. Если None, то не добавляется в запрос
        "duration": 7,                  # длительность программы в днях
        "start_day_offset": 0,          # смещение времени начала программы в днях (0 - с сегодняшнего дня, 1 - завтра, 2 - послезавтра и пр.)
        "time_zone": "+03:00",          # таймзона для запроса. Учитывается сервером для определения времени начала программы
        "chans_per_request": 3,         # кол-во каналов в одном запросе (больше - меньше запросов к серверу, но и больше потребление памяти)
        "chans_logos": True,            # добавлять в XMLTV ссылки на логотипы каналов (<icon>)
        "chans_logos_url": None,        # если задан URL, то при добавлении ссылок на логотипы каналов (<icon>) URL создается след. образом: <chans_logos_url><channel-id><chans_logos_ext>. Для добавления в XMLTV собственных лого каналов. Если None - ссылки на логотипы берутся с tv.yandex.ru
        "chans_logos_ext": ".jpg",      # расширение для файлов пользовательских логотипов каналов
        "additional_tags": True,        # добавлять в XMLTV дополнительные тэги передач. Если выставить в False, то размер программы значительно сократится.
        "additional_tags_dict": {       # дополнительные тэги передач. tvheadend показывает в EPG только: <desc>, <category>, <icon>
            "date": False,
            "desc": True,
            "category": True,
            "country": False,
            "icon": True,
            "credits": False,
        },
        "xmltv_id_suffix": ".yatv",     # суффикс добавляется к XMLTV-id каналов. Можно использовать при составлении XMLTV из разных источников, чтобы не повторялись числовые id. C цифровыми id граббер не проходит валидацию (XMLTV id д.б. в DNS-подобном синтаксисе: 146.yatv и пр.)
        "channels_from_tvh": False,     # брать каналы из конфигурации tvheadend вместо каналов из конфига
        "tvh_channels_dir": "/home/hts/.hts/tvheadend/channel/config",   # директория с конфигами каналов tvheadend
        "replace_categories": True,     # замена категорий передач для tvheadend (чтобы отображались в EPG)
        "proxies": None,                # прокси серверы для граббера. Прим.: "proxies": {"http": "http://192.168.0.1:8080", "https": "http://192.168.0.1:8080"}
        "ssl_unverified": False,        # в случае проблем с сертификатами SSL выставить в True
        "channels_ids": {               # id необходимых каналов. Если не укзаны ({}), то запрашиваются все каналы
            "146": None,
            "82": None,
            "529": None,
            "765": None,
        },
        "channels_titles": {            # то же, что и channels_ids, но вместо id названия каналов. По названиям граббер сам найдёт id, если есть совпадения (регистр не важен)
            "TV-5": None,
            "CBS Reality": None,
            "Nickelodeon": None,
            "Fox": 2,                   # время передач канала Fox пересчитывается со смещением 2 (+2 часа ко времени передач). Можно задавать отрицательные значения: -2 и т.д.
            "Hollywood HD": None,
        },
    }


### Подробнее про настройки

Если Вам нужна программа для всех доступных каналов, то просто оставьте словари `channels_ids` и `channels_titles` пустыми:

    "channels_ids": {}
    "channels_titles": {}

Параметр конфига `additional_tags_dict` даёт возможность выбрать дополнительные тэги передач для XMLTV. Tvheadend отображает тэги: `<desc>`, `<category>`, `<icon>`. Но можно создать максимально полный XMLTV из тех данных, что предоставляет api tv.yandex.ru. Размер итогового файла, в этом случае, увеличивается очень существенно. Также, можно наоборот убрать всё кроме названия передачи (`"additional_tags": False`) и уменьшить вывод XMLTV в разы.

Если при запуске граббера не задан путь к конфиг. файлу (`--config-file`), то скрипт пытается найти конфиг по следующим путям: `./tv_grab_ru_yatv.cfg`, `/home/hts/tv_grab_ru_yatv.cfg`, `/etc/tv_grab_ru_yatv.cfg`, `/opt/etc/tv_grab_ru_yatv.cfg`. Если поместить конфиг по одному из этих путей, то нет необходимости указывать стартовый параметр `--config-file` (полезно для старых версий tvheadend, например из Entware, которые не позволяют задавать параметры запуска для граббера).


### Как увидеть список всех доступных каналов tv.yandex.ru?

    /usr/bin/tv_grab_ru_yatv --configure
    /usr/bin/tv_grab_ru_yatv --configure --region 213   # для региона 213 (Москва)

Выведет в консоль пример стандартного конфига с полным набором доступных каналов.




