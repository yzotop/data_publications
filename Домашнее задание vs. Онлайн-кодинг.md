---
author: Kashnitsky
tags:
  - data
link: https://vas3k.club/post/18668/
source: vastrik
company: Google
---
В комментариях к [посту](https://vas3k.club/post/1819470/) о том, как chatGPT почти крякнул домашнее задание на позицию ML инженера, наметились зачатки спора про домашние задания ("take-home") против алгоритмических собеседований с написанием кода в режиме реального времени ("leetcode"). Разверну мысль, возможно, защищая непопулярное мнение: мне домашние задания куда больше по душе.

Ругают вообще-то оба этих способов отбора кандидатов, домашние задания - в основном за неоплачиваемое время кандидатов и нагрузку во внерабочие часы, литкод - за проверку специфичных навыков. Так что тут выбор из двух зол. И мой выбор - домашние задания. Поскольку, про минусы литкода только ленивый не писал, буду больше про take-home.

## [Плюсы take-home в сравнении с алгоритмическими собеседованиями](https://vas3k.club/post/18668/#Pliusy-take-home-v-sravne)

### [Плюс 1. Куда ближе к реальной работе](https://vas3k.club/post/18668/#Plius-1-Kuda-blizhe-k-rea)

Тут вряд ли кто поспорит, завернуть модельку в API и докер, обсудить model lifecycle management и дашборды с метриками - бесконечно ближе к реальной работе MLE, чем пресловутые BFS и DFS (уж красно-черные деревья такой мем и стереотип, что даже не хотел упоминать).

### [Плюс 2. Меньше стресса](https://vas3k.club/post/18668/#Plius-2-Menshe-stressa)

Кандидат работает в нестрессовой обстановке. Про стресс собесов вживую сто раз говорилось, просто отметим, что не спеша делать задание дома или на работе (кек) - куда лучше.

### [Плюс 3. Проверка навыков приоретизации](https://vas3k.club/post/18668/#Plius-3-Proverka-navykov)

Хорошее задание дается из расчета на 3-6 часов работы. Это значит не 40, а реально 3-6 часов. В таком ограничении я вижу только плюс - можно увидеть, что кандидат считает важным, а что нет. Для примера можно представить идеальный парсер какого-то сайта и хреновую архитектуру приложения и, наоборот, продуманную архитектуру и базовый quick & dirty парсер.

### [Плюс 4. (опц.) Качественный фидбек](https://vas3k.club/post/18668/#Plius-4-opts-Kachestven)

Это в общем не факт, что верно, тут скажу за себя: я даю кандидатам подробный фидбэк по заданию, как по ключевым решениям, так и по коду. Джунам такой фидбек будет сильно полезнее, чем стандартный фидбэк после запоротого алгоритмического собеса или отписка «надо бы структуры данных поботать».

### [Плюс 5. Часть портфолио](https://vas3k.club/post/18668/#Plius-5-Chast-portfolio)

По согласованию с работодателем сделанное домашнее задание можно поместить в портфолио проектов. С сравнении с сотнями часов, потраченных на литкод, это выглядит плюсом, таким же, как от пет-проджектов.

И, конечно, минусы. Приведу и те, которые традиционно считаются минусами, со своими контраргументами.

### [Минус 1. «Кандидаты работают за бесплатно»](https://vas3k.club/post/18668/#Minus-1-Kandidaty-rabo)

Хех, давайте поговорим об этом. Во-первых, если постараться, можно и оплатить работу кандидатов, что, думаю, в средней компании сделать сложновато, но можно.

Во-вторых, будем честны, больше половины заданий настолько плохи, что фидбек очень ценен для кандидата, а говорить об оплате времени даже как-то нелепо, когда на выходе что-то несуразное, вот уж реально chatGPT лучше сделает.

Так что средний кандидат тратит время на take-home, получает опыт и фидбек, по мне – fair enough. Не вижу особого отличия от 100 часов, убитых на литкод, и собес, запоротый за 10 минут, где попадается hard или крепкий medium (см. плюс первый, take-home задания хотя бы куда ближе к жизни, чем литкод).

### [Минус 2. Работа во внеурочное время](https://vas3k.club/post/18668/#Minus-2-Rabota-vo-vneur)

Часто можно слышать, что «бедные кандидаты должны работать над заданием во внерабочее время». Серьезно? А литкод? В рабочее время? Тебе в принципе надо сильно постараться, чтоб сменить работу. И хорошо бы не лицемерить, когда человек ищет работу, он уже отбирает немного времени и от работы в текущей компании, опять же будь то take-home, литкод или какой угодно другой собес.

### [Минус 3. Задания утекают в открытый доступ](https://vas3k.club/post/18668/#Minus-3-Zadaniia-utekaiut)

Да, вот это серьезный минус, тут можно частично разрулить проблему путём NDA, но задания все равно будут утекать, с этим нельзя что-либо сделать. А новые задания каждый раз легко не придумаешь, когда надо (к слову, это же - наибольшая сложность открытых повторяющихся курсов типа [mlcourse.ai](https://mlcourse.ai/)).

### [Минус 4. Вряд ли для синьоров](https://vas3k.club/post/18668/#Minus-4-Vriad-li-dlia-sin)

Вот это да, проблемка, точнее, ограничение - take-home хороши для найма стажеров, джунов, мидлов, но вот господа синьоры, весьма вероятно, ответят «нет у меня времени на это ваше задание». Но по мне это и не такая уж проблема, у синьоров можно другие вещи спрашивать, с ними на первый план выходят поведенческие собеседования (писал про них [тут](https://t.me/new_yorko_times/6)) и истории из жизни.

### [Минус 5. chatGPT](https://vas3k.club/post/18668/#Minus-5-chatGPT)

Да, это надо переосмыслить то, как chatGPT [почти справился](https://vas3k.club/post/1819470/) с моим заданием на MLE. Я пока вот что намыслил: chatGPT не убил мое задание. Да, с этим ассистентом задание можно сделать на 90%, окей. Тогда просто объявляю прямо в формулировке задания, что можно пользоваться chatGPT, идем дальше, обсуждаем все вокруг. Опять же, приближено к реальной работе.

### [Минус 6. Можно скатать](https://vas3k.club/post/18668/#Minus-6-Mozhno-skatat)

Да, никак не проверишь, сделал ли за тебя друг твое задание. Но у меня take-home всегда сопровождается собесом с презентацией решения, и кажется, я с высокой точностью могу понять, когда человек презентует не свой код. Но да, false positives в этой части отбора возможны.

Резюмируя, я бы не противопоставлял эти два вида отбора кандидатов (так что формат поста – не баттл), но мне ближе домашние задания. Их, впрочем, можно совместить и с простым "литкодом" уровня 1000-го числа Фибоначчи или даже fizz-buzz, чтоб как можно быстрее нулёвых отсеять. К слову, с Амазоном у меня было и домашнее задание (tech talk), и простой литкод, и стандартные вопросы по теме, и куча времени для behavioral.