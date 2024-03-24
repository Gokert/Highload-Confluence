# Проектирование высоконагруженного пространства для командной работы

Курсовая работа в рамках 3-го семестра программы по Веб-разработке ОЦ VK x МГТУ им. Н.Э. Баумана (ex. "Технопарк") по дисциплине "Проектирование высоконагруженных сервисов"

#### Автор - [Андрей Мышляев](https://park.vk.company/profile/a.myshliaev/ "Страница на портале VK x МГТУ")
#### Задание - [Методические указания](https://github.com/init/highload/blob/main/homework_architecture.md)

#### Содержание:
1. [Тема, функционал и аудитория](#1)
2. [Расчет нагрузки](#2)
3. [Расчет глобальной нагрузки](#3)
4. [Расчет локальной нагрузки](#4)

## Часть 1. Тема и целевая аудитория <a name="1"></a>

### Тема курсовой работы - **"Проектирование сервиса для командной работы"**
Confluence — это удобное рабочее пространство для удаленных команд, в котором участники могут хранить знания и сотрудничать друг с другом. В приложении Confluence Cloud можно легко записывать возникшие идеи, создавать и редактировать страницы и совместно работать с командой практически на любом устройстве.


### Ключевой функционал сервиса
- Поиск и навигация по документам.
- Просмотр документов.
- Создание и редактирование документов.

### Целевая аудитория
- Весь мир
- 25 млн пользователей в месяц (MAU) [^1]
- 0.84 млн пользователей в день (DAU) [^1]

## Часть 2. Расчет нагрузки <a name="2"></a>

### Продуктовые метрики

### Среднее количество действий пользователя по типам в день.
Для подсчета некоторых метрик воспользуемся похожим по функционалу сайту [en.wikipedia.org](https://stats.wikimedia.org/#/metrics/en.wikipedia.org).
У en.wikipedia.org в день [68 млн](https://stats.wikimedia.org/#/en.wikipedia.org/reading/unique-devices/normal|line|2-year|(access-site)~mobile-site*desktop-site|daily) пользователей, тогда коэффициент равен: 68 / 0.84 = 76.

В день происходит [2 млн просмотров страниц](https://hypestat.com/info/atlassian.com). На основе личного опыта предположим, что около 1 страницы пользователь находит по поиску, а навигация по сайту 2.38 - 1 = 1.38.
Тогда получается, что на поиск уходит 0.84 млн просмотров страниц, а на переход по ссылкам и навигацию внутри сайта уходит 1.16 млн просмотров.

#### Создание и редактирование страниц.
Ежесуточно на сайте en.wikipedia.org количество редактирований равняется [114 тысяч](https://stats.wikimedia.org/#/en.wikipedia.org/contributing/user-edits/normal|bar|2-year|(page_type)~content*non-content|daily), тогда: 114 000 / 76 = 1500 редактирований в день.
Ежедневно на сайте en.wikipedia.org создаётся [5 тысяч](https://stats.wikimedia.org/#/en.wikipedia.org/contributing/new-pages/normal|bar|2-year|~total|daily) новых документов, откуда находим: 6 000 / 76 = 80 новых документов в день.

| Действие пользователя  | Количество в   день |
|------------------------|---------------------|
| Поиск                  | 840 000             |
| Навигация              | 1 160 000           |
| Просмотр документов    | 2 000 000           |
| Создание документов    | 80                  |
| Редактирование         | 1 500               |


Всего на сайте en.wikipedia.org существует [7 млн](https://pageviews.wmcloud.org/siteviews/?platform=all-sites&source=unique-devices&start=2024-01-25&end=2024-02-25&sites=en.wikipedia.org) статей. Откуда находим примерное количество статей: 7 млн / 76 = 0.1 млн.

Всего на сайте en.wikipedia.org существует [914 тысяч](https://pageviews.wmcloud.org/siteviews/?platform=all-sites&source=unique-devices&start=2024-01-25&end=2024-02-25&sites=en.wikipedia.org) картинок. Откуда находим примерное количество картинок: 0.914 / 76 = 0.012 млн. Тогда среднее количество картинок на статью равняется 0.012 / 0.1 = 0.12.
Оценим средний размер PNG картинки: (1920 * 1080 * 24) / (8 * 1024 * 1024) * 0.4 (среднее сжатие) = 2.4 МБ

В среднем на статью приходится [620 слов](https://thequickadvisor.com/how-many-links-does-wikipedia-have/), а средняя длина слов равняется 5 буквам. 1 символ в Unicode кодируется 2 байтами. Статья также имеет ID - 16б (UUID).
В среднем каждая вправка занимает [25 МБ](https://stats.wikimedia.org/#/en.wikipedia.org/content/net-bytes-difference/normal|bar|2-year|~total|daily).

Общий объём памяти: 
```
(100 000*620*5*2 + 100 000*16 + 12 000*1920*1080*24*0.4)/(8*1024*1024*1024) = 27.9 ГБ
```
Средний вес одного документа: 
```
27.9 /100 000 = 0.00028ГБ = 280 КБ
```

#### Сводная таблица продуктовых метрик
| Характеристика                                           |   Метрика   |
|:---------------------------------------------------------|:-----------:|
| MAU                                                      |     25M     |
| DAU                                                      |    0.84М    |
| Общее число документов                                   |    0.1 М    |
| Среднее время просмотра страницы                         |   5.26 м    |
| Средний размер документа                                 |   280 КБ    |
| Средний размер вправки документа                         |    25 МБ    |
| Среднее количество действий по просмотру документов      |  2 M/сутки  |
| Среднее количество действий по созданию документов       |  80 /сутки  |
| Среднее количество действий по редактированию документов | 1 500/сутки |

### Технические метрики:

Размер хранения текста: 
```
(100 000*620*5*2 + 100 000*16)/(8*1024*1024*1024) = 0.07 ГБ
```

Размер хранения картинок: 
```
(12 000*1920*1080*24*0.4)/(8*1024*1024*1024) = 27.8 ГБ 
```

RPS по просмотру документов: 
```
2 000 000 / (60 * 60 * 24) = 23
```

RPS по созданию документов: 
```
80 / (60 * 60 * 24) = 0.001
```

RPS по редактированию документов: 
```
2 000 000 / (60 * 60 * 24) = 0.017
```

Возьмем небольшой коэффициент k=1,5 отличия от среднего трафика для получения пикового трафика по просмотру документов: 
```
23 * 0.28 МБ * 1.5 = 9.7 МБ/с
```

Возьмем небольшой коэффициент k=1,5 отличия от среднего трафика для получения пикового трафика по созданию документов: 
```
0.001 * 0.28 МБ * 1.5 = 0.0004 МБ/с
```

Возьмем небольшой коэффициент k=1,5 отличия от среднего трафика для получения пикового трафика по редактированию документов: 
```
0.017 * 25 МБ * 1.5 = 0.64 МБ/с
```

Пиковое потребление в течение суток:
```
9.7 + 0.0004 + 0.64 = 10.34 МБ/с
```

Cуммарный суточного трафик: 
```
(23 * 0.28 + 0.001 * 0.28 + 0.017 * 25) * (24 * 60 * 60) = 593 160 МБ/сутки = 593.17 ГБ/сутки
```

#### Сводная таблица технических метрик
| Характеристика                      |     Метрика     |
|:------------------------------------|:---------------:|
| Размер хранения текста              |     0.07 ГБ     |
| Размер хранения картинок            |     27.8 ГБ     |
| Пиковое потребление в течении суток |  10.34 МБ/сек   |
| Суммарный суточный                  | 593.17 ГБ/сутки |
| RPS по просмотру документов         |       23        |
| RPS по созданию документов          |      0.001      |
| RPS по редактированию документов    |      0.017      |


## Часть 3. Расчет глобальной нагрузки <a name="3"></a>

## Часть 4. Расчет локальной нагрузки <a name="4"></a>


## Список источников:
[^1]: [Статистика Atlassian](https://www.atlassian.com/ru/customers/the-telegraph)

[^2]: [Статистика Hypestat](https://hypestat.com/info/confluence.atlassian.com)

[^3]: [Данные по макс размеру](https://confluence.atlassian.com/confkb/how-to-detect-5mb-of-text-on-page-858576591.html)

[^4]: [Hypestat](https://hypestat.com/info/atlassian.com)

[^5]: [Общая статистика сайта en.wikipedia.org](https://stats.wikimedia.org/#/metrics/en.wikipedia.org)

[^6][Картинки в wikipedia](https://en.wikipedia.org/wiki/Special:MediaStatistics)
