---
link: https://habr.com/ru/articles/833494/
tags:
  - data
source: habr
---
[GitHub - ArtUkrainskiy/PythonConsoleMinesweeper: Minesweeper in 66 Lines of Python and Automatic Solver](https://github.com/ArtUkrainskiy/PythonConsoleMinesweeper)

Идея написать данную статью пришла после прочтения статьи [Реализация сапёра в 100 строках чистого Ruby](https://habr.com/ru/companies/ruvds/articles/830778/). Во-первых, мне показалось, что 100 строк кода многовато для такой простой по механике игры. Я бы мог написать более компактное решение на чистом С. Во-вторых, реализация не совсем корректна: в оригинальной игре нельзя проиграть первым ходом, более того, первая открытая ячейка не должна иметь в соседних ячейках мину.

Помимо реализации самой головоломки, было бы интересно написать алгоритм, который её решает. Для этого создадим вероятностный алгоритм, который хорошо с этим справляется.

### Определим требования

Перед тем как приступить к написанию кода, зафиксируем требования к реализации:

- Программа представляет собой консольный вариант головоломки Сапёр. Игровое поле печатается в stdout, колонки и поля нумеруются, каждый шаг консоль очищается от предыдущих данных.
    
- При запуске программа принимает в качестве аргументов размер поля (ширину и высоту) и количество мин. По умолчанию, без передачи аргументов, создается игровое поле размером 10x10 с 10 минами.
    
- Управление происходит путем передачи команд в стандартный ввод (stdin) в формате `row col`.
    
- Первым ходом нельзя проиграть, т.е. ячейка, выбранная в первом ходе, не должна содержать мину. Также соседние ячейки не должны содержать мины.
    
- Победа засчитывается, когда на игровом поле все незаминированные ячейки раскрыты, поражение — когда игрок открывает ячейку с миной. Последним ходом выводится соответствующее сообщение о победе или поражении и полностью открытое игровое поле.
    
- Необходимо реализовать алгоритм решения головоломки. На каждом шаге программа печатает текущее состояние игры с сопутствующей справочной информацией.
    

Требования получились весьма формальными, в самой реализации я не буду стремиться к минимальному количеству строк кода, а постараюсь соблюсти баланс между компактностью, читаемостью и производительностью.

### Проектирование реализации

Для начала хорошо бы разобраться, что собой представляет "Сапёр" с точки зрения игровой логики. Основой игры является двумерное поле произвольного размера, клетки которого могут содержать мины. При старте игры все клетки закрыты, задача игрока — открыть все клетки, не содержащие мины, и не "подорваться". Когда клетка открыта, на её месте появляется счётчик, указывающий количество мин в соседних клетках. Если вокруг клетки нет мин, автоматически раскрываются все соседние клетки. На основании этой информации игрок должен принять решение, какую клетку открывать следующей.

Итак, нам необходимо хранить состояние всех клеток игрового поля. Каждая клетка может быть:

- открытой или закрытой
    
- содержать или не содержать мину
    
- знать количество мин в соседних клетках
    

Поскольку наша приоритетная цель — написать компактный алгоритм, можем рассмотреть два варианта хранения состояния клетки: структура с соответствующими полями или один байт данных, в который мы закодируем все состояния.

**Структура данных**

```
class Cell:    def __init__(self):        self.is_mine = False        self.revealed = False        self.adjacent_mines = 0
```

Тут все интуитивно понятно: три поля на каждую характеристику, легко читать и работать с этим. Всего 5 строк кода.

**Один байт на клетку**

При необходимости мы можем закодировать состояние клетки всего в один байт. Хоть это и будет сложнее для восприятия, но правила такие:

- Отрицательные числа от -1 до -9 будут представлять закрытые незаминированные клетки, где абсолютное значение числа минус один указывает на количество соседних мин.
    
- Положительные числа от 1 до 9 будут представлять открытые клетки.
    
- Значение 0 будет обозначать клетку с миной.
    

Поскольку открытие клетки с миной будет считаться финальным шагом, нам не нужно различать открытые или закрытые заминированные клетки.

Я выберу вариант с хранением состояния в одном байте. Осталось определить, в какой структуре данных будем хранить состояние поля. Первое, что приходит в голову, это, конечно же, двумерный массив. Однако есть альтернатива — плоский список, а Python просто король списков, поэтому никаких двумерных массивов.

### Пишем код

Объявим класс `Minesweeper`, его поля и методы:

```
class Minesweeper:    def __init__(self, rows, cols, mines):        self.size = rows * cols        self.rows = rows        self.cols = cols        self.mines = mines        self.board = [-1] * (self.size)  # Инициализация значениями -1        self.first_step = True    def play(self):      pass        def place_mines(self, safe_row, safe_col):      pass        def print_board(self, reveal_mines=False):      pass        def get_neighbors(self, idx):      pass        def reveal(self, idx):      pass
```

Конструктор принимает в качестве аргументов размер поля и количество мин, а также инициализирует список ячеек значением `-1`. Это значение соответствует закрытой клетке с нулевым количеством мин по соседству. Для этого мы просто умножаем список из одного элемента на количество необходимых элементов.

Для работы с плоским списком нам нужны следующие простые формулы.

Индекс элемента по его двумерным координатам:

```
index = row * cols + col
```

Столбец элемента по его индексу:

```
col = index % cols
```

Строка элемента по его индексу:

```
row = index // cols
```

Зная это, можно написать метод получения соседних клеток по индексу клетки:

```
def get_neighbors(self, index):    # Вычисляем строку и столбец из индекса    row = index // self.cols    column = index % self.cols    # Инициализируем список для хранения индексов соседних клеток    neighbors = []        # Проходим по всем возможным соседям    for i in range(row - 1, row + 2):  # r - 1, r, r + 1        for j in range(column - 1, column + 2):  # c - 1, c, c + 1            # Проверяем, чтобы не выйти за пределы поля            if 0 <= i < self.rows and 0 <= j < self.cols:                # Проверяем, чтобы не добавить саму клетку                if i != row or j != column:                    # Вычисляем индекс соседа и добавляем его в список                    neighbor_index = i * self.cols + j                    neighbors.append(neighbor_index)        return neighbors
```

Получилось довольно многословно. В Python есть множество инструментов для удобной работы со списками, включая их генерацию. Существует так называемый [list comprehensions](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions), это не то же самое, что обычные [генераторы](https://docs.python.org/3/reference/expressions.html#generator-expressions). Различие в том, что генераторы создают последовательности значений по мере необходимости (лениво), тогда как list comprehensions создают и размещают их в памяти сразу. Отличить генератор от list comprehensions просто: выражение генератора заключается в круглые скобки, а list comprehensions — в квадратные.

```
def get_neighbors(self, idx):    r, c = divmod(idx, self.cols)    return [i * self.cols + j for i in range(r - 1, r + 2) for j in range(c - 1, c + 2)            if 0 <= i < self.rows and 0 <= j < self.cols and (i != r or j != c)]
```

Обратите внимание, все циклы и условия мы перенесли без изменений.

Пришло время реализовать метод размещения мин на игровом поле. Здесь также не обойдется без генераторов списков:

```
def place_mines(self, safe_row, safe_col):    def count_mines(idx):        return sum(1 for ni in self.get_neighbors(idx) if self.board[ni] == 0)    safe = set(self.get_neighbors(safe_row * self.cols + safe_col) + [safe_row * self.cols + safe_col])    possible = [(i, j) for i in range(self.rows) for j in range(self.cols) if (i * self.cols + j) not in safe]    list(map(lambda pos: self.board.__setitem__(pos[0] * self.cols + pos[1], 0), sample(possible, self.mines)))    self.board = [-1 * (count_mines(idx) + 1) if cell != 0 else cell for idx, cell in enumerate(self.board)]
```

Разберем, что тут происходит:

Для начала нам необходимо определить безопасную зону. Метод принимает координаты безопасной клетки, получает список её соседей и создает список индексов клеток `safe`, в которых мы не будем размещать мины.

Следующей строкой мы генерируем список индексов клеток `possible`, в которых можно разместить мину. Для этого генерируем список индексов всех клеток на игровом поле, которые не находятся в безопасной зоне.

Теперь время размещать мины. Ранее мы пришли к соглашению о том, что клетка, содержащая мину, принимает значение `0`. Чтобы выбрать N случайных элементов списка, воспользуемся функцией `random.sample`. Эта функция стандартной библиотеки `random` принимает в качестве аргумента любую итерируемую последовательность элементов и число, определяющее количество уникальных элементов, которые будут выбраны из переданной последовательности. Мы используем `map`, чтобы применить лямбда-функцию ко всем элементам списка, и `list`, чтобы `map` выполнил всю работу.

Далее нам нужно подсчитать количество мин в соседних клетках. Для этого снова используем генератор списка и для каждого элемента существующего списка проверяем, является ли клетка миной (значение равно нулю), и считаем мины в соседних клетках.

Теперь, когда игровое поле готово, можем перейти к написанию игрового цикла.

Метод `play` должен обрабатывать пользовательский ввод, при первом ходе размещать мины, открывать клетки, осуществлять проверку условий победы и поражения. Строк кода у нас получится больше всего.

```
def play(self):    while True:        print("\033[H\033[J", end="")  # очищаем терминал        self.print_board()        try:            row, col = map(int, input("Enter row and column (0-9): ").split())            assert 0 <= row < self.rows and 0 <= col < self.cols        except (ValueError, AssertionError):            print("Invalid input. Please enter numbers between 0 and 9.")            continue        idx = row * self.cols + col        if self.first_step:            self.place_mines(row, col)            self.first_step = False        if self.board[idx] == 0:            print("You hit a mine! Game Over.")            self.print_board(reveal_mines=True)            break        self.reveal(idx)        if all(cell >= 0 for cell in self.board):            print("Congratulations! You've cleared the minefield.")            self.print_board(reveal_mines=True)            break
```

Условие победы очень простое: значения всех клеток на доске должны быть больше либо равны нулю.

В методе `reveal`, который открывает клетку, у нас есть два варианта реализации. Поскольку, если открытая клетка не содержит мин по соседству, нам нужно открыть все соседние клетки, в голову приходит рекурсия. Однако с этим следует быть осторожным, так как при большом размере игрового поля и низкой плотности мин мы можем легко достигнуть максимальной глубины рекурсии. По умолчанию глубина рекурсии в Python ограничена 1000 вызовами. Альтернативой будет использование стека, в который мы будем помещать индексы клеток для раскрытия:

```
def reveal(self, idx):    stack = [idx]    while stack:        current_idx = stack.pop()        self.board[current_idx] = abs(self.board[current_idx])        if self.board[current_idx] - 1 == 0:            stack.extend(ni for ni in self.get_neighbors(current_idx) if self.board[ni] < 0)
```

В начале мы добавляем в стек, который представлен списком, индекс переданной клетки. Далее, пока в стеке есть элементы, мы достаем их. Чтобы клетка считалась открытой, убираем знак в её значении. Если вокруг нет мин, добавляем в стек соседние клетки.

Осталось реализовать метод, который печатает игровое поле.

```
def print_board(self, reveal_mines=False):    max_col_digits = len(str(self.cols - 1))    max_row_digits = len(str(self.rows - 1))    print(" " * (max_row_digits + 2) + " ".join(f"{i:>{max_col_digits}}" for i in range(self.cols)))    for i in range(self.rows):        row = ['X' if reveal_mines and value == 0 else               str(abs(value) - 1) if reveal_mines and value < 0 else               '*' if value <= 0 else str(value - 1)               for value in (self.board[i * self.cols + j] for j in range(self.cols))]        print(f"{i:>{max_row_digits}}: " + " ".join(f"{cell:>{max_col_digits}}" for cell in row))
```

Метод принимает параметр `reveal_mines`, который определяет, будут ли раскрыты позиции мин на поле. Чтобы столбцы и строки были выровнены независимо от их числа, подсчитывается количество цифр в номерах строк и столбцов. Эти данные используются для вставки нужного количества пробелов.

Первой строкой выводится заголовок с нумерацией столбцов: `" " * (max_row_digits + 2)` создаёт начальный отступ, затем номера столбцов печатаются с помощью [f-строк](https://docs.python.org/3/reference/lexical_analysis.html#f-strings).

Далее, в цикле для каждой строки составляется список отображаемых значений клеток с использованием генератора списка:

- Если `reveal_mines is True` и клетка содержит мину, выводится `X`.
    
- Если `reveal_mines is True`, но клетка не раскрыта, выводится её значение.
    
- Если клетка не раскрыта, выводится пустая строка.
    
- Во всех остальных случаях выводится значение клетки (число мин в соседних клетках).
    

Осталась инициализация и запуск:

```
if __name__ == "__main__":    rows, cols, mines = (int(arg) for arg in sys.argv[1:]) if len(sys.argv) > 1 else (10, 10, 10)    game = Minesweeper(rows, cols, mines)    game.play()
```

Программа ожидает три аргумента: количество строк, столбцов и мин. Если аргументы не переданы, используются значения по умолчанию.

На этом реализация самой игры завершена, и весь код уместился всего в 66 строк!

Вот полный листинг кода:

```
import sysfrom random import sampleclass Minesweeper:    def __init__(self, rows, cols, mines):        self.size = rows * cols        self.rows = rows        self.cols = cols        self.mines = mines        self.board = [-1] * self.size        self.first_step = True    def play(self):        while True:            print("\033[H\033[J", end="")  # очищаем терминал            self.print_board()            try:                row, col = map(int, input("Enter row and column (0-9): ").split())                assert 0 <= row < self.rows and 0 <= col < self.cols            except (ValueError, AssertionError):                print("Invalid input. Please enter numbers between 0 and 9.")                continue            idx = row * self.cols + col            if self.first_step:                self.place_mines(row, col)                self.first_step = False            if self.board[idx] == 0:                print("You hit a mine! Game Over.")                self.print_board(reveal_mines=True)                break            self.reveal(idx)            if all(cell >= 0 for cell in self.board):                print("Congratulations! You've cleared the minefield.")                self.print_board(reveal_mines=True)                break    def print_board(self, reveal_mines=False):        max_col_digits = len(str(self.cols - 1))        max_row_digits = len(str(self.rows - 1))        print(" " * (max_row_digits + 2) + " ".join(f"{i:>{max_col_digits}}" for i in range(self.cols)))        for i in range(self.rows):            row = ['X' if reveal_mines and value == 0 else                   str(abs(value) - 1) if reveal_mines and value < 0 else                   '*' if value <= 0 else str(value - 1)                   for value in (self.board[i * self.cols + j] for j in range(self.cols))]            print(f"{i:>{max_row_digits}}: " + " ".join(f"{cell:>{max_col_digits}}" for cell in row))    def place_mines(self, safe_row, safe_col):        def count_mines(idx):            return sum(1 for ni in self.get_neighbors(idx) if self.board[ni] == 0)        safe = set(self.get_neighbors(safe_row * self.cols + safe_col) + [safe_row * self.cols + safe_col])        possible = [(i, j) for i in range(self.rows) for j in range(self.cols) if (i * self.cols + j) not in safe]        list(map(lambda pos: self.board.__setitem__(pos[0] * self.cols + pos[1], 0), sample(possible, self.mines)))        self.board = [-1 * (count_mines(idx) + 1) if cell != 0 else cell for idx, cell in enumerate(self.board)]    def get_neighbors(self, idx):        r, c = divmod(idx, self.cols)        return [i * self.cols + j for i in range(r - 1, r + 2)                for j in range(c - 1, c + 2)                if 0 <= i < self.rows and 0 <= j < self.cols and (i != r or j != c)]    def reveal(self, idx):        stack = [idx]        while stack:            current_idx = stack.pop()            self.board[current_idx] = abs(self.board[current_idx])            if self.board[current_idx] - 1 == 0:                stack.extend(ni for ni in self.get_neighbors(current_idx) if self.board[ni] < 0)if __name__ == "__main__":    rows, cols, mines = (int(arg) for arg in sys.argv[1:]) if len(sys.argv) > 1 else (10, 10, 10)    game = Minesweeper(rows, cols, mines)    game.play()
```

Теперь можно запустить игру:

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/1e4/e76/d10/1e4e76d106604a4a2f7f94356687d0cb.png)

### Алгоритм решения головоломки

Теперь, когда мы создали компактную и эффективную реализацию «Сапёра» на Python, пора разработать алгоритм для автоматического решения головоломки. В этой части статьи мы создадим вероятностный алгоритм, который будет искать безопасные ходы и решать головоломку, минимизируя риск подрыва на мине.

Эвристический алгоритм решения основывается на вероятностных и логических выводах. Вот общий подход к его реализации:

1. Начать с открытия произвольной клетки.
    
2. Оценить открывшиеся клетки и соотнести количество соседних нераскрытых клеток с числом мин по соседству.
    
3. Если количество соседних нераскрытых клеток равно числу мин по соседству, пометить их флагом как заминированные.
    
4. Если количество соседних клеток, помеченных флагом, равно числу мин по соседству, раскрыть оставшиеся соседние клетки как гарантированно безопасные и вернуться к шагу 2.
    
5. Если конфигурация поля не позволяет найти гарантированно безопасные клетки, провести анализ вероятности содержания мин в нераскрытых клетках на основании информации о количестве мин вокруг раскрытых клеток.
    
6. Открыть клетку с минимальной вероятностью содержания мины. Если клетка безопасна, перейти к шагу 2. Если клетка содержит мину — игра проиграна.
    
7. Если количество нераскрытых клеток равно количеству оставшихся мин на доске, игра выиграна.
    

Теперь перейдём к реализации: создадим класс `MinesweeperSolver`, расширяющий базовый класс `Minesweeper`.

```
class MinesweeperSolver(Minesweeper):    def __init__(self, rows, cols, mines):        super().__init__(rows, cols, mines)        self.probabilities = [0] * self.size        self.flagged = [False] * self.size        self.place_mines(int(rows / 2), int(cols / 2))        self.reveal(int(rows / 2 * cols + cols / 2))
```

В конструкторе мы инициализировали списки для хранения вероятностей и флагов, а также сразу раскрыли первую клетку. Для этого выбрана центральная клетка, так как она обычно предоставляет наибольшее количество информации о состоянии поля.

Метод, который анализирует состояние поля и открывает клетки:

```python
def analyze_and_reveal(self):    to_reveal = []    self.probabilities = [0] * self.size    flag_placed = False    for idx, cell in enumerate(self.board):        if cell <= 1:  # клетка не раскрыта или по соседству 0 мин            continue        unrevealed_neighbors = [n for n in self.get_neighbors(idx) if self.board[n] <= 0]        flag_neighbors_count = sum(self.flagged[n] for n in self.get_neighbors(idx))        #  если количество мин вокруг равно значению клетки то помечаем соседей флагом        if cell - 1 == len(unrevealed_neighbors):            for un_idx in unrevealed_neighbors:                if not self.flagged[un_idx]:                    self.flagged[un_idx] = True                    flag_placed = True        for un_idx in unrevealed_neighbors:            # если соседняя клетка не помечена флагом            if not self.flagged[un_idx]:                if cell - 1 == flag_neighbors_count and un_idx not in to_reveal:                    # если количество помеченных флагом клеток вокруг равно значению клетки                    # добавляем ее в список для раскрытия                    to_reveal.append(un_idx)                # подсчитываем вероятность клетки быть заминированной на основании ее значения                # и количества нераскрытых и не помеченных флагом клеток вокруг                self.probabilities[un_idx] += (cell - 1) / (len(unrevealed_neighbors))    if to_reveal:        # открываем гарантированно безопасные клетки        print("Reveal safe cells:", [divmod(safe_idx, self.cols) for safe_idx in to_reveal])        list(map(self.reveal, to_reveal))    elif flag_placed:        # если безопасных клеток нет, но какая-то клетка была помечена флагом, следует проанализировать доску снова        return True    else:        # находим клетку с минимальной вероятностью быть заминированной и пытаемся открыть        probably_safe = (i for i, p in enumerate(self.probabilities) if p > 0 and not self.flagged[i])        idx = min(probably_safe, key=lambda i: self.probabilities[i], default=None)        if idx is not None:            print("Try reveal:", divmod(idx, self.cols), self.board[idx])            if self.board[idx] == 0:                return False            self.reveal(idx)        else:            return False    return True
```

Метод `solve` в цикле вызывает `analyze_and_reveal` и анализирует условия победы и поражения:

```
def solve(self):    while (not_failed := self.analyze_and_reveal()) and not all(cell >= 0 for cell in self.board):        self.print_board()    if not_failed:        print("Congratulations! You've cleared the minefield.")    else:        print("Auto-solver hit a mine! Game Over.")     
```

Алгоритм получился довольно простым, и его можно улучшить, например, используя метод [Монте-Карло](https://ru.wikipedia.org/wiki/%D0%9C%D0%B5%D1%82%D0%BE%D0%B4_%D0%9C%D0%BE%D0%BD%D1%82%D0%B5-%D0%9A%D0%B0%D1%80%D0%BB%D0%BE). Чтобы оценить, насколько хорошо текущая реализация справляется с решением головоломки, предлагаю провести бенчмарк. Для этого протестируем конфигурации с размером поля 10x10 и количеством мин от 10 до 35. Мы проведем 5000 итераций для каждой конфигурации и вычислим процент успешных решений. Результаты можно визуализировать с помощью `matplotlib`, построив график зависимости процента успешных решений от количества мин.

Код теста:

```
if __name__ == "__main__":  tries = 5000  success_rates = []  mine_counts = range(10, 36)  for m in mine_counts:      rows, cols, mines = (int(arg) for arg in sys.argv[1:]) if len(sys.argv) > 1 else (10, 10, m)      stat = 0      for i in range(tries):          solver = MinesweeperSolver(rows, cols, mines)          stat += solver.solve()      success_rate = stat / tries * 100      success_rates.append(success_rate)      print(m, success_rate)  # Построение графика  plt.figure(figsize=(8, 4))  plt.plot(mine_counts, success_rates, marker='o')  plt.title('Success Rate of Minesweeper Solver')  plt.xlabel('Number of Mines')  plt.ylabel('Success Rate (%)')  plt.ylim(0, 100)  plt.grid(True)  plt.show()
```

Результат:

![image.png](https://habrastorage.org/r/w1560/getpro/habr/upload_files/c22/d89/5a2/c22d895a2e8f3715f7c85da58cbb9b6a.png)

Результаты получились довольно хорошими. Стандартная конфигурация 10x10 с 10 минами решается успешно примерно в 95% случаев. Для конфигураций с 20 минами вероятность успешного решения составляет около 30%, а при 30 и более минах алгоритм уже не справляется. Однако метод Монте-Карло может улучшить результаты, поэтому для тех, кто дочитал статью до конца, предлагаю челендж: реализуйте алгоритм, решающий головоломку "Сапёр" с использованием метода Монте-Карло или другого метода, который показывает лучшие результаты по сравнению с текущим!

Код проекта доступен на [GitHub](https://github.com/ArtUkrainskiy/PythonConsoleMinesweeper).