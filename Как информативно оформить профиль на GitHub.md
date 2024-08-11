---
tags:
  - data
source: habr
link: https://habr.com/ru/articles/813399/
data_type:
  - DevOps
---
[[Работаем с Git первые шаги в GitHub]]
[[Мегагайд - культура работы с Git]]

Тетрадь, дневник — ваше лицо. А круто оформленный профиль на гитхабе — статус вашей занятости. Чем больше участий в проектах, тем безработнее... Пока молодые специалисты оформляют свои страницы с "Lib-Meta-Neo ML-Scientist 10 years of expirience" на LinkedIN настоящий амбассадор HR и трудового найма бегут на GitHub. Именно там выискиваются самые закостенелые гики программирования, вносящие тридцать пять тысяч коммитов в безбюджетные опенсорс проекты; именно там рождаются гении, разрабатывающие AAA-проекты геймдева на ассемблере. 

Все это шутки. 

Но реальность такова, что многие из рекрутеров не против оценить ваш профиль. Подавать себя, как в маркетинге, важно. И неплохо бы сразу представить всю статистику развернуто перед глазами, чтобы бедный HR не искал ваши коммиты, а гордо проведенные тысячи часов в GitHub не остались за кадром. Каждый проект служит материальным доказательством способностей разработчика, позволяя потенциальным соавторам или работодателям оценить его стиль программирования, навыки решения задач и умение управлять проектами. 

Для этого на гитхабе есть много утилит, которые помогают оформить Readme личного профиля. Но для начала давайте быстренько его создадим. 

### Как создать личный Readme

1. Войдите в свой аккаунт на GitHub и нажмите кнопку "New" в правом верхнем углу страницы. Введите название для вашего репозитория (например, "username.github.io", где "username" - ваш логин на GitHub). Убедитесь, что галочка "Initialize this repository with a README" отмечена.

2. После создания репозитория вас перекинет на страницу с содержимым вашего нового репозитория. Нажмите на ссылку "README.md", чтобы открыть файл для редактирования.

_PS. В Readme можно использовать как Markdown, так и HTML. Никаких невероятных навыков работы с версткой для создания профиля не нужно._ 

4. После того как вы добавили необходимую информацию, прокрутите вниз страницы и найдите раздел "Commit changes". Введите заголовок коммита и описание изменений, затем нажмите кнопку "Commit new file".

5. Для того чтобы ваш профиль был доступен по URL-адресу вида "username.github.io", включите GitHub Pages для вашего репозитория. Для этого перейдите в раздел настроек вашего репозитория, выберите раздел "Pages" и активируйте опцию "Source" для вашей ветки "main" или "master".

После активации GitHub Pages профиль будет доступен по URL-адресу "username.github.io". 

Все готово для залива нашей невероятной статистики. 

Но прежде можно попробовать провести самую базовую настройку профиля и создадим простой заголовок профиля на HTML. 

|   |
|---|
|`<html lang="en">   <head>       <meta charset="UTF-8">       <meta name="viewport" content="width=device-width, initial-scale=1.0">       <title>GitHub Profile Header</title>       <link rel="stylesheet" href="styles.css">   </head>   <body>      <div class="header">       <h1>My GitHub Profile</h1>       <p> Ya krutoy specialist !</p>   </div>      </body>   </html>`|

В этом примере мы создаем блок заголовка с классом .header. Он имеет фоновый цвет 24292e (стандартный цвет для заголовков на GitHub), белый цвет текста, внутренние отступы в 20 пикселей и выравнивание текста по центру.

Конечно, у многих из нас "Ya krutoy specialist!" вызовет шквал эмоций и желаний моментально нанять специалиста в свой штат, определив заранее лидом всего отдела. Но лучше показать как можно больше статистики для "убедительности". Например, продемонстрировать свою эффективность и тысячи часов, проведенных на Github. 

### У меня нет времени на это все. Хочу сразу и быстро

Окей, тогда для самых занятых небольшая подборка инструментов с созданием профиля по шаблону. 

**Три плагина для шаблонного профиля и даже обычных репозиториев**

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/58b/952/1a7/58b9521a7dbe6a859ac93ac81ce59381.png)

1. [**GitHub Profile README Generator**](https://github.com/abhisheknaiidu/awesome-github-profile-readme)

Тут перед нами относительно простой шаблон, куда можно вписать немного информации о себе и получить статистику с минимальной активностью. 

_Склонируйте репозиторий:_

|                                                                                |
| ------------------------------------------------------------------------------ |
| `git clone https://github.com/rahuldkjain/github-profile-readme-generator.git` |

Перейдите в рабочий каталог:

|   |
|---|
|`cd github-profile-readme-generator`|

Установите зависимости:

|   |
|---|
|`npm install`|

Запустите приложение:

|   |
|---|
|`npm start`|

2. [Awesome README](https://github.com/matiassingers/awesome-readme)

Этот плагин предлагает различные шаблоны и примеры для создания привлекательного и информативного README файла. Вы можете выбрать подходящий шаблон и настроить его под свои нужды. 

|                                         |
| --------------------------------------- |
| `npm install --save-dev awesome-readme` |

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/007/a65/4fb/007a654fb9a78644428ce195b60ffd88.png)

3. [Readme.so](http://readme.so/)

Это веб-приложение, которое позволяет создавать README файлы с помощью визуального редактора. Вы можете выбирать различные секции, такие как описание, установка, использование, лицензия и т.д., и настраивать их содержимое. Результат можно скопировать в ваш README файл.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/a22/a11/deb/a22a11deb1ec014d10925338d17bd5ed.png)

4. [Profileme](https://www.profileme.dev/) Это бесплатный помощник для создания красивых GitHub-профилей. В нем можно выбрать один из более 60 языков оформления и настроить различные элементы. 

|   |
|---|
|`npm install --save-dev readme-generator`|

### Статистика в профиле – признак ума

Общий вид добавления любого плагина в Гитхаб-readme. 

|   |
|---|
|`[![название плагина](ссылка на плагин/?user=сюда вписываем юзернейм)](ссылка на репозиторий плагина)`|

Пример:

`[![GitHub Streak](https://streak-stats.demolab.com/?user=DenverCoder1)](https://git.io/streak-stats)`

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/d41/dfc/5fe/d41dfc5fe033a919dbf8b145991eb9f6.png)

1. [**GitHub Readme Activity Graph**.](https://github.com/Ashutosh00710/github-readme-activity-graph) Он отображает информацию о количестве коммитов, сделанных пользователем в его репозиториях на GitHub за определённый период времени. Цвет ячейки указывает на количество коммитов за  день: яркие зелёные ячейки обозначают дни с большим количеством коммитов, тогда как более тёмные ячейки указывают на менее активные дни. Можно поиграть с графиком, наведя курсор мыши на конкретную ячейку, чтобы увидеть подробную информацию о количестве коммитов за этот день. 

Добавляем по шаблону выше… 

|   |
|---|
|`markdown   ### 📈 GitHub Activity Graph:   ![Anurag's GitHub activity graph](https://activity-graph.herokuapp.com/graph?user`|

2. [**Metrics.**](https://github.com/lowlighter/metrics) Изометрический календарь ваших коммитов, языковой, географический анализ профиля, отслеживание "кодерских" привычек, отображение лицензий репозитория, реакции и комменты, подписчики, выкладка с донатами, дискуссии, ачивки, последняя активность, рандомный код из ваших репозиториев, список проектов, список топ треков со Spotify, время подъема; статистика с Leetcode, Steam, шахмат и даже, господи, город из вашей активности — все в этом репозитории.  Более чем 30 различных плагинов и 200 параметров для настройки динамической инфографики. 

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/55f/39d/8fc/55f39d8fcef1eff27e2c4b1a58391928.png)

3.  [GitHub Stats Card](https://github.com/anuraghazra/github-readme-stats?tab=readme-ov-file)

Этот плагин позволяет отображать основную статистику профиля, включая количество репозиториев, звезд, подписчиков и т.д. 

|   |
|---|
|`markdown   ![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=anuraghazra&show_icons=true&theme=radical)`|

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/2d9/dce/970/2d9dce9701f5d3f98bf21227007e8706.png)

4. [Top Languages Card](https://github.com/anuraghazra/github-readme-stats)

Этот плагин показывает топ языков программирования, на которых вы пишете код. 

|   |
|---|
|`markdown   [![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=anuraghazra&layout=compact)](https://github.com/anuraghazra/github-readme-stats)`|

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/227/97b/868/22797b868dba6918b66e3821e849dda9.png)

5.  [GitHub Streak Stats](https://git.io/streak-stats)

Этот плагин отображает ваш текущий стрик коммитов. 

|   |
|---|
|`markdown   ### 🔥 GitHub Streak Stats   [![GitHub Streak](https://github-readme-streak-stats.herokuapp.com/?user=anuraghazra&theme=dark)](https://git.io/streak-stats)`|

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/1e6/5c8/d2e/1e65c8d2e629835c2d3243f94de5e0d8.png)

6.  [GitHub Trophy](https://github.com/ryo-ma/github-profile-trophy)

Этот плагин показывает ваши достижения в виде трофеев. 

|   |
|---|
|`markdown   [![trophy](https://github-profile-trophy.vercel.app/?username=anuraghazra&theme=onedark)](https://github.com/ryo-ma/github-profile-trophy)`|

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/fa0/f22/748/fa0f22748d6f421dcccd25c652ee8829.png)

7. [Codewars](https://github.com/estraviz/codewars) — виджет, показывающий количество решенных задач и актуальный уровень пользователя на платформе. Ссылки на карточки можно найти в своем профиле, нажав на кнопку «View Profile Badges» под аватаркой. Кастомизации нормальной нет, но есть три варианта размеров виджета.

|   |
|---|
|`[![codewars](https://www.codewars.com/users/username/badges/large)](https://www.codewars.com/users/username)`|

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/b37/203/003/b37203003376ddeb008020edc2cbb26d.png)

  

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/89c/022/495/89c02249517ffcf5785acca6e543dfda.png)

8. [LeetCode Readme Stats](https://github.com/JacobLinCool/LeetCode-Stats-Card) Все, наверное, слышали про литкод, амбассадор топовых задачек для программистов, так сказать. Ну так вот с LeetCode Readme Stats можно выводить данные о количестве решенных задач с уровнями сложности. Также у автора имеется инструкция по настройке отслеживания статистики с помощью GitHub Actions. 

Но любой профиль украсит интересное изображение, гифка или анимация

**Топ инструментов, чтобы украсить ваш профиль:**

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/907/61d/0c4/90761d0c4f80cbaacc98864a86e48fb8.png)

1. [itHub Skyline 3D](https://skyline.github.com/)

Этот инструмент генерирует 3D-визуализацию ваших вкладов в GitHub, создавая уникальный и привлекательный элемент профиля. Достаточно зайти на сайт и сгенерировать отчеты в .stl и загрузить на GitHub. Каждое "здание" в 3D-модели соответствует ежедневным вкладам пользователя.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/1ef/9fe/092/1ef9fe0920276fe854e8c08367e06f99.png)

2. [GitHub Profile Views Counter](https://github.com/antonkomarev/github-profile-views-counter)

Плагин добавляет счетчик посетителей на ваш профиль GitHub, позволяя вам отслеживать вовлеченность с вашей страницей.

|   |
|---|
|`![](https://komarev.com/ghpvc/?username=your-github-username&color=green)`|

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c10/e24/af6/c10e24af6fce4a9650a1a38b6495e534.png)

3. [GitHub Profile Summary Cards](https://profile-summary-for-github.com/)

Этот инструмент генерирует набор карточек-сводок, которые предоставляют обзор активности, включая метрики по репозиториям, звездам, подписчикам и другим характеристикам.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/668/55d/b46/66855db46de0282c80adc31fe598a6da.png)

4. [GitHub Readme Quotes.](https://github.com/PiyushSuthar/github-readme-quotes) Этот плагин добавляет случайную вдохновляющую или мотивирующую цитату в ваш README-файл GitHub. “Жжем сердца документацией!”

|   |
|---|
|`![Quotes](https://quotes-github-readme.vercel.app/api?type=horizontal&theme=dark)`|

Пользуемся. В любом случае никто не запрещает создать самый серый профиль и забыть про этот ваш опенсорс сервис, открыть для себя настоящие нишевые проекты за деньги. Мы привели список интересных плагинов, которые помогут сделать ваш профиль информативнее, привлекательнее и профессиональнее. Челюсть у HR не отпадет, но он будет знать – вы прекрасно справились с задачками “Easy” на литкоде и Markdown-разметкой.