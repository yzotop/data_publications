---
tags:
  - data
link: https://habr.com/ru/companies/megafon/articles/795261/
source: habr
author: Ryblov
data_type:
  - DS
---


Привет! Меня зовут Артем. Я работаю Data Scientist'ом в компании МегаФон (платформа для безопасной монетизации данных [OneFactor](https://nn.megafon.ru/onefactor/)). Мы строим скоринговые (credit scoring), лидогенерационные (lead generation) и антифрод (anti-fraud) модели на телеком данных, а также делаем гео-аналитику (geoanalytics).

В январе 2022 года я создал репозиторий, чтобы хранить там ссылки на материалы по различным темам: подготовка к собеседованиям, computer science, data science, git, математика и многие другие. За все это время репозиторий разросся до таких размеров (и по количеству материалов, и по набору топиков), что я пришел к выводу, что нужно каким-то образом структурировать эту информацию.

В итоге, блок с материалами для подготовки к собеседованиям я решил выделить в отдельный репозиторий, а для того, чтобы было понятно как эти материалы использовать, было решено написать серию статей, первую из которых вы читаете прямо сейчас.

![Типичный процесс интервью глазами Dall-E 3](https://habrastorage.org/r/w1560/getpro/habr/upload_files/4fa/235/1a6/4fa2351a69bddfb86ee8992bd52ac5d8.jpeg "Типичный процесс интервью глазами Dall-E 3")

Типичный процесс интервью глазами Dall-E 3

Обычно собеседование на позицию Data Scientist состоит из следующих секций (порядок может быть разным):

- **Live Coding**
    
- [Классическое машинное обучение](https://habr.com/ru/companies/megafon/articles/800919/)
    
- Специализированное машинное/глубокое обучение
    
- Дизайн систем машинного обучения (middle+/senior)
    
- Поведенческое интервью (middle+/senior)
    

В данной статье разберемся что такое _live coding_ интервью и как к нему готовиться.

Материал в первую очередь будет полезен Data Scientist'ам и ML инженерам, при этом некоторые разделы, например, [Алгоритмы и структуры данных](https://habr.com/ru/companies/megafon/articles/795261/#%D0%B0%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC%D1%8B-%D0%B8-%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D1%8B-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85-%D1%8D%D1%85-%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D0%B8%D0%BA%D0%B0) подойдут всем IT специалистам, которым предстоит пройти секцию _live coding_.

Замечания

## Содержание

- [Подготовка к алгоритмическому интервью](https://habr.com/ru/companies/megafon/articles/795261/#%D0%BF%D0%BE%D0%B4%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D0%BA%D0%B0-%D0%BA-%D0%B0%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%BC%D1%83-%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%B2%D1%8C%D1%8E)
    
- [Материалы](https://habr.com/ru/companies/megafon/articles/795261/#%D0%BC%D0%B0%D1%82%D0%B5%D1%80%D0%B8%D0%B0%D0%BB%D1%8B)
    
    - [Алгоритмы и структуры данных](https://habr.com/ru/companies/megafon/articles/795261/#%D0%B0%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC%D1%8B-%D0%B8-%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D1%8B-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85-%D1%8D%D1%85-%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D0%B8%D0%BA%D0%B0)
        
    - [Программирование на Python](https://habr.com/ru/companies/megafon/articles/795261/#%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BD%D0%B0-python-%D0%BA%D1%83%D0%B4%D0%B0-%D0%B6%D0%B5-%D0%B1%D0%B5%D0%B7-%D0%BD%D0%B5%D0%B3%D0%BE)
        
    - [SQL](https://habr.com/ru/companies/megafon/articles/795261/#sql-%D1%82%D0%BE%D0%B6%D0%B5-%D0%BD%D1%83%D0%B6%D0%BD%D0%BE)
        
    - [Решение практической Data Science задачи](https://habr.com/ru/companies/megafon/articles/795261/#%D1%80%D0%B5%D1%88%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BF%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B9-data-science-%D0%B7%D0%B0%D0%B4%D0%B0%D1%87%D0%B8-%D0%B0-%D1%87%D1%82%D0%BE-%D1%82%D0%B0%D0%BA-%D0%BC%D0%BE%D0%B6%D0%BD%D0%BE-%D0%B1%D1%8B%D0%BB%D0%BE)
        
    - [Гибрид](https://habr.com/ru/companies/megafon/articles/795261/#%D0%B3%D0%B8%D0%B1%D1%80%D0%B8%D0%B4-%D0%B2%D1%81%D0%B5-%D0%B2%D0%B5%D0%B7%D0%B4%D0%B5-%D0%B8-%D1%81%D1%80%D0%B0%D0%B7%D1%83)
        
- [Learning How to Learn](https://habr.com/ru/companies/megafon/articles/795261/#learning-how-to-learn)
    
- [Подведем итоги](https://habr.com/ru/companies/megafon/articles/795261/#%D0%BF%D0%BE%D0%B4%D0%B2%D0%B5%D0%B4%D0%B5%D0%BC-%D0%B8%D1%82%D0%BE%D0%B3%D0%B8)
    
- [Что дальше?](https://habr.com/ru/companies/megafon/articles/795261/#%D1%87%D1%82%D0%BE-%D0%B4%D0%B0%D0%BB%D1%8C%D1%88%D0%B5)
    

## Подготовка к алгоритмическому интервью

В рамках live coding секции на собеседовании у вас могут проверить знание:

- Алгоритмов и структур данных
    
- Программирования
    
- SQL
    
- Решения практических Data Science задач
    

Алгоритмическое интервью - один из наиболее распространенных форматов live coding этапа в западных [BigTech](https://ru.wikipedia.org/wiki/GAFAM) компаниях и теперь повсеместно встречается в российском "бигтехе", поэтому в этом разделе мы более подробно остановимся на нем. Про другие форматы можно почитать в разделе [Материалы](https://habr.com/ru/companies/megafon/articles/795261/#%D0%BC%D0%B0%D1%82%D0%B5%D1%80%D0%B8%D0%B0%D0%BB%D1%8B).

#### Знакомство

В течение одной алгоритмической секции от вас будет ожидаться решение 1-2 алгоритмических задач (примеры таких задач можно найти на [LeetCode](https://leetcode.com/problemset/algorithms/)) за 60 - 90 минут. В некоторых компаниях, например в Яндексе, таких секций может быть несколько.

Если до этого момента вы еще ни разу не проходили данную секцию и не решали алгоритмические задачи, то рекомендую начать со следующих ресурсов:

- ⭐ [«Подготовка к алгоритмическому собеседованию»](https://practicum.yandex.ru/algorithms-interview/) от Яндекс Практикума.  
    Он позволит:
    
    - Понять структуру алгоритмических собеседований, требования и критерии оценки.
        
    - Проверить свой уровень знаний алгоритмов и структур данных.
        
    - Попрактиковаться на реальных задачах с собеседований.
        
- [Грокаем алгоритмы. Иллюстрированное пособие для программистов](https://www.piter.com/product/grokaem-algoritmy-illyustrirovannoe-posobie-dlya-programmistov-i-lyubopytstvuyuschih-2) / [Grokking Algorithms. An illustrated guide for programmers and other curious people](https://www.manning.com/books/grokking-algorithms)  
    Данная книга является очень простым и дружелюбным введением в алгоритмы. Я бы не стал опираться на нее в подготовке к собеседованию, но как введение в тему вполне подойдет. Читая эту книгу, я замечал в ней много ошибок, поэтому стоит сверяться со своей интуицией на сайте автора в разделе [Errata](https://www.adit.io/errata.html).
    

#### План подготовки

После того, как вы поняли что такое алгоритмы и как проходят такие секции можно приступить к подготовке. И тут, как мне кажется, есть три основных подхода:

- _Фундаментальный_  
    Прохождение курсов и чтение книг по алгоритмам и структурам данных.
    
- _Практический_  
    Решение задач на различных платформах: [LeetCode](https://leetcode.com/), [Codewars](https://www.codewars.com/), [HackerRank](https://www.hackerrank.com/) и тд.
    
- _Гибридный  
    _Изучение теории и мгновенное применении ее на практике.
    

Я предпочитаю последний - на нем и сфокусируемся.

Ниже рассмотрим 3 плана действий (roadmap) для изучения алгоритмов (подготовки к алгоритмическому интервью).

#### План подготовки от NeetCode

Начнем со следующего ⭐ [roadmap](https://neetcode.io/roadmap)'а:

![NeetCode's roadmap](https://habrastorage.org/r/w1560/getpro/habr/upload_files/811/1b1/4d5/8111b14d5cd8d92f52c29721b2a949f2.png "NeetCode's roadmap")

NeetCode's roadmap

На изображении выше я выделил блоки и расставил приоритеты - именно в таком порядке автор [рекомендует](https://www.youtube.com/watch?v=tohk1ETx3ZM) знакомиться с этими темами.

Когда мы нажимаем на один из топиков в roadmap'е, то получаем набор тем для изучения (в данном случае это _Dynamic Arrays_ (динамические массивы), _Hash Usage_ (использование хеширования), _Hash Implementation_ (имплементация хеширования), _Prefix Sums_ (префиксные суммы)), а ниже видим задачи для практики с делением на уровни, видео-объяснением и решением на Python. При этом, если мы нажмем на задачу, то нас перекинет на сайт [LeetCode](https://leetcode.com/problemset/algorithms/).

![Содержание блока Arrays & Hashing](https://habrastorage.org/r/w1560/getpro/habr/upload_files/e62/1d6/9a2/e621d69a2edc24d2f339d8266b29fcbd.png "Содержание блока Arrays & Hashing")

Содержание блока Arrays & Hashing

#### План подготовки от Владимира Балуна

Алгоритм YouTube несколько месяцев назад предложил мне [видео](https://www.youtube.com/watch?v=JjsSssoVZD8), из которого я узнал про еще один roadmap для подготовки и собрал всю необходимую информацию [здесь](https://ryblov.notion.site/Vladimir-Balun-s-Roadmap-063c32655232458e8cc47e0225dfc170?pvs=4):

- [**Видео**](https://www.notion.so/ryblov/Vladimir-Balun-s-Roadmap-063c32655232458e8cc47e0225dfc170?pvs=4#3cca06ba40864e259c4f4d76e2bd0c6a): полезные для подготовки видео с канала автора
    
- ⭐ [**Список задач**](https://www.notion.so/ryblov/Vladimir-Balun-s-Roadmap-063c32655232458e8cc47e0225dfc170?pvs=4#9fc4a3ccfe3a4bcab01c4e684f7378a6): преобразовал [доску miro](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbGdjay1LSEN1UXZGTEduX3NJVVZnWUd1OTU4d3xBQ3Jtc0tsYUp1c3hXR3RMZENUS1ljc0VLRUVnOS1uNHI3cEROQ3JRZ0k0S05aeEdRekpJckNodWxHYkltS2lWcktld0ZIby1nWFotTGhfdzJuaFZXTkFnTXlsRi1yS1p6U292UUtLX2JRek1tTDAtaWFZa0pqMA&q=https%3A%2F%2Fmiro.com%2Fwelcomeonboard%2FVWY1RzF1ZFE5RDZPSlZOdWlSQ3FqUFdzNFdEWjZ1MWkyZHhiOTNiT0JlTUpleGZzRmdDYmZrU0dhSXNxbWJZcHwzNDU4NzY0NTM4MjgyMjUzMDgzfDI%3D%3Fshare_link_id%3D822094860689&v=JjsSssoVZD8) (miro board) из видео в удобный список с гиперссылками.
    
- ⭐ [**Программа курса**](https://www.notion.so/ryblov/Vladimir-Balun-s-Roadmap-063c32655232458e8cc47e0225dfc170?pvs=4#30eb0e6b01c24ae0bb9ee0e9e6427856): прикрепил программу авторского курса, которую мы будем использовать для подготовки.
    

#### План подготовки от EDA Academy

Еще один [roadmap для изучения алгоритмов](https://ryblov.notion.site/EDA-s-Roadmap-d082f650bb434c9abd979cd862817be8?pvs=4) от [EDAcademy](https://t.me/eda_academy).

### Что же со всем этим делать?

Рекомендую сделать файл, в котором вы будете отслеживать свой прогресс и сохранять необходимую информацию.

Я пришел к следующей структуре:

- Весь файл поделен на уровни (аналогично блокам из плана подготовки от NeetCode)
    
- Внутри уровня находятся темы
    
- Каждая тема поделена на два раздела
    
    - Теория  
        Сохраняем теорию, чтобы при необходимости повторить
        
    - Практика  
        Сохраняем ссылки на задачи и решение. Для каждой задачи можем коротко описать подход
        

В качестве примера прикладываю [шаблон](https://ryblov.notion.site/Roadmap-ecd62466c5f9480dbb0354d768b94cad?pvs=4) с первыми двумя блоками, который можно дублировать и использовать.

### Материалы

#### Алгоритмы и структуры данных (эх, классика)

В предыдущей секции были разобраны подходы к изучению алгоритмов и структур данных, рассмотрены 3 плана для подготовки и предложен шаблон для создания собственного roadmap'а.

Не хватает только материалов для заполнения персонализированного роадмапа, которые мы рассмотрим ниже в этом разделе.

#### Задачники

- [LeetCode](https://leetcode.com/)
    
    - ⭐ [Leetcode Patterns](https://seanprashad.com/leetcode-patterns/)  
        Список вопросов с делением по темам, сложности и компаниям (которые спрашивают эти вопросы на собеседовании) + советы насчет того, какой алгоритм выбрать при первом взгляде на условия задачи.
        
    - [LeetCode Explore](https://leetcode.com/explore/)  
        Мини-курсы по алгоритмам (и другим темам) от LeetCode
        
- [Codewars](https://www.codewars.com/)
    
- [HackerRank](https://www.hackerrank.com/)
    
- [CodeAbbey](https://www.codeabbey.com/index/task_list)
    
- [Codeforces](https://codeforces.com/)
    
- [Яндекс.Кон­тест](https://contest.yandex.ru/)
    
- [Задачи проекта "Школа программиста" красноярского краевого дворца пионеров](https://acmp.ru/article.asp?id_text=7)
    
- [Задачи московского центра непрерывного математического образования](https://informatics.msk.ru/)
    
- [Другие](https://en.wikipedia.org/wiki/Competitive_programming#Online_platforms)
    

#### Книги

Подготовка к собеседованию:

- [Elements of Programming Interviews in Python: The Insiders' Guide](https://elementsofprogramminginterviews.com/)  
    Данная книга содержит более 250 алгоритмических задач с решениями по основным темам, о которых мы говорили выше. Есть версии для C++ и Java.
    
- [Cracking the Coding Interview](https://www.crackingthecodinginterview.com/)  
    Отличается от предыдущей более подробным введением и дополнительными вопросами по языкам программирования и поведенческому интервью.
    

Изучение алгоритмов:

- [Томас Кормен. Алгоритмы: построение и анализ](https://www.dialektika.com/books/978-5-8459-1794-2.html) / [Introduction to Algorithms](https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/)  
    Классический труд по алгоритмам.
    
- [Дасгупта С., Пападимитриу Х., Вазирани У. Алгоритмы](https://biblio.mccme.ru/node/13590/shop) / [S. Dasgupta, C. H. Papadimitriou, and U. V. Vazirani. Algorithms](http://algorithmics.lsi.upc.edu/docs/Dasgupta-Papadimitriou-Vazirani.pdf)
    
- Другие:
    
    - Никлаус Вирт. Алгоритмы и структуры данных.
        
    - Дж. Макконнелл. Основы современных алгоритмов.
        
    - Седжвик Роберт. Алгоритмы на C++ / Седжвик Роберт, Уэйн Кевин. Алгоритмы на Java
        
    - Альфред Ахо, Джон Хопкрофт, Джеффри Ульман. Структуры данных и алгоритмы
        

#### Курсы

- [Основы алгоритмов](https://education.yandex.ru/handbook/algorithms)  
    Бесплатный курс (хендбук) от Яндекса. Лучше, чем Яндекс я этот ресурс не резюмирую:  
    "С помощью этого хендбука вы научитесь проектировать, оптимизировать, комбинировать и отлаживать алгоритмы — причем без привязки к какому-либо языку программирования. Кроме теории мы собрали и практические задания разного уровня сложности, а также подготовили систему автоматической проверки эффективности алгоритмов — всё это поможет вам закрепить и отточить новые навыки"
    
- Алгоритмы: теория и практика: [Методы](https://stepik.org/course/217/info) + [Структуры данных](https://stepik.org/course/1547/info)  
    Классные курсы для тех, кто хочет разобраться с алгоритмами и структурами данных, но, по моему мнению, это чересчур для собеседований (кроме Яндекса).
    
- [Introduction To Algorithms](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/video_galleries/lecture-videos/)  
    Вводный курс, охватывающий элементарные структуры данных (динамические массивы, кучи, сбалансированные двоичные деревья поиска, хеш-таблицы) и алгоритмические подходы к решению классических задач (сортировка, поиск по графам, динамическое программирование). Введение в математическое моделирование вычислительных задач, а также общие алгоритмы, алгоритмические парадигмы и структуры данных, используемые для решения этих задач. Подчеркивает взаимосвязь между алгоритмами и программированием, а также знакомит с основными показателями производительности и методами анализа этих проблем.
    
- Материалы от Яндекса:
    
    - [Тренировки по алгоритмам 1.0](https://yandex.ru/yaintern/algorithm-training_1)
        
    - [Тренировки по алгоритмам 2.0](https://yandex.ru/yaintern/algorithm-training_2)
        
    - [Тренировки по алгоритмам 3.0](https://yandex.ru/yaintern/training/algorithm-training_3)
        
    - [Тренировки по алгоритмам 4.0](https://yandex.ru/yaintern/training/algorithm-training_4)
        
    - [Тренировки по алгоритмам 5.0](https://yandex.ru/yaintern/algorithm-training)
        
    - [Интенсив по алгоритмам](https://www.youtube.com/playlist?list=PLQC2_0cDcSKAzLqqXUidKBJsy1Im44aOo)
        
    
    Видео с лекциями + домашние задания с разбором
    
- [Курс «Алгоритмы и структуры данных» (яндекс практикум)](https://practicum.yandex.ru/algorithms/) `paid`
    
- [Подготовься к алгоритмическому собеседованию за 9 недель (balun.courses)](https://balun.courses/courses/algorithmic_interview) `paid`
    
- [Алгоритмическое программирование](https://algoprog.ru/) `paid   `Курс по олимпиадному программированию от А до Я. Для собеседования избыточно (кроме Яндекса), но первые два уровня могут быть хорошим введением в алгоритмические задачи.
    
- [Algorithms and Data Structures for Beginners](https://neetcode.io/courses/dsa-for-beginners/0) `paid` + [Advanced Algorithms](https://neetcode.io/courses/advanced-algorithms/0) `paid`  
    Курсы от создателя первого плана подготовки (NeetCode). Объясняет простым языком и закрепляет практикой.
    
- [Алгоритмические задачи с собеседований](https://stepik.org/course/126012/) `paid`
    

#### Разное

Советы:

- ⭐ [Решение алгоритмических задач](https://btseytlin.github.io/parts/3_interviewing/technical_interview.html#id7)  
    Советы по решению задач. Хорошо дополняет, а где-то и повторяет, бесплатный курс от яндекс практикума, про который я писал в разделе [Знакомство](https://habr.com/ru/companies/megafon/articles/795261/#%D0%97%D0%BD%D0%B0%D0%BA%D0%BE%D0%BC%D1%81%D1%82%D0%B2%D0%BE). Советую также прочитать всю методичку, а не только этот раздел.
    
- [Coding Interview Guide](http://patrickhalina.com/posts/coding-interview-guide/)  
    Советы по прохождению алгоритмической секции.
    

Структуры данных:

- [Data Structures Reference](https://www.interviewcake.com/data-structures-reference)  
    Краткий справочник затрат по Big O ([«O» большое](https://ru.wikipedia.org/wiki/%C2%ABO%C2%BB_%D0%B1%D0%BE%D0%BB%D1%8C%D1%88%D0%BE%D0%B5_%D0%B8_%C2%ABo%C2%BB_%D0%BC%D0%B0%D0%BB%D0%BE%D0%B5)) нотации и основных свойств каждой структуры данных.
    
- [An Executable Data Structures Cheat Sheet for Interviews](https://algodaily.com/lessons/an-executable-data-structures-cheat-sheet)  
    Справочник по структурам данных с реализацией на разных языках программирования и оценкой сложности.
    

Телеграм каналы:

- [Алгоритмы - Собеседования, Олимпиады, ШАД](https://t.me/algoses)  
    Телеграм канал, в котором выкладывают алгоритмические задачи (с решениями).
    

Репозитории:

- [Leetcode company-wise questions](https://github.com/MysteryVaibhav/leetcode_company_wise_questions)  
    Репозиторий, содержащий список вопросов (в разрезе компаний), доступных на премиум-версии Leetcode.
    
- [The Algorithms](https://github.com/TheAlgorithms)  
    Репозитории для изучения структур данных и алгоритмов и их реализации на любом языке программирования.
    

Остальное:

- [Алгоритмы и структуры данных простыми словами](https://codonaft.com/%D0%B0%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC%D1%8B-%D0%B8-%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D1%8B-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85-%D0%BF%D1%80%D0%BE%D1%81%D1%82%D1%8B%D0%BC%D0%B8-%D1%81%D0%BB%D0%BE%D0%B2%D0%B0%D0%BC%D0%B8/)  
    Подборка материалов для изучения алгоритмов и структур данных + видео на тему мотивации для их изучения.
    
- [Алгоритмика](https://ru.algorithmica.org/)  
    Материалы различных CS-курсов, проводящихся в Tinkoff Generation (анализ алгоритмов, структуры данных, динамическое программирование, математика, графы, геометрия, строки, разное).
    
- [Algorithmic concepts](https://superstudy.guide/algorithms-data-structures/foundations/algorithmic-concepts/)  
    Шпаргалка с теорией по алгоритмам и структурам данных.
    
- ⭐ [Algorithmic Thinking](https://labuladong.gitbook.io/algo-en/)  
    В статьях на этом сайте рассматриваются различные подходы к изучению алгоритмов. Все они проиллюстрированы с помощью задач с LeetCode'а, причем указаны не только решения, но и объяснения ПОЧЕМУ решение работает и КАК вы тоже можете его понять.
    

### Программирование на Python (куда же без него)

Для Data Scientist'а язык программирования _Python_ (иногда _R_) - это рабочий инструмент, с помощью которого проводится анализ данных, поэтому важно хорошо в нем разбираться, чтобы писать "чистый" и оптимальный код.

Чтобы проверить эти знания, на собеседовании вас могут спросить такие базовые теоретические вопросы, как:

- Какие типы данных есть в Python?
    
- Какие структуры данных есть в Python?
    
- Отличия типов и структур данных
    
- Асимптотика основных операций в Python
    

Также нередко предлагают оценить что будет выведено после выполнения ячейки кода/функции.

Например:

```
# Что выведет код?D = {}A = DB = D.copy()B[ 'b' ] = 2A[ 'b' ] = 3print(D, B, A)
```

Во время ответа на этот вопрос собеседуемый должен проявить свои знания о способах копирования объектов в Python и различиях между ними (shallow vs deep copy), а также о изменяемости типов данных в Python (mutable vs immutable).

Для подготовки к данной секции рекомендую ознакомиться с материалами ниже.

#### Чистый код

- ⭐ [Мартин Р. Чистый код: создание, анализ и рефакторинг](https://www.piter.com/product/chistyy-kod-sozdanie-analiz-i-refaktoring-biblioteka-programmista-45ccca) / [Robert C. Martin. Clean Code: A Handbook of Agile Software Craftsmanship](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
    
- Стив Макконнелл. Совершенный код. Мастер-класс / [Steve McConnell. Code Complete: A Practical Handbook of Software Construction](https://www.amazon.com/Code-Complete-Practical-Handbook-Construction/dp/0735619670)
    

#### Теория

- [Основы Python](https://education.yandex.ru/handbook/python)  
    Хендбук по Python поможет овладеть основным синтаксисом и принципами языка, а также разными подходами к программированию.
    
- [Программирование на Python](https://stepik.org/course/67/info)  
    Вводный курс по Python. Курс посвящен базовым понятиям и элементам языка программирования Python (операторы, числовые и строковые переменные, списки, условия и циклы). Курс является вводным и наиболее подойдет слушателям, не имеющим опыта написания программ ни на одном из языков программирования.
    
- [Python: основы и применение](https://stepik.org/course/512/info)  
    Курс посвящен базовым принципам языка Python и программирования в целом. Подойдет тем, кто уже может писать простейшие программы на Python или тем, кто до этого программировал на других языках.
    
- Поколение Python:
    
    - [Курс для начинающих](https://stepik.org/course/58852)
        
    - [Курс для продвинутых](https://stepik.org/course/68343)
        
    - [Курс для профессионалов](https://stepik.org/course/82541) `paid`
        
    
    Просто отличные курсы, которые становились лучшими бесплатными/платными курсами на Stepik в течение 2020-2022 годов.
    
- [CS50’s Introduction to Programming with Python](https://cs50.harvard.edu/python/2022/)  
    Введение в программирование с использованием языка Python от [гарвардского университета](https://ru.wikipedia.org/wiki/%D0%93%D0%B0%D1%80%D0%B2%D0%B0%D1%80%D0%B4%D1%81%D0%BA%D0%B8%D0%B9_%D1%83%D0%BD%D0%B8%D0%B2%D0%B5%D1%80%D1%81%D0%B8%D1%82%D0%B5%D1%82).
    
- [Python Tutorial for Beginners (with mini-projects)](https://www.youtube.com/watch?v=qwAFL1597eM)
    

#### Вопросы

- Python: вопросы на собеседовании:
    
    - [Часть I. Junior](https://pythonist.ru/python-voprosy-sobesedovaniya-chast-i-junior/)
        
    - [Часть II. Middle](https://pythonist.ru/python-voprosy-sobesedovaniya-chast-ii-middle/)
        
    - [Часть III. Senior](https://pythonist.ru/python-voprosy-sobesedovaniya-chast-iii-senior/)
        
- [53 Python Interview Questions and Answers](https://towardsdatascience.com/53-python-interview-questions-and-answers-91fa311eec3f)
    
- [Python вопросы с собеседований](https://t.me/python_job_interview)  
    Вопросы по Python + алгоритмические задачи.
    

#### Практика

- [Задачи по Python и машинному обучению](https://t.me/python_tasks)  
    Каждый день в канале выкладываются различные задачи со звездочкой, которые заставляют подумать.
    

#### Разное

- [Efficient Python Tricks and Tools for Data Scientists](https://khuyentran1401.github.io/Efficient_Python_tricks_and_tools_for_data_scientists/README.html)  
    Классная онлайн-книга, содержащая различные методы и инструменты для более эффективного использования Python.
    

### SQL (тоже нужно)

Владение SQL (Pandas/PySpark) также часто проверяется во время live coding интервью, потому что это необходимый навык для специалиста по данным.

В рамках этой секции на собеседовании вам могут задать теоретические вопросы, например:

- Что делает `Union`?
    
- В чем разница между `INNER JOIN` и `LEFT` / `RIGHT JOIN`?
    
- Что за `NoSQL` базы данных? В чем их принципиальное отличие от `SQL`?
    
- Что такое под-запрос и для чего они нужны?
    

Также могут предложить решить 1 или 2 задачи на написание SQL запроса (или совершение аналогичных преобразований в `Pandas`/`PySpark`/`name_your_framework`), например:

Дана таблица **transactions**:

|   |   |   |
|---|---|---|
|id|date|income|
|1|2021-04-01|22000|
|2|2021-04-02|11100|
|3|2021-04-11|64000|
|4|2021-05-04|23000|
|5|2021-06-17|20000|
|6|2021-06-18|7900|
|7|2021-06-19|32000|
|8|2021-07-12|17000|
|9|2021-07-23|14600|
|10|2021-01-12|26300|
|11|2021-08-11|10000|

_Для текущего месяца посчитайте скользящее среднее дохода за 3 предыдущих месяца._

Во время ответа на этот вопрос собеседуемый должен проявить свои знания о группировке данных с помощью оператора `GROUP BY`, функциях `MONTH`/`SUM`/`AVG`, оконных функциях и вариантах ограничения фрейма `ROWS BETWEEN`.

Для подготовки к данному типу интервью рекомендую ознакомиться с материалами ниже.

#### Курсы

- [Интерактивный тренажер по SQL](https://stepik.org/course/63054/info)  
    Минимально необходимые теоретические выкладки + много практики. Из минусов могу отметить большое количество запросов на корректировку данных (`CREATE TABLE`, `INSERT INTO`, `UPDATE`, `DELETE`), которые необходимо написать для успешного завершения курса.
    
- ⭐ [Пакет SQL курсов](https://stepik.org/course/61247/info) `paid`:
    
    - [Основы SQL](https://stepik.org/course/51562/info) `paid`
        
    - [Продвинутый SQL](https://stepik.org/course/55776/info) `paid`
        
    - [Проектирование баз данных](https://stepik.org/course/51675/info) `paid`
        
    
    Все в этих курсах мне понравилось, кроме того, что они платные и нет теории в виде текста (все в видео формате)
    
- [PostgreSQL Tutorial for Beginners](https://www.youtube.com/watch?v=SpfIwlAYaKk)  
    Вводный курс в PostgreSQL. Кажется, что он сильно проще, чем курсы выше. Также из минусов стоит отметить отсутствие практики.
    
- [Оконные функции SQL](https://stepik.org/course/95367) `paid`  
    18 уроков и 56 практических заданий на оконные функции. Минус - курс платный.
    
- [SQL Tutorial](https://mode.com/sql-tutorial/)  
    Огромный туториал/гайд с теорией и практикой.
    
- [The Ultimate SQL Guide](https://blog.count.co/the-ultimate-sql-guide/)  
    Красивый туториал с теорией для изучения/повторения основ SQL.
    

#### Практика

- ⭐ [Онлайн тренажер SQL Academy](https://sql-academy.org/)  
    Удобный и бесплатный тренажер с большим количеством практических задач.
    
- [Ace the SQL Interview](https://datalemur.com/questions?category=SQL)  
    37 задач на SQL, которые встречались на собеседованиях в топовые компании.
    
- ⭐ [Симулятор SQL](https://karpov.courses/simulator-sql)  
    Бесплатный симулятор SQL (более 120 задач).
    
- Другие тренажеры:
    
    - [Practice SQL](https://www.sql-practice.com/)
        
    - [SQLBolt. Learn SQL with simple, interactive exercises](https://sqlbolt.com/)
        
    - [SQL Tutorial by w3schools](https://www.w3schools.com/sql/)
        
    - [PostgreSQL Exercises](https://pgexercises.com/)
        

### Решение практической Data Science задачи (а что так можно было?)

Иногда в рамках данной секции вместо решения алгоритмической задачи или написания SQL вопроса предлагается более практическое задание, а именно решение прикладной задачи по анализу данных:

- End-to-end: Начиная от EDA (Exploratory Data Analysis, разведочный анализ данных), до построения модели.  
    Обычно такую задачу дают в виде тестового задания, потому что она предполагает от нескольких часов до нескольких дней на решение.
    
- Укороченный формат:
    
    - Предлагается код (предположим, код для построения модели), который нужно проанализировать, найти ошибки и предложить улучшения.
        
    - Предлагается решить одну из частей полной задачи по анализу данных, например, провести EDA или feature engineering (создание и модификация признакового пространства).
        
    
    Данный формат не занимает много времени, поэтому используется во время собеседования.
    

Подготовкой к данному виду live coding собеседования является практика решения Data Science задач на работе и/или самостоятельно. В этом вам помогут материалы, которые я указал ниже.

#### Анализ кода

- ⭐ [Annotated Research Paper Implementations: Transformers, StyleGAN, Stable Diffusion, DDPM/DDIM, LayerNorm, Nucleus Sampling and more](https://nn.labml.ai/)  
    Коллекция реализаций нейросетевых архитектур на PyTorch с комментариями.
    
- Библиотеки  
    Нужна насмотренность, поэтому читаем документацию и пытаемся понять реализацию алгоритмов в популярных ML/DL библиотеках:
    
    - [NumPy](https://numpy.org/)
        
        - [100 numpy exercises](https://github.com/rougier/numpy-100)
            
    - [Pandas](https://pandas.pydata.org/)
        
    - [scikit-learn](https://scikit-learn.org/stable/)
        
    - [PyTorch](https://pytorch.org/)
        
- ⭐ [PEP 8 – Style Guide for Python Code](https://peps.python.org/pep-0008/) + [How to Write Beautiful Python Code With PEP 8](https://realpython.com/python-pep8/)  
    Гид по стилю программ на Python, чтобы ваш код было приятно читать и легко поддерживать.
    

#### Практика

Для практики рекомендую делать pet-проекты (начать можно с этой статьи - [Data Science Pet Projects. FAQ](https://habr.com/ru/companies/ods/articles/681718/)).

- ⭐ [Kaggle Learn](https://www.kaggle.com/learn)  
    Kaggle бесплатно предоставляет доступ к своим коротким, но интересным курсам. Вот лишь некоторые из них:
    
    - [Intro to Machine Learning](https://www.kaggle.com/learn/intro-to-machine-learning)
        
    - [Intermediate Machine Learning](https://www.kaggle.com/learn/intermediate-machine-learning)
        
    - [Feature Engineering](https://www.kaggle.com/learn/feature-engineering)
        
    - [Machine Learning Explainability](https://www.kaggle.com/learn/machine-learning-explainability)
        
- [Kaggle Competitions](https://www.kaggle.com/competitions)  
    Можно участвовать, как в учебных соревнованиях, направленных на тренировку и развитие навыков, так и в соревнованиях, за победу в которых дают призы.
    
- ⭐ [The Kaggle Book](https://www.kaggle.com/discussions/general/320574)  
    Внутри книги подробно описан анализ соревнований, примеры кода, end-to-end пайплайны, а также все идеи, предложения, лучшие практики, советы и рекомендации, которые Лука Массарон с Конрадом Банахевичом собрали за годы соревнований на Kaggle (более 22 лет).
    
- [Симулятор ML](https://karpov.courses/simulator-ml) `paid   `Платный практико-ориентированный курс, на котором можно решить [большое количество](https://karpov.courses/simulator-ml/programm) ML задач.
    

### Гибрид (все, везде и сразу)

На собеседовании, например, вас могут попросить решить:

- Несколько алгоритмических задач
    
- Одну алгоритмическую задачу и одну на SQL
    
- Дать тестовое задание, связанное с ML и SQL
    
- И так далее
    

Поэтому при подготовке советую не пренебрегать ни одним из пунктов, указанных выше.

## Learning How to Learn

Чтобы надолго запомнить всю эту информацию и не сойти с ума можно использовать технику [интервального повторения](https://ru.wikipedia.org/wiki/%D0%98%D0%BD%D1%82%D0%B5%D1%80%D0%B2%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5_%D0%BF%D0%BE%D0%B2%D1%82%D0%BE%D1%80%D0%B5%D0%BD%D0%B8%D1%8F), а для реализации этой техники программу ⭐ [Anki](https://ru.wikipedia.org/wiki/Anki).

Создаете deck (топик), добавляете туда материал, который хотите запомнить, например алгоритмические задачи, и повторяете его, когда программа предложит.

Также рекомендую книги:

- ⭐ [Марк Макдэниэл, Генри Рёдигер, Питер Браун. Запомнить все. Усвоение знаний без скуки и зубрежки](https://alpinabook.ru/catalog/book-zapomnit-vse/) / [Make it Stick: The Science of Successful Learning by Peter C. Brown, Henry L. Roediger, III, and Mark A. McDaniel](https://www.retrievalpractice.org/make-it-stick)
    
- [Барбара Оакли, Бет Роговски, Терренс Джей Сейновски. Научить невозможному. Как помочь ученикам освоить любой предмет и не бояться экзаменов](https://eksmo.ru/book/nezdravyy-smysl-kak-s-pomoshchyu-neyronauk-izuchat-novoe-ITD1216969/) / [Uncommon Sense Teaching. Practical Insights in Brain Science to Help Students Learn by Barbara Oakley PhD, Beth Rogowsky EdD, Terrence J. Sejnowski](https://barbaraoakley.com/books/uncommon-sense-teaching/)
    

Эти книги будут полезны всем, кто хочет эффективно усваивать новые знания и использовать их на практике.

## Подведем итоги

Основная идея данной статьи заключается в том, чтобы использовать доступные ресурсы в интернете (а именно, планы и программы, курсы, видео, статьи, блог-посты и т.д.) для того, чтобы:

1. Подготовить собственный план изучения материала  
    Свои любимые материалы я выделил ⭐
    
2. Наполнить этот план теорией и практикой
    
3. Достичь своей цели (изучение алгоритмов / поиск работы), при этом сохранив все материалы на будущее, как reference (все равно придется повторять при устройстве на следующую работу)
    

Собранные в этой статье материалы будут полезны при подготовке к собеседованиям на [различные позиции](https://job.megafon.ru/) в Big Data МегаФон.

А если вы только начинаете свою карьеру в Data Science, то обратите внимание на стажировки в крупных компаниях, на которых вы сможете не только прокачать свои знания, но также получить крутой опыт применения теории на практических задачах бизнеса. В МегаФоне пример такой стажировки - это акселератор (пишите на [почту](mailto:career@megafon.ru) с темой письма "_стажировка в big data"_), с помощью которого ежегодно находят свою первую работу специалисты по работе с данными (Data Scientists), аналитики (Data Analysis) и дата инженеры (Data Engineers).

## Что дальше?

В следующей статье разберем материалы для подготовки к секции по классическому машинному обучению.

Актуальные ресурсы для этой серии статей вы сможете найти в репозитории [Data Science Resources](https://github.com/Extremesarova/ds_resources), который будет поддерживаться и обновляться. Также вы можете подписаться на мой телеграм-канал [Data Science Weekly](https://t.me/data_science_weekly), в котором я каждую неделю делюсь интересными и полезными материалами.

Если вы знаете какие-нибудь классные ресурсы, которые я не включил в этот список, то прошу написать о них в комментариях.

P.S. Благодарю [Дарью Шатько](https://habr.com/ru/users/DariaSatco/) за редактуру и вычитку этого поста!