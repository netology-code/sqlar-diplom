# Итоговая работа по курсу "Администрирование баз данных"

* [Цель проекта](#цель-курсового-проекта)
* [Чек-лист готовности к выполнению проекта](#Чек-лист-готовности-к-выполнению-проекта)
* [Инструменты и дополнительные материалы, которые пригодятся для выполнения итоговой работы](#Инструменты-и-дополнительные-материалы-которые-пригодятся-для-выполнения-итоговой-работы)
* [Задание](#задание)
* [Замечания к заданию](#замечания-к-заданию) 
* [Этапы выполнения проекта](#этапы-выполнения-проекта) 
* [Тестирование](#тестирование) 
* [Правила приема проекта](#правила-приема-проекта)
* [Критерии оценки проекта](#критерии-оценки-проекта)

### Цель курсового проекта

1. Научиться рассчитывать и создавать структуру БД
2. Проработать логику разделения полномочий и прав доступа
3. Получить практические навыки прогнозирования возможных метрик производительности и потребление ресурсов БД с учетом планируемого профиля нагрузки

------

### Чек-лист готовности к выполнению проекта
1. Установлена PostgreSQL 15 версии.
2. Запущен экземпляр базы данных в режиме чтение запись.

------

### Инструменты и дополнительные материалы которые пригодятся для выполнения итоговой работы

1. [Документация](https://postgrespro.ru/docs/postgresql/15/diskusage) по использованию диска 
2. [Документация](https://postgrespro.ru/docs/postgresql/15/storage) по внутреннему физическому хранению БД 

------

### Задание 

Банк, в котором вы работаете, запускает новый проект по внедрению SMART POS-терминалов.
Ожидается успешное внедрение устройства на рынке и развитие клиентской сети (компании использующие терминалы в своих финансовых расчетах).
Руководитель проекта планирует, что данные по терминалам (фин. операции) нужно будет выгружать в корпоративное хранилище, 
также требуется предусмотреть выделение архивных исторических данных в архив для отделения старых данных от БД в целях оптимизации.
 
Имеются следующие бизнес требования:

1. Запускается система SMART POS-терминалов, разрабатываемая сторонней внешней компанией.
2. Предполагаемое количество клиентов на этапе стабильной работы системы в течении первого года 10 компаний, 
каждая может иметь в среднем 5-10 терминалов, при общем расчете до 100 терминалов итого. 
Система должна выдерживать потенциально генерируемую нагрузку с учетом плюс 30% от стандартной нагрузки (поправка на пиковые моменты). 
Интенсивность работы терминалов: предполагается получать в среднем 1 транзакцию в минуту.
3. Необходимо будет реализовать выгрузку данных в корпоративное хранилище (другая БД на платформе PostgreSQL) для отдела DWH (Корпоративное Хранилище).
4. Необходимо обеспечить возможность взаимодействия нескольких команд банка (клиентов базы данных) с различными наборами доступов к данным.
5. К базе данных будут подключаться пользователи из команды “поддержки пользователей системы” (это внутреннее приложение, разрабатываемое командой 
разработки в самом банке, но другого подразделения).
6. Обеспечить возможность восстановления работы системы в случае аварийной ситуации (физическое повреждение сервера БД) в течении суток с возможной потерей данных максимально в 24 часа.
7. Логирование всех SQL-операций выполняющихся более 500ms
  
---

### Замечания к заданию

1. В рамках текущего проекта, рассматривается возможность использования только одного дата центра и не рассматривается отказоустойчивость с 
использованием резервного сервера (с потоковой репликацией), для пилотного первого года это дорого.
2. Расчет тестирования на потенциальную нагрузку требуется сделать гипотетически, представить логику размышления, как планируется это обеспечить, без проведения практических изысканий. Но требуется оценить количество сессий возникающих при работе с БД. 
Терминалы подключаются к серверу приложения, которое в свою очередь открывает сессию от каждого терминала в БД, серверов приложения может быть больше одного.
3. Ресурс, куда будут складываться резервные копии, автоматом копируется на ленточный носитель и подразумевается, что является надежным хранилищем данных при любых аварийных сценариях.
4. Количественные значения, которые не даны по условию задачи, требуется додумать самостоятельно, так как они могут помочь в создании модели системы и более подходящему к реалиям ситуации, исходя из ваших размышлений (результирующие значения будут оцениваться исходя не из количества, а из логики размышления).
5. Структуру БД (состав и количество таблиц) требуется составить из логики, что БД должна содержать необходимый минимум информации - описание общей модели объектов и их взаимодействия, например:
   - терминал - физические данные устройства, 
   - клиент в системе - оформленная компания как потребитель услуги использования терминалов, 
   - торговая точка клиента - магазин, 
   - платеж - финансовая транзакция.
6. Клиентов (команды), взаимодействующих с БД, можно предложить своих, так как это очень индивидуально, но минимум это должны быть: 
    - сами терминалы (короткие платежные операции: продажа, отмена продажи), 
    - команда поддержки пользователей (небольшие отчеты по клиенту), 
    - команда разработки (владелец системы для наката обновлений), 
    - администратор БД, 
    - команда Корпоративного хранилища.
  
---

### Этапы выполнения проекта

1. Провести расчет требуемых ресурсов сервера для обеспечения бесперебойной работы БД в течении одного года:
   - необходимое количество процессоров и оперативной памяти
2. Провести анализ структуры БД и ее объектов:
   - структура таблиц
   - присутствие необходимых индексов и сиквенсов, если требуется
3. Определить необходимую минимальную конфигурацию отказоустойчивости:
   - минимально необходимую  конфигурацию экземпляра БД WAL-логирования - режим архивирования
   - конфигурация резервного копирования
4. Определить источники потребителей ресурсов БД и спрогнозировать их профиль нагрузки:
   - определить какие источники как будут подключаться к БД (возможность использования балансировщика коннектов, напрямую или еще как)
   - определить необходимые минимумы конфигурации экземпляра БД для подключения пользователей к БД (параметры конфигурации БД)
5. Предусмотреть возможность выгрузки необходимых данных в Корпоративное хранилище:
   - определиться с таблицами содержащими данные для Корпоративного хранилища
   - предусмотреть возможность репликации таблиц данных в Корпоративном хранилище
   - настроить необходимую конфигурацию экземпляра БД и самих таблиц (параметры и свойства таблицы для беспрепятственной выгрузки данных средствами PostgreSQL в другую БД PostgreSQL)
6. Рассчитать минимальный необходимый план доступов для команд потребителей ресурсов БД:
   - определиться, какие команды будут взаимодействовать с БД
   - определиться, какие доступы нужны для команд
   - рассчитать и создать необходимые роли доступов для команд
   - создать необходимых пользователей с предоставлением им прав
   - создать политику дефолтных прав доступа
7. Предусмотреть настройку логирования событий чекпоинта, скорости выполнения запросов/команд в БД.

---

###  Тестирование

1. Выполнить тестирование создания резервной копии и восстановления БД с нее. База данных должна открыться как результат в режиме чтения и записи.
2. Проверить работы механизма логирования.
3. Проверить старт, остановку сервиса БД и доступность БД для работы, как через локальный адрес 127.0.0.1, так и через внешний адрес сервера доступный для подключения серверам приложения, согласно разработанной вами модели подключения пользователей к БД (балансировщик или на прямую).
 
---

###  Правила приема проекта

По результатам выполнения работы необходимо предоставить файлы, содержащие следующую информацию:
1. Дамп структуры БД с данными.
2. Команды выполнения резервного копирования.
3. Файл конфигурации параметров экземпляра БД.
4. По пункту 1,4 (этапов выполнения проекта) предоставить текст расчета или анализ-вывод, что требуется сделать и какие количественные показатели требуются.
5. По пункту 5 предоставить команды, которые будут выполнены для настройки репликации.
6. Любая другая информация на ваше усмотрение, что поможет корректно и полно оценить весь объем работы.

---

### Критерии оценки проекта

1. Создана новая продуктивная БД.
2. Заведены необходимые пользователи и роли, которым выданы требующиеся для работы привилегии.
3. Выполнена настройка режима работы архивации журналов WAL.
4. Сконфигурирована процедура выполнения резервного копирования.
5. Настроен необходимый уровень журналирования.
