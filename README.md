# Задание 1 — найди ошибки

В этом репозитории находятся материалы тестового задания "Найди ошибки" для [14-й Школы разработки интерфейсов](https://academy.yandex.ru/events/frontend/shri_msk-2018-2) (осень 2018, Москва, Санкт-Петербург, Симферополь).

Для работы тестового приложения нужен Node.JS v9. В проекте используются [Yandex Maps API](https://tech.yandex.ru/maps/doc/jsapi/2.1/quick-start/index-docpage/) и [ChartJS](http://www.chartjs.org).

## Задание

Код содержит ошибки разной степени критичности. Некоторые из них — стилистические, а другие — даже не позволят вам запустить приложение. Вам нужно найти все ошибки и исправить их.

Пункты для самопроверки:

1. Приложение должно успешно запускаться.
1. По адресу http://localhost:9000 должна открываться карта с метками.
1. Должна правильно работать вся функциональность, перечисленная в условиях задания.
1. Не должно быть лишнего кода.
1. Все должно быть в едином codestyle.

## Запуск

```
npm i
npm start
```

При каждом запуске тестовые данные генерируются заново случайным образом.

# Решение

Склонировав проект, установив зависимости и запустив проект, у меня в консоли появилось одно предупреждение:

    WARNING in ./src/index.js 4:2-9
    "export 'default' (imported as 'initMap') was not found in './map'
     @ multi (webpack)-dev-server/client?http://localhost:9000 ./src/index.js

По описанию сразу понимаю, что в проекте применяется модульный подход организации JS-кода. 

Предупреждение гласит, что экспорт по умолчанию (импортируемый как `initMap`) не найден по адресу `./map`. Предупреждение указывает на файл с адресом `./src/index.js`, значит, идем искать проблему в файл `index.js`.

Заглянув в код модуля `index.js`, вижу следующую первую строку:

    import initMap from "./map";

`initMap` импортируется без фигурных скобок, а значит, это значение по умолчанию. Загляну в map.js, чтобы убедиться в том, что там находится именно экспорт по умолчанию. 
И что я вижу на пятой строке файла map.js?

    export function initMap(ymaps, containerId) {

Вижу то, что это вовсе не “экспорт по умолчанию”. Почему? Потому что после слова `export` отсутствует слово `default`. А вот если поставить после export слово `default`, то значение станет «экспортом по умолчанию». И тогда значение можно импортировать без фигурных скобок.
Как буду решать вопрос? Могу поставить `default` после `export` в map.js – первое решение. А могу просто взять в фигурные скобки `initMap` при импорте в index.js. Я выберу второй вариант. Так как в данном случае это не так важно какой способ выбрать: «экспорт по умолчанию» – это «синтаксический сахар» и его я могу использовать на свое усмотрение. 
Эта проблема решена. Идем дальше.  


Теперь ошибок я не вижу ни в своей консоли, ни в консоли Chrome DevTools, только сообщение “inited”, что, судя по коду, хорошо. Однако карта, по-прежнему, на странице не отображается.
Первым делом я загляну во вкладку Elements в DevTools: навожу на контейнер для карты – div с идентификатором map – и вижу, что его высота равна нулю. Логично, что и у его потомка ymaps инлайново высчитываемая высота тоже равна нулю пикселей. Иду в файл index.css и задаю высоту в 100% для уже существующего селектора `html, body, #map`:

    html,
    body,
    #map {
        margin: 0;
        padding: 0;
        height: 100%;
    }

Отлично! Теперь карта отображается на странице и занимает 100% ширины и высоты окна браузера. Условие “всю область экрана занимает интерактивная карта Москвы” теперь можно считать выполненным.  


Идем дальше. Следующее условие тоже не выполняется:
“На карте отображаются места размещения базовых станций.
- Если на небольшом пространстве много объектов, они объединяются в кластер. 
- При клике на кластер карта масштабируется для просмотра объектов, входящих в него.”

Дело в том, что метки вообще отсутствуют на карте. Обращусь за информацией к вкладке Network в DevTools, чтобы узнать какая информация приходит ко мне в браузер. Там выбираю только одну вкладку XHR (эта вкладка дает возможность смотреть запросы, которые браузер отправляет серверу из JS-скриптов). Перезагружаю страницу и вижу два запроса: один из них идет по адресу http://localhost:9000/api/stations, что вижу во вкладке Header. Открыла посмотреть тело ответа во вкладке Preview: вижу массив из 721 объекта, и у каждого объекта есть поля с долготой и широтой. Предполагаю, что это и есть те самые метки, которые мне нужны.
Возвращаюсь в редактор: послнотекстовым поиском по всему проекту ищу слово stastions. Нахожу шесть соответствий, перехожу в файл backend-stub.js: на первой строке импортируется файл /generate-data, а на третьей строке используется метод `generateData()`, на седьмой строке вижу, что объявлен route, на которой происходит запрос. На восьмой строчке вижу, что возвращаются преобразованные в JSON данные, ранее сгенерированные методом `generateData()`. Значит, надо смотреть в импортированный файл, чтобы убедиться, что это действительно нужные мне метки. Перехожу в файл generate-data.js и на 18 строке вижу экспорт `generateData`. Внутри которого вижу цикл, который генерирует 721 запись, а внутри цикла вижу, что координаты генерируются случайным образом вокруг координаты 55.755222 37.62102. А на 23 строке, скорее всего, герерируется серийного номера вышки с помощью библиотеки faker.js. Далее мне нужно найти где используется этот route. Захожу в api.js.и вижу что `route` `/api/stations` используется в функции `loadList`. А внутри функции вижу, что полученный ответ преобразется в другой объект с помощью импортированного метода `mapServerData` из mappers.js. Иду смотреть mapper.js и вижу, что внутри функции mapServerData возвоащается новый объект, который содержит в себе коллекцию сущностей с геометками, которую будет можно передать в `ObjectManager`, который в свою очередь и должен работать с моими метками, судя по документации: https://tech.yandex.ru/maps/doc/jsapi/2.1/ref/reference/ObjectManager-docpage/
Возвращаюсь в передыдущий файл в api.js и хочу полнотекстовым поиском найти где используется `loadList`, для того чтобы узнать куда передаются эти трансформированные объекты. Вижу, что импориирется в map.js и на 24 строке вызывается промисом, и когда промис закончит работу, в ObjectManager будут добавлены объекты как будут получены.

    loadList().then(data => {
        debugger; //поставив вот тут debugger^ честно убеждаюсь, что данные получены и смапленны в FeatureCollection с точками
        objectManager.add(data);
    });

Здесь все хорошо, проблем не вижу. На 12 строке вижу, что, действительно, ObjectManager объявлен. Сверяю с примером из документации: замечаю в примере из документации, что ObjectManager просто напросто не подключается. Добавляю следующую на 22 строку в map.js следующее:

    myMap.geoObjects.add(objectManager);

Судя по документации, я сделала все верно. В чем же тогда дело? Я, по-прежнему, не вижу меток на карте.

Интуиция и немалый опыт поисков чужих ошибок подсказывает мне, что карту стоит отдалить и посмотреть, не "потерялись" ли мои карты в другом месте. И о чудо! Мои метки "убежали" в Иран. Минус одна проблема: метки есть. Но теперь это значит, что у них проблемы с координатами. Иду в mappers.js и сразу предполагаю что координаты могли быть просто перепутаны. На одиннадцатой строке пробую поменять местами долготу и широту:

    coordinates: [obj.lat, obj.long]

И действительно, долгота и широта перепутаны. Метки есть и они на своих местах - задание выполнено.

Идем дальше. 
Следующее требование звучит так:
" Неисправные станции обозначаются на карте красным цветом, исправные — синим.
- Используя фильтр, можно отобразить на карте объекты с нужным состоянием — например, отобразить только неисправные.
- Если неисправный объект входит в кластер, то иконка кластера должна показывать, что в нем есть неисправная станция."
Кластеры у нас есть, но у нас нет никаких обозначений на них, что где-то значатся неисправные станции. Почитав код кластеров и прошерстив документацию, нахожу несостыковку: дело в том, что в map.js на 24 строке стиль кластера переобъявляется и закрашивается полностю зеленым:

    objectManager.clusters.options.set('preset', 'islands#greenClusterIcons');

И это при том, что стиль кластера был объявлен выше. И кластер должен был выглядеть как круговая диаграмма (pie chart):

      const objectManager = new ymaps.ObjectManager({
        clusterize: true,
        gridSize: 64,
        clusterIconLayout: 'default#pieChart', // <-- стиль кластера объявлен
        clusterDisableClickZoom: false,
        geoObjectOpenBalloonOnClick: false,
        geoObjectHideIconOnBalloonOpen: false,
        geoObjectBalloonContentLayout: getDetailsContentLayout(ymaps)
    });

Я комментирую переобъявление стилей кластера, а точнее вот эту строку: `objectManager.clusters.options.set('preset', 'islands#greenClusterIcons');`. И теперь мои кластеры стали показывать находятся ли неисправные станции в этой местности. Теперь неисправные станции обозначаются на карте красным цветом, исправные — синим. Это требование можем считать выполненым.

И последнее требование:
"При клике на метку базовой станции появляется попап с информацией о ней: серийный номер, состояние, количество активных дронов, график нагрузки."
Мне кажется, под попапом в данном котексте говорится о балуне метки. Да и в коде все указывает на то, что речь идет именно о балуне. Почитав документацию и посмотрев примеры, я окончательно убедилась, что балун объявлен верно. Еще немного почитав код я нашла ошибку. Ошибка пряталась в файле details.js в двух строках:

    build: () => {

и:

    clear: () => {

А затем внутри ипсользовался this. Но стрелочная функция не создаёт свой контекст исполнения, а заимствует this из внешней функции, в которой она определена. Поэтому заменив стрелочную функцию на обычную, можно решить проблему проблему.

Ура, балун отображается. Но теперь не хватает последнего: графика. Захожу снова во вкладку Network в DevTools и вижу, что координаты для графика приходят в виде массива. Значит, данные приходят и ошибка не здесь. Иду в файл chart.js и вижу точки - все верно. Иду в map.js и вижу, что то, что получили передаём объекту как модель для отображения во вьюхе, которая рендерится вот здесь:

    if (!obj.properties.details) {
      loadDetails(objectId).then(data => {
        obj.properties.details = data;
        objectManager.objects.balloon.setData(obj);
      });
    }

Далее иду в файл details.js и на 21 строке вижу canvas, в котором как раз и отрисоывывается нужный мне график:

    <canvas class="details-chart" width="270" height="100" />

По классу делаю поиск по всему проекту и в этом же файле ниже вижу, что функция подключена из другого файла:

     if (details) {
        const container = this.getElement().querySelector('.details-chart');

        this.connectionChart = createChart(
        container,
        details.chart,
        details.isActive
        );
    }

Иду в chart.js и вижу, что в стороннюю библиотеку Chart.js мы передаём то, что получили от бэка. Ставлю debugger и убеждаюсь в этом. Значит, дело не в этом. 
Захожу в документацию: https://www.chartjs.org/docs/latest/
Смотрю в пример: передаётся дата - выглядт очень похоже на мой код. Спускаюсь к options'aм, обращаю внимание на параметр `max: 0` у меня в коде - это ограничение по y-оси, то есть это значит, что значение выше нуля не показывать. А у нас передаются количество подключений к вышке, которые всегда выше ноля, следовательно - ошибка. 
Убираю `max: 0` и график появился.

Теперь у меня работает все так, как говорится в требованиях. Но меня все еще смущает слово попап в требованиях и наличие файла popup.js, который нигде не используется. Надо в этом разобраться: иду в код этого файла.
Выглядит так, будто файл popup.js не из этого проекта. Верстка имеет классы station-info, которые нигде больше не встречаются в проекте. Также popup.js работает с неким объектом, который должен иметь следующую структуру:

    var obj = {
    isActive: Boolean,
    drones: Array,
    id: Integer,
    serialNumber: string,
    connections: Integer
    }

Очень похоже на то, что я должна получать с сервера, но в backend нет ничего, что бы отдавало массив дронов. 
Значит, я могу смело удалить этот файл из проекта.