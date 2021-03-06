---
last\_modified: 2015-02-23 16:27:08
tags: Perl
author: perlnews
title: Ускорение сигнатуры функции в Perl
---

Функции с экспериментальной поддержкой сигнатур, которые появились в Perl 5.20
работали медленнее, чем обычные функции. Например, если сравнить функции
`plain` и `sig`, то функции с сигнатурой потребуется выполнить на 60% больше
инструкций:

    sub plain {
        my ($a, $b, $c) = @_;
        ...
    }
    
    sub sig ($a, $b, $c) {
        ...
    }

Дейв Митчелл провёл большую работу по оптимизации сигнатур и, в частности, по
присвоению значений по умолчанию, например:

    sub foo ($a, $b, $c = 1) {
        ...
    }

В результате этой работы появилась новая операция `OP_SIGNATURE`, которая
призвана заменить множество отдельных операций по присвоению значений
параметров. Далее чуть подробнее о том, что это даёт.

---

Бенчмарки показывают, что с подобной оптимизацией функции с сигнатурами
становятся быстрее на 35% по сравнению с обычными функциями. Дэйв также добавил
возможность оптимизации обычных функций в функции с сигнатурами (если включить
опцию сборки `-DPERL_FAKE_SIGNATURE`). Это означает, что если компилятор
встретит такую функцию:

    sub foo {
        my ($x, $y, $z) = @_;
        ...
    }

То он автоматически её приведёт к виду:

    sub foo ($x, $y, $z) {
        ...
    }

Что приведёт к ускорению её работы на 35% без какого-либо вмешательства со
стороны программиста.

Зачастую функция вызывается с параметрами всегда одно и того же типа. Например,
методы класса всегда имеют первый параметр в виде ссылки на объект.
`OP_SIGNATURE` видит, что первый параметр это ссылка и выполняет более быстрые
операции создания лексической переменной вместо более дорогого вызова
`sv_setsv()`.

На данный момент код новой операции помещён в ветке
`smoke-me/davem/op_signature3` и не включён в blead. Поскольку чуть ранее в
blead была объявлена заморозка на добавление новых фич, пока трудно сказать
появится ли эта оптимизация в грядущем Perl 5.22.

[Источник
новости](http://www.nntp.perl.org/group/perl.perl5.porters/2015/02/msg226044.html)
