# Data Science Avito

Данный проект представляет собой сборник утилит для анализа, визуализации и процессинга данных объявлений Авито.

## Реализованные инструменты

В данном списке находятся все подпроекты, с помощью которых происходит анализ и визуализация данных

* [MCDA Flat Finder](https://github.com/kubikrubikvkube/data_science/blob/master/README.md#mcda-flat-finder)
* [New Residential Areas Heatmap](https://github.com/kubikrubikvkube/data_science#new-residential-areas-heatmap)

***
## MCDA Flat Finder
В этом проекте реализован алгоритм [мультикритериального анализа решений](https://en.wikipedia.org/wiki/Multiple-criteria_decision_analysis) для выбора оптимальной квартиры для покупки в городе Санкт-Петербург.
User-Interface базируется на [Jupyter Notebook](https://jupyter.org/) и [Jupyter Widgets](https://ipywidgets.readthedocs.io/en/stable/)
Общий вид программы
![UI](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_0.png)
При нажатии на кнопку **Обновить базу данных с Авито** скрипт использует Avito private API для того чтобы получить все объявления о продаже недвижимости за последние 30 дней.
![Update_DB](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_6.png)

Это занимает около 10-20 минут. Все объявления сохраняются в SQLite базу данных, в одну таблицу с примитивной структурой

![SQLite_Structure](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_7.png)

В таблице находится около 50 000 записей, данные выглядят таким образом

![SQLite_Table_Data](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_8.png)

Где id - ключ объявления, идентичный ключу в базе данных Avito.

При нажатии на кнопку **Обновить СSV файл с геоданными** скрипт выполняет расчёт геопараметров для каждого объявления о продаже недвижимости и сохраняет их в CSV файл для дальнейшего анализа с помощью PANDAS.
![Analyze_Geodata](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_9.png)

Формат CSV файла:

| item.natural_id  | item.price | item.closest_metro_distance.m | item.distance_to_city_center.m | item.square_meters |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| 117459  | 3100000  | 2322.9588777099475 | 7049.398285855972 | 25.0 |
|117459|3100000|2322.9588777099475|7049.398285855972|25.0|
|207635|1800000|444.1950510152622|9658.967118430239|24.7|
|277212|7500000|2465.026781234918|11371.925201938595|60.0|
|280535|3257396|7698.607259706862|15850.820627283993|36.8|

Где 
* item.natural_id - ID объявления
* item.price - стоимость квартиры (руб)
* item.closest_metro_distance.m - расстояние в метрах до ближайшей станции метро рассчитанное с помощью [GeoPy](https://geopy.readthedocs.io/en/stable/#module-geopy.distance)
* item.distance_to_city_center.m - расстояние в метрах до центра города (за 'центр' взяты координаты метро Невский Проспект)
* item.square_meters - площадь квартиры (м²)

После обновления этих данных скрипт полностью готов к расчёту лучших вариантов квартир из доступных на Авито. Переходим непосредственно к заданию критериев поиска.

 Вкладка "Цены"
 ![Prices](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_4.png)
 На данной вкладке можно задать желаемую минимальную и/или максимальную цену квартиры. Эта вкладка необходима для фильтрации тех предложений, которые вписываются в рамки исходного бюджета.
 
 Вкладка "Веса"
 ![Weights](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_3.png)
 Данная вкладка необходима для настройки [весов](https://ru.wikipedia.org/wiki/%D0%92%D0%B5%D1%81%D0%BE%D0%B2%D0%B0%D1%8F_%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F) при выборе оптимального предложения квартиры. Именно заданные веса и определяют оптимальность окончательных предложений для конечного пользователя. 
 
 Вкладка "Ограничения"
 
 ![Limitations](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_5.png)
 Параметры на этой вкладке задают опциональные ограничения при фильтрации исходной выборки. Пользователю известно, какая минимальная площадь квартиры его устроит. Минимальная и максимальная площадь задаётся для фильтрации вариантов, которые заведомо неприемлимы. Аналогично и с расстоянием до метро - минимальное расстояние может быть задано для фильтрации заведомо фейковых объявлений (когда указан адрес в нескольких остановках от метро, но координаты указаны в 5-10 метрах от станции), а также для случаев, когда расстояние до метро важно конечному пользователю.
 
Что будет если попробовать оставить все значения средними?

![Average_Analyze](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_11.png)

Лучшее объявление, удовлетворяющее нашему запросу:

![Average_Analyze_2](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_12.png)

По мнению системы, квартира-студия через дорогу от метро Гостиный Двор является оптимальным вариантом среди 50 000 объявлений по соотношению цена/близость к метро/близость к центру. Это объявление действительно выглядит достаточно сбалансированным вариантом.

Допустим мы хотим приобрести квартиру максимальной площади, наиболее близко расположенную к метро, и в данном случае нам не важна близость центра и мы согласны на вариант в спальном районе.

![Close_to_metro](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_13.png)

![Close_to_metro2](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_14.png)

Система предлагает нам вариант трёхкомнатной квартиры через дорогу от метро Автово общей площадью 78.2м² , что впринципе согласуется с нашими целями.

Возьмём более экзотический вариант - мы хотим приобрести наиболее дешёвое, и наиболее просторное жильё. В данном случае мы не можем рассчитывать на близость к метро, или же близость к центру города и ориентируемся на вариант "большой старый дом, до которого можно добраться только на автомобиле".

![Luxury_house2](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_15.png)

![Luxury_house2](https://github.com/kubikrubikvkube/data_science/blob/master/docs/images/mcda_flat_finder_16.png)

В данном случае системы выбрала для нас 6(!) комнатную квартиру в бывшем бараке, который находится на окраине Ржевского лесопарка, недалеко от КАД. Этот вариант однозначно неудобен в плане транспортной доступности и находится на старой окраине города, но прекрасно удовлетворяет нас по цене, и уж тем более по размеру доступной площади.

### Использованные библиотеки
* [PANDAS](https://pandas.pydata.org/)
* [Scikit-Criteria](http://scikit-criteria.org/en/latest/)

***
## New Residential Areas Heatmap

### Использованные библиотеки
* [PANDAS](https://pandas.pydata.org/)
* [ipyleaflet](https://ipyleaflet.readthedocs.io/en/latest/)
