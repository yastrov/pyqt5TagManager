UTF-8, Line end CRLF

Вступление:
С одной стороны желательно попробовать написать это самому. С другой стороны некоторые вещи пришлось сейчас (или когда-то раньше) гуглить и собирать из примеров методом "проб".
Что-то читал ещё в давние годы.
Так что некоторый вариант (на случай, если получится совсем "быдлокод", или будет непонятно как
сделать сохранение в файл, перетаскивание с помощью мыши...)

Эта версия работает удовлетворительно, и на предмет утечек памяти проверена. (Может утечь, только если упадёт.)

Дополнительные примеры можно посмотреть:
https://github.com/baoboa/pyqt5/tree/master/examples

Описание:

main.py - главный файл, "точка входа"/запуска программы.

TagManagerMainWidget.py - Главное окно (виджет) программы - отображение дерева, реакция на кнопки
"открыть/загрузить" и т.п. (кнопки "Переместить элемент выше/ниже" рааботает некорректно.)

TagTreeModel.py - Модель данных (взаимодействует с окном (виджетом) на предмет "как отображать" и "что делать, если юзер...")
(кнопки "Переместить элемент выше/ниже" рааботает некорректно на уровне функций moveUp в TagTreeModel.)

TagTreeItem.py - Фактически наш типовой элемент дерева, графа. Объект TagTreeItem описывает наш тег, и имеет ссылки на родительский элемент, и на потомков. Таким образом, вкладывая одни TagTreeItem в другие мы создаём древовидную структуру, как в иерархиях.
Некоторые вещи из модели мы выносим сюда для удобства.

Так же, как генеалогическое древо любой сложности состоит из типовых элементов "человек".

examples\TagTreeItem_weakref_example.py - фрагмент класса TagTreeItem, если бы его переписали на weakref
(Так конечно потретится больше памяти, но есть надежда гарантировать себя от утечек даже без моих ухищрений с ручной очисткой данных в модели. И может даже при падениях. Впрочем, ухищрения тоже лишними не будут.)

DataStoreHelper - папка, содержащая классы для чтения/записи данных в файлы. Написанное мной на основе набросков из examplesCSV_XML_SQL, частично тестировано. Примерно в таком виде хорошие программисты исползуют импорт/экспорт в своих программах. (Один из них и используется для сохранения сейчас. Заменить одно на другое - минутное дело.)

ColorHelper - классы для отображения одинаковых тегов одним цветом (в качестве примера 2 варианта).

examples\CSV_XML_SQL - примеры работы с форматами CSV,XML,SQL написанными мной, и пара примеров из сети, адаптированных под Qt5. (Самостоятельны. Имеет ценнность только как примеры и иллюстрации.)


Послесловие:
Да, с некоторыми вещами я обращался менее строго, чем нужно было бы (например прямой доступ к переменным другого класса "снаружи" считается плохим тоном, но это частности. Тем более на этом этапе.)
Впрочем, моей задачей была реализация тех вещей, которые требуют опыта или знаний. Подгонки, комбинрование, улучшения и т.п. в задачу не входило.

Остаётся открытым и вопрос, как лучше (соотновшение надёжность/экономия памяти) решать: TagTreeItem_weakref_example.py или TagTreeItem.py (экономнее, но надёжность ниже)

Можно (и по хорошему стоит) обдумать ещё раз функции удаления, на предмет не забыли ли мы удалить ссылку на родитиеля в каком-то элементе, например. Или на потомков. Или где-то дублируется код
(в разных функциях похожие куски кода можно вынести)...

-----------
Иногда шёл на оптимизацию. Хотя и не слишком (вообще многие прогеры считают это глупой "экономией на мелочах"). Например объекты типа списков или словарей можно.. закэшировать.
(Наа самом деле на каждой операции "точка" (официально называется "оператор доступа") мы теряем несколько тактов.)

def foo1_slow(self):
    return self.node_list[1] + self.node_list[2]

def foo1_faster(self):
    node_list = self.node_list
    return node_list[1] + node_list[2]

Функции классов или объектов тоже можно кэшировать, если осторожно:

def foo2_slow(data_list):
    node_list = []
    for elem in data_list:
        node_list.append(elem)
    return node_list

def foo2_faster(data_list):
    node_list = []
    foo = node_list.append
    for elem in data_list:
        foo(elem)
    return node_list

Можно было бы вообще заменить констуркцию for на [elem for elem in data_list],
которая сразу "соберёт" список:

def foo2(data_list):
    return [elem for elem in data_list]

Но это уже частности.

П.С.
Есть некоторый риск запутаться с QModelIndex (и это видится самым сложным: они используют в двух случаях два индекса: когда запрашивают данные - то индекс указывает на сам элемент, в некоторых случаях - на элемент верхнего уровня). Впрочем, возможно это
от невнимательности (с которой последнее время проблема).

Про тонкости работы с памятью: https://habrahabr.ru/post/210304/
