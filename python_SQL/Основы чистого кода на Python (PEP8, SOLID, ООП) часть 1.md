---
tags:
  - data
link: https://habr.com/ru/articles/836678/
source: habr
---
Когда вы уже написали несколько своих небольших пет-проектов, вы начинаете понимать что чистый код, архитектура и другие паттерны программирования начинают иметь смысл. В масштабируемых, командный или коммерческих проектах это несет особую ценность. Изучив эти принципы, новички получат представление о построении надежных, гибких и легко тестируемых приложений, что позволит им сохранить ясность кодовой базы и возможность ее сопровождения по мере роста их проектов.

В этой статье мы разберем: что такое PEP8, poetry, как создавать архитектуру python-приложения, какие существуют методологии Driven Development и как писать чистый код на Python.

Современные реалии требуют высокой читаемости кода - на работе или Open Source, а может даже в личных пет-проектах. Ведь проект скорее всего придется переписывать и расширять, и без банальных знаний принципов ООП может получиться спагетти-код, который полон костылей и чрезмерной сложностью. Конечно, за счет чистой архитектуры может незначительно понизиться производительность приложения, поэтому иногда приходиться жертвовать читабельностью, или наоборот, скоростью работы проекта.

Давайте начнем с малого - с дзена питона. Именно в нем даются основные идеи по написанию кода на python:

```
Красивое лучше, чем уродливое.
Явное лучше, чем неявное.
Простое лучше, чем сложное.
Сложное лучше, чем запутанное.
Плоское лучше, чем вложенное.
Разреженное лучше, чем плотное.
Читаемость имеет значение.
Особые случаи не настолько особые, чтобы нарушать правила.
При этом практичность важнее безупречности.
Ошибки никогда не должны замалчиваться.
Если они не замалчиваются явно.
Встретив двусмысленность, отбрось искушение угадать.
Должен существовать один и, желательно, только один очевидный способ сделать это.
Хотя он поначалу может быть и не очевиден, если вы не голландец.
Сейчас лучше, чем никогда.
Хотя никогда зачастую лучше, чем прямо сейчас.
Если реализацию сложно объяснить — идея плоха.
Если реализацию легко объяснить — идея, возможно, хороша.
Пространства имён — отличная штука! Будем делать их больше!
```

Принцип "Должен существовать один и, желательно, только один очевидный способ сделать это." является отсылкой на язык Perl, девизом которого был "There is more than one way to do it" (Существует более одного способа сделать это).

Кстати, код дзена питона противоречит дзену питона:

```
s = """Gur Mra bs Clguba, ol Gvz CrgrefOrnhgvshy vf orggre guna htyl.Rkcyvpvg vf orggre guna vzcyvpvg.Fvzcyr vf orggre guna pbzcyrk.Pbzcyrk vf orggre guna pbzcyvpngrq.Syng vf orggre guna arfgrq.Fcnefr vf orggre guna qrafr.Ernqnovyvgl pbhagf.Fcrpvny pnfrf nera'g fcrpvny rabhtu gb oernx gur ehyrf.Nygubhtu cenpgvpnyvgl orngf chevgl.Reebef fubhyq arire cnff fvyragyl.Hayrff rkcyvpvgyl fvyraprq.Va gur snpr bs nzovthvgl, ershfr gur grzcgngvba gb thrff.Gurer fubhyq or bar-- naq cersrenoyl bayl bar --boivbhf jnl gb qb vg.Nygubhtu gung jnl znl abg or boivbhf ng svefg hayrff lbh'er Qhgpu.Abj vf orggre guna arire.Nygubhtu arire vf bsgra orggre guna *evtug* abj.Vs gur vzcyrzragngvba vf uneq gb rkcynva, vg'f n onq vqrn.Vs gur vzcyrzragngvba vf rnfl gb rkcynva, vg znl or n tbbq vqrn.Anzrfcnprf ner bar ubaxvat terng vqrn -- yrg'f qb zber bs gubfr!"""d = {}for c in (65, 97):    for i in range(26):        d[chr(i+c)] = chr((i+13) % 26 + c)print("".join([d.get(c, c) for c in s]))
```

Хотя иногда некоторые фразы шуточные, но явно понятно, что это основные принципы и отличия кода на python.

В этой статье я хочу рассмотреть несколько идей по написанию чистой архитектуры Python-приложений, по возможности покажу примеры кода и многое другое.

Среди основных правил, стандартов по написанию кода можно выделить Clean Code и PEP8.

Чистый Код (Clean Code) - это код, который просто читать и просто изменять. Определение было введено Робертом Мартином в начала 2000-х и описано в его одноимённой книге. Оно появилось, как противоположность плохому или “грязному” кода. Характеристики Чистого Кода: Хорошо решает одну задачу и является таким решением, к которому нечего добавить.

В основе чистого кода лежит несколько принципов:

- Чистый код важен так же как и производительность, функциональность, оптимизация и дебаг.
    
- Код относительно легко читается любым разработчиком - будь это сеньор или джун.
    
- Код можно расширить и изменить в любой момент, и сделать это сможет любой разработчик.
    
- Код не должен иметь проблемы, сюрпризы. Только то, что он и должен делать по алгоритму, то что от него ожидается.
    
- Автор кода должен заботиться о нем, комментировать, тестировать и оптимизировать.
    

[Основные правила PEP8](https://pep8.org/) — это набор рекомендаций по оформлению кода на Python, который помогает сделать код более читаемым и понятным. PEP8 — это просто "style guide".

PEP8 важен для написания качественного кода на Python по нескольким причинам. Во-первых, он помогает сделать код более читаемым и понятным для других программистов, которые могут работать с вашим кодом. Это особенно важно, если вы работаете в команде или если ваш код будет использоваться другими людьми.

Во-вторых, соблюдение стандартов PEP8 может помочь сделать ваш код более консистентным. Это означает, что ваш код будет выглядеть более единообразно и просто, что упрощает его понимание и обслуживание.

В-третьих, соблюдение стандартов PEP8 может помочь обнаружить ошибки и потенциальные проблемы в вашем коде. Например, если вы используете нестандартное именование переменных или не соблюдаете правила отступов, это может привести к ошибкам или проблемам при чтении вашего кода.

Поговорим об импортах и самом стиле кода.

Структура импортов обычно такая:

```
Встроенные библиотеки;

Внешние библиотеки;

Локальные модули;
```

Например:

```
from enum import Enumfrom string import ascii_lowercasefrom rich import printfrom mymodule.logging import Logger
```

Разные типы библиотек отделяются одним абзацем, а также желательно сортировать по алфавиту. Также желательно не использовать конструкцию `from module import *`.

По функциям и классам - их надо отделять. По два абзаца сначала и в конце:

```
my_var = 'VARIABLE'class ExampleClass:	# ...def main():	# ...if __name__ == '__main__':	main()
```

Условный оператор в конце - это конструкция нужна для того, чтобы вызывать какие либо функции только при прямом запуске скрипта, а не через импорт.

## Наименование

Начнем с наименования разных объектов. Пройдемся по базе:

- классы - CamelCase
    
- переменные - snake_case в нижнем регистре
    
- константы - snake_case, но в верхнем регистре
    
- функции - snake_case в нижнем регистре
    

Но на этом, конечно, правила не заканчиваются.

Переменные описываются именем существительным. Они должны отвечать на вопросы для чего они используются, какую роль выполняют, как они используются в дальнейшем.

```
APPLES_IN_BOX = 10bananas_out_box = 10class GeometricFigure:	def __init__(self, name: str, x_coord: int, y_coord: int):		self.name = name		self.x_coord = x_coord		self.y_coord = y_coord	def change_coordinates(self, new_x_coord: int, new_y_coord: int):		self.x_coord = new_x_coord		self.y_coord = new_y_coord
```

Названия должны быть понятными. Если наименование переменной требует комментария - значит наименование не достаточно понятное. Но это не значит что код не надо комментировать - надо сделать его максимально понятным и со стороны документации, и со стороны кода.

Наименования должны быть едины. То есть не надо употреблять в названиях синонимы, нужно использовать одно слово. Например, не надо использовать в контексте подарка слова gift и present, лучше использовать одно слово gift. То есть соблюдайте соглашение об именах. Соблюдение принятого соглашения об именах важно для устранения путаницы, когда другие разработчики работают над вашим кодом. И это относится к именованию переменных, файлов, функций и даже структур каталогов.

```
CLIENT_ID = 0# Неправильноdef get_client_id():	return CLIENT_IDdef get_customer_first_name():	return 'Николай'# Правильноdef get_client_id():	return CLIENT_IDdef get_client_first_name():	return 'Николай'
```

Также названия должны быть подробные и описательные, которые легко читаются. Названия должны быть информативными. Но также помните, длинные имена не всегда описательные.

```
# Неправильноc = ['Лондон', 'Москва', 'Париж', 'Варшава']# илиeuropean_cities_capitals_name_list_in_russian = ['Лондон', 'Москва', 'Париж', 'Варшава']for i in c:	print(i)# Правильноfrom typing import Listcities: List[str] = ['Лондон', 'Москва', 'Париж', 'Варшава']for city in cities:	print(f'Город: {city}')
```

Переменные отражают то, где они используются, а не что реализуют. Следует избегать назначение в качестве переменных символы l (строчная буква эль), O (заглавная латинская буква «o») и I (латинская буква «ай»).

В принципе, правила наименования в PEP8 и Clean Code не сильно различаются. Но в Python есть понятие "скрытых", приватных методов, которые начинаются со знака нижнего подчеркивания (`_`),

Также стоит поговорить о Magic Numbers - числах, которые непонятно откуда взялись. Самый простой пример:

```
from random import randint# Неправильноdef rolling_dice():	return randint(1, 6)# Правильноdice_sides_count = 6def rolling_dice():	return randint(1, dice_sides_count)
```

Конечно же есть исключения, числа, которые могут быть понятны по контексту. Например 1024 - это число байтов.

Названия неиспользуемых переменных заменяются на нижнее подчёркивание. То есть:

```
for _ in range(10):	print('Hi!')
```

Поговорим немного о функциях. В программировании функции находятся на втором уровне абстракции. Они описывают поведение переменных в динамике. Динамика определяется преобразованием входящих данных в исходящие. Входящие данные описываются аргументами (их сравнение будет чуть позже), а исходящие данные с помощью встроенного оператора return.

Есть миф, что функция не должна содержать больше одного блока `try/except`. Он происходит из того, что каждая функция или метод должны делать что-то одно. Это ссылается на принципы UNIX:

- Пишите программы (классы, функции) которые выполняют только одну задачу - и выполняют ее хорошо.
    
- Пишите программы для совместной работы.
    
- Программы взаимодействуют, используя универсальный текстовый интерфейс, а классы — код. Тип — универсальный интерфейс взаимодействия функций и классов.
    

И из-за этого можно подумать что обработка ошибок тоже должна происходить в отдельной функции.

```
from datetime import datetimedef _write_current_datetime_to_file(filepath: str) -> bool:	with open(filepath, 'w') as file:		file.write(f'Current datetime: {datetime.now()}\n')	return Truedef handle_error_for_write_current_datetime_to_file(filepath: str):	try:		write_current_datetime_to_file(filepath)	except FileNotFoundError:		print('That file already did not exist.')
```

В коде нам пришлось сделать функцию записи текущей даты скрытым, и вызывать ее в функции-обработчике.

Ваши функции должны быть простыми и компактными, но это не значит, что они всегда должны делать что-то одно (как бы вы это ни определяли). Вполне нормально, если ваши функции содержат несколько команд try-except и эти команды не охватывают весь код функции.

Но как вы видите, это может создать определенную путаницу в коде, и наименование функции обработки ошибок будет слишком длинным. Поэтому я думаю, что можно использовать несколько блоков обработки исключений в функции. Конечно, многое зависит от контекста и задач, но если код остается читабельным, то можно обойтись и без дополнительных вспомогательных функций.

Со стороны чистого кода, я бы реализовал код выше так:

```
from pathlib import Pathfrom datetime import datetimedef write_current_datetime_to_file(filepath: str) -> bool:	"""	Функция для записи текущей даты и времени в файл.	:param filepath: путь до файла в виде строки	:return: True если успешно записано, в противном случае False	"""	filepath = Path(filepath)	if not filepath.exists():		print('That file already did not exist.')		return False	with open(filepath, 'w') as file:		file.write(f'Current datetime: {datetime.now()}\n')	return True
```

В Python рекомендуется использовать для путей не строки а объекты класса Path. И поэтому вместо обработки исключений мы используем проверку, существует ли файл, и если нет - то возвращаем False.

По функциям также есть мнение, что аргументы-флаги не нужны. То есть:

```
def example_function(flag_argument: bool):	if flag_argument:		# ...	else:		# ...
```

Суть этого заключается в том, что функция должна выполнять только одну задачу. Но из-за аргумента флага теперь две задачи. Иногда да, лучше создать две функции, а иногда можно обойтись одной. Но есть такие моменты, когда можно использовать аргументы флаги, например создавать две функции сортировки данных по дате - в одной сортировка по порядку, а в другом сортировка в обратном порядке, почему бы не использовать одну функцию?

Следующий важный принцип - избегайте дезинформации. Например:

```
from datetime import datetime# Неправильноdef get_date() -> str:	return datetime.now()# Правильноdef get_current_datetime() -> str:	return datetime.now().strftime('%Y-%m-%d %H:%M:%S')
```

Здесь же можно и рассказать о том, что имена функций должны соответствовать их функционалу, а также быть простыми в произношении, иначе вашим коллегам не будет понятно о чем вы говорите.

Также соблюдайте части речи в коде. Классы и переменные должны быть существительными. Значения в Enum классе должны быть прилагательными.

```
from enum import Enumclass CarColor(Enum):	BLACK = 0	GREY = 1	RED = 2	YELLOW = 3	GREEN = 4class Car:	def __init__(self, brand_name: str, model_name: str, color: CarColor):		self.brand_name = brand_name		self.model_name = model_name		self.color = color
```

Нужно избегать таких слов, как Manager/Prosessor/Info/Data, так как они не дают никакой конкретики. Лучше использовать более конкретные словосочетания, например DatabaseConnectionManager.

Код должен быть написан так, чтобы мы его могли прочитать как книгу, как прозаическое произведение.

Также желательно избегать отрицательные функции (условия). То есть:

```
# Неправильноdef not_bigger_than_zero(num: float):	if num > 0:		return False	else:		return Trueif not not_bigger_than_zero(1):	print('bigger!')# Правильноdef bigger_than_zero(num: float):	if num > 0:		return True	else:		return Falseif bigger_than_zero(1):	print('bigger!')
```

Также хорошим тоном является писать короткие функции. Если функция длинная, то ее стоит разбить на несколько других функций. Иначе даже с комментариями код будет понятен только вам (а может даже спустя какое-то количество времени даже вы запутаетесь в своем коде). Функции должны использовать принцип разделения ответственности: одна функция, одна задача.

В Python есть полезная вещь - аннотации типов, type hints. Type hints позволяет указать, какой тип данных будет у переменной. Это полезно использовать, ведь всегда можно понять какой тип данных должен быть передан или возвращен:

```
def is_valid_username_length(username: str) -> bool:	"""	Функция проверки длины имени пользователя.	:param username: имя пользователя	:return: True если больше 4, в противном случае False	"""	if len(username) < 4:		return False	return True
```

Также в определении аннотаций типов поможет встроенный модуль `typing` в python:

```
from typing import Tuple, List, Dict, Unionexample_tuple: Tuple[int] = (1, 2, 3)example_list: List[int] = [1, 2, 3]example_dict: Dict[int, str] = {1: '1', 2: '2', 3BAR_ALIGN_LEFT: '3'}example_union: Union[int, float] = 1 # или 1.0
```

Благодаря этому модулю мы можем прямо указать, что будет в кортеже, списке или словаре. А Union это type hint, когда переменная может иметь несколько значений - в нашем примере int или float. Также вместо Union можно банально использовать следующую конструкцию:

```
example_union: int | float = 1.0 # или 1 # Данная конструкция может быть использована только в python>=3.10
```

Аннотации типов помогают при создании библиотек.

Выше мы говорили о приватных методах. Так вот, в классе приватные методы должны отображаться ниже остальных методов. Желательно использовать следующую структуру класса:

1. `__new__` (если используется)
    
2. `__init__`
    
3. Остальные магические методы
    
4. Public-методы
    
5. Protected-методы
    
6. Private-методы
    

## Комментарии и докстринги (PEP 257)

Комментарии - база, документация - сила. Но иногда отсутствие комментариев лучше, чем плохие комментарии.

Вообще, существует множество разных правил документирования кода - и вы вольны использовать что хотите. Но в некоторых случаях бывает так, сами вы выбрали его, или его выбрали за вас, это лучший способ из оставшихся. Но это зависит целиком от вас.

Нельзя недооценивать важность написания читаемого кода, который является синонимом качественного документирования кода. На данный момент в Python нет «идеального» способа написания докстрингов (строк документации), так же как и нет единого стиля, которого можно придерживаться.

Строка документации - это одна или несколько строк в начале функции. Используется тройные литералы (`"""<docstring>"""`).

Только в случае, если это первый оператор в функции, он может быть распознан компилятором байт-кода Python и доступен как атрибуты объекта времени выполнения с помощью метода `__doc__` или функции help().

```
def say_phrase_to_users(phrase: str, users: list):	"""	Функция для того чтобы обратиться к пользователям в клиенте.	:param phrase: фраза для обращения.	:param users: список пользователей.	"""	for user in users:		print(f'{phrase}, {user}')	print(say_phrase_to_users.__doc__)say_phrase_to_users('Привет', ['Антон', 'Олег', 'Джон', 'Хабраюзер'])help(say_phrase_to_users)
```

```
Привет, Антон
Привет, Олег
Привет, Джон
Привет, Хабраюзер

	Функция для того чтобы обратиться к пользователям в клиенте.

	:param phrase: фраза для обращения.
	:param users: список пользователей.
	
```

- Все модули, классы, методы и функции, включая конструктор `__init__` в пакетах, должны иметь строки документации.
    
- Описания пишутся с заглавной буквы и включают пунктуацию в конце предложения.
    
- Всегда окружайте строки документации двойными кавычками по три раза.
    
- В конце докстринга пустая строка не ставится.
    

Однострочный докстринг прописывает функцию или действие метода как команду, а не как описание функции: """Do this, return that""".

Для написания докстрингов желательно использовать стиль сфинкса (Sphinx). Стиль Sphinx использует синтаксис облегченного языка разметки reStructuredText (reST).

Пример функции:

```
def calculate_percent(num: int | float, percent: int | float) -> int | float:	"""	Эта функция высчитывает процент из числа.		:param num: сумма для высчитывания.	:type num: int | float	:param percent: процент из суммы который надо высчитать.	:type percent: int | float	:rtype: int | float	:return: процент от числа	"""	percentage = (percent * num) / 100	return percentage
```

В Sphinx используется такой же, как и в большинстве языков программирования синтаксис: keyword(reserved word). Наиболее важные ключевые слова:

- param и type: значение параметра и тип его переменной;
    
- return и rtype: возвращаемое значение и его тип;
    
- :raises: описывает любые ошибки, которые возникают в коде;
    
- .. seealso::: информация для дальнейшего чтения;
    
- .. notes::: добавление заметки;
    
- .. warning::: добавление предупреждения.
    

Хотя порядок этих ключевых слов не является фиксированным, (опять же) принято придерживаться вышеуказанного порядка на протяжении всего проекта. Записи seealso, notes и warning не являются обязательными.

## Принципы ООП

Объектно-ориентированная парадигма имеет несколько принципов:

- Данные структурируются в виде объектов, каждый из которых имеет определенный тип, то есть принадлежит к какому-либо классу.
    
- Классы – результат формализации решаемой задачи, выделения главных ее аспектов.
    
- Внутри объекта инкапсулируется логика работы с относящейся к нему информацией.
    
- Объекты в программе взаимодействуют друг с другом, обмениваются запросами и ответами.
    
- При этом объекты одного типа сходным образом отвечают на одни и те же запросы.
    
- Объекты могут организовываться в более сложные структуры, например, включать другие объекты или наследовать от одного или нескольких объектов.
    

Давайте для начала напишем ООП-код и разберем его:

```
from enum import Enumimport datetimeDAYS_IN_YEAR = 365class FuelType(Enum):	"""	Enum-класс с типами топлива	"""	GASOLINE = "Бензин"	DIESEL = "Дизель"	ELECTRIC = "Электричество"	HYBRID = "Гибрид"class VehicleStatus(Enum):	"""	Enum-класс с статусом состояния транспорта	"""	IDEAL = "Идеал"	LIKE_NEW = "Как новая"	USED = "Поддержанный"	DETERIORATING = "Плохой"	URGENT_REPAIR = "Нужен ремонт"	BROKEN = "Сломана окончательно"class Engine:	"""	Класс, представляющий собой двигатель	"""	def __init__(self, model_name: str, fuel_type: FuelType, max_speed_in_km: float, acceleration_time_in_seconds: float, 				max_mileage_in_km: float, fuel_consumption: float, max_fuel_capacity: float):		self.__fuel_type = fuel_type		self.__model_name = model_name		self.__max_speed_in_km = max_speed_in_km		self.__acceleration_time_in_seconds = acceleration_time_in_seconds		self.__max_mileage_in_km = max_mileage_in_km		self.current_mileage_in_km = 0.0		self.__fuel_consumption = fuel_consumption		self.__max_fuel_capacity = max_fuel_capacity		self.current_fuel_level = 0.0	@property	def fuel_type(self) -> FuelType:		return self.__fuel_type	@property	def model_name(self) -> str:		return self.__model_name	@property	def max_speed_in_km(self) -> float:		return self.__max_speed_in_km	@property	def acceleration_time_in_seconds(self) -> float:		return self.__acceleration_time_in_seconds	@property	def max_mileage_in_km(self) -> float:		return self.__max_mileage_in_km	@property	def fuel_consumption(self) -> float:		return self.__fuel_consumption	@property	def max_fuel_capacity(self) -> float:		return self.__max_fuel_capacity	def get_remaining_mileage(self) -> float:		"""		Функция получения остатка доступного киломентража		"""		return self.__max_mileage_in_km - self.current_mileage_in_km	def refuel(self, amount: float):		"""		Заправка двигателя		"""		self.current_fuel_level = min(self.current_fuel_level + amount, self.__max_fuel_capacity)	def drive(self, distance: float) -> bool:		"""		Поездка		"""		if distance <= self.get_remaining_mileage() and distance <= self.current_fuel_level / self.__fuel_consumption * 100:			self.current_mileage_in_km += distance			self.current_fuel_level -= distance / 100 * self.__fuel_consumption			return True		return Falseclass TransportVehicle:	"""	Класс, представляющий собой транспорт	Каждое транспортное средство имеет следующие параметры:	 + vehicle_type - тип транспорта	 + brand_name - имя бренда-производителя	 + model_name - имя модели	 + release_year - год выпуска	 + purchase_price - цена покупки	 + purchase_date - дата покупки	 + warranty_time_in_days - срок действия гарантии в днях	 + engine - объект класса двигателя	 + condition_percentage - процент состояния	 + condition_status - статус состояния	"""	def __init__(self, vehicle_type: str, brand_name: str, model_name: str, release_year: int,				purchase_price: int, purchase_date: datetime.date, warranty_time_in_days: int,				engine: Engine, condition_percentage: float):		self.vehicle_type = vehicle_type		self.brand_name = brand_name		self.model_name = model_name		self.release_year = release_year		self.purchase_price = purchase_price		self.purchase_date = purchase_date		self.warranty_time_in_days = warranty_time_in_days		self.engine = engine		self.condition_percentage = condition_percentage		self.condition_status = self._get_vehicle_condition_status().value	def drive(self, distance):		"""		Поездка. Вызываем метод из Engine и выводим дополнительную информацию		"""		if self.engine.drive(distance):			print(f'Было преодалено {distance}км. Текущий пройденный километраж: '\				f'{self.engine.current_mileage_in_km}/{self.engine.max_mileage_in_km} '\				f'(осталось {self.engine.get_remaining_mileage()}), остаток топлива: '\				f'{self.engine.current_fuel_level}')			return True		else:			print(f'{self.vehicle_type} заглох. Текущий пройденный километраж: '\				f'{self.engine.current_mileage_in_km}/{self.engine.max_mileage_in_km} '\				f'(осталось {self.engine.get_remaining_mileage()}), остаток топлива: '\				f'{self.engine.current_fuel_level}')			return False	def refuel(self, amount: float):		"""		Заправка. Вызываем метод из Engine и выводим дополнительную информацию		"""		print(f'Заправили {amount} топлива.')		self.engine.refuel(amount)	def get_info(self):		"""		Информация о траспорте		"""		description = f'Транспортное средство типа "{self.vehicle_type}": {self.brand_name}' \					f' {self.model_name} {self.release_year} года выпуска (куплена в {self.purchase_date.year} '\					f'году за {self.purchase_price}$). Текущее состояние - {self.condition_status} '\					f'({self.condition_percentage}). Двигатель {self.engine.model_name}: тип топлива '\					f'{self.engine.fuel_type.value}, максимальная скорость {self.engine.max_speed_in_km}км/ч, '\					f'время разгона до 100км {self.engine.acceleration_time_in_seconds}сек, максимальный '\					f'километраж {self.engine.max_mileage_in_km}км, максимальное количество топлива '\					f'{self.engine.max_fuel_capacity} литров, расход {self.engine.fuel_consumption} на 100км.'		return description	def _get_vehicle_condition_status(self):		if self.condition_percentage >= 90:			return VehicleStatus.IDEAL		elif self.condition_percentage >= 80:			return VehicleStatus.LIKE_NEW		elif self.condition_percentage >= 50:			return VehicleStatus.USED		elif self.condition_percentage >= 30:			return VehicleStatus.DETERIORATING		elif self.condition_percentage >= 10:			return VehicleStatus.URGENT_REPAIR		else:			return VehicleStatus.BROKEN	def get_remaining_warranty_time(self) -> str:		"""		Метод получения остатка срока действия гарантии.		Return:			строка с информацией о сроке действии гарантии или сообщением что он истек.		"""		today_datetime = datetime.date.today()		warranty_end_date = self.purchase_date + datetime.timedelta(days=self.warranty_time_in_days)		remaining_warranty_time = warranty_end_date - today_datetime		if remaining_warranty_time.days < 0:			return f'Срок действия гарантии ({self.warranty_time_in_days} дней) истек'		else:			return f'Срок действия гарантии истечет через {remaining_warranty_time.days} дней'class Car(TransportVehicle):	def __init__(self, brand_name: str, model_name: str, release_year: int,				purchase_price: int, purchase_date: datetime.date, warranty_time_in_days: int,				engine: Engine, condition_percentage: float):		super().__init__('Автомобиль', brand_name, model_name, release_year, purchase_price,						purchase_date, warranty_time_in_days, engine, condition_percentage)	def drive(self, distance):		if self.engine.drive(distance):			print(f'Автомобиль проехал {distance}км. Текущий пройденный километраж: {self.engine.current_mileage_in_km}/{self.engine.max_mileage_in_km} (осталось {self.engine.get_remaining_mileage()}), остаток топлива: {self.engine.current_fuel_level}')			self.condition_percentage -= (distance * 0.1) / 100			self.condition_status = self._get_vehicle_condition_status().value			return True		else:			print(f'Автомобиль заглох. Текущий пройденный километраж: {self.engine.current_mileage_in_km}/{self.engine.max_mileage_in_km} (осталось {self.engine.get_remaining_mileage()}), остаток топлива: {self.engine.current_fuel_level}')			return Falseclass Truck(TransportVehicle):	def __init__(self, brand_name: str, model_name: str, release_year: int,				purchase_price: int, purchase_date: datetime.date, warranty_time_in_days: int,				engine: Engine, condition_percentage: float):		super().__init__('Грузовик', brand_name, model_name, release_year, purchase_price,						purchase_date, warranty_time_in_days, engine, condition_percentage)	def drive(self, distance):		if self.engine.drive(distance):			print(f'Грузовик проехал {distance}км. Текущий пройденный километраж: {self.engine.current_mileage_in_km}/{self.engine.max_mileage_in_km} (осталось {self.engine.get_remaining_mileage()}), остаток топлива: {self.engine.current_fuel_level}')			self.condition_percentage -= (distance * 0.1) / 100			self.condition_status = self._get_vehicle_condition_status().value			return True		else:			print(f'Грузовик заглох. Текущий пройденный километраж: {self.engine.current_mileage_in_km}/{self.engine.max_mileage_in_km} (осталось {self.engine.get_remaining_mileage()}), остаток топлива: {self.engine.current_fuel_level}')			return Falseclass Helicopter(TransportVehicle):	def __init__(self, brand_name: str, model_name: str, release_year: int,				purchase_price: int, purchase_date: datetime.date, warranty_time_in_days: int,				engine: Engine, condition_percentage: float):		super().__init__('Вертолет', brand_name, model_name, release_year, purchase_price,						purchase_date, warranty_time_in_days, engine, condition_percentage)	def drive(self, distance):		if self.engine.drive(distance):			print(f'Вертолет пролетел {distance}км. Текущий пройденный километраж: {self.engine.current_mileage_in_km}/{self.engine.max_mileage_in_km} (осталось {self.engine.get_remaining_mileage()}), остаток топлива: {self.engine.current_fuel_level}')			self.condition_percentage -= (distance * 0.1) / 100			self.condition_status = self._get_vehicle_condition_status().value			return True		else:			print(f'Вертолет заглох. Текущий пройденный километраж: {self.engine.current_mileage_in_km}/{self.engine.max_mileage_in_km} (осталось {self.engine.get_remaining_mileage()}), остаток топлива: {self.engine.current_fuel_level}')			return Falsetruck_engine = Engine("TruckEngine X100", FuelType.DIESEL, 180, 30, 800000, 25, 250)truck = Truck('MAZ', 'KAMAZ', 2015, 10000, datetime.date(2000, 3, 3), DAYS_IN_YEAR * 20, truck_engine, 90)print(truck.get_remaining_warranty_time())print(truck.get_info())truck.refuel(100)while truck.drive(100):	print('Проезжаем 100км...')print(truck.get_info())# >>> выводСрок действия гарантии (7300 дней) истекТранспортное средство типа "Грузовик": MAZ KAMAZ 2015 года выпуска (куплена в 2000 году за 10000$). Текущее состояние - Идеал (90). Двигатель TruckEngine X100: тип топлива Дизель, максимальная скорость 180км/ч, время разгона до 100км 30сек, максимальный километраж 800000км, максимальное количество топлива 250 литров, расход 25 на 100км.Заправили 100 топлива.Грузовик проехал 100км. Текущий пройденный километраж: 100.0/800000 (осталось 799900.0), остаток топлива: 75.0Проезжаем 100км...Грузовик проехал 100км. Текущий пройденный километраж: 200.0/800000 (осталось 799800.0), остаток топлива: 50.0Проезжаем 100км...Грузовик проехал 100км. Текущий пройденный километраж: 300.0/800000 (осталось 799700.0), остаток топлива: 25.0Проезжаем 100км...Грузовик проехал 100км. Текущий пройденный километраж: 400.0/800000 (осталось 799600.0), остаток топлива: 0.0Проезжаем 100км...Грузовик заглох. Текущий пройденный километраж: 400.0/800000 (осталось 799600.0), остаток топлива: 0.0Транспортное средство типа "Грузовик": MAZ KAMAZ 2015 года выпуска (куплена в 2000 году за 10000$). Текущее состояние - Как новая (89.60000000000002). Двигатель TruckEngine X100: тип топлива Дизель, максимальная скорость 180км/ч, время разгона до 100км 30сек, максимальный километраж 800000км, максимальное количество топлива 250 литров, расход 25 на 100км.
```

Мы имеем класс двигателя, который передается в класс транспортного средства. И также у нас не просто транспорт - а специальные классы разных типов (автомобиль, вертолет, грузовик), которые наследуются от базового класса транспорта.

Engine имеет приватные параметры, которые можно получить по property-функции,

ООП основывается на четырех фундаментальных принципах: инкапсуляции, наследовании, полиморфизме и абстракции.

Инкапсуляция – механизм сокрытия деталей реализации класса от других объектов. Достигается путем использования модификаторов доступа public, private и protected, которые соответствуют публичным, приватным и защищенным атрибутам.

Наследование – процесс создания нового класса на основе существующего класса. Новый класс, называемый подклассом или производным классом, наследует свойства и методы существующего класса, называемого суперклассом или базовым классом.

Полиморфизм – способность объектов принимать различные формы. В ООП полиморфизм позволяет рассматривать объекты разных классов так, как если бы они были объектами одного класса.

Абстракция – процесс определения существенных характеристик объекта и игнорирования несущественных характеристик. Это позволяет создавать абстрактные классы, которые определяют общие свойства и поведение группы объектов, не уточняя детали каждого объекта.

Одна из основных целей использования абстракции в ООП – повышение гибкости и упрощение разработки. Абстрактный подход помогает создавать интерфейсы и классы, которые определяют только те свойства и методы, которые необходимы для выполнения определенной задачи. Это позволяет создавать более гибкие и масштабируемые приложения, которые легко поддаются изменению и расширению.

Для работы с абстрактными классами в Python используют модуль abc. Он предоставляет:

- `abc.ABC` – базовый класс для создания абстрактных классов. Абстрактный класс содержит один или несколько абстрактных методов, то есть методов без определения (пустых, без кода). Эти методы необходимо переопределить в подклассах.
    
- `abc.abstractmethod` – декоратор, который указывает, что метод является абстрактным. Этот декоратор применяется к методу внутри абстрактного класса. Класс, который наследует свойства и методы от абстрактного класса, должен реализовать все абстрактные методы, иначе он также будет считаться абстрактным.
    

```
from abc import ABC, abstractmethodclass Recipe(ABC):    @abstractmethod    def cook(self):        passclass Entree(Recipe):    def __init__(self, ingredients):        self.ingredients = ingredients    def cook(self):        print(f"Готовим на медленном огне смесь ингредиентов ({', '.join(self.ingredients)}) для основного блюда")class Dessert(Recipe):    def __init__(self, ingredients):        self.ingredients = ingredients    def cook(self):        print(f"Смешиваем {', '.join(self.ingredients)} для десерта")class Appetizer(Recipe):    passclass PartyMix(Appetizer):    def cook(self):        print("Готовим снеки - выкладываем на поднос орешки, чипсы и крекеры")
```

В этом примере наряду с абстракцией используются концепции полиморфизма и наследования.

Наследование заключается в том, что подклассы Entree, Dessert и PartyMix наследуют абстрактный метод cook() от абстрактного базового класса Recipe. Это означает, что все они имеют ту же сигнатуру (название и параметры) метода cook(), что и абстрактный метод, определенный в классе Recipe.

Полиморфизм проявляется в том, что каждый подкласс класса Recipe реализует метод cook() по-разному. Например, Entree реализует cook() для вывода инструкций по приготовлению основного блюда на медленном огне, а Dessert реализует cook() для вывода инструкций по смешиванию ингредиентов десерта. Эта разница в реализации является примером полиморфизма, когда различные объекты могут рассматриваться как объекты, которые относятся к одному типу, но при этом ведут себя по-разному.

### SOLID

SOLID - это самый популярный принцип ООП.

Вот как расшифровывается акроним SOLID:

- S: Single Responsibility Principle (Принцип единственной ответственности). Каждый класс или модуль в программе должен иметь только одну причину для изменения.
    
- O: Open-Closed Principle (Принцип открытости-закрытости). Программные сущности (классы, модули, функции и т.п.) должны быть открыты для расширения, но закрыты для изменения.
    
- L: Liskov Substitution Principle (Принцип подстановки Барбары Лисков). Объекты в программе должны быть заменяемыми на экземпляры их подтипов без изменения корректности программы.
    
- I: Interface Segregation Principle (Принцип разделения интерфейса). Клиенты не должны зависеть от интерфейсов, которые они не используют.
    
- D: Dependency Inversion Principle (Принцип инверсии зависимостей). Зависимости внутри системы должны строиться на основе абстракций, а не деталей.
    

Цель использования принципов SOLID — упростить разработку, сделать её более гибкой и устойчивой к ошибкам.

#### Принцип единственной ответственности

Одна функция, класс, программа или сервис - одна задача. Одна функция не должна быть ответственна за 2 задачи (кроме как в некоторых случаях, всегда надо в первую очередь руководствоваться разумом).

Если класс имеет несколько задач, и потребуется изменить одну задачу - придется изменять весь класс.

```
# Плохой кодclass User:	def __init__(self, name: str, password: str, email: str):		self.name = name		self.password = password		self.email = email		self.email_is_valid = self.validate_email()	def save_in_db(self):		# ...	def validate_email(self):		# ...# Хороший кодclass Validator:	# ...	def check_email(self):		# ...	# ...class DatabaseManager:	# ...	def create_user(self, name, password, email):		# ...	# ...class User:	# ...	def __init__(self, name: str, password: str, email: str):		self.name = name		self.password = password		self.email = email 	# ...validator = Validator()db_manager = DatabaseManager()name = 'John'password = 'qwerty'email = 'johnny@example.com'if validator.check_email(email):	db_manager.create_user(name, password, email)
```

Каждый класс выполняет одну задачу. Валидатор проверяет почту, менеджер БД работает с моделями, а класс пользователя и есть модель.

#### Принцип открытости-закрытости

Программные сущности (классы, модули, функции) должны быть открыты для расширения, но не для модификации.

```
# Плохой кодclients = {	'John': 'VIP',	'Jane': 'favorite',	'Max': 'plain'}for client_name, discount_level in clients.items():	if discount_level == 'VIP':		print(f'{client_name} has discount 50%')	elif discount_level == 'favorite':		print(f'{client_name} has discount 25%')	else:		print(f'{client_name} has discount 10%')
```

То есть в коде выше, который является плохим, нам придется изменять сам код и его структуру для его расширения. То есть для того чтобы расширить, надо модифицировать, а это не соответствует принципу открытости/закрытости.

```
# Хороший кодclass Client:	def __init__(self, name):		self.name = name		self.discount = 10class FavoriteClient:	def __init__(self, name):		self.name = name		self.discount = 25class VIPClient:	def __init__(self, name):		self.name = name		self.discount = 50clients = [VIPClient('John'), FavoriteClient('Jane'), Client('Max')]for client in clients:	print(client.discount)
```

#### Принцип подстановки Барбары Лисков

Необходимо, чтобы подклассы могли бы служить заменой для своих суперклассов.

Цель этого принципа заключаются в том, чтобы классы-наследники могли бы использоваться вместо родительских классов, от которых они образованы, не нарушая работу программы. Если оказывается, что в коде проверяется тип класса, значит принцип подстановки нарушается.

Плохой код:

```
class Client:	def __init__(self, name: str, status_level: int=0):		self.status_level = status_level		self.name = nameclass VIPClient(Client):	def __init__(self, name: str):		super().__init__(name, 1)class SuperDuperPuperUltraProVIPClientGoldEdition(Client):	def __init__(self, name: str):		super().__init__(name, 2)clients = [VIPClient('Anton'), SuperDuperPuperUltraProVIPClientGoldEdition('Oleg')]for client in clients:	if client.status_level == 1:		print('discount 10%')	elif client.status_level == 2:		print('discount 20%')
```

Хороший код:

```
class Client:	def __init__(self, name: str, status_level: int=0):		self.status_level = status_level		self.name = name	def print_discount(self):		print(f'discount {10 * self.status_level}')class VIPClient(Client):	def __init__(self, name: str):		super().__init__(name, 1)class SuperDuperPuperUltraProVIPClientGoldEdition(Client):	def __init__(self, name: str):		super().__init__(name, 2)clients = [VIPClient('Anton'), SuperDuperPuperUltraProVIPClientGoldEdition('Oleg')]for client in clients:	client.print_discount()
```

#### Принцип разделения интерфейса

Создавайте узкоспециализированные интерфейсы, предназначенные для конкретного клиента. Клиенты не должны зависеть от интерфейсов, которые они не используют.

Этот принцип не будем рассматривать на примере, т.к. для питона он не имеет значения из за отсутствия интерфейсов.

#### Принцип инверсии зависимостей

Объектом зависимости должна быть абстракция, а не что-то конкретное.

1. Модули верхних уровней не должны зависеть от модулей нижних уровней. Оба типа модулей должны зависеть от абстракций.
    
2. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.
    

В процессе разработки программного обеспечения существует момент, когда функционал приложения перестаёт помещаться в рамках одного модуля. Когда это происходит, нам приходится решать проблему зависимостей модулей. В результате, например, может оказаться так, что высокоуровневые компоненты зависят от низкоуровневых компонентов.

Вот пример кода, удовлетворяющего данному принципу

```
class Connection:	# ...	def __init__(self, hostname: str, port: int):		self.hostname = hostname		self.port = port	def request(self):		# ...	# ...class HTTPRequest(Connection):	def __init__(self, hostname: str, port: int):		super().__init__(hostname, port)	def request(self):		# ...		status_code = 200		return status_codeclass HTTPSRequest(Connection):	def __init__(self, hostname: str, port: int, ssl_context):		super().__init__(hostname, port)		self.ssl_context = ssl_context	def request(self):		# ...		status_code = 200		return status_code
```

Кроме того, стоит отметить, что следуя принципу инверсии зависимостей, мы соблюдаем и принцип подстановки Барбары Лисков.

## Заключение

Это конец первой части. Во второй мы рассмотрим архитектуру создания приложения, линтеры, инструменты и утилиты, как создать python-пакет, методологии разработки и паттерны проектирования ПО на python. Впереди еще много троп, протоптанных или нелюдимых.

Я был бы рад, если вы присоединитесь к [моему телеграм каналу про Python](https://t.me/+nU8zAJFiW-xlMzg6), если вам не трудно.

### Источники

- [Самоучитель Python - абстракция и полиморфизм](https://proglib.io/p/samouchitel-po-python-dlya-nachinayushchih-chast-19-osnovy-oop-abstrakciya-i-polimorfizm-2023-04-24)
    
- [Принципы SOLID, о которых должен знать каждый разработчик](https://habr.com/ru/companies/ruvds/articles/426413/)
    
- [Докстринги Python - NoP](https://nuancesprog.ru/p/8140/)