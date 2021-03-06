---
title: "Как я не попал в Яндекс"
date: 2019-06-26T20:41:57+03:00
draft: false
toc: false
images:
tags: 
---
Эта статья о том, как обычный PHP разработчик готовился к собеседованию и чуть не попал в Яндекс в сентябре 2018 года. Перед тем как описывать сам процесс прохождения собеседования, я бы хотел рассказать, что происходило за месяц до собеседования.

## За месяц до собеседования
В этот момент я еще не думал о Яндексе и Яндекс обо мне. Я желал просто сменить свою текущую работу. Своё желание о смене работы я заранее огласил своему руководству и со спокойной душой обновил резюме, начитался как проходить собеседования и стал искать работу. Новую работу нашел довольно быстро.

На новом месте я успел проработать примерно 2 недели. Прошел онбординг и в целом всё устраивало. Но в этот момент мне пишет СТО с предыдущей работы и спрашивал меня, знаю ли хороших Python разработчиков. Я в шутку ему ответил: "Нет, не знаю, но могу сам попробовать". В итоге оказалось, что к СТО обратились знакомые из Яндекса, и он решил спросить меня, так как знал моё увлечение к Python.

Я стал размышлять над предложением. Я понимал, что завалю собеседование, возможно с позором, что возможно буду самым худшим, который когда либо собеседовался в Яндексе. Ведь даже Python не является моим основным инструментом, использовал только для своих проектов. Я понимал, что возможно могу поставить крест на текущей работе, где успел проработать 2 недели. Но я решил, что лучше попробовать, лучше получить хоть какой-то опыт, проверить свои силы, чем просто спасовать. В итоге я согласился на предложение.

## Подготовка к собеседованию
После переписки с СТО с предыдущей работы началась моя подготовка к собеседованию в Яндекс. Со мной связалась HR. Она рассказала про все преимущества, какие есть в Яндексе. Рассказала, как проходит само собеседование, и само собеседование назначили в Москве через 2 недели.

Далее мне предстояло пообщаться с СТО на текущей работе, чтобы отпросится на собеседование, так как собеседование было в будни и плюс в Москву нужно было добраться на поезде. Первая реакция СТО была немного агрессивная, он подумал, что я решил сменить работу и что мне тут не нравится. В итоге, когда я ему всё честно объяснил, что основная моя мотивация это проверка моих скиллов, то он понял и предложил помощь. Он в итоге меня встретил в Москве и подвез до собеседования, за это я ему очень благодарен.

Возвращаясь непосредсвенно к моей подготовке, то меня ждали две тяжелые недели, за это время я успел:
-	Прочитать книгу A Bite of Python
-	Каждый день по 2 часа решал задачки на HackerRank
-	Выучил наизусть 5-6 основных алгоритмов сортировки
-	Тренировался писать код на листочке

На всё это требовалось много времени, примерно по 4 часа в день выделял на подготовку. Чем ближе к собеседованию, тем больше понимал, что как много я не знаю. Но в последние два дня решил себя не мучить и дать голове отдохнуть и переварить всю полученную информацию.


## Собеседование
В Московском офисе меня ждало 4 секции, то есть 4 собеседования с 12:00 и до 18:00:

```
13:00 - 14:00 – Первая секция с написанием кода 
14:00 - 15:00 – Вторая секция с написанием кода 
15:00 - 16:00 - Обед с hr
16:00 - 17:00 - Менеджерская секция
17:00 - 18:00 - Финальная секция с кодом
```

Сразу оговорюсь, я не буду писать ответы в виде кода на свои задачи. Так как мои решения могут быть не самыми лучшими или вообще неправильными.

### Первая секция с написанием кода
Первая секция была довольна простая, и это было хорошо для разогрева. Мы много общались про Python. Потом мне дали посмотреть код, и я должен был провести code review. Сказать где ошибка закралась, сказать свои пожелания и улучшения, которые можно сделать. И в конце уже была задача:


> Нужно написать функцию, которая принимает путь к директории и возвращает, сколько занимает места на диске переданная директория.

<details>
<summary>Подсказки</summary>

Символические ссылки нужно игнорировать.  
Отслеживать inode - если inode уже была, больше учитывать её не нужно.

</details>  

Эту задачу я сделал, но не учел ряд моментов, которые отобразил в подсказке. Интервьюер стал мне задавать наводящие вопросы “а что если” и в итоге смог дойти до решения. Моё решение выглядело примерно как тут: https://stackoverflow.com/a/12984676/7772173

Итог: секция успешная


### Вторая секция
Вторая секция с кодом была труднее. Сразу дали задачку:


> Нужно написать функцию, у которой есть два аргумента:
> -	Первый аргумент массив уникальных идентификаторов баннеров
> -	Второй аргумент целое число K
> 
> Функция должна вернуть массив, содержащий K уникальных баннеров 

Эту задачу мы разбирали всю секцию. Моё первое решение было довольно простое: на каждом шаге от 1 до K вычисляем рандомный индекс из входящего массива, если индекс уже встречался, то еще раз перезапускаем рандом. Решение плохое, так как если K стремится к длине входящего массива, то рандом будет всё чаще и чаще вызываться.
Мы долго общались, я предлагал разные алгоритмы и структуры данных. И в итоге у меня получилось озвучить удовлетворяющий требованиям алгоритм. Код написал на листке бумаги и на удивление написал его без ошибок.

<details>
<summary>Подсказка</summary>

Нужно удалять элементы из входящего массива, когда через рандом находим нужный элемент. Но удаление из массива O(n), чтобы удалить за O(1) нужно найденное значение поменять местами с последним элементом в массиве. И тогда удалить последний элемент массива можно за O(1).

</details>  

После завершения этой секции у меня остались смешанные чувства. С одной стороны я смог дойти до правильного решения, а с другой стороны мне потребовались подсказки и скорее всего, должна была быть еще одна задача, но мы не успели по времени.

Итог: секция успешная


### Третья секция
Менеджерская секция была для меня небольшим отдыхом от задач. Я общался с людьми из той команды, которая хотела меня видеть. Были вопросы о моём опыте, о моих целях и планах. Также были вопросы, как я подхожу к выполнению задач, например: `Что ты делаешь, если видишь, что не успеваешь сделать задачу в срок?`

Итог: секция успешная

### Четвертая секция
Финальная секция с кодом была самой сложно и самой важной. Меня предупредили, что если я не пройду эту секцию, то меня в любом случае не возьмут.

Первая задача было следующая:
> Нужно написать функцию, которая принимает две строки, а в результате возвращает True или False.
> 
> Например:
> -	если str_1 = ‘ctr’ и str_2 = ‘catrow’, то нужно вернуть True
> -	если str_1 = ‘ctr’ и str_2 = ‘cartow’, то нужно вернуть False

Я знал, что есть целый класс алгоритмов для поиска подстроки в строке. Но я ни одного не знал. Но на удивление со второй попытки мне удалось изложить алгоритм, который понравился интервьюеру.

<details>
<summary>Подсказка</summary>

Мой алгоритм использовал счетчик, который указывал на индекс первой строки. И в конце нужно сверить счетчик и длину первой строки, если счетчик меньше, чем длина строки, то возвращаем False, иначе True

</details>  


Вторая задача была следующая:
> Дается массив каждый элемент, которого состоит из двух чисел - координаты X и Y. Этот массив это облако точек на координатной плоскости.
> Нужно написать функцию, которая возвращает True, если через это облако точек можно провести вертикальную асимптоту, и если все точки симметричны относительно этой асимптоты.

С этой задачкой я был в тупике. Я не мог даже предоставить простого решения. Но в итоге с помощью интервьюера получилось хоть какое-то решение. Далее нужно было его оптимизировать и тут у меня были сложности. Иногда я путался в вычислении сложности алгоритма, не мог подобрать нужные структуры данных. Но под конец интервью я смог на словах проговорить оптимизированных алгоритм, но не успел написать реализацию.

<details>
<summary>Подсказка</summary>

Чтобы вычислить координату X вертикальной асимптоты, нужно найти середину между максимальным и минимальным X в облаке точек.
Нужно учесть, что точки могут накладываться друг на друга.
Если у точки совпадает координата X с координатой вертикальной асимпоты, то точку можно не учитывать, так как она симметрична относительно себя.
Можно облако точек преобразовать в хеш таблицу, где ключ это X, а значение тоже хеш-таблица, где ключ Y, а значение счетчик, показывающий кол-во точек в этой координате. Далее работать с этой хеш-таблицей и удалять симметричные точки, если хеш-таблица в конце пустая, значит все точки симметричные.

</details>  

Итог: секцию провалил

## После собеседования
После собеседования меня встретила HR. Она пообещала как можно скорее дать результаты собеседований. У меня были ощущения, что я не прошел собеседование. Последняя и важная секция выдалась плохой для меня. Сами результаты собеседований мне сообщили довольно быстро, спустя 3 дня. И как ожидалось, завалил последнюю секцию. Общий мотив, то что мне нужно прокачивать алгоритмы.

## Вывод
Меня не взяли. Я прекрасно понимаю, что крупным компаниям лучше не взять хорошего программиста, чем взять плохого. Но я доволен этим опытом. Я действительно горд собой. За 2 недели подготовки мне получилось многое узнать, и в итоге чуть не попал в Яндекс. Это был челендж для меня, и я считаю, что с ним справился.


