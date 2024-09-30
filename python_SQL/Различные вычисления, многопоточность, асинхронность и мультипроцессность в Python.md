---
link: https://habr.com/ru/companies/sberbank/articles/829098/
tags:
  - data
source: habr
code_language:
  - python
company: Sber
author: Sber
---
Всем привет! Меня зовут Дмитрий Первушин, я лидер Python-компетенций трайба ИСУ в Сбере. 

Эта статья рассчитана на людей, которые уже знакомы с Python, хотя бы на уровне junior+. Я объясню, какие есть отличия и особенности в многопоточности, асинхронности и мультипроцессности в Python, где и когда они используются. Как говорится в пословице: «Всё познаётся в сравнении», именно в таком стиле я подготовил примеры. Кроме этого, буду специально делать ошибки и рассматривать неправильные подходы, чтобы можно было сразу разобраться, убедиться и запомнить, почему так делать нельзя и какой другой подход в этом случае нужно использовать.

## Виды нагрузок и подходы

Выбор между процессами, потоками или применением асинхронного подхода зависит в первую очередь от нагрузки. Она бывает двух видов:

- **CPU bound** — нагрузка на процессор, когда он активно работает, например, при выполнении математических расчётов или вычислениях в «тяжёлых» компьютерных играх.
    
- **I/O bound** — процессор ожидает операции ввода-вывода, не слишком интенсивно работая. Из примеров можно привести запросы к базам данных или API каких-нибудь сервисов, то есть к внешним ресурсам. Другими словами это нагрузка, длительность обработки которой не зависит от скорости работы процессора.
    

**Примечание**: [GIL](https://en.wikipedia.org/wiki/Global_interpreter_lock) (Global Interpreter Lock) — способ синхронизации потоков, который используется в некоторых интерпретируемых языках программирования.

Есть три основных подхода к обработке нагрузки:

- **Многопроцессность** (multiprocessing). У каждого процесса своя область памяти. Более того, каждый процесс — это, в сущности, отдельный интерпретатор Python со своим GIL и соответствующими расходами ресурсов. Дальше в статье мы это проверим.
    
- **Многопоточность** (threading). Все потоки используют общую память. Нагрузка вида CPU bound обрабатывается процессором в один поток — самое интересное, что это работает так именно в Python. В различных языках программирования это может быть реализовано по-разному, увидим это на примере C++. Для обработки нагрузки I/O bound GIL фактически отключается и позволяет работать параллельно, убедимся и в этом.
    
- **Асинхронность** (asyncio). В этом случае используют конкурентные потоки, которые выполняются НЕ параллельно. Можно сказать, что асинхронность — это параллельное ожидание. Работа в один поток накладывает некоторые ограничения на вид нагрузки. Например, главный поток асинхронного алгоритма можно заблокировать, и ниже мы посмотрим, как этого избежать и к чему может привести, если всё же сделать неправильно.
    

### Нагрузка вида CPU bound

**Примечание**: длительность работы алгоритмов может сильно отличаться в зависимости от производительности процессора и системы в целом, особенно когда речь идёт о нагрузке CPU bound.

Начнём с самого простого примера, а именно с алгоритма, который работает в один процесс в котором только один поток. В качестве нагрузки будем суммировать квадратные корни последовательности из 100 млн чисел. Это хорошая нагрузка CPU bound, заодно замерим длительность обработки, чтобы сравнить с другими алгоритмами.

Простой алгоритм на Python

```
import mathimport timefrom typing import ListLOAD = 100_000_000def summarizator(arr: List[int]) -> int:    res = 0    for n in arr:        res += math.sqrt(n)    return resarr = list(range(LOAD))start = time.time()res = summarizator(arr)stop = time.time()print(f"Simple result: {int(res)}, time {stop - start}")
```

Чтобы убедиться, что все алгоритмы работают корректно, будем также выводить сумму корней нашей последовательности. Самый простой подход:

`Simple result: 666666661666, time 7.731382608413696`

Логично предположить, что если распределить нагрузку между двумя процессами, то каждый из них подсчитает по 50 млн чисел последовательности. И благодаря этому длительность работы, должно быть, снизится, ну может и не в два раза, но в полтора уж точно, подумает начинающий программист. Ну что ж проверим это:

Алгоритм на Python, использующий процессы

```
import mathimport timefrom multiprocessing import Manager, Processfrom typing import ListN_JOBS = 2LOAD = 100_000_000def summarizator(arr: List[int], i: int, summ: List[int]) -> None:    begin = int(i * LOAD / N_JOBS)    end = int((i + 1) * LOAD / N_JOBS)    res = 0    for k in range(begin, end):        res += math.sqrt(arr[k])    summ[i] = resarr = list(range(LOAD))start = time.time()with Manager() as manager:    summ = manager.list([0] * N_JOBS)    processes = [None] * N_JOBS    for i in range(N_JOBS):        processes[i] = Process(target=summarizator, args=(arr, i, summ))        processes[i].start()    for i in range(N_JOBS):        processes[i].join()    res = sum(summ)stop = time.time()print(f"{N_JOBS} processes result: {int(res)}, time {stop - start}")
```

Общая длительность:

`2 processes result: 666666661666, time 8.201336145401001`

«Ой!», — скажет начинающий программист, — «как же так получилось?». Расчёт занял больше времени, чем простой алгоритм. А получилось так из-за того, что создавать процессы дорого по ресурсам, потому что у каждого процесса в Python свой интерпретатор и свой GIL. Распараллеливать выгодно, когда расходы на создание дополнительных процессов не превышают выгоды от их совместной работы.

Проверим это и запустим наш алгоритм в четыре процесса `(N_JOBS = 4)`, в результате получим:

`4 processes result: 666666661666, time 4.886035442352295`

Да, теперь действительно быстрее, что и требовалось доказать.

Рассмотрим теперь работу алгоритма, разбивающего нагрузку **на потоки**. В теории мы знаем, что он отработает в один процесс и GIL будет позволять работать только одному потоку, блокируя другие.

Алгоритм на Python, использующий потоки

```
import mathimport timefrom threading import Lock, Threadfrom typing import ListN_JOBS = 4LOAD = 100_000_000def summarizator(arr: List[int], i: int, summ: List[int], lock: Lock) -> None:    begin = int(i * LOAD / N_JOBS)    end = int((i + 1) * LOAD / N_JOBS)    res = 0    for k in range(begin, end):        res += math.sqrt(arr[k])    with lock:        summ[i] = resarr = list(range(LOAD))start = time.time()summ = [0] * N_JOBSthreads = [None] * N_JOBSlock = Lock()for i in range(N_JOBS):    threads[i] = Thread(target=summarizator, args=(arr, i, summ, lock))    threads[i].start()for i in range(N_JOBS):    threads[i].join()res = sum(summ)stop = time.time()print(f"Threads result: {int(res)}, time: {stop - start}")
```

Мы убедились в том, что GIL действительно блокирует параллельное выполнение CPU bound потоков, что видно по длительности работы алгоритма:

`Threads result: 666666661666, time 10.382546186447144`

Проверим, а так ли это работает в других языках программирования, например, в C++.

Алгоритм на С++, использующий потоки

```
#include <iostream>#include <thread>#include <chrono>#include <cmath>#include <vector>using namespace std;using namespace std::chrono;static const int N_JOBS = 4;static const int LOAD = 100000000;void summarizator(int *arr, int i, double *summ){    int begin = i * LOAD / N_JOBS;    int end = (i + 1) * LOAD / N_JOBS;    double res = 0;    for (int k = begin; k < end; k++)    {        res += sqrt(arr[k]);    }    summ[i] = res;}int main(){    int *arr = new int[LOAD];    for (int i = 0; i < LOAD; i++)    {        arr[i] = i;    }    auto start = system_clock::now();    thread threads[N_JOBS];    double summ[N_JOBS];    for (int i = 0; i < N_JOBS; i++)    {        threads[i] = thread(summarizator, arr, i, summ);    }    for (int i = 0; i < N_JOBS; i++)    {        threads[i].join();    }    double res = 0;    for (int i = 0; i < N_JOBS; i++)    {        res += summ[i];    }    auto stop = system_clock::now();    auto elapsed = duration_cast<milliseconds>(stop - start);    cout << "Threads result (C++): " << long(res) << ", time: " << elapsed.count() / 1000.0 << "\n";    return 0;}
```

Этот же алгоритм в C++ отработал как минимум в 10 раз быстрее:

`Threads result (C++): 666666661666, time: 0.322`

**Примечание**: в Python очень много «плюшек»: «резиновые» списки, сборщик мусора и много других удобных вещей, позволяющих быстро на нём разрабатывать, но скорость работы из-за этого, конечно, уменьшается.

Алгоритм в четыре потока отработал честно параллельно, в этом можно убедиться, если посмотреть на гистограмму загрузки процессора (потоки 1, 3, 5 и 6 полностью загружены).

![Гистограмма программы htop, эта утилита используется в основном в Linux-машинах](https://habrastorage.org/r/w1560/getpro/habr/upload_files/cb4/ba4/dab/cb4ba4dab52322c6c0cc3d6f0518d030.png "Гистограмма программы htop, эта утилита используется в основном в Linux-машинах")

_Гистограмма программы_ [_htop_](https://en.wikipedia.org/wiki/Htop)_, эта утилита используется в основном в Linux-машинах_

### Нагрузка вида I/O bound через потоки

Зачем же нам тогда в Python распараллеливание на потоки? Как я уже говорил, GIL блокирует нагрузку CPU bound в один поток, а вот нагрузка вида I/O bound будет работать действительно параллельно.

Для удобства я написал в веб-фреймворке FastAPI простой эндпоинт, который ждёт одну секунду и после этого возвращает ответ:

Простой эндпоинт на FastAPI

```
import asyncioimport uvicornfrom fastapi import FastAPIapp = FastAPI()@app.get("/")async def read_root():    await asyncio.sleep(1)    return {"status": "ok"}if __name__ == "__main__":    uvicorn.run(app, host="0.0.0.0", port=23555)    
```

Для проверки напишем на Python алгоритм, который будет последовательно посылать запросы в одном процессе и только в одном потоке, и замерим длительность его исполнения:

Простой алгоритм на Python с последовательными запросами

```
import timeimport requestsLOAD = 5def get_request(begin: int, end: int):    for i in range(begin, end):        response = requests.get("http://localhost:23555")        print(i, response.json())start = time.time()get_request(0, LOAD)stop = time.time()print(f"Request simple, time: {stop - start}")
```

Что и следовало ожидать, простой алгоритм отработал за 5 секунд:

`Request simple, time: 5.030400514602661`

Разобьём теперь нагрузку на два потока `(N_JOBS = 2)` и посмотрим, сколько в таком случае понадобится времени:

Алгоритм на Python, разделяющий запросы на потоки

```
import timefrom threading import Threadimport requestsN_JOBS = 2LOAD = 5def get_request(begin: int, end: int):    for i in range(begin, end):        response = requests.get("http://localhost:23555")        print(i, response.json())start = time.time()threads = [None] * N_JOBSfor i in range(N_JOBS):    begin = int(i * LOAD / N_JOBS)    end = int((i + 1) * LOAD / N_JOBS)    threads[i] = Thread(target=get_request, args=(begin, end))    threads[i].start()for i in range(N_JOBS):    threads[i].join()stop = time.time()print(f"Request {N_JOBS} threads, time: {stop - start}")
```

Скорость работы действительно увеличилась:

`Request 2 threads, time: 3.0286645889282227`

То есть в первую и вторую секунду обработалось по два запроса, а в третью — оставшийся один.

Попробуем тогда разделить нагрузку на пять потоков `(N_JOBS = 5)`:

`Request 5 threads, time: 1.016575574874878`

Ясно, что с увеличением количества потоков длительность работы не уменьшится, потому что у нас всего пять запросов, которые мы можем запустить одновременно, и каждый из них работает не меньше секунды.

### Нагрузка вида I/O bound через асинхронное исполнение

Как всегда, начнём с простого алгоритма. Перепишем его в этот раз через отдельные методы, в каждом из которых будет один запрос к нашему API, и будем последовательно их запускать:

Простой алгоритм на Python с последовательными запросами, разбитый на два метода

```
import timeimport requestsdef get_request() -> dict:    response = requests.get("http://localhost:23555")    return response.json()def first():    data = get_request()    print(1, data)def second():    data = get_request()    print(2, data)def main():    first()    second()start = time.time()main()stop = time.time()print(f"Request without async/await, time: {stop - start}")
```

Два метода, которые мы синхронно вызвали последовательно отработали за две секунды, как мы и ожидали:

`Request without async/await, time: 2.0116055011749268`

Чтобы понять работу асинхронного алгоритма, Гвидо Ван Россум советовал мысленно убрать все ключевые слова `async` и `await`. Только мы сделаем наоборот и их добавим:

Асинхронный алгоритм на Python

```
import asyncioimport timeimport aiohttpasync def get_request() -> dict:    async with aiohttp.ClientSession() as session:        async with session.get("http://localhost:23555") as response:            return await response.json()async def first():    data = await get_request()    print(1, data)async def second():    data = await get_request()    print(2, data)async def main():    await first()    await second()start = time.time()asyncio.run(main())stop = time.time()print(f"Request with async/await, time: {stop - start}")
```

`Request with async/await, time: 2.013671875`

«Ой!», — опять воскликнет наш начинающий друг, — «мы же столько всего сделали, понаписали везде ключевых слов `async/await`, использовали асинхронные библиотеки `asyncio` и `aiohttp`, а длительность работы не уменьшилась, как же так получилось?». Алгоритм отработал асинхронно и верно. Чтобы методы `first` и `second` отработали конкурентно, их просто нужно правильно запустить, например через `asyncio.gather`, или можно завернуть их в `asyncio.Task`. Сделаем так и посмотрим, что будет:

Асинхронный алгоритм на Python с конкурентным исполнением

```
import asyncioimport timeimport aiohttpasync def get_request() -> dict:    async with aiohttp.ClientSession() as session:        async with session.get("http://localhost:23555") as response:            return await response.json()async def first():    data = await get_request()    print(1, data)async def second():    data = await get_request()    print(2, data)async def main():    # await asyncio.gather(first(), second())    task1 = asyncio.Task(first())    task2 = asyncio.Task(second())    await task1    await task2start = time.time()asyncio.run(main())stop = time.time()print(f"Request with concurrent async/await, time: {stop - start}")
```

И действительно, получилось быстрее. Два запроса, каждый из которых работает не меньше одной секунды, конкурентно исполнились вместе чуть более чем за одну секунду:

`Request with concurrent async/await, time: 1.011049509048462`

Помните, что асинхронные алгоритмы работают в один поток — главный поток исполнения, — и блокировать его никак нельзя. Другими словами, нельзя включать в асинхронный код какую либо значительную нагрузку вида CPU bound, это может плохо закончиться, потому что пока выполняется эта нагрузка, все конкурентные задачи будут ждать окончания её работы.

Проверим это. Начинающий программист может подумать, что задачи исполняются как-то так конкурентно, можно сказать одновременно, с его точки зрения — вполне себе параллельно. И он, конечно, подумает, что если добавить нагрузку CPU bound в методы `first` и `second`, например `time.sleep(1)`, то весь алгоритм отработает за две секунды:

Асинхронный алгоритм на Python с заблокированным главным потоком исполнения

```
import asyncioimport timeimport aiohttpasync def get_request() -> dict:    async with aiohttp.ClientSession() as session:        async with session.get("http://localhost:23555") as response:            return await response.json()async def first():    time.sleep(1)    data = await get_request()    print(1, data)async def second():    time.sleep(1)    data = await get_request()    print(2, data)async def main():    # await asyncio.gather(first(), second())    task1 = asyncio.Task(first())    task2 = asyncio.Task(second())    await task1    await task2start = time.time()asyncio.run(main())stop = time.time()print(f"Request with bad concurrent async/await, time: {stop - start}")
```

«Ой!», — опять воскликнет наш юный друг, — «алгоритм отработал за три секунды»:

`Request with bad concurrent async/await, time: 3.0200273990631104`

Причём две секунды он последовательно спал, а запросы делал на третьей, что и требовалось доказать.

## Заключение

На простых примерах мы разобрали виды нагрузок и подходы к их исполнению. Я специально добавил неправильные примеры, чтобы было наглядно понятно, почему так делать нельзя и как делать нужно. То есть здесь мы разбирали не синтаксис, а принцип работы.

Спасибо за внимание.