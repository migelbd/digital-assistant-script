# Сценарий поведения цифрового помощника

В данном проекте ведется работа по разработке описания сценария цифрового помощника. При подготовке структуры описания
сценария ключевым критерием является соблюдение баланса между гибкостью настройки сценария и простотой его описания.

## Цифровой помощник

Цифровой помощник - это цифровая система, которая предоставляет полезную автоматизацию взаимодействия в формате
диалога на естественном языке. Предпочтительным средством коммуникация для человека является диалог, поэтому цифровые 
помощники как средство взаимодействия с информационными системами довольно востребованы.

Цифровой помощник объединяет в себе три независимых слоя:

* описание поведения
* исполнитель поведения
* средство диалога

### Описание поведения (сценарий)

Поведение цифрового помощника зависимо от реакции использующего его пользователя, поэтому описание поведения
помощника названо сценарием. Под сценарием понимается совокупность алгоритмов поведения, которые последовательно
исполняются в зависимости от реакции пользователя.

Форматом записи сценария выбран json-формат как наилучший формат, предоставляющий баланс для одновременного понимания
вычислительной машиной и человеком. Кроме того структура json может быть описана в его же формате с помощью
[json scheme](https://json-schema.org/). В проекте в качестве артефакта представлена json-схема описания сценария
([схема](scenario.schema.json)).

Так как цифровой помощник представляет собой информационную систему, то поведение ее естественно описывать
опираясь на принципы структурного программирования. Структурное программирование предполагает, что описание
поведения состоит всего из трех структур управления:

* последовательности (sequence)
* ветвления (selection)
* цикла (iteration)

Учитывая что необходимо описывать сценарий, введены дополнительные структуры реакции пользователя:

* команда (command)
* ответ (reply)
* выбор из предлагаемых вариантов (choice)

#### Последовательность

Сценарий описывает в виде последовательности операций ответную реакцию помощника на действие пользователя. Описание
вполне отдельной операции из последовательности назовем актом ([/$def/act](scenario.schema.json)).

Акты исполняются последовательно в логике их типов:

* сообщение пользователю message ([/$def/message](scenario.schema.json));
* изменение контекста ([/$def/context](scenario.schema.json)),
* ветвление selection ([/$def/selection](scenario.schema.json)).

##### Сообщение пользователю

Сообщение пользователю является базовой операцией. Собственно через отправку сообщений и реализуется диалоговая
форма взаимодействия со стороны помощника. Сообщение формируется из шаблона фразы на естественном языке пользователя,
в которой могут быть использованы переменные. Шаблон фразы задается в атрибуте message акта сообщения. В момент
формирования сообщения в шаблон подставляются значения переменных.

Вместе с сообщением можно отправить варианты выбора choices. Варианты задаются через массив из выбора choice 
либо через переменную типа ключ-значение. Выбор choice определяется значением value и текстом text.

При задании вариантов выбора через переменную в параметрах value и text выбора доступна зарезервированная переменная
$$choice (см ниже). В параметре choice можно задать шаблон для value и text, которые будут заполняться при каждой
итерации перебора указанной переменной. Если параметр choice не задан, то value = $$choice[key] и text = $$choice[value]

При выборе пользователем варианта выбранный вариант доступен в следующем акте через зарезервированную переменную
$$choice (см ниже)

##### Изменение контекста

Совокупность значений переменных сценария является контекстом его исполнения. Управление контекстом описывается в
специальном акте context. Управление контекстом осуществляется за счет изменения значений переменных. Изменение
контекста может задаваться строго в сценарии либо запрашиваться у стороннего сервиса.

Обращение к стороннему сервису доступно в формате REST и jsonRPC.

##### Ветвление

Ветвление предоставляет вариативность исполнения сценария. Если условие, заданное в параметре if истинно, то
исполняется последовательность успеха then, иначе последовательность неуспеха else. Последовательность неуспеха может
быть не определена.

#### Команды

Команды являются исходными точками для начала исполнения сценария помощником и всегда доступны пользователю.
Команда сбрасывает статус исполнения сценария до первого акта команды.

#### Цикл

В сценарии цикл применяется в сценарии как итерация по переменной типа ключ-значение.

* определение списка вариантов для выбора клиенту, заданный через переменную, в акте типа message.
* цикличное повторение последовательности

## Переменные сценария

Переменные сценария бывают пользовательские и зарезервированные. При использовании переменных необходимо перед именем
переменной прописывать знак $.

### Пользовательские переменные

Пользовательские переменные используются в шаблонах сообщений, условных сообщениях, вариантах выбора и при
присвоении значения. Возможные пользовательские переменные задаются в описании сценария в разделе vars
([/vars](scenario.schema.json)).

#### Имя переменной

Имя переменной имеет специальную структуру и ограничения. Имя переменной может состоять из букв, цифр, символа нижнего
подчеркивания и символов квадратных скобок, которые допустимо использовать только особым образом (см ниже Нотация
квадратных скобок).

Переменные принимают значения простых типов:

* integer (целочисленный),
* number (число, включая все действительные числа),
* string (строковый),
* boolean (true или false).

Так же переменные можно организовать в структуры типа ключ-значение.

#### Нотация квадратных скобок

Переменные, имя которых задано нотацией квадратных скобок, можно использовать как одну переменную списка. Списки
могут быть именованные и нумерованные.

**Именованные списки**

Именованные списки формируются через указание свойства в квадратных скобках.

После основного имени переменной в квадратных скобках указывается имя свойства переменной.
Например, Переменная[Свойство].
При этом ограничение на имя свойства такое же как и для имени простой переменной.

Вложенность свойств может быть произвольной. Например, Переменная[Свойство][ЕщеОдноСвойство].

Если две и более переменных имеют одинаковую часть имени до закрывающейся квадратной скобки, то эти переменные можно
использовать как список.
Например, если заданы две переменной
Переменная[Свойство][ОдноСвойство]
и
Переменная[Свойство][ДругоеСвойство],
то в можно использовать переменные как список
$Переменная[Свойство]

**Нумерованные списки**

Нумерованные списки задаются через указание пустых квадратных скобок

Переменная[]
Переменная[]

Нумерованные переменные могут быть использованы только как список.

### Зарезервированные переменные

Зарезервированные переменные принимают определенные значения в зависимости от этапа исполнения сценария. Значения
зарезервированным переменным присваиваются либо при исполнении акта либо по итогу исполнения предыдущего акта.
Во втором случае назначенные значения зарезервированных переменных доступны только в следующем акте. Если требуется
использование значения зарезервированной переменной присвойте ее пользовательской переменной.

При указании зарезервированных переменных
При использовании зарезервированных переменных необходимо перед именем переменной прописывать двойной знак $$.

Ниже перечислены зарезервированные переменные

#### reply

Переменная доступна после исполнения акта типа message в следующем акте. Исполнение акта message завершается ответом
пользователя. Ответ пользователя доступен через свойства этой переменной

* $$reply[message] - текст сообщения;
* $$reply[files] - массив ссылок на файлы, переданные в сообщении пользователем;
* $$reply[choice] - переменная содержит значение выбора, сделанного пользователем из предоставленных вариантов:
* $$reply[choice][text] - текст выбора;
* $$reply[choice][value] - значение выбора.

#### choice

Переменная доступна в акте message, в котором используется список вариантов выбора listChoices (
[/$def/listChoices](scenario.schema.json)). Для формирования списка вариантов выбора с использованием цикла по
переменной. При этом в шаблоне варианта выбора можно использовать переменные:

* $$choice[key] - ключ элемента списка;
* $$choice[value] - значение элемента списка.

#### sequence

Переменная доступна в цикле последовательности ([/$def/sequenceCycle](scenario.schema.json))

* $$sequence[key] - ключ элемента списка;
* $$sequence[value] - значение элемента списка.

### Присвоение значения переменной через вызов внешнего endpoint'а

В определенных случах требуется особый расчет значения переменной, алгоритм расчета которого не описан в сценарии. Для
этого служит особый тип присвоения переменной ([/$def/endpointAssign](scenario.schema.json)). При исполнении
акта присвоения через вызов внешнего endpoint'а на указанный адрес endpoint'а отправляется post запрос с телом,
содержащим массив переменных с текущими значениями, в формате json. В ответ с кодом 200 должно вернуться значение,
которое присваивается указанной переменной.

## Исполнитель поведения

Описание сценария использует исполнитель сценария для реализации поведения цифрового помощника. Описание сценария
не ограничивает выбор языка программирования или технологию реализации исполнителя. Реализация предполагается в виде
отдельных приложений, плагинов или модулей.

В ходе взаимодействия цифрового помощника с пользователем исполнение
сценария находится на определенном акте. Нахождение исполнения сценария на определенном этапе назовем статусом
исполнения сценария. Статус исполнения сценария хранится для каждого отдельного пользователя. Статус определяется
текущим актом и контекстом. Под контекстом понимается совокупность значений переменных сценария.
