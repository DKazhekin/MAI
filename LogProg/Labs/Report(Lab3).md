#№ Отчет по лабораторной работе №3
## по курсу "Логическое программирование"

## Решение задач методом поиска в пространстве состояний

### студент: Кажекин Д.А.

## Результат проверки

| Преподаватель     | Дата         |  Оценка       |
|-------------------|--------------|---------------|
| Сошников Д.В. |              |               |
| Левинская М.А.|              |               |

> *Комментарии проверяющих (обратите внимание, что более подробные комментарии возможны непосредственно в репозитории по тексту программы)*


## Введение

Пролог очень мощный инструмент для решения задач поиска в пространстве состояний в силу того, что можно легко организовать различные алгоритмы обхода всевозможных исходов (поиск в дереве в глубину, ширину и итеративное углубление). Примерами таких задач является нахождение выхода из лабиринта, задачи типа "коза-волк-капуста" и т.д.

## Задание

Три миссионера и три каннибала хотят переправиться с левого берега реки на правый. Как это сделать за минимальное число шагов, если в их распоряжении имеется трехместная лодка и ни при каких обстоятельствах (в лодке или на берегу) миссионеры не должны оставаться в меньшинстве.

## Принцип решения

1. Сперва я решил определить всевозможные состояния (передвижения) Миссионеров и Каннибалов (передвижения с левого берега на правый и наоборот). Не будем, конечно же, забывать, что всего возможно 8 вариантов комбинаций состава пассажиров в лодке, причем Миссионеров должно быть большинство, либо не быть вообще.
Реализованы состояния с помощьью предиката move(A, B), где A - список, отражающий количество Миссионеров (M1), Каннибалов(С1) на левом берегу до отправления лодки и M2, C2 - после. Первые 8 предикатов отражают передвижение с левого берега(tmp1) на правый(tmp), а следующие за ними 8 - с правого (tmp) на левый (tmp1).

P.S. В ходе решения задачи было выяснено, что в списках А и В необходимо третьим элементом добавить "инициализатор" левого (tmp1) и правого (tmp) берегов. Без них программа выдавала решения из двух ходов (Сначала переправляются 3 Каннибала, а следом 3 Миссионера, но лодка не может не везти ни одного пассажира)

```prolog
move([M1, C1, tmp1], [M2, C1, tmp]) :-
	M1 > 0,
	M2 is M1 - 1.
move([M1, C1, tmp1], [M2, C1, tmp]) :-
	M1 > 1,
	M2 is M1 - 2.
move([M1, C1, tmp1], [M2, C1, tmp]) :-
	M1 > 2,
	M2 is M1 - 3.
move([M1, C1, tmp1], [M1, C2, tmp]) :-
	C1 > 0,
	C2 is C1 - 1.
move([M1, C1, tmp1], [M1, C2, tmp]) :-
	C1 > 1,
	C2 is C1 - 2.
move([M1, C1, tmp1], [M1, C2, tmp]) :-
	C1 > 2,
	C2 is C1 - 3.
move([M1, C1, tmp1], [M2, C2, tmp]) :-
	M1 > 0,
	C1 > 0,
	M2 is M1 - 1,
	C2 is C1 - 1.
move([M1, C1, tmp1], [M2, C2, tmp]) :-
	M1 > 1,  C1 > 0,
	M2 is M1 - 2,
	C2 is C1 - 1.
move([M1, C1, tmp], [M2, C1, tmp1]) :-
	M1 < 3,
	M2 is M1 + 1.
move([M1, C1, tmp], [M2, C1, tmp1]) :-
	M1 < 2,
	M2 is M1 + 2.
move([M1, C1, tmp], [M2, C1, tmp1]) :-
	M1 < 1,
	M2 is 3 + M1.
move([M1, C1, tmp], [M1, C2, tmp1]) :-
	C1 < 3,
	C2 is C1 + 1.
move([M1, C1, tmp], [M1, C2, tmp1]) :-
	C1 < 2,
	C2 is C1 + 2.
move([M1, C1, tmp], [M1, C2, tmp1]) :-
	C1 < 1,
	C2 is C1 + 3.
move([M1, C1, tmp], [M2, C2, tmp1]) :-
	M1 < 3,
	C1 < 3,
	M2 is M1 + 1,
	C2 is C1 + 1.
move([M1, C1, tmp], [M2, C2, tmp1]) :-
	M1 < 2,  C1 < 3,
	M2 is M1 + 2,
	C2 is C1 + 1.
```
2. Следующим шагом организована проверка большинства Миссионеров на левом и правом берегах после переезда пассажиров, в связи с условием задачи.

```prolog
check([Ml, Cl, _]) :-	% Проверка на правом берегу
	(Cl =< Ml ; Ml = 0),
	Cr is 3 - Cl, Mr is 3 - Ml, 
	(Cr =< Mr ; Mr = 0).

check2([Ml, Cl, _]) :-	% Проверка на левом берегу
	Cr is 3 - Cl,
	Mr is 3 - Ml,
	(Cr =< Mr ; Mr = 0).
```

3. Дальше представлены алгоритмы деревьев обхода состояний:

Поиск в глубину и вспомогательный предикат соотвественно:

```prolog
dfs :-
	write("Output: "),
	nl,
	path1([[3, 3, tmp1]], [0, 0, tmp], Res),
	prints(Res).

path(X, Y, P) :- path1([X], Y, P). 
path1([X|T], X, [X|T]).
path1(P, Y, R) :-
	prolong(P, P1),
	path1(P1, Y, R).
```

```prolog
prolong([X|T], [Y,X|T]) :-
	move(X, Y),
	check(Y),
	check2(X),
	not(member(Y, [X|T])).
```
Поиск в ширину:

```prolog
bfs :-
	write("Output: "),
	nl,
	path2([[[3, 3, tmp1]]], [0, 0, tmp], Res),
	prints(Res).


path2([[Y|T]|_], Y, [Y|T]).
path2([P|QL], X, R) :-
	findall(Z, prolong(P, Z), T),
	append(QL, T, QO), !,
	path2(QO, X, R).
path2([_|T], Y, R) :- 
	path2(T, Y, R).
```

Поиск с итерационным углублением и вспомогательный предикат соответственно:

``` prolog
it_dive :-
	write("Output: "),
	nl,
	iter(L),
	path3([[3, 3, tmp1]], [0, 0, tmp], Res, L),
	prints(Res).

path3([X|T], X, [X|T], 0).
path3(P, Y, R, N) :- 
	N > 0,
	prolong(P, P1),
	N1 is N - 1,
	path3(P1, Y, R, N1).
```

```prolog
iter(1).
iter(M) :- 
	iter(N),
	M is N + 1.
```

4. И последний предикат, помогающий нам с выводом ответа в терминал:

```prolog
prints([_]).
prints([Y, X|T]) :-
	prints([X|T]),
	move(X, Y),
	write(X),
	write('===='),
	write(Y),
	nl.
```

## Результаты

1. Поиск в глубину

```prolog
Output:
[3,3,tmp1]====[3,1,tmp]
[3,1,tmp]====[3,2,tmp1]
[3,2,tmp1]====[0,2,tmp]
[0,2,tmp]====[2,2,tmp1]
[2,2,tmp1]====[1,1,tmp]
[1,1,tmp]====[3,1,tmp1]
[3,1,tmp1]====[0,1,tmp]
[0,1,tmp]====[1,1,tmp1]
[1,1,tmp1]====[0,0,tmp]
```
Здесь мы видим, что длина пути равна 9.
Для примера разберем пару строк вывода решения:
В первой строке видно, что начальное количество Миссионеров и Каннибалов было [3,3], после переправы стало [3,1], это значит что 2 Каннибала теперь на правом берегу. Во второй строке видим что из пололжения [3,1] мы перешли в состояние [3,2], это значит, что 1 Каннибал перебрался на левый берег на втором шаге. И т.д.

2. Поиск в ширину

```prolog
Output: 
[3,3,tmp1]====[3,1,tmp]
[3,1,tmp]====[3,2,tmp1]
[3,2,tmp1]====[0,2,tmp]
[0,2,tmp]====[0,3,tmp1]
[0,3,tmp1]====[0,0,tmp]
```

3. Поиск с итеративным углублением

```prolog
Output: 
[3,3,tmp1]====[3,1,tmp]
[3,1,tmp]====[3,2,tmp1]
[3,2,tmp1]====[0,2,tmp]
[0,2,tmp]====[0,3,tmp1]
[0,3,tmp1]====[0,0,tmp]
```
Таблица

```prolog
! Алгоритм поиска |  Длина найденного первым пути  |  Время работы  |
|-------------------------------------------------------------------|
| В глубину       |             9                  |     0.012       |
| В ширину        |             5                  |     0.015       |
| ID              |             5                  |     0.013       |
```
## Выводы

Данная лабораторная работа была для меня очень интересной, потому что я научился реализовывать алгоритмы обхода деревьев на языке Пролог, немного соприкоснувшись с Искусственным Интеллектом (поиск в пространстве состояний). Мне удалось укрепить навык разработки Пролог программ и подытожить усвоенные теоретические знания. Пришлось задуматься над логикой представления состояний move(A,B) для корректной работы программы, что в конце концов удалось. 

1. Алгоритм поиска в глубину удобно использовать в том случае, когда нам необходимо найти хотя бы какое-то решение и нет необходимости, чтобы оно было самым коротким
2. Алгоритм поиска в ширину пригождается в случае, если мы хотим обнаружить самое короткое решение, а также в случае, если у нас есть циклы.
3. Алгоритм итеративного углубления рационален, когда мы заранее знаем на каком уровне "лежит" наше решение. Мне кажется, что очень показательным является пример с кубиком Рубика, который мы прокрутим 5 раз. Тогда, с помощью последнего алгоритма мы с легкостью восстановим исходное положение, зная, что решение лежит на высоте дерева = 5.

В моем случае, более эффективными оказались алгоритмы поиска в ширину и с итерационным углублением, не сильно отстающих по времени от поиска в глубину, но нашедших путь почти в два раза короче.