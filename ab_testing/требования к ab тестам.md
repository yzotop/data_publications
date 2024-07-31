---
tags:
  - data
data_type:
  - AB tests
---


-   Календарь АБ тестов, где указаны даты, аудитории, экраны (группа экранов), на которых проводится тест.
- Указание на владельца АБ теста.
- сортировка АБ тестов на главной странице.
- механика выбора слоев для тестов и отображения этой информации на календаре (чтобы можно было разные зависимые АБ тесты на разных аудиториях крутить)
- Механика автоматического расчета метрик в АБ тесте и дашборд с результатами
- Добавление в раздел “эксперименты” типа эксперимента: тест/фича тогл (тест это АБ тест, фича тогл - это ситуация, когда “эксперимент” раскатан на 100% для 1 группы)
- при переводе в статус выполнено добавить поле для аналитика с описанием итогов эксперимента, для единой статистики
- Калькулятор для расчета выборки под АБ тест (встроенный в платформу или сделанный на стороне дашборда в табло)
- нам скорее нужны слои в АБ платформе, чтобы аффектящие тесты технически нельзя было запустить друг поверх друга![](data:image/png;base64,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) (выбор слоев решает аналитик на этапе дизайна)
- на группу “контроль” не приходит физически “тоггл”. то есть мы не умеем логировать/обрабатывать конкретную группу “control”. обход сейчас сущестует: делаем доп группу.
- люди используют аб-платформу как инструмент включения/выключения фичи. непонятно какой из экспериментов - чисто для раскатки, а какой настоящий аб.
- огромная каша в тестах/фичах/аудиториях. нужно уметь ориентироваться, уметь отделять тестовое-дебажное-ненастоящее. не могу удобно увидеть 1) все свои тесты 2) все тесты моей вертикали 3) все тесты конкретного человека
- Нет оповещений по срокам тестов запущенных - когда закончится или когда наберется аудитория (если новеньких набирать)
- нет возможности командой управлять экспериментом. тестировщик+разработчик+аналитик+продакт вместе работают и тратятся силы лишние на коммуникацию. редактировать эксп может только 1 человек - создатель.
- нельзя делить аудиторию более аккуратно: по наличию ивента определенного (пример “хоть раз делал заказ в intercity”) или по метрике с датами (пример “колво заказов больше 5 за последнюю неделю”). Приходится на анализе результатов отфильтровывать ненужных пользователей. Но они были задействованы в эксперименте просто так. Так можно выделить только “опытных”, например.
- непонятно насколько большую аудиторию мы отобрали по фильтрам в аудитории.
- не могу понять - моя аудитория уже в каком-то тесте участвует?
- не могу понять - на какую метрику влияет эксперимент, запущенный кем-то. не могу сделать вывод о допустимости пересечения аудиторий. приходится лезть в конфльенс либо в личку.
- не знаю кому писать, если вижу чей-то тест
- не знаю к какой вертикали относится чей-то тест
- все тесты по-разному описаны в конфлюенсе - неудобно читать и разбираться. не вижу такой инфы удобно на платформе аб.
- не вижу течения экспериментов чужих - хочу ссылки на мониторинги.
- матстатные приколы хочу: помочь в подсчете срока проведения, объема аудитории, подсчет метрик, подсчет p-value, оценка стат.значимости, проверка АА на выбранной аудитории
- нет правил нейминга фичей, аудиторий, экспериментов. непонятно что означают названия