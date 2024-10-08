---
link: https://habr.com/ru/companies/ozontech/articles/839214/
tags:
  - data
company: Ozon
author: Ozon
source: habr
data_type:
  - DevOps
---
Всем привет! Меня зовут Саша, я руковожу группой разработки Composer Core в Ozon Tech. В этой статье я расскажу о том, как устроена пользовательская часть одного из ведущих российских маркетплейсов, в развитии которой на момент написания статьи участвуют сотни специалистов из десятков команд. При наличии такого количества людей разработка новой функциональности, без риска сломать уже существующую, является достаточно сложной задачей.

Поделюсь подходами, которые позволили нам организовать взаимодействие большого количества сервисов доменных команд для формирования общих ответов пользователю. При этом менять содержимое страниц можно буквально по щелчку пальцев, а значит, быстро адаптироваться к постоянно меняющимся требованиям бизнеса.

Продукт, который мы разработали, вряд ли когда-нибудь станет open-source-проектом, так как он слишком зависит от специфики инфраструктуры Ozon Tech. Но основные принципы могут быть полезны при проектировании похожих систем.

Если говорить о нашем подходе в двух словах, то он предполагает разбиение большой и сложной системы на составляющие, которые можно представить в виде кубиков. Каждый кубик находится в зоне ответственности конкретной команды. Кубики скрывают в себе сложную реализацию, позволяя не погружённым в разработку менеджерам создавать инструкции. А далее на основании инструкций из кубиков можно собрать нечто большое и осмысленное, что будет решать проблему конечного пользователя.

В первом разделе мы пройдём путь разбиения маркетплейса на составляющие. Я объясню, зачем вообще мы это делаем и какую пользу из этого извлекаем, а также отвечу на вопросы, как и почему мы за несколько лет пришли именно к таким кубикам, из которых строится наш продукт. Во втором разделе я расскажу про формирование инструкций: из чего они состоят, кто их пишет и какие проблемы они решают. В третьем разделе я объясню, как ядро системы собирает из кубиков по инструкциям ответы пользователю и почему оно делает это наиболее эффективным способом. В заключении я подведу итог: расскажу, какие преимущества мы получили от внедрения данного подхода и какие проблемы хотели бы учесть ещё на самых ранних этапах развития продукта. По ходу статьи я буду также выделять требования к нашему продукту, которые формировались в процессе его разработки.

Кому будет интересна статья? Я думаю, что всем любознательным инженерам, которым интересно устройство крупных информационных систем, объединяющих в себе сотни сервисов. Кому будет полезна статья? Разработчикам и архитекторам, перед которыми стоит задача проектирования Backend-Driven UI-системы или задача распределённого выполнения запросов, при обработке которых данные собираются из зависимых друг от друга источников.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/cbf/7cb/67c/cbf7cb67c03a23e32b21182647dcdc41.png)

## Разбираем Ozon

Что мы представляем, когда говорим о витрине интернет-магазина? Конечно же, в первую очередь в голову приходят карточки товаров, каталог, поиск, корзина, личный кабинет и так далее. При этом большинство разделов маркетплейса содержат элементы сразу из нескольких предметных областей. Например, главная страница может содержать карточки товаров с кнопкой добавления в корзину, поиск по всем товарам, рекламные баннеры, иконку пользователя, отображающую краткую информацию о нём при наведении, и многое другое. Такая ситуация в одном разделе сайта или мобильного приложения накладывает определённые ограничения на архитектуру системы в целом. 

Предположим, что завтра от бизнеса придёт запрос на масштабное изменение главной страницы. Например, нужно будет сделать полный редизайн для неавторизованных пользователей. Для разработчиков это будет означать, что нужно изменить код компонента системы, который отвечает за раздел главной страницы. В случае с небольшим сервисом подход деления на разделы сработал бы, но в разработке Ozon участвуют порядка 2000 человек из сотни доменных команд, каждая из которых отвечает за свою предметную область. Поэтому любые изменения кода раздела будут требовать согласования с другими участниками, а это усложнит и замедлит разработку новых фич. Становится очевидно, что простое деление на разделы нам не подходит из-за слишком больших трудозатрат, которые потребуются для согласования изменений. 

Напрашивается применение принципа «разделяй и властвуй» по отношению к самим разделам. Разделим одну сущность — страницу, которая находится на пересечении зон ответственности разных доменов, — на небольшие сущности — виджеты, каждый из которых находится в зоне ответственности только одного домена. Конечно же, сама сущность страницы полностью не исчезает, так как нам всё ещё нужно где-то хранить структуру виджетов и их настройки. Таким способом мы только отделили сущность страницы от логики доменных команд. Разделение страницы на виджеты — это первый этап деления маркетплейса на кубики.

Теперь попробуем разобраться, что представляет собой виджет. У любого виджета есть визуальное представление — то, что мы видим, взаимодействуя с приложением или сайтом, и состояние — данные, которые нужны для отображения, но независимые от клиента. 

Например, у нас есть виджет синей кнопки с текстом «Добавить в корзину». В этом случае информация о тексте и цвете будет состоянием виджета, а готовая HTML-разметка кнопки для вставки на сайт — его визуальным представлением.

Если вспомнить стандартное разделение систем на бэкенд и фронтенд, то мы приходим к тому, что компонент системы, отвечающий за вычисление состояния виджета, — это его бэкенд, а компонент, отвечающий за вычисление его визуального представления, — фронтенд.

В дальнейшем сервис, который отвечает за логику обработки виджета, мы будем называть **виджет-провайдером**, процесс вычисления состояния виджета — **процессингом**, а процесс получения его визуального представления — **рендерингом**.

На этом этапе мы разобрали страницу на несколько десятков виджетов. Теперь, если к нам придёт запрос главной страницы от браузера пользователя, каждый виджет-провайдер должен будет вычислить состояния своих виджетов, которые впоследствии будут отданы на рендеринг для получения итогового HTML-документа. При таком подходе каждому виджет-провайдеру для процессинга нужны данные. Например, если мы хотим отобразить виджет профиля пользователя с его именем, то само имя нужно откуда-то взять. Если виджет выглядит по-разному для пользователей из разных городов, то при процессинге нужно получить локацию.

При таком подходе первые проблемы появляются, когда одни и те же данные нужны разным виджетам. Предположим, что пять виджетов из разных доменов на главной странице захотели использовать выбранную пользователем валюту в логике своего отображения. Так как за процессинг каждого виджета отвечает отдельный сервис, в сервис валют придут пять запросов. Для небольших систем это может быть приемлемо, но в случае с крупным маркетплейсом, где только входящий пользовательский трафик достигает нескольких сотен тысяч запросов в секунду, команда домена валюты будет крайне недовольна таким подходом к проектированию архитектуры. Плюс перед тем как получить валюту пользователя, нужно получить данные о самом пользователе от сервиса авторизации, а потом получить выбранную пользователем локацию от сервиса адресов. В итоге хайлоад из миллионов RPS между сервисами обеспечен.

Сразу возникает вопрос: если валюта пользователя нужна сразу нескольким виджетам на одной странице, то зачем получать её в каждом виджет-провайдере? Почему бы не вычислить эти данные один раз и не растиражировать всем потребителям в рамках запроса одной страницы? Всего лишь нужен сервис, который будет координировать все запросы данных от виджет-провайдеров. Далее мы будем называть его **core-сервисом**. Сервисы же, которые поставляют данные, мы будем называть **resolve-провайдерами**. Остаётся только понять, какие данные каким виджетам нужны и кто их может предоставить.

Итак, вот какие основные требования к системе накопились у нас к этому моменту:

> 1. Виджет-провайдеры как компоненты системы могут добавляться, изменяться и удаляться по причинам, не зависящим от core-сервиса, так как находятся в зоне ответственности своих доменных команд.
>     
> 2. Виджеты для процессинга могут запрашивать необходимые им данные у поставщиков.
>     
> 3. Ядро системы отвечает за получение запрошенных данных от источников и тиражирует их всем потребителям.
>     

Для удовлетворения всем этим требованиям в основу системы лёг контракт — описание того, какому интерфейсу должен соответствовать каждый компонент системы при его интеграции. Контракт известен всем участникам процесса, в том числе и core-сервису. Для описания контракта мы решили использовать Google Protocol Buffers ([protobuf](https://protobuf.dev/)). В нашем случае получился обычный proto-файл — и сгенерированные из него proto-структуры, которые интегрируются в каждый компонент системы как отдельные модули. Теперь если core-сервис имеет список интегрированных в систему сервисов-провайдеров, то может обращаться к ним по gRPC ([фреймворк удалённого вызова процедур](https://grpc.io/), который работает поверх protobuf), ничего не зная об их внутренней логике.

Вернёмся к нашему делению маркетплейса на кубики. Основной задачей было понять, как разделять абстрактные данные так, чтобы с ними было удобно работать. В первой итерации было принято, что элемент деления — сам запрос данных у сервиса. Список доступных resolve-провайдеров был указан в контракте ядра, а результат вычисления данных записывался в отдельное поле универсальной структуры запроса. Виджет-провайдеры при регистрации в системе указывали в контракте, какие resolve-провайдеры нужно вызвать для правильной работы виджетов, которые они предоставляют. Core-сервис при этом хранил информацию о том, в какой последовательности нужно вызывать resolve-провайдеры. 

В целом такой подход оказался рабочим. Виджет-провайдеры получали запрошенные данные для вычисления состояний виджетов, а нагрузка на поставщиков данных не росла в геометрической прогрессии из-за количества виджетов на странице. Казалось бы, самая сложная проблема решена и мы получили высокую скорость разработки виджетов. Но внезапно поддержка и увеличение количества источников данных усложнились в разы. Любое изменение в поставляемых данных требовало обновления контракта, ядра системы, а также потребителей, которые это изменение запросили. А необходимость менять поставляемые данные для виджетов возникала регулярно. Кубики, на которые мы разделили систему, оказались всё ещё слишком большими, из-за чего их поддержка и разработка были слишком дорогими.

Поэтому к уже существующим требованиям мы добавили следующие:

> 4. Resolve-провайдеры, как и виджет-провайдеры, могут добавляться, изменяться и удаляться по причинам, не зависящим от core-сервиса системы.
>     
> 5. Сами данные абстрактны для core-сервиса системы. Ядро не знает, что за информация содержится внутри данных, но знает обо всех зависимостях между resolve-провайдерами, которые эти данные поставляют.
>     
> 6. Изменение потребностей resolve-провайдеров не требует изменений в ядре системы.
>     

Следующей попыткой абстрагировать данные от контракта и ядра стало решение принять за минимальный кубик отдельную сущность — параметр. Параметр в системе имеет своё имя, тип и значение. Для вычисления параметра используется его описание — информация о том, какой поставщик данных его вычисляет и значения каких параметров для этого нужны. Виджеты, соответственно, в своём описании также могут запрашивать любые параметры, зарегистрированные в системе. Core-сервис, зная потребности виджетов и параметров, умеет вычислять их и тиражировать потребителям через контракт, в котором указана только универсальная структура хранения данных без конкретных полей.

Тут стоит уточнить, что параметры могут быть статическими — парситься из входящего HTTP-запроса (например, query-параметры) и динамическими — вычисляться resolve-провайдерами (например, user_id), но структура и контракт параметров — одинаковые.

Данное решение нас полностью устроило — и параметр стал минимальным кубиком в нашей системе кубиков, из которых мы собираем ответы пользователю. Сам по себе формат передачи данных в параметрах не накладывает на поставщиков данных ограничения размера параметров. Например, ничего не мешает сервису корзины отдавать информацию о товарах одним параметром с типом JSON, при этом сервис авторизации может сообщать авторизован запрос пользователя или нет через параметр с булевым типом. 

Виджеты отвечают за внешний вид интерфейса, а параметры позволяют получить данные, на основании которых он формируется. Чтобы пользователь мог взаимодействовать с полученным интерфейсом, не хватает ещё одного ключевого кубика — **действия**. Действием мы будем считать любое взаимодействие пользователя с сервером, например добавление товара в корзину. Если говорить совсем простым языком, то действие — это вызов какого-либо метода бэкенда. Так как совершение действия также может требовать данных, предоставляемых сервисами-поставщиками, мы выделим его в отдельную сущность со своей конфигурацией. И, соответственно, у каждого действия будет сервис-поставщик, который его обрабатывает.

Таким образом, в результате декомпозиции мы выделили три элемента нашей системы: параметры, виджеты и действия. Этих трёх сущностей оказалось достаточно, чтобы сформировать витрину маркетплейса; и я думаю, что многие из современных интернет-сервисов также могут быть представлены как их совокупность.

На изображении ниже сайт открыт в режиме отладки.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/951/c45/a0d/951c45a0dafae3b1cafebecdf103a9a3.png)

## Учимся собирать Ozon

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/436/2ba/0d0/4362ba0d08c25d7ea4189cb2ea4f919e.png)

Теперь ядро нашей системы знает, из каких кубиков состоит Ozon. У нас есть информация обо всех виджетах, параметрах и действиях. Также мы знаем, какой сервис в системе какие кубики предоставляет и как эти кубики связаны между собой. 

Нужно определить набор инструкций — правил, по которым ядро будет собирать витрину из заданных элементов. И далее я расскажу о том, как задаётся конфигурация для сборки страниц и какие преимущества мы получаем от разбиения системы на небольшие части.

### Как выбрать нужную страницу

Сначала определим, что такое страница и что видит пользователь, когда открывает какую-нибудь страницу Ozon. 

У любой страницы есть два основных элемента: визуальная часть — композиция из виджетов, которую далее мы будем называть **шаблоном**, и правило, по которому этот шаблон выбирается. 

Скорее всего, вы уже задаётесь вопросом: зачем всё так усложнять? У каждой страницы ведь есть адрес, который её определяет. Почему бы просто не использовать URL? Открывает пользователь страницу [https://ozon.ru](https://ozon.ru/)/ — так давайте сразу покажем ему шаблон главной. И так действительно можно было бы делать, если бы Ozon не был такой крупной и вариативной системой. Сейчас у нас есть десятки (если уже не сотни) различных вариантов компоновки главной страницы. Визуальное представление только домашней страницы может кардинально отличаться в зависимости от того, зарегистрирован пользователь или нет, в каком A/B-эксперименте он находится, из какого города открывает сайт и так далее. И эти условия могут меняться по несколько раз в день. 

Конечно же, логика выбора страницы на основе гибких правил добавляет новые требования к ядру системы, так как именно оно определяет, какие виджеты будут отрисовываться на странице:

> 7. Композиция виджетов, которую видит пользователь, определяется набором гибких условий и не требует изменений в коде сервисов при редактировании.
>     
> 8. Параметры, по которым создаются условия, могут в любой момент добавляться и изменяться.
>     

Если попробовать описать условия простым языком, то получится нечто вроде следующего набора правил: 

1. Если A = B1 и C = D1, то выбираем шаблон X.
    
2. Если A = B2, то выбираем шаблон Y.
    
3. Если C = D2, то выбираем шаблон Z.
    

Тут мы уже начинаем получать первые значительные бонусы от того, что система поделена на элементы, одним из которых является параметр. Так как имя параметра и его тип известны заранее, для него не составит труда сформировать условие. А так как параметры в системе могут в любой момент добавляться и удаляться, нужно только определить список тех из них, которые будут доступны для создания условия выбора шаблона.

Не буду вдаваться в детали устройства алгоритма движка выбора правил. Он претерпел множество изменений, прежде чем стал таким, как сейчас. Скажу лишь, что проблему конфликтов правил мы решили, добавив приоритет каждому параметру, а для отладки того, какие правила могут «выиграть», мы используем специальный инструмент.

Для создания и редактирования условий выбора страниц у нас есть специальная админка, которая предоставляет удобный и понятный интерфейс для пользователей, далёких от написания кода. Далее основных пользователей этой админки я буду называть **контент-менеджерами**, так как именно они отвечают за настройку контента, который видит конечный пользователь Ozon на витрине маркетплейса.

### Как скомпоновать виджеты

Поговорим немного о том, как именно контент-менеджеры собирают страницы из виджетов. Я уже упоминал, что разделение маркетплейса на кубики позволяет нам постоянно переиспользовать единожды разработанные элементы. Поэтому разработанный виджет баннера для категорийной страницы может быть в любой момент размещён на странице какой-нибудь акции без изменений в коде. 

Композиция виджетов, или шаблон — это последовательный список выбранных виджетов, которые будут отображены на странице, и заданные настройки для каждого из них. Некоторые виджеты могут являться контейнерами для других, поэтому шаблон также можно представить в виде древовидной структуры. 

Пользователю админки при создании или изменении шаблона доступен список зарегистрированных в системе виджетов. К нему также применяются фильтры по платформе и уровню доступа пользователя. В описании виджета также содержится список настроек каждого виджета, которые можно изменить, например значение текстового поля или высота. 

Счастье было бы неполным, если бы, помимо доменных виджетов, у нас не было ещё и системных, которые позволяют гибко настраивать логику отображения различных частей шаблона. Такие виджеты предоставляются ядром системы, которое отвечает за их обработку. Этот подход иногда ещё сравнивают с программированием на шаблоне. 

Один из самых частоиспользуемых системных виджетов — Condition (виджет-условие). Он, как вы уже, наверное, догадались, позволяет в зависимости от выполнения условия скрывать определенную группу виджетов. Параметрами для условия выступают те же сущности, что и при создании правил определения страниц, что позволяет настраивать скрытие виджетов максимально гибко. Например, можно показывать кнопку только авторизованным пользователям или показывать рекламный баннер в определённое время, меняя конфигурацию шаблона без изменения кода сервисов.

Есть ещё несколько интересных системных виджетов. TryCatch позволяет менять одни виджеты на другие, если в результате обработки шаблона произошла ошибка. Together скрывает сразу всю пачку виджетов, если хотя бы один из них не получилось обработать. Они могут использоваться, например, в тех случаях, когда несколько виджетов представляют ценность для пользователя, только если отображаются вместе. И отдельного внимания заслуживает виджет Paginator, благодаря которому мы можем организовать пагинацию любого количества страниц на сайте.

Инструмент редактирования шаблонов, с одной стороны, предоставляет контент-менеджеру возможность быстро внести изменения в отображаемый контент и запустить новую акцию, которая принесёт компании много денег. Но, с другой стороны, случайно удалив из шаблона виджет корзины пользователя, можно сделать неработоспособным весь маркетплейс. Чтобы этого избежать, все шаблоны версионируются, позволяя пользователю админки с необходимым уровнем доступа совершить откат нажатием одной кнопки. А для публикации изменений на особо важных страницах предусмотрен отдельный процесс ревью и получения подтверждений от ответственных лиц.

Таким образом, к нашим кубикам системы — действиям, виджетам и параметрам — добавляются шаблоны и настройки страниц — инструкции, в которых указано, как именно из кубиков ядро системы должно собирать страницы при запросах пользователя.

Ниже приведён пример части настроенного шаблона главной страницы.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/78e/1e7/ae3/78e1e7ae352050c644e76e9898708915.png)

## Собираем Ozon

В этой части статьи я расскажу, как именно ядро системы собирает страницы, которые видит пользователь, из кубиков, разработанных доменными командами, в соответствии с инструкциями, настроенными в админке контент-менеджерами.

### Парсим входящий запрос

Входными данными для ядра системы является HTTP-запрос, который пришёл из браузера или мобильного приложения пользвоателя. Но базовым элементом, на основании которого строится страница, является параметр. Поэтому и HTTP-запрос тоже должен быть преобразован в набор параметров. Например, запрос страницы [https://www.ozon.ru/my/favorites?category=9200](https://www.ozon.ru/my/favorites?category=9200) будет преобразован в следующий набор параметров:

- “request.method” = “GET”
    
- “page.type” = “account”,
    
- “account.page” = “favorites”,
    
- “request.url.query.category” = 9200
    
- и так далее. 
    

Для cookie и заголовков — аналогичным образом. Суть этого этапа состоит в том, чтобы перейти к работе только с параметрами, абстрагировавшись от конкретных значений запроса.

Для разных URL могут потребоваться разные настройки парсинга, поэтому за их конфигурацию отвечает наша специальная внутренняя админка. Её настройки для примера выше могут выглядеть следующим образом:

- Для Host = “*” и Path = “/my/*”
    
    - положить конкретное значение “account” в параметр “page.type”,
        
    - взять значение из Path по ключу “*” и положить в параметр “account.page”,
        
    - взять значение из Query по ключу “category” и положить в параметр “request.url.query.category”.
        
- Взять значение из заголовка “User-Agent” и положить в параметр “request.header.user_agent”.
    
- И так далее.
    

Я думаю, вы уже примерно поняли суть. Любой заполненный на этом этапе параметр можно использовать для создания настроек страниц, для условий в системном виджете Condition, а также запросить в сервисе-поставщике для обработки другой сущности (действия, виджета или другого параметра).

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/ed9/8c2/ba8/ed98c2ba85f768ac757c0d4350b74e4e.png)

### Строим граф микросервисов

Переходим к одному из самых интересных этапов сборки страниц — построению графа микросервисов. Параметры и их зависимости представляют собой граф, в котором параметры — это вершины, а зависимости между параметрами — рёбра. Сам граф всех параметров статичен — он меняется только тогда, когда меняются потребности в конфигурации какого-нибудь параметра. При этом ядро всегда должно иметь максимально быстрый доступ к этому графу, то есть хранить его у себя в памяти.

Виджет-провайдеры должны получить все параметры, которые они запросили, чтобы вычислить состояния виджетов на выбранной ядром странице. При обработке запроса пользователя core-сервис обходит граф параметров сверху вниз, начиная с параметров, запрошенных виджетами, и строит граф сервисов, которые нужно вызвать для разрешения всех зависимостей.

Ниже приведён пример графа сервисов для запроса страницы.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/751/524/5d4/7515245d48452da3b2540600979c0c43.png)

Разумеется, мы хотим, чтобы вызывались только те сервисы, данные которых нужны для отрисовки страницы. Было бы странно и неэффективно вызывать сервис, данные которого необходимы только для отображения карточки товара, при открытии главной страницы. Но возникает вопрос: как тогда использовать правила выбора шаблона на основе параметров внешних сервисов? Если мы хотим, чтобы главная страница выглядела по-разному для пользователей из Москвы и Санкт-Петербурга, то что будет являться причиной вызова сервиса адресов? Ведь на этапе выбора страницы мы ещё не знаем, какие виджеты будут отрисовываться и, соответственно, их потребности.

Если мы попробуем сначала вычислить все параметры, которые используются в правилах, то рискуем опять прийти к тому, что при открытии главной странице будет вызван какой-нибудь тяжёлый сервис, данные которого нужны только при открытии личного кабинета. При этом, если сначала мы попробуем вычислить значение параметра с названием города для выбора страницы, а потом виджету на странице потребуется название улицы, это приведёт к двойному запросу к сервису адресов, что крайне нежелательно при наших нагрузках. Значит, к требованиям к core-сервису добавляются новые:

> 9. Сервис — поставщик данных не должен вызываться при открытии страницы, если его данные на ней не нужны.
>     
> 10. Сервис — поставщик данных не должен вызываться при открытии страницы больше одного раза.
>     

В первой итерации проектирования мы решили, что сначала нужно сгруппировать правила выбора страницы по неким признакам. Главное условие для объединения правил в группу — это участие тяжёлых сервисов в выборе страницы. Также правила должны группироваться на основании статической информации — той, которую мы уже распарсили из пришедшего HTTP-запроса. Этими признаками стали:

1. Тип страницы — что-то вроде префикса пути (path) запроса, который мы вычисляем с помощью регулярного выражения. Таким образом, всё, что имеет path — “/”, будет иметь тип страницы — «главная», “/product/*” — «карточка товара», “/my/*” — «личный кабинет» и так далее.
    
2. Платформа — тип клиента, от которого пришёл запрос. Мы разделили запросы от браузера, мобильного браузера и мобильного приложения.
    

Теперь, если у нас есть список шаблонов, которые могут быть выбраны в рамках конкретной группы страниц, мы можем без труда составить один общий список потребностей виджетов и параметров, объединив его со списком параметрами, которые нужны для выбора страницы. Даже не нужно формировать граф для каждого запроса пользователя — этим может заниматься отдельный фоновый процесс.

Это решение позволило нам сформировать отдельный граф сервисов-поставщиков для каждой комбинации типа страницы и платформы. Таким образом, при получении на вход запроса главной страницы от браузера, обход графа не требовал вызова тяжёлого сервиса, необходимого для запроса карточки товара из мобильного приложения. Но для полного соблюдения второго требования об однократном вызове сервиса-поставщика пришлось пойти на компромисс. Сервис всё ещё мог вызываться впустую, если в результате выбора страницы был выбран шаблон, в котором не было потребителей данных этого сервиса. И это стало главным недостатком нашего первого подхода к формированию графов, так как таких вызовов было больше десятков тысяч в секунду.

Во второй итерации мы попробовали вызывать сначала те сервисы, данные которых нужны для выбора страницы, а затем — сервисы, данные которых нужны виджетам в выбранном шаблоне, то есть разделили граф на две фазы обработки запроса: выбор страницы и обработка шаблона. Но проблема заключалась в том, чтобы на первой фазе запрашивать у сервисов те данные, которые могут потребоваться сервисам во время второй, и при этом учесть все транзитивные зависимости между параметрами. При таком подходе алгоритм формирования графа всё ещё получался слишком сложным и запутанным.

Нашей ошибкой была попытка решения проблемы путём формирования графа с нуля, забыв, что на самом деле полный граф сервисов с учётом всех зависимостей между параметрами у нас уже был сформирован в первой итерации. Нужно было всего лишь спуститься по готовому графу, удалив те вершины, которые не нужны для выбора страницы. Получившийся после этого граф стал графом для первой фазы обработки запроса. В нём были учтены все вызовы сервисов и все параметры, которые могли бы понадобиться сервисам во время второй фазы. Граф для второй фазы сформировать было ещё проще: удалить из готового графа те вершины, которые мы обошли во время первой фазы. 

На изображении ниже продемонстрировано деление графа resolve-провайдеров на две фазы (отделены вертикальной чертой).

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/684/2ee/e03/6842eee0376375beb77110c996875764.png)

Скорее всего, алгоритм получился не оптимальным, но нам были важны в первую очередь простота и понятность. А так как формированием графов занимается фоновый процесс, конечный пользователь не ждёт, пока ядро сформирует граф для его запроса. 

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/030/e4a/226/030e4a226fe368c12f926e72e2e8af83.png)

### Обрабатываем и рисуем виджеты

После парсинга входящего запроса, построения графа и вызова всех сервисов — поставщиков данных ядро имеет весь необходимый набор параметров для того, чтобы приступить к процессингу виджетов — получению их состояний для последующего рендеринга. Перед этим должны быть обработаны все системные виджеты Condition — чтобы не процессить те виджеты, которые будут скрыты в итоговом шаблоне.

Из полезных фич этого этапа стоит выделить передачу <head> части HTML-страницы браузеру до того, как начнётся обработка виджетов для формирования <body>. Тут нам ничего не мешает заранее сообщить клиенту скрипты, которые будут использоваться, и начать их загрузку, так как список виджетов уже известен. Такое несложное действие позволит ускорить получение клиентом первого байта от сервера на пару десятков миллисекунд.

На процессинг виджетов требуется разное время. Есть обязательные виджеты, без которых страница теряет ценность для пользователя, например виджет с ценой в карточке товара. И есть виджеты, которые по каким-то причинам не могут быть асинхронными, но которыми мы можем пренебречь, например рекламные баннеры. Поэтому мы решили разделить виджеты на три группы по степени их важности: must_have, nice_to_have и optional. У каждой группы виджетов на странице есть свой тайм-аут. Благодаря такому механизму загрузку виджета с названием товара мы будем ждать даже семь секунд, а рекламный баннер будет скрыт, если не успеет запроцесситься за две секунды (при условии, что остальные виджеты уже загрузились).

Процессинг виджетов от разных сервисов-поставщиков выполняется параллельно, что позволяет ускорить ответ пользователю, а разделение по тайм-аутам делается с помощью gRPC-стримов. Исключение составляют только случаи, когда требуется сделать несколько итераций для виджета TryCatch. 

Финальным этапом построения страницы является рендеринг — превращение данных о состоянии виджетов в разметку и красивую картинку, которую увидит пользователь. В случае с мобильным приложением вся магия отрисовки происходит внутри него, поэтому на выходе бэкенд отдаёт данные в формате JSON. С браузером дела обстоят чуть интереснее, потому что рендеринг происходит на стороне сервера параллельно для каждого виджета. При этом в случае деградации запроса мы можем перейти на рендеринг на стороне клиента.

Остаётся только дополнить страницу асинхронными виджетами и добавить действия всем кнопкам. Обработка виджетов и действий выполняется примерно одинаково. В обоих случаях клиент на основании информации из переданного ему шаблона принимает решение о том, чтобы сделать запрос к бэкенду. Core-сервис понимает, какую сущность у него запросили, выполняет обработку графа запрошенных сущностью параметров и возвращает ответ сервиса-поставщика клиенту.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/9dd/6cb/812/9dd6cb81293a49427159c433bf5013b5.png)

## Итоги

Подведём итоги. Используя такой подход, у нас получилось объединить в одну упорядоченную систему порядка 150 сервисов от 70 доменных команд (и с ростом Ozon их число постоянно увеличивается). Количество шаблонов витрины уже превысило 50 000, и для создания новых страниц контент-менеджерам доступно более 3000 виджетов. 

Сервисы взаимодействуют друг с другом только согласно строго зафиксированному контракту. Поэтому любой новый виджет может наиболее эффективным образом получить доступ к 700 статическим и 500 динамическим параметрам от поставщиков данных. А интеграция виджета в страницы сайта и приложения не требует изменения кода и занимает считанные часы, большая часть которых уходит на согласование с владельцами страниц.

Разработанный нами продукт решает две ключевые задачи: организация эффективного взаимодействия между сервисами доменных команд и формирование BDUI-страниц без изменений в коде клиента и сервера. 

Преимущества, которые мы получили от внедрения такого подхода:

1. Высокая скорость разработки и внедрения новых фич. Минимум времени на согласование и разрешение конфликтов между доменными командами.
    
2. Высокая эффективность обработки запросов в высоконагруженной системе. Практически нет ненужных запросов одних и тех же данных, а вызовы независимых друг от друга сервисов выполняются параллельно. Плюс единая точка обработки запросов позволяет внедрять общие для всех оптимизации, такие как раздельная отдача <head> и <body> или кеширование параметров.
    
3. Простота получения общих данных на бэкенде. Для потребителя данных не имеет значения, как будет вычислена выбранная валюта на сайте и тем более что для её вычисления нужно сначала узнать локацию пользователя, которой вдобавок требуются данные о самом пользователе. Ядро системы позволяет скрыть обход графа параметров за простым интерфейсом указания зависимости от конкретного параметра.
    
4. Возможность изменения контента на страницах без изменения кода. Помимо того что нам не нужно менять код клиента благодаря BDUI, нам ещё и необязательно менять код на стороне провайдера. Контентом и компоновкой страниц управляют из одного места — специальной админки, которая спроектирована для не погруженных в технические детали пользователей.
    

Я бы ещё очень долго и с радостью мог рассказывать о таких крутых фичах ядра, как:

1. Кэширование параметров, которое экономит нам миллионы RPS. 
    
2. Трекинговая аналитика, которая проходит через ядро и позволяет анализировать действия пользователя на сайте. 
    
3. Наш кластер ClickHouse, который позволяет собирать Time Series-метрики сущностей системы и получает на вход десятки миллионов событий в секунду. 
    
4. Ленивый маршалинг, который экономит нам сотни ядер на сериализации и десериализации данных. 
    
5. Инструмент анализа графа всех сущностей с сравнением изменений по времени.
    

Но давайте оставим эти темы на будущее, так как редактор посоветовал мне ограничиться 20-ю страницами.

Думаю, что с ростом популярности BDUI и микросервисной архитектуры такая задача, как динамический сбор данных из разных зависящих друг от друга источников, будет встречаться всё чаще. Сейчас, когда наша система насчитывает тысячи интегрированных сущностей, уже очень сложно вносить изменения в какие-то базовые принципы. Поэтому хочется порекомендовать разработчикам, столкнувшимся с подобной задачей, следующее:

1. Как ни странно, документация. Если участники системы интегрируют свои сервисы независимо от вас, обязывайте их чётко документировать все предоставляемые ими сущности. Если ваша система разделена на небольшие кусочки, которые могут быть использованы кем-то ещё, они должны быть хорошо задокументированы, иначе новый участник системы при интеграции своего сервиса не сможет найти нужную ему часть. Спустя несколько лет существования системы сегодня нам приходится тратить существенные ресурсы на сбор информации о находящихся в ней данных и их предназначении.
    
2. Кэширование. Учитывайте кеширование ответов от участников системы на первых этапах проектирования. Стремитесь к тому, чтобы запросы к сервисам содержали наименее вариативные данные — и только те, что нужны для обработки. Конечно, это только звучит просто и на практике очень маленькие кубики сделать не получится. Но если вы разрешите всем запрашивать сырой URL запроса, который частенько содержит рандомно сгенерированный GET-параметр, то кэширование можно сразу будет иметь плохой хит-рейт. Также лучше сделать кэширование обязательной, а не опциональной фичей — тогда участники системы будут сразу учитывать это ограничение при проектировании своей части.
    
3. Чётко разделяйте бизнес-логику и логику сервиса-платформы. Если вы добавите даже один маленький if-чик для какой-нибудь бизнес-фичи в core-сервисе, вы рискуете создать ограничение для всех новых участников системы. Лучше перенести всю бизнес-логику в отдельный сервис, который реализует единый для всех контракт, даже если в моменте это потребует чуть больше трудозатрат и, возможно, будет менее эффективно из-за сетевых вызовов.
    
4. Инструменты отладки. Это именно то, во что вам стоит вкладываться с самого начала. Знать, как устроена система, будете только вы и ваша команда (даже через несколько лет). Если вы не хотите утонуть в бесконечных вопросах о том, почему кому-то не приходят данные или виджет не отображается на странице, предоставьте вашим пользователям нормальные инструменты, с помощью которых они смогут проследить все этапы обработки пользовательского запроса и получить промежуточные данные.
    

Надеюсь, наш опыт будет вам полезен и у вас получится создать ещё более крутую систему, не допустив наших ошибок.