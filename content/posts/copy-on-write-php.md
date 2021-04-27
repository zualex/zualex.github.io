---
title: "Copy-on-write в PHP"
date: 2021-04-27T14:14:23+03:00
draft: false
tags:
  - php
---

Copy-on-write или копирование при записи — один из способов управлением памятью. Но перед тем как давать какие-то определения, предлагаю рассмотреть пример:

```
function handle(array $array) {
    $result = [];
    // ...
    return $result;
}

$largeArray = getLargeArray();
handle($largeArray);
```

В данном примере есть функция handle. В эту функцию передаётся массив большого размера. По умолчанию в PHP передача аргументов происходит по значению. Это означает, что если изменить значение аргумента внутри функции, то вне функции значение всё равно останется прежним. Другими словами внутри функции используется копия переменной, но для создания копии требуется выделить память.

**Вопрос: в целях оптимизации стоит ли передать аргумент по ссылке handle(array &$array)?**

На самом деле ответ зависит от того, что происходит внутри функции handle.

## Чтение аргумента
Представим, что функция handle только читает значения из $array.
```
function handle(array $array) {
    $result = [];
    foreach($array as $row) {
        $result[] = $row['id'];
    }
	
    return $result;
}
```

В данном случае внутри функции handle не произойдет копирования переменной $array. Переменные $array и $largeArray ссылаются на одну и ту же запись zval.

zval - если упрощенно, то это контейнер, в котором хранится переменная.

```
$value = 'string'; // создается контейнер zval_1
$copyValue = $value; // используется контейнер zval_1

$value = 'new string'; // создается контейнер zval_2

unset($value); // удаляется zval_2
unset($copyValue); // удаляется zval_1
```

При создании новой переменной $value создается новый контейнер zval, для простоты обозначим, как zval_1. На следующей строке переменной $copyValue присваивается значение $value. Интуитивно может показаться, что в этот момент будет выделена память, но на самом деле $copyValue только лишь ссылается (не путать со ссылками PHP, которые начинаются со знака &) на тот же zval контейнер, что и $value. Новый контейнер будет создан только в том случае, если переменную $value или $copyValue изменить.

## Модификация аргумента
Теперь рассмотрим ситуацию, когда в функции handle переменная $array модифицируется.
```
function handle(array $array) {
    unset($array[0]);

    return $array;
}
```

В данном случае произойдет копирование переменной, то есть создание нового контейнера zval с выделением памяти.

## Copy-on-write
Суть подхода сopy-on-write или копирование при изменении заключается в том, что при чтении переменных используется общая копия, в случае изменения переменной — создается новая копия.

## Замеры памяти
Проведём тест, попробуем увидеть, что при чтении аргумента используется общая копия, а при модификации выделяется память.

```
<?php

function printMemory(string $header) {
    $memoryPeak = memory_get_peak_usage();
    
    echo $header . PHP_EOL;
    echo 'Peak usage: ' . round($memoryPeak / 1024) . 'KB of memory ' . PHP_EOL . PHP_EOL;
}

function handleRead(array $array) {
    $result = [];
    foreach($array as $row) {
        $result = 1; // чтобы выделенная память на $result не повлияла на замер
    }
	
    return $result;
}

function handleWrite(array $array) {
    unset($array[0]);

    $result = [];
    foreach($array as $row) {
        $result = 1; // чтобы выделенная память на $result не повлияла на замер
    }
}

$largeArray = range(0, 500000);
printMemory('create $largeArray');

handleRead($largeArray);
printMemory('handleRead');

handleWrite($largeArray);
printMemory('handleWrite');
```

Результат для PHP 7.4 - 8.0:
```
create $largeArray
Peak usage: 16764KB of memory

handleRead
Peak usage: 16765KB of memory

handleWrite
Peak usage: 33153KB of memory
```

Из результата видно, что при изменении аргумента, была выделена память, а при чтении нет.

## Передача объекта в качестве аргумента
Стоит рассмотреть случай передачи объекта в качестве аргумента. И посмотреть, применяется ли для данного случая механизм copy-on-write. Рассмотрим следующий пример:

```
function printMemory(string $header) {
    $memoryPeak = memory_get_peak_usage();
    
    echo $header . PHP_EOL;
    echo 'Peak usage: ' . round($memoryPeak / 1024) . 'KB of memory ' . PHP_EOL . PHP_EOL;
}

$object = new stdClass;
$object->list = range(0, 500000);

function handle(stdClass $stdClass) {
    unset($stdClass->list[0]);
}

echo 'Before: ' . count($object->list) . PHP_EOL;

handle($object);

echo 'After: ' . count($object->list) . PHP_EOL;
```

Результат:
```
Before: 500001
Memory before handle
Peak usage: 16764KB of memory 

After: 500000
Memory after handle
Peak usage: 16765KB of memory 
```

В данном примере, переданный объект модифицируется внутри функции. Следуя, описанной выше логике, то в методе должна создаться копия $stdClass. Но если посмотреть на результат выполнения скрипта, то видно, что $object изменился, несмотря на то, что передавали переменную по значению, а память, по сути, осталось неизменной.

Складывается впечатление, что объект передаётся по ссылке, а не по значению, но это не совсем верно. На самом деле при передаче объекта в качестве аргумента передаётся только ID объекта. Содержимое объекта хранится отдельно и доступ можно получить по ID. Из-за этого, если что-то изменить внутри объекта, то это доступно и внутри функции и вне функции. Более детально можно прочитать документации PHP: [Объекты и ссылки](https://www.php.net/manual/ru/language.oop5.references.php).

То есть в данном примере не нужно передавать объект по ссылке, так как передается всего лишь ID объекта.

## Вывод
В подавляющем большинстве не нужно передавать аргумент по ссылке. Так как редко оперируем большими по памяти переменными. И аргументы в большинстве случаев используются только на чтение. Если речь идет про передачу объектов, то и в этом случае не нужно передавать аргумент по ссылке, так как передаётся только идентификатор объекта.

Другими словами не нужно сейчас бежать, и срочно что-то менять в вашей коде и в вашем подходе. Продолжаем писать код как, обычно, но уже более осознанно.

## Источники
- [PHP Internals Book - Memory management](https://www.phpinternalsbook.com/php5/zvals/memory_management.html)
- [Henry. 2011. *Henry @ Web Apps: PHP copy on write - how PHP manages variable memory*](zotero://select/items/_SN9JCSZQ)
- [PHP - Объекты и ссылки](https://www.php.net/manual/ru/language.oop5.references.php)
