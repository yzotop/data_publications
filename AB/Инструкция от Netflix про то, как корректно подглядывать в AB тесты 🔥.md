---
tags:
  - data
author: Balandin
source: telegram
link: https://t.me/suranalytics/196
data_type:
  - AB tests
ab_type: Peeking
---

[ab\_testing/sequential\_testing\_at\_netflix\_simulation.ipynb at main · YuriyBalandin/ab\_testing · GitHub](https://github.com/YuriyBalandin/ab_testing/blob/main/sequential_testing_at_netflix_simulation.ipynb)

У Netflix недавно вышло две статьи в их блоге (по мотивам нескольких научных). Они рассказывают, как применяют sequential testing для непрерывных данных и для счетчиков (counting processes, под них, например, попадают данные из распределения Пуассона, такие как кол-во логинов или покупок подписок). И это позволяет непрерывно мониторить p-value, и при этом еще и ускорять тесты. 

Но зачем это нужно? Все вы наверное знаете (надеюсь), что в классическом fixed-horizon подходе нельзя подглядывать в A/B-тесты и останавливать их как только p-value пробил уровень значимости. Коротко - принимая решение таким образом и каждый раз подглядывая в тест (собираясь при этом принять решение) мы как-бы проводим тест снова, и при каждом подглядывании у нас есть вероятность совершить ошибку первого рода. И если так делать, то ошибка первого рода будет не на заданном вами уровне, а гораздо выше (условно, не 0.05, а 0.5) и вы гораздо чаше будете раскатывать тесты, в которых изменений на самом деле нет. 

Тем не менее, иногда подглядеть хочется, и по тесту надо принять решение как можно быстрее. Пример из статьи Netflix - раскатка нового software, которое может повлиять на важные метрики, например play-delay (время от нажатия плей до начала воспроизведения контента). Хочется как можно быстрее понять, что новый софт не сломал ничего в пользовательском опыте. Для таких непрерывных метрик в Netflix придумали метод постоянного сравнения разницы распределений, подробнее читайте в первой статье.   

Во второй статье рассказывается про то, как обходиться с так называемыми counting processes, которые могут моделироваться распределением Пуассона. Это может быть кол-во логинов, кол-во воспроизведений контента или ошибок (и это мы хотели бы отловить как можно скорее). Элегантность их метода в том, что требуется только для числа для постоянного мониторинга p-value - кол-во событий из контроля и из теста. А еще есть код на GitHub и вы можете попробовать метод на своих данных. 

🔎 Я проверил на моделировании, ошибка первого рода действительно сохраняется при постоянном мониторинге. При это мощность таких методов меньше и они позволяют быстро отлавливать только довольно большие разницы. Вот мой код на github (https://github.com/YuriyBalandin/ab_testing/blob/main/sequential_testing_at_netflix_simulation.ipynb). 

Сами статьи: 
- Часть 1 (https://netflixtechblog.com/sequential-a-b-testing-keeps-the-world-streaming-netflix-part-1-continuous-data-cba6c7ed49df)
- Часть 2 (https://netflixtechblog.com/sequential-testing-keeps-the-world-streaming-netflix-part-2-counting-processes-da6805341642)

Код на github (https://gist.github.com/michaellindon/5ce04c744d20755c3f653fbb58c2f4dd) ко второй части. 

Научные статьи с разобранными в блоге методах: 
- Rapid Regression Detection in Software Deployments through Sequential Testing (https://arxiv.org/pdf/2205.14762) 
- Anytime-Valid Inference For Multinomial Count Data (https://openreview.net/pdf?id=a4zg0jiuVi)