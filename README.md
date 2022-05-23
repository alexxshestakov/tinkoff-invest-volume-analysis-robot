# tinkoff-invest-volume-analysis-robot
## Робот на основе объемного анализа для Тинькофф Инвестиций

![Background image](/preview.png)

### О приложении
Кластерный анализ и профиль рынка позволяет определить действительную заинтересованность участников рынка.<br/> 
Был реализован один из методов работы с профилем рынка - реакция на максимальный горизонтальный объем внутри дня за выбранный период.<br/>
Проведен анализ истории в промежутке с 04-05-22 по 20-05-22. Реализованная стратегия дает **положительное математическое ожидание** при подборе параметров (результаты представлены в `./logs/statistics`).<br/> 
Все тесты проведены на трех инструментах в равных условиях (на одинаковых параметрах).

Основной объем работы был заложен в математический аппарат.<br/> 
Робот был протестирован **только** в песочнице.<br/>
Будьте придельно внимательны, если решите воспользоваться текущим решением!

### Инструкция по запуску
Версия python 3.8
```
$ python -m venv ./venv
$ ./venv/Scripts/activate
$ pip3 install -r ./requirements.txt
```

### Запуск робота
```
$ python ./trading_robot.py
```

### Запуск анализа истории
- предварительно распаковать архив `./data/trades.zip`;
- проверить указанные пути к историческим файлам для `./tests/test_profile_touch_strategy.py`;
- выполнить: `python ./tests/test_profile_touch_strategy.py`

### Возможности робота
- накопление истории по обезличенным сделкам по указанным инструментам;
- тестирование алгоритма на истории* (без учета комиссии и проскальзываний);
- просмотр результата анализа на графике;
- восстановление данных за промежуток времени при сбоях;
- анализ актива и торговля его фьючерса в реальном времени;
- управление настройками алгоритма и возможность его полной замены;
- выступать в роли робота с открытием позиций или только советника;

### Алгоритм 
1. Строим объемные уровни за выбранный ТФ.
2. Если цена, через заданный промежуток времени, подходит к одному из объемных уровней:
- сверху вниз, то рассматривается ТВ в лонг;
- снизу вверх, то рассматривается ТВ в шорт.
5. Если сформировалась ТВ, то анализируем предыдущую закрытую свечу (ТФ свечи указывается в настройках):
- если в свече преобладают лонгисты и максимальный объем свечи снизу, то открывается позиция в лонг; 
- если в свече преобладают шортисты и максимальный объем свечи сверху, то открывается позиция в шорт.
5. Устанавливается S/L и цели в соответствии с заданными настройками.
- стоп-лосс задается на уровне приложения (серверный стоп-лосс не выставляется); 
- закрытие сделок осуществляется обратным ордером.
6. Осуществляется уведомление через telegram-бота.

### Визуализация алгоритма (добавить скриншоты)
- осуществляется в режиме тестирования истории при включении параметра в настройках;
- данные графика обновляются каждый час (для уменьшения нагрузки, т.к. обработка истории быстрее, по сравнению с реальными торгами)
- график является интерактивным, позволяет рассматривать участки графика в большем/меньшем масштабе;
- графически представлена следующая информация:
  - свечи с заданным периодом (ТФ);
  - профиль рынка за указанный период времени;
  - максимальный объем в свечах;
  - маркер с указанием точек входа;

### Структура приложения
- /data - исторические данные по сделкам;
- /logs - логи приложения и статистика по сделкам;
- /jupyter - проект в jupiter notebook для тестирования алгоритмов;
- /domains - сущности для удобного взаимодействия между методами;
- /services - сервис для работы со сделками и уведомлениями;
- /strategies - стратегии робота;
- /tests - модульное тестирование кода и стратегии;
- /utils - вспомогательные методы для сервисов и стратегии;
- /visualizers - графические визуализаторы стратегии;
- settings.py - общие настройки робота и настройки стратегии; 
- trading_robot.py - точка входа для запуска робота;

### Общие параметры
| Свойство             | Описание                                                                          | Рекомендуемые значения |
|----------------------|-----------------------------------------------------------------------------------|------------------------|
| TOKEN                | Токен профиля в Тинькофф Инвестиций                                               | -                      |
| IS_SANDBOX           | Используется песочница или реальный счет                                          | -                      |
| ACCOUNT_ID           | Счет для торговли в зависимости от выбранного режима работы                       | -                      |
| INSTRUMENTS          | Массив анализируемых инструментов                                                 | -                      |
| CAN_OPEN_ORDERS      | Приложение выступает в роли советника или робота с открытием позиций              | -                      |
| CAN_REVERSE_ORDER    | Признак переворачивания позиции, если определена ТВ в противоположное направление | True                   |
| COUNT_LOTS           | Количество лотов на 1 точку входа                                                 | > 1                    |
| COUNT_GOALS          | Количество целей на 1 точку входа                                                 | 2                      |
| FIRST_GOAL           | Соотношение к стоп-лоссу                                                          | 3                      |
| GOAL_STEP            | Размер шага для очередной цели                                                    | 0.5                    |
| PERCENTAGE_STOP_LOSS | Процент, на который устанавливается стоп-лосс                                     | 0.03                   |
| IS_SHOW_CHART        | Отображение графика при анализе                                                   | -                      |
| NOTIFICATION         | Токен бота и id чата для уведомлений в телеграм                                   | -                      |

### Параметры стратегии
| Свойство                      | Описание                                                                           | Рекомендуемые значения |
|-------------------------------|------------------------------------------------------------------------------------|------------------------|
| PROFILE_PERIOD                | Период профиля рынка                                                               | 1h                     |
| SIGNAL_CLUSTER_PERIOD         | ТФ сигнальной свечи для рассмотрения ТВ                                            | 5min                   |
| FIRST_TOUCH_VOLUME_LEVEL      | Время в минутах, через которое можем рассматривать первое касание объемного уровня | 90                     |
| SECOND_TOUCH_VOLUME_LEVEL     | Время в минутах для последующих касаний объемного уровня                           | 5                      |
| PERCENTAGE_VOLUME_LEVEL_RANGE | Процент, на который цена может превысить или не дойти до объемного уровня          | 0.03                   |

### Планы по развитию

#### Минимизировать риски стратегии
- текущие настройки алгоритма часто дают убыточные сделки, когда цена движется в узком диапазоне; 
- провести более подробное тестирование истории на разных параметрах: ТФ свечей, ТФ объемных уровней;

#### Архитектура
- хранение данных в файлах оказалось не лучшим решением (todo причины);
- оптимизировать расчеты стратегии:
  - тестирование истории 1 инструмента за 1 день занимает от 1 до 5 минут;
  - в случае сбоя приложения и повторного запуска, стенд с 1 ядром нагружается до 99% (за счет загрузки и объединении истории);

#### Реализовать алгоритмы
- вход по тренду с ориентированием на максимальный объем текущего дня;
- вход против тренда с ориентированием на максимальный объем текущего дня;
- реакция на максимальный горизонтальный объем предыдущего дня;
- вход на обновление минимума/максимума текущего дня / предыдущих дней;

#### Управление позицией
- закрывать позицию частями;
- добавить трейлинг-стоп;
- открывать позицию частями: по рынку и лимитными ордерами;
- закрывать позицию полностью, если ситуация на рынке изменилась на противоположную;
- входить на повышенный/уменьшенный объем, если алгоритм дает идеальные/слабые условия;

#### Удобство взаимодействия
- реализовать api для робота;
- реализовать web-интерфейс;