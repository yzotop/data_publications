---
tags:
  - data
data_type:
  - AB tests
link: https://habr.com/ru/articles/781060/
author: Yuldashev
source: habr
---
Задача оценки нововведений в онлайн и мобильных приложениях возникает повсеместно. Один из наиболее надёжных и популярных способов решения этой задачи - двойной слепой рандомизированный эксперимент, также известный как [АБ-тест](https://en.wikipedia.org/wiki/A%2FB_testing).

На тему АБ-тестирования доступны как статьи на Хабре, так и целые книги (неполный список литературы в конце). В основе АБ-теста лежит следующая идея - случайно разделить пользователей на две или более группы, в одной из которых исследуемая функциональность выключена, а в других - включена. Затем можно сравнить метрики и сделать выводы.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/373/19d/af1/37319daf15238b1cc8ee7bd11c0d1279.png)

Основной математический инструментарий, применяемый в A/B-тестировании, уже реализован в стандартных библиотеках, таких как SciPy, Statsmodels, Scikit-learn, Pandas и других. Кроме того, существуют готовые коробочные решения, включая открытый исходный код, которые объединяют возможности включения и отключения функциональности для различных пользователей, расчёт метрик и вывод результатов статистических критериев.

Это приводит к естественному желанию полностью автоматизировать процесс дизайна, сопровождения и анализа A/B-тестов, чтобы снизить нагрузку на аналитиков компании.

### Полная автоматизация

Зачем каждый раз после окончания очередного АБ-теста писать новый Jupyter ноутбук, если часть кода, связанная с вычислением p-value и обработкой выбросов, повторяется из раза в раз? Если описать все часто используемые метрики, то необязательно каждый раз скачивать данные и запускать Т-тест или бутстрап для подсчета p-value. Разумно проводить эти расчёты автоматически сразу для всех метрик и тестов. После тщательной проверки на исторических данных, синтетических данных и в АА тестах можно организовать небольшой сервис для их мониторинга и запуска.

Тогда роль аналитика практически полностью исключается, особенно если выдать доступы на мониторинг и запуск заказчикам, например менеджерам. Таким образом, заказчики после реализации новой идеи могут самостоятельно создать АБ тест, разбить пользователей на желаемые группы и самостоятельно визуально убедиться, что новая фича "прокрасила" часть метрик - обнаружена статистическая значимость.

Из статистической значимости набора метрик можно сделать вывод об успешности идеи и её реализации. Таким образом время аналитика можно потратить на более важные задачи.

Однако, непродуманная автоматизация может привести к неожиданным проблемам.

### Проблемы аналитиков

Первая проблема, с которой приходится сталкиваться, - это рост числа параллельно выполняемых тестов. Не все заказчики знают, что происходит в соседних командах, и тесты начинают пересекаться и влиять друг на друга. С этим можно справиться достаточно просто, если ввести визуальный контроль за выполняемыми тестами и дополнительные инструкции. Например, при создании нового теста необходимо убедиться, что он не влияет на уже выполняющиеся, а общее количество параллельно выполняемых тестов не должно превышать утвержденной величины, например, 20.

Каждая новая функциональность имеет свою специфику и добавляет свои метрики. С какого-то момента количество метрик и данных становится настолько велико, что вычисление доверительных интервалов и p-value бутстрапированием становится невыносимо долгим. В таких случаях приходится переходить к аккуратному применению [Пуассоновского бутстрапирования](http://www.med.mcgill.ca/epidemiology/hanley/reprints/bootstrap-hanley-macgibbon2006.pdf), [t-тест](https://en.wikipedia.org/wiki/Student's_t-test), [дельта метода](https://habr.com/ru/companies/X5Tech/articles/740476) и других продвинутых подходов. Код становится сложным инженерным решением, требующим комбинацию хороших навыков разработчика, дата инженера и аналитика. Процесс добавления новых метрик усложняется, возрастает нагрузка на команду девопсов.

### Проблемы заказчиков

Давайте рассмотрим А/Б-тестирование с точки зрения заказчиков и тех, кто управляет процессом создания новых фичей. Обычно на таких должностях работают люди с хорошими софт скилами и объяснить, почему нужен доступ в систему управления и мониторинга АБ тестов большой проблемы не составляет.

Дальше процесс максимально прост и понятен - создать новый тест, выбрать его конфигурацию, запустить на максимально возможном количестве метрик и пользователей. В процессе, если в какой-то момент метрики ухудшились, то можно подождать ещё. Как только появилась стат. значимость необходимого набора метрик можно остановить тест, сформировать автоматический отчёт и подкрепить тем самым продуктовую идею. Такой путь намного проще и быстрее, чем общение с дотошным аналитиком, вечно сомневающемся в любом результате. То есть происходит манипуляция данными и выбор метрик для получения ожидаемого результата.

Таким образом, даже технически подкованный инициатор АБ теста, осознанно или неосознанно, подвергается высокому риску сделать ложноположительный вывод. Скорее всего, помимо АБ теста имеется ряд предпосылок и аргументов, подтверждающих, что новая функциональность будет положительно влиять на продукт, а положительный результат вознаграждается гораздо лучше, чем никакой.

### Примеры

Рассмотрим примеры проблем, с которыми заказчики и аналитики могут столкнуться на практике






### Итоги

Мы обсудили несколько важных проблем, с которыми можно столкнуться при автоматизации А/Б-тестирования.

- Множественного тестирования
    
- Подглядывание за результатами
    
- Возможные трудности при исключении выбросов
    
- Сильная связанность метрик
    
- Некорректный выбор статистических критериев
    
- Неравномерное деление на группы.
    

Часть проблем можно решить технически, например применять поправки Бонферони, sequential testing (mSPRT), тесты на равномерность групп, запретить пользоваться ненадежными критериями, однообразно исключать выбросы и т.д. Однако, опытный менеджер с хорошо развитым социальным капиталом без больших проблем найдет способ их обойти. Тогда на первый план выходит зрелость менеджмента и организации процесса принятия решений.

Будем рады услышать ваш положительный и отрицательный опыт по этому вопросу - будет как нельзя кстати для подготовки следующей заметки.

### Авторы

[Marat Yuldashev](https://linkedin.com/in/marat-yuldashev)
[Roman Rudnitskiy](https://www.linkedin.com/in/rudnitskiy)
[Mikhail Tretyakov](https://www.linkedin.com/in/m-tretyakov)

### Ссылки на материалы

#### Видео

[https://en.wikipedia.org/wiki/A%2FB_testing](https://en.wikipedia.org/wiki/A%2FB_testing)
[https://www.youtube.com/watch?v=YeHx5AspRA4](https://www.youtube.com/watch?v=YeHx5AspRA4)
[https://www.youtube.com/watch?v=b6L6P2Bzaq4](https://www.youtube.com/watch?v=b6L6P2Bzaq4)

#### Книги

Trustworthy Online Controlled Experiments**,** Ron Kohavi , Diane Tang, Ya Xu

#### Статьи 

Underpowered A/B Tests – Confusions, Myths, and Reality [https://blog.analytics-toolkit.com/2020/underpowered-a-b-tests-confusions-myths-reality/#authorStart](https://blog.analytics-toolkit.com/2020/underpowered-a-b-tests-confusions-myths-reality/#authorStart)

Zhou J., Lu J., Shallah A. All about sample-size calculations for A/B testing: Novel extensions and practical guide //arXiv preprint arXiv:2305.16459. – 2023.

Howard S. R. et al. Time-uniform, nonparametric, nonasymptotic confidence sequences. – 2021.

Kohavi R., Deng A., Vermeer L. A/b testing intuition busters: Common misunderstandings in online controlled experiments //Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining. – 2022. – С. 3168-3177.

Dmitriev P., Wu X. Measuring metrics //Proceedings of the 25th ACM international on conference on information and knowledge management. – 2016. – С. 429-437.

Gupta S. et al. Top challenges from the first practical online controlled experiments summit //ACM SIGKDD Explorations Newsletter. – 2019. – Т. 21. – №. 1. – С. 20-35.

Fabijan A. et al. Diagnosing sample ratio mismatch in online controlled experiments: a taxonomy and rules of thumb for practitioners //Proceedings of the 25th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining. – 2019. – С. 2156-2164.

Howard S. R., Ramdas A. Sequential estimation of quantiles with applications to A/B testing and best-arm identification //Bernoulli. – 2022. – Т. 28. – №. 3. – С. 1704-1728.

Scott S. L. Multi‐armed bandit experiments in the online service economy //Applied Stochastic Models in Business and Industry. – 2015. – Т. 31. – №. 1. – С. 37-45.

Johari R. et al. Peeking at a/b tests: Why it matters, and what to do about it //Proceedings of the 23rd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining. – 2017. – С. 1517-1525.

Johari R. et al. Always valid inference: Continuous monitoring of a/b tests //Operations Research. – 2022. – Т. 70. – №. 3. – С. 1806-1821.

#### Калькуляторы

- [https://socioline.ru/rv.php](https://socioline.ru/rv.php)
- Optimizely Sample Size Calculator: [https://www.optimizely.com/sample-size-calculator/](https://www.optimizely.com/sample-size-calculator/)
- VWO A/B Test Duration Calculator: [https://vwo.com/ab-test-duration/](https://vwo.com/ab-test-duration/)
- AB Testguide Sample Size Calculator: [https://www.abtestguide.com/ab-test-sample-size-calculator/](https://www.abtestguide.com/ab-test-sample-size-calculator/)
- AB Tasty Sample Size Calculator: [https://www.abtasty.com/sample-size-calculator/](https://www.abtasty.com/sample-size-calculator/)
- Evan Miller Sample Size Calculator: [https://www.evanmiller.org/ab-testing/sample-size.html](https://www.evanmiller.org/ab-testing/sample-size.html)
- Mindbox [https://mindbox.ru/tools/ab-test-calculator/](https://mindbox.ru/tools/ab-test-calculator/)