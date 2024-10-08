---
tags:
  - data
data_type:
  - Analytics
link: https://habr.com/ru/articles/836776/
source: habr
---
Я обожаю RPG, меня привлекают их богатый сюжет, стратегическая глубина и захватывающие миры. Также меня восхищают data-driven подходы к разработке. Они не только улучшают логическую структуру игровых механик, но и гарантируют, что каждый элемент игры сбалансирован и вносит значимый вклад в опыт игрока. Баланс - один из самых сложных аспектов разработки игр, поскольку он требует тщательного внимания к взаимодействию игровых механик. Сегодня я расскажу о том, как использовать линейную алгебру для баланса стоимости предметов в игре.

## В чём проблема?

Допустим, мы добавили в нашу игры базовые предметы, которые добавляют скорость атаки, урон, ману и т.д. На основе таких предметов довольно легко создать новые: например, в игре существует меч, дающий + 3 урона, стоит он 30 монет. Не составит труда оценить меч, дающий +9 урона: единица урона стоит 10 монет, следовательно, второй меч будет стоить примерно 90 монет. О том, почему примерно, мы поговорим дальше.

Но что если вы набросали несколько предметов, дающих разное количество характеристик, протестировали их, но их стоимость нельзя явно выразить через эти характеристики? Рассмотрим упрощенный пример:

- Топор берсерка: дает 9 единиц урона и 2 единицы скорости атаки, стоит 40 монет,
    
- Кинжал трикстера: 4 урона и 8 скорости атаки, стоит 35 монет,
    
- Меч тамплиера: 5 урона и 5 скорости атаки, стоит 38 монет.
    

Это оружие можно представить в виде линейного уравнения:

![\begin{cases} 9x+2y = 40 \\ 4x +8y = 35 \\ 5x+5y = 38 \end{cases}](https://habrastorage.org/getpro/habr/upload_files/ce2/d2c/526/ce2d2c52621b73062f4877890221ebd0.svg)

Эта система не имеет решения, значит ли это что мы что-то сделали не так и теперь мы не можем балансить игру? Не совсем.

![Система не имеет решения](https://habrastorage.org/r/w1560/getpro/habr/upload_files/34b/c45/3fe/34bc453fef3536dc2ba9e1ac869a2176.png "Система не имеет решения")

Система не имеет решения

## Применим линейную алгебру

Давайте попробуем решить эту систему уравнений иначе. Для этого запишем её в матричной форме:

![A \cdot \vec{x}  = \vec{b},](https://habrastorage.org/getpro/habr/upload_files/286/b94/207/286b94207cb94a85b6ef671f776e984d.svg)

где 𝐴- матрица коэффициентов, 𝑥 - вектор переменных, и 𝑏 - постоянный вектор.

![A = \begin{bmatrix} 9 & 2 \\ 4 & 8\\ 5 & 5 \end{bmatrix}, \vec{x} = \begin{bmatrix} x\\ y\end{bmatrix}, \vec{b} = \begin{bmatrix} 40\\ 35\\38\end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/c3e/163/b60/c3e163b60fcab2490dd66745ec69b553.svg)

Обычно такие системы решаются с помощью обратных матриц:

![\vec{x} = A^{-1} \cdot \vec{b},](https://habrastorage.org/getpro/habr/upload_files/c79/850/290/c79850290cd921a7056b5ddec1b8d8ac.svg)

Дело в том, что все нижеописанные операции можно автоматизировать в каком-нибудь wolfram или математических библиотеках для python, поэтому, если вам неинтересно как именно мы пытаемся найти решение системы уравнений, прочитайте спойлер и листайте до следующего заголовка).

Hidden text

Чтобы определить, имеет ли данная система решение, мы исследуем ранг матрицы A и ранг расширенной матрицы Если ранг A меньше ранга расширенной матрицы, то система противоречива и не имеет решения. Запишем расширенную матрицу:

![\begin{bmatrix}\begin{array}{c|c}         A & \vec{b}  \end{array} \end{bmatrix} =\begin{bmatrix}\begin{array}{cc|c}         9 & 2 & 40 \\  4 & 8 & 35\\  5 & 5 & 38 \\     \end{array} \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/2a8/65f/eab/2a865feab41101df0e90e23748816e20.svg)

Чтобы найти ранг, мы можем выполнить операции над строками матрицы ![𝐴](https://habrastorage.org/getpro/habr/upload_files/f09/5cc/c36/f095ccc362a9b65f379f6fd210b9d1d0.svg), чтобы свести ее к ступенчатому виду:

Вычтем 4/9 раз первый столбец из второго, получим:

![\begin{bmatrix} 9 & 2 \\ 0 & \frac{68}{9}\\ 5 & 5 \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/aa9/077/abf/aa9077abfe51cf9ac3e2aff043b82b1f.svg)

Вычтем 5/9 раз первый столбец из третьего, получим:

![\begin{bmatrix} 9 & 2 \\ 0 & \frac{68}{9}\\ 0 & \frac{40}{9} \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/fee/0c6/504/fee0c6504553edbe843195aa76f3edf1.svg)

Эта ступенчатая форма строк показывает, что матрица ![𝐴](https://habrastorage.org/getpro/habr/upload_files/811/25c/963/81125c96307160c959089e2870e14d0a.svg) имеет полный ранг (ранг 2), что означает, что исходные уравнения линейно независимы.

Теперь рассчитаем ранг расширенной матрицы ![\begin{bmatrix}\begin{array}{c|c}         A & \vec{b}  \end{array} \end{bmatrix}.](https://habrastorage.org/getpro/habr/upload_files/74c/85b/1fc/74c85b1fc27366e896cb40a58c225eac.svg) Для этого также вычтем ![\frac{4}{9}](https://habrastorage.org/getpro/habr/upload_files/3e5/4c6/7c7/3e54c67c702b2abebf1eb04843cd59b0.svg) раз первый столбец из второго и ![\frac{5}{9}](https://habrastorage.org/getpro/habr/upload_files/b0a/fbd/2f1/b0afbd2f1c2d1d119773c6acee856891.svg) раз из третьего:

![\begin{bmatrix}\begin{array}{cc|c}         9 & 2 & 40 \\  0 & \frac{68}{9} & \frac{-55}{9}\\  0 & \frac{40}{9} & \frac{-82}{9} \\     \end{array} \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/b56/2b0/aa1/b562b0aa19afed3c3dc4bfe87d853ec4.svg)

Упростим матрицу ещё сильнее: вычтем из третьего ряда второй, умноженный на ![\frac{40}{68}](https://habrastorage.org/getpro/habr/upload_files/e3e/561/f99/e3e561f99780741cc268064840db37c1.svg):

![\begin{bmatrix}\begin{array}{cc|c}         9 & 2 & 40 \\  0 & \frac{68}{9} & \frac{-55}{9}\\  0 & 0 & \frac{-26}{17} \\     \end{array} \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/c22/f5a/ea0/c22f5aea04ac909576234db58c5de094.svg)

Ранг дополненной матрицы ![\begin{bmatrix}\begin{array}{c|c}         A & \vec{b}  \end{array} \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/082/23e/ec6/08223eec6dfce8d32f7a8c486f3c2d78.svg)равен 3, что больше, чем ранг ![𝐴](https://habrastorage.org/getpro/habr/upload_files/35d/ce8/3bf/35dce83bf435ca8c62255fb092d54e61.svg), который равен 2.

Собственно, мы ещё раз доказали, что система уравнений противоречива и не имеет решения.

## Так что же делать или Мура-Пенроуза

Мы исследовали эту систему уравнений со всех сторон, убедились в том, что она не имеет решений, неужели мы не сможем теперь высчитывать стоимость новых предметов в игре исходя только из их характеристик? Конечно, сможем. Именно на тот случай, когда не существует явного решения, человечество придумало псевдо-решения.

Давайте сначала разберёмся, что именно значит "система не имеет решения". Матричная запись ![\vec{x} = A^{-1} \cdot \vec{b}](https://habrastorage.org/getpro/habr/upload_files/da9/743/50b/da974350bf75aa26bf58ace473746239.svg) предполагает поиск обратной матрицы ![A^{-1}](https://habrastorage.org/getpro/habr/upload_files/c02/127/f72/c02127f72c92e705d1a1c8bb213004b8.svg), но когда система не имеет решения, этой матрицы просто не существует. Однако, в начале-середине прошлого века два математика Элиаким Мур и Роджер Пенроуз сформулировали концепцию псевдообратной матрицы.  

Псевдообращение можно понимать как решение задачи поиска наиболее близкого приближения к правильному решению (простите за тавтологию) системы линейных уравнений по методу наименьших квадратов.

Если говорить по-человечески: хоть мы и не можем найти решение системы уравнений (то есть стоимость каждой характеристики оружия), просто потому что его нет, но мы можем найти наиболее правдоподобное, которое будет наиболее близким к каждому из решений для всех уравнений.

То есть теперь мы имеем следующую формулу:

![\vec{x} = A^{+} \cdot \vec{b},](https://habrastorage.org/getpro/habr/upload_files/568/139/ee3/568139ee3537209c8e8c29946eb83921.svg)

Дальше мы найдём псевдообратную матрицу. Если вам неинтересен и этот процесс, то снова прочитайте спойлер и листайте к результату.

Hidden text

![A^{+} = \begin{bmatrix} 0.1201 & −0.0399 & 0.0157 \\ −0.0753 & 0.1182 & 0.0411 \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/452/ca1/523/452ca1523e3751bf117b7564a91981ca.svg)

Начнём с формулы расчёта псевдообратной матрицы

![A^{+} = (A^{T}A)^{-1}A^{T}](https://habrastorage.org/getpro/habr/upload_files/ea0/deb/83a/ea0deb83a389e87c3220b5e51e916d86.svg)

Посчитаем ![A^{T}A](https://habrastorage.org/getpro/habr/upload_files/91c/d02/cb5/91cd02cb5c7822baf087cd0ed084d793.svg):

![A^{T}A = \begin{bmatrix} 9 & 4 & 5 \\ 2 & 8 & 5\end{bmatrix} \begin{bmatrix} 9 & 2\\ 4 & 8 \\ 5 & 5 \end{bmatrix} = \begin{bmatrix} 122 & 86\\ 86 & 93 \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/5a2/651/232/5a26512326d515031fcf17d313ed8adb.svg)

Теперь перейдём к (![A^{T}A)^{-1}:](https://habrastorage.org/getpro/habr/upload_files/bca/ddf/6a4/bcaddf6a4edb4e4a570c4f00b89ceb00.svg)

![(A^{T}A)^{-1} = \frac{1}{det(A^{T}A)} adj(A^{T}A),](https://habrastorage.org/getpro/habr/upload_files/dfb/3dd/af3/dfb3ddaf37781183f743f0c876e3e599.svg)

где ![det(A^{T}A)](https://habrastorage.org/getpro/habr/upload_files/a88/4fc/937/a884fc937fff838c72ecab32cffd5d5d.svg)- определитель ![A^{T}A](https://habrastorage.org/getpro/habr/upload_files/9b1/4da/f91/9b14daf911e90f683f92259b4617ddb8.svg), а ![adj(A^{T}A)](https://habrastorage.org/getpro/habr/upload_files/0ec/b56/0a0/0ecb560a04dbfced57e0149ad45c31c4.svg) - сопряжённая матрица

![det(A^{T}A) = 122\cdot93−86\cdot86=4042−7396=−3354 \\ adj(A^{T}A) = \begin{bmatrix} 93 & -86\\ -86 & 122 \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/f62/83f/b14/f6283fb1402d369f95d991303cd9b33a.svg)

Ну и отсюда следует, что:

![(A^{T}A)^{-1} = \frac{1}{−3354} \begin{bmatrix} 93 & -86\\ -86 & 122 \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/079/a1b/b7a/079a1bb7abc6efeb82bab3b771290f7c.svg)

Теперь вычислим саму псевдообратную матрицу:

![A^{+} =\frac{1}{−3354} \begin{bmatrix} 93 & -86\\ -86 & 122 \end{bmatrix}  \begin{bmatrix} 9 & 4 & 5 \\ 2 & 8 & 5\end{bmatrix} \\ A^{+} = \begin{bmatrix} 0.1201 & −0.0399 & 0.0157 \\ −0.0753 & 0.1182 & 0.0411 \end{bmatrix}](https://habrastorage.org/getpro/habr/upload_files/e4c/421/0fb/e4c4210fb3d914d427ce18f276134833.svg)

## Результат

значения, которые мы получаем из ![\vec{x}](https://habrastorage.org/getpro/habr/upload_files/67e/275/59c/67e27559c3fbe72c19d16ae16f08e89d.svg) представляют собой оптимальную стоимость каждой единицы урона и скорости атаки, рассчитанную по методу наименьших квадратов. Эти значения могут не полностью совпадать с первоначальной стоимостью предметов, но они предлагают последовательную стратегию ценообразования, которая наилучшим образом уравновешивает различные атрибуты всех предметов.

Приближенное решение с использованием псевдоинверсии Мура-Пенроуза дает нам следующую стоимость характеристик:

- Стоимость единицы урона (![x](https://habrastorage.org/getpro/habr/upload_files/6a8/ddc/5bf/6a8ddc5bf56b76394c60897356d95308.svg)): примерно 4.01 монеты
    
- Стоимость единицы скорости атаки (![y](https://habrastorage.org/getpro/habr/upload_files/6b0/1b9/cd4/6b01b9cd474c6c2a195f003b9f995413.svg)): примерно 2.68 монеты
    
    Эти значения говорят о том, что для баланса затрат по всем предметам каждое очко урона должно стоить 4.01 монеты, а каждое очко скорости атаки 2.68 монеты.
    

## Вывод

Используя псевдоинверсию Мура-Пенроуза, мы можем устранить несоответствие в исходной стоимости предметов и найти приближенное решение, которое уравновешивает стоимость каждого атрибута для всех предметов.

Однако, помните, что результаты, полученные в результате вычислений не обязательно должны быть истиной в последней инстанции. Экспериментируйте, накладывайте на вычисленные стоимости предметов шум, тестируйте это всё в игре, ведь в конце концов главное - не справедливость распределения стоимости атрибутов и максимальная выверенность всех метрик, а фан от игры.

Если вам интересны такого рода посты, можете подписаться на моq [telegram](https://t.me/robowasps), там я пишу посты поменьше