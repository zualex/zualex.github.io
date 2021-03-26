---
title: "PHP производительность array_merge в цикле"
date: 2021-03-24T11:47:29+03:00
draft: false
toc: false
images:
tags: 
  - php
---

Если использовать array_merge в цикле, то phpStrom + плагин [Php Inspections (EA Extended)](https://plugins.jetbrains.com/plugin/7622-php-inspections-ea-extended-) даёт подсказку: `[EA] 'array_merge(...)' is used in a loop and is a resources greedy construction.` Стало интересно, насколько  сильно влияет на произовдительность использование array_merge в цикле и какие есть альтернативные решения.

Рассмотрим пример. Имеется список товаров следующей структуры:
```
$products = [
    ['id' => 1, 'tags' => ['tag_1', 'tag_2', 'tag_3']],
    ['id' => 2, 'tags' => ['tag_1', 'tag_2', 'tag_3']],
];
```

Нужно получить список всех тегов.

В экспериментах решил проверить скорость скрипта при количестве в 10, 250, 500, 1000, 2000, 5000, 10000, 50000 товаров, и также изменять количество тегов внутри каждого товара: 1, 3, 6 тегов.

## Объединение массивов
Выделил следующие способы объединить два и более массивов в один:
- array_merge
- Оператор ...
- Оператор +
- array_replace
- foreach

Рассмотрим каждый из способов с примерами кода.

#### Использование array_merge
```
$tags = [];
foreach($products as $product) {
    $tags = array_merge($tags, $product['tags']);
}
```
https://gist.github.com/zualex/39b3dfe9503b69468307f420089d0c42

#### Распаковка массива через оператор ...
```
$tags = [];
foreach($products as $product) {
    $tags[] = $product['tags'];
}
$tags = array_merge(...$tags);
```
https://gist.github.com/zualex/c1bd85a1c2fee5b4c362953211d976ea

#### Использование оператора +
При использовании оператора + если совпадают индексы, то в результирующем массиве будут только элементы из массива слева от оператора. Чтобы этого избежать потребуется переделать структуру хранения товаров, чтобы в списке тегов был уникальный индекс.

```
$products = [
    ['id' => 1, 'tags' => ['tag_1' => tag_1', 'tag_2' => 'tag_2', 'tag_3' => 'tag_3']],
    ['id' => 2, 'tags' => ['tag_1' => tag_1', 'tag_2' => 'tag_2', 'tag_3' => 'tag_3']],
];

$tags = [];
foreach($products as $product) {
    $tags += $product['tags'];
}
```
https://gist.github.com/zualex/902a6f9e9d1f23a7da7cdd3bebf72812

#### Использование array_replace
В данном примере тоже потребуется использовать структуру хранения товаров, как при использовании оператора +.

```
$tags = [];
foreach($products as $product) {
    $tags = array_replace($tags, $product['tags']);
}
```
https://gist.github.com/zualex/b656a5f12992f96edaacf1bb22cd0bf7

#### Использование foreach
```
$tags = [];
foreach($products as $product) {
    foreach($product['tags'] as $tag) {
        $tags[] = $tag;
    }
}
```
https://gist.github.com/zualex/0d101fb8b48701352117cfc253be2762

## Анализ производительности
Результаты прогонов в этой таблице: https://docs.google.com/spreadsheets/d/1Va7pg5iaPbXMxkbcQ5sOTco0d316IUzLCOcHq1CrzRo/edit?usp=sharing

Эксперименты показали, что для PHP 8.0 и 7.4 нет существенной разницы между версиями. Из-за этого примеры будут для PHP 7.4.

![График PHP 7.4 ...](/static/php-array_merge/PHP_7.4_....png)
Для 6 тегов нет расчетов из-за memory limit во время прогона тестов.

![График PHP 7.4 +](/static/php-array_merge/PHP_7.4_+.png)
![График PHP 7.4 foreach](/static/php-array_merge/PHP_7.4_foreach.png)

Как видно из графиков, `...`, `+`, `foreach` имеют линейную зависимость.

![График PHP 7.4 array_merge](/static/php-array_merge/PHP_7.4_array_merge.png)
![График PHP 7.4 array_replace](/static/php-array_merge/PHP_7.4_array_replace.png)

 `array_merge`, `array_replace` - видна квадратичная зависимость. В экспериментах данные имеются только для 10000.
 
 ### Сравним вместе
 Теперь попробуем совместить графики, чтобы увидеть общую картину. Сравнивать буду для PHP 7.4 с использованием 3-х тегов.
 
График для кол-во от 10 до 500 товаров.
![График PHP 7.4 всё вместе от 10 до 500 товаров](/static/php-array_merge/PHP_7.4_10_500.png)

Как видно, уже начиная от 250, а может и ранее (в экспериментах делал тесты для 10, 250, 500 товаров), `array_merge` и `array_replace` отрабатывают дольше всех.

Теперь если посмотреть для 50000 товаров.
![График PHP 7.4 всё вместе от 10 до 50000 товаров](/static/php-array_merge/PHP_7.4_10_50000.png)
`array_merge` и `array_replace` в рамках теста, смог замерить только для 10000 товаров. Разница существенна, на порядки.

### А что с памятью?
![График PHP 7.4 память](/static/php-array_merge/PHP_7.4_memory.png)
С памятью всё ок, все подходы практически одинаково используют память.

### Самый быстрый подход
![PHP 7.4 сравнение быстрых подходов](/static/php-array_merge/7.4_compare_fast.png)
В данных эксперимента самым быстрым оказался обычный foreach, но разница столь не существенна, что думаю не стоит делать разницы между `...`, `+` и `foreach` .

## А если без цикла?
Для полноты картины посмотрим на те же подходы, если без цикла объединить 3 больших массива по 100000 элементов в каждом.

```
$big1 = range(0, 100000);
$big2 = array_fill(100000, 100000, 'string');
$big3 = range(200000, 100000);
```

#### Без цикла array_merge
```
$result = array_merge($big1, $big2, $big3);
```
https://gist.github.com/zualex/a9f536006c1a24f7c4d4888077dc9fc0

#### Без цикла Распаковка массива через оператор ...
```
$result = [...$big1, ...$big2, ...$big3];
```
https://gist.github.com/zualex/a35cd71c38480658a1bbaf6559c4c2fe

#### Без цикла оператор +
```
$result = $big1 + $big2 + $big3;
```
https://gist.github.com/zualex/6e6d9d584cc979b59da517f9078e6559

#### Без цикла array_replace
```
$result = array_replace($big1, $big2, $big3);
```
https://gist.github.com/zualex/4e770098f40f2f101138dda7251f32ca

#### Без цикла foreach
```
$result = [];
foreach ($big1 as $key => $value) {
    $result[$key] = $value;
}
foreach ($big2 as $key => $value) {
    $result[$key] = $value;
}
foreach ($big3 as $key => $value) {
    $result[$key] = $value;
}
```
https://gist.github.com/zualex/4e770098f40f2f101138dda7251f32ca

### Анализ данных без циклов
#### Скрость
![График PHP 7.4 без цикла скорость](/static/php-array_merge/PHP_7.4_Without_loop_Speed.png)
#### Память
![График PHP 7.4 без цикла память (KB)](/static/php-array_merge/PHP_7.4_Without_loop_Memory.png)

Как видно, `array_merge` быстрее всех справляется с объединением больших массивов без использования циклов. По памяти Чуть экномичнее `foreach` и `array_replace`


## Вывод
`array_merge` и `array_replace` не нужно использовать в циклах, даже в маленьких. При решении данной задачи лучше выбрать  `...`, `+` или `foreach`. Но стоит не забывать отличия в работе `array_merge` и `+`.

Но если стоит задача объединить массивы, когда нет циклов, то лучше использовать `array_merge`.
 

 ## Ресурсы:
 - [Разница между array_merge и + (оператор плюс) в PHP](http://langtoday.com/?p=393)
 - [[PHP 7] Merging arrays with array_merge() and spread syntax](https://www.kindacode.com/article/merging-arrays-in-php-7/)
- [No Array_merge In Loops](https://github.com/dseguy/clearPHP/blob/master/rules/no-array_merge-in-loop.md)
- [Zendframework Issues #5716: Zend\Loader\ClassMapAutoloader - Performance improvement](https://github.com/zendframework/zendframework/issues/5716)
- [Исходный код array_merge: php-src/ext/array.c -> PHPAPI int php_array_merge](https://github.com/php/php-src/blob/master/ext/standard/array.c#L3544)

