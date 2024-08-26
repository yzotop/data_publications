---
tags:
  - data
link: https://habr.com/ru/companies/amvera/articles/834582/
source: habr
---
Всем привет! В этой статье я решил показать один из методов парсинга на Python на примере маркетплейса Wildberries.

Суть подхода в том, что мы будем не разбирать запрошенную html страницу по ссылке, а будем использовать API сайта, который используется сервисом для получения и отображения всех товаров требуемой категории.

В проекте будут использоваться следующие библиотеки:

1. [requests](https://pypi.org/project/requests/) - для парсинга данных API.    
2. [aiogram 3.10.0](https://pypi.org/project/aiogram/) - одна из самых популярных библиотек для разработки telegram ботов.

### Работа проекта

Приложение будет отправлять GET запрос к API, получая на выходе данные о карточках товара. Далее мы отфильтруем требуемые данные, такие как название бренда, название товара и т.п., после чего "завернем" приложение в telegram-бота. И естественно не забудем про деплой.

Wildberries, как и другие крупные сервисы могут блокировать IP бота-парсера, поэтому я предлагаю использовать proxies.

#### Разбор API

Как было уточнено выше - wildberries использует API для получения интересующей нам информации на странице. Давайте перейдем на сайт и откроем любую категорию. Пусть это будет Электроника -> Гарнитура и наушники.

Теперь клавишей откроем инструмент разработчика, переключимся во вкладку network для вывода запросов, которые отправляются сайтом и выберем фильтр Fetch/XHR. Перезагрузим страницу.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/25b/817/d52/25b817d524c3621b45c0746987ee9ce2.png)

Здесь нас интересует запрос, начинающийся на _catalog?_. Кликаем на него, и, изучая содержимое во вкладке Preview, можем перейти по ключу data - products и увидеть содержащиеся внутри данные продуктов!

Теперь мы знаем как выглядит запрос и можем перейти к написанию кода, так как совсем скоро нам понадобятся данные из DevTools.

### Пишем основную логику проекта

Сначала импортируем необходимую библиотеку - requests, объявим переменную для прокси и зададим структуру [main.py](http://main.py/).

```
import requests

proxies = 'ЗАПИШИТЕ-СЮДА-СВОЙ-ПРОКСИ-В-НУЖНОМ-ФОРМАТЕ'

def get_category():
	pass 
	
def format_items(response):
    pass
            
def main():
    pass

if __name__ == '__main__':
    main()
```

Как вы видите, будет использоваться 3 основных функции:

- В get_category() мы укажем url и headers, которые мы получим, **скопировав curl (bash) запроса** через DevTools. И вернем response запроса GET в json:
    

```
def get_category():
    url = 'https://catalog.wb.ru/catalog/electronic14/v2/catalog?ab_testing=false&appType=1&cat=9468&curr=rub&dest=-1185367&sort=popular&spp=30'
    
    headers = {
        'Accept': '*/*',
        'Accept-Language': 'ru-RU,ru;q=0.9,en-US;q=0.8,en;q=0.7',
        'Connection': 'keep-alive',
        'DNT': '1',
        'Origin': 'https://www.wildberries.ru', 
        'Referer': 'https://www.wildberries.ru/catalog/elektronika/igry-i-razvlecheniya/aksessuary/garnitury',
        'Sec-Fetch-Dest': 'empty',
        'Sec-Fetch-Mode': 'cors',
        'Sec-Fetch-Site': 'cross-site',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36',
        'sec-ch-ua': '"Not)A;Brand";v="99", "Google Chrome";v="127", "Chromium";v="127"',
        'sec-ch-ua-mobile': '?0',
        'sec-ch-ua-platform': '"Windows"',
    }
    
    response = requests.get(url=url, headers=headers, proxies=proxies)
    
    return response.json()
```

- format_items() - принимает response запроса. Здесь с помощью цикла **for** пробежимся по данным товаров, предварительно сделав проверки на наличие самих товаров, и запишем все в products:
    

```
def format_items(response):
    products = []
    
    products_raw = response.get('data', {}).get('products', None)
    
    if products_raw != None and len(products_raw) > 0:
        for product in products_raw:
            print(product.get('name', None))
            products.append({
                'brand': product.get('brand', None),
                'name': product.get('name', None),
                'id': product.get('id', None),
                'reviewRating': product.get('reviewRating', None),
                'feedbacks': product.get('feedbacks', None),
            })
            
            
    return products
```

Получаем products_raw (все продукты) с помощью метода get в response. Нам нужно дойти до products, который мы видели в инструменте разработчика в браузере. Для этого сначала получаем содержимое data, а после - самого products.

Что за метод GET и как с ним работать? Сначала берется любой ключ, в нашем случае - это product. И в методе get, первым параметром указывается название следующего ключа/переменной, содержащие в себе какое-то значение. Вторым параметром указывается то, что будет возвращено, если такого ключа/переменной не найдется. Если найдено - возвращается словарь с содержащимися внутри ключа/переменной данными. С помощью append мы записываем словарь с данными в массив products, объявленный в начале функции.

- в main() вызовем функции и попробуем вывести данные карточек:
    

```
def main():
    response = get_category()
    products = format_items(response)
    
    print(products)
```

Теперь можно запустить. На выходе успешно получаем выделенные данные!

## Адаптируем код для работы бота

Когда логика парсера готова, мы можем написать бота. Он будет простым - бот не будет давать пользователю выбрать категорию, а просто отправит 10 карточек по выбранной нами категории.

Добавим нужные импорты:

```
import osimport asyncioimport timeimport loggingfrom aiogram import Bot, Dispatcher, typesfrom aiogram.filters import CommandStartfrom aiogram.enums.parse_mode import ParseModefrom aiogram.types.inline_keyboard_button import InlineKeyboardButtonfrom aiogram.utils.keyboard import InlineKeyboardBuilder
```

Запустим logging и объявим класс бота с диспетчером и переменную прокси:

```
logging.basicConfig(level=logging.INFO)

proxies = os.getenv('PROXIES')

bot = Bot(os.getenv("TOKEN"))
dp = Dispatcher()
```

Так как мы будем загружать бота на облако, я рекомендую использовать переменные окружения.

Теперь немного изменим main(), сделаем его асинхронным и вместо обычного вызова переменной main, используем asyncio для запуска асинхронных функций:

```
async def main():    await bot.delete_webhook(drop_pending_updates=True)    await dp.start_polling(bot)    if __name__ == '__main__':    asyncio.run(main())
```

И самое главное: обработчик команды /start:

```
@dp.message(CommandStart)async def start(message: types.Message):    response = get_category()    products = format_items(response)        items = 0        for product in products:        text=f"<b>Категория</b>: Гарнитуры и наушники\n\n<b>Название</b>: {product['name']}\n<b>Бренд</b>: {product['brand']}\n\n<b>Отзывов всего</b>: {product['feedbacks']}\n<b>Средняя оценка</b>: {product['reviewRating']}"                builder = InlineKeyboardBuilder()        builder.add(InlineKeyboardButton(text="Открыть", url=f"https://www.wildberries.ru/catalog/{product['id']}/detail.aspx"))                await message.answer(text, parse_mode=ParseMode.HTML, reply_markup=builder.as_markup())                if items >= 10:            break        items += 1                time.sleep(0.3)
```

Здесь мы просто заносим все полученные при парсинге данные в самодельную карточку в виде сообщения бота и добавляем кнопку для перехода на страницу товара.

Бот отправляет до 10 самых популярных товаров (фильтр можно задать в параметрах url в первой функции) с перерывами в 0.3 секунды, чтобы API telegram не жаловался на слишком частые отправки сообщений.

Пока бот будет запускаться локально, можно вписать токен и прокси прямо в код, но в дальнейшем лучше использовать переменные окружения.

Запускаем, и при команде старт видим, что все работает!

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/85b/a1c/edb/85ba1cedb70a4e7cbb2c839f543b4c86.png)

Весь код выглядит так:

```
import requests
import os
import asyncio
import time

import logging

from aiogram import Bot, Dispatcher, types
from aiogram.filters import CommandStart
from aiogram.enums.parse_mode import ParseMode
from aiogram.types.inline_keyboard_button import InlineKeyboardButton
from aiogram.utils.keyboard import InlineKeyboardBuilder

logging.basicConfig(level=logging.INFO)

proxies = os.getenv('PROXIES')

bot = Bot(os.getenv("TOKEN"))
dp = Dispatcher()

def get_category():
    url = 'https://catalog.wb.ru/catalog/electronic14/v2/catalog?ab_testing=false&appType=1&cat=9468&curr=rub&dest=-1185367&sort=popular&spp=30'
    
    headers = {
        'Accept': '*/*',
        'Accept-Language': 'ru-RU,ru;q=0.9,en-US;q=0.8,en;q=0.7',
        'Connection': 'keep-alive',
        'DNT': '1',
        'Origin': 'https://www.wildberries.ru', 
        'Referer': 'https://www.wildberries.ru/catalog/elektronika/igry-i-razvlecheniya/aksessuary/garnitury',
        'Sec-Fetch-Dest': 'empty',
        'Sec-Fetch-Mode': 'cors',
        'Sec-Fetch-Site': 'cross-site',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36',
        'sec-ch-ua': '"Not)A;Brand";v="99", "Google Chrome";v="127", "Chromium";v="127"',
        'sec-ch-ua-mobile': '?0',
        'sec-ch-ua-platform': '"Windows"',
    }
    
    response = requests.get(url=url, headers=headers, proxies=proxies)
    
    return response.json()

def format_items(response):
    products = []
    
    products_raw = response.get('data', {}).get('products', None)
    
    if products_raw != None and len(products_raw) > 0:
        for product in products_raw:
            products.append({
                'brand': product.get('brand', None),
                'name': product.get('name', None),
                'id': product.get('id', None),
                'reviewRating': product.get('reviewRating', None),
                'feedbacks': product.get('feedbacks', None),
            })
            
    return products
            
@dp.message(CommandStart)
async def start(message: types.Message):
    response = get_category()
    products = format_items(response)
    
    items = 0
    
    for product in products:
        text=f"<b>Категория</b>: Гарнитуры и наушники\n\n<b>Название</b>: {product['name']}\n<b>Бренд</b>: {product['brand']}\n\n<b>Отзывов всего</b>: {product['feedbacks']}\n<b>Средняя оценка</b>: {product['reviewRating']}"
        
        builder = InlineKeyboardBuilder()
        builder.add(InlineKeyboardButton(text="Открыть", url=f"https://www.wildberries.ru/catalog/{product['id']}/detail.aspx"))
        
        await message.answer(text, parse_mode=ParseMode.HTML, reply_markup=builder.as_markup())
        
        if items >= 10:
            break
        items += 1
        
        time.sleep(0.3)
            
async def main():
    await bot.delete_webhook(drop_pending_updates=True)
    await dp.start_polling(bot)
    
if __name__ == '__main__':
    asyncio.run(main())
```

### Деплой бота в Amvera

Разворачивать нашего бота мы будем в сервисе [Amvera](https://amvera.ru/?utm_source=habr&utm_medium=article&utm_campaign=bot-parser).

[Amvera](https://amvera.ru/?utm_source=habr&utm_medium=article&utm_campaign=bot-parser) - одно из лучший решений для быстрого деплоя ботов из-за своей простоты в загрузке файлов (через git или интерфейс) и простоты настройки (вам не нужно настраивать виртуальную машину, зависимости и т.д.). Единственное, что надо настроить - переменные окружения, задать параметры в конфигурации и создать файл зависимостей (requirements.txt). Хоть это и звучит страшно, но на деле делается в пару кликов.

В Amvera вы сможете быстро обновлять код через git (консольную утилиту или интерфейс IDE) буквально за 3 команды.

Еще одна из интересных особенностей Amvera - то, что деньги не будут сгорать, если проект не работает из-за почасовой тарификации. Если ваше приложение будет работать час - спишется за час. 20 минут - спишется за 20 минут и так далее. Цена в тарифе указана, если приложение будет работать весь месяц без остановок.

Для начала зарегистрируемся по [ссылке](https://amvera.ru/?utm_source=habr&utm_medium=article&utm_campaign=bot-parser). После подтверждения номера телефона и почты, вам начисляется **111 рублей на баланс**.

Открываем страницу проектов и нажимаем кнопку “Создать”. В открывшемся окне прописываем название проекта на латинице или кириллице, выбираем понравившийся тариф, тип сервиса оставляем “Приложение”.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/5ea/4b3/b04/5ea4b3b042143b55299ff6ea9204597b.png)

Нажимаем далее. Теперь доступно окно загрузки данных. Можно загрузить сейчас через интерфейс, а можно через Git. Я рекомендую использовать Git, если в будущем будут обновления проекта.

Выбор не важен - вы в любом случае сможете пользоваться и интерфейсом и Git.

Нажимаем далее и нас встречает окно с конфигурацией. Это и есть инструкции для запуска проекта. Все просто - Выбираем окружение Python, инструмент pip, указываем версию python, название запускаемого скрипта и все. Больше нам ничего настраивать не нужно.

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/fb2/f26/481/fb2f264813fc0586d874122dcf824f32.png)

Завершаем настройку проекта и теперь нам остается только создать файл зависимостей и загрузить данные.

Файл зависимостей создается просто - нужно указать название библиотеки и ее версию в формате

```
библиотека1==версия1
библиотека2==версия
```

В нашем случае requirements.txt

```
aiogram==3.10.0
requests==2.32.3
```

#### Финальная настройка и доставка кода в репозиторий (GIT)

Перед загрузкой файлов нужно настроить переменные окружения. Нужно зайти на страницу проекта и перейти во вкладку “Переменные”, где создать секреты с указанным в коде названием и нужным значением.

Мы готовы к загрузке кода! Если хотите - можно быстро загрузить через интерфейс сайта во вкладке “Репозиторий”, но я воспользуюсь git.

Я покажу последовательность команд для первого коммита и отправки (пуша) файлов:

1. `git init` - инициализирует git локально
    
2. `git remote add amvera https://git.amvera.ru/Ваш_Ник/Имя_проекта` - команда для подключения к репозиторию Amvera. Команду можно найти во вкладке “Репозиторий”
    
3. `git add .` - добавляет все файлы в локальном репозитории
    
4. `git commit -m "Комментарий"` - первый коммит
    
5. `git push amvera master` - пуш в репозиторий.
    

Иногда могут возникнуть проблемы. [Здесь](https://docs.amvera.ru/applications/git/freq-errors.html) собраны возможные ошибки и решения, связанные с Git.

При пуше автоматически начинается сборка. Если вы загрузили через интерфейс - перейдите во вкладку “Конфигурация” и нажмите кнопку “Собрать”. Важно именно собирать проект при обновлении кода/конфигурации.

## Итог

Когда приложение собралось - вам осталось дождаться запуска бота и проверить его работоспособность.

Cегодня мы научились парсить страницу, а точнее использовать для этого API маркетплейса, получая данные о карточках, добавили функционал в бота и научились развертывать минимальное приложение в [Amvera](https://amvera.ru/?utm_source=habr&utm_medium=article&utm_campaign=bot-parser).