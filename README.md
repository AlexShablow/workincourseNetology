# Курсовая работа Нетология
## Итоговый DT-файл информационной базы находится в блоке Realeses данного репозитория под именем Release-ItFirm
## Описание задачи
Развить созданную конфигурацию "Управление ИТ-фирмой", обеспечив возможность учета хозяйственных операций на управленческом плане счетов, расчет и начисление заработной платы, а также взаимодействие сотрудников с использованием задач и процессов.

Итогом станет конфигурация, на базовом уровне содержащая все элементы управленческих учетных систем на платформе 1С:Предприятие.

## Технические требования
Программный код всех модулей должен быть оформлен в соответствии со стандартами разработки на платформе 1С:Предприятие по ссылке: https://its.1c.ru/db/v8std#content:456:hdoc
Недопустимо выполнять запросы в цикле, в том числе неявные при обращении через точку к реквизиту ссылки. То есть обращение Строка.Номенклатура.ТипНоменклатуры недопустимо.
При работе с регистрами расчета необходимо получать данные для расчета через виртуальные таблицы. Недопустимо использовать объектную модель для обращения к регистрам расчета.

## Функциональные требования

Выгрузка информационной базы (файл с расширением dt), включающий демоданные и конфигурацию с именем "УправлениеИТФирмой" содержащую:

### Перечисление ЮридическоеФизическоеЛицо

#### Со значениями
ЮридическоеЛицо, ФизическоеЛицо

### Справочник Контрагенты

#### С реквизитами
ПолноеНаименование, ИНН, КПП, EMail, Телефон, ФактическийАдрес, ЮридическийАдрес, Тип (ПеречислениеСсылка.ЮридическоеФизическоеЛицо), Ответственный (СправочникСсылка.Сотрудники).
Типы должны быть подходящими, их длина и точность - разумно достаточными.

#### С формой элемента

КПП должен быть доступен только для контрагентов-юридических лиц. Доступность должна отрабатываться как при изменении типа, так и при перечитывании элемента справочника из ИБ.

При изменении наименования полное наименование должно заполняться с разворачиванием распространенных сокращений организационно-правовых форм (например, *АО "Вектор"* должно превращаться в *Акционерное общество "Вектор"*).

Должна быть реализована проверка корректности ИНН как для физических, так и для юридических лиц. Проверять нужно и при попытке записи (с выдачей предупреждения и отказом), и при изменении ИНН (с подсветкой поля ввода и/или выводом текста ошибки рядом с ним).

#### С формой списка

В форме списка должны присутствовать все существенные реквизиты в разумном порядке.

#### С модулем объекта

В коде которого определено заполнение по умолчанию:
* Тип - Юридическое лицо.
* Ответственный - текущий сотрудник из параметра сеанса **ТекущийСотрудник**.

### Перечисление Пол

#### Со значениями
Мужской, Женский

### Справочник Сотрудники

#### С реквизитами
EMail, ДатаРождения, ИдентификаторПользователяИБ, Оклад, Пол, СтавкаЧаса, Телефон.
Типы должны быть подходящими, их длина и точность - разумно достаточными.
Для сумм нужно использовать определяемый тип.

#### С модулем менеджера

В котором переопределено получение формы объекта в зависимости от права доступа "Администрирование".

#### С формой элемента ФормаАдминистратора

Которая открывается для пользователей с правом доступа "Администрирование".

В которой, помимо реквизитов сотрудника, есть флажок "Вход разрешен" и поля ввода "Имя для входа" и "Пароль", отражающие свойства пользователя ИБ.

При записи сотрудника из этой формы при необходимости должно выполняться:
* создание пользователя ИБ с ролью **БазовыеПрава**;
* изменение его пароля и имени для входа;
* при снятии флажка - отключение стандартной аутентификации.

Рядом с полем ввода пароля должна быть команда "Случайный пароль", генерирующая случайный пароль и показывающая его пользователю.

#### С формой элемента ФормаПользователя

Которая открывается для пользователей без права доступа "Администрирование" и содержит элементы управления для реквизитов сотрудника, упорядоченные по смыслу.

### Константу и функциональную опцию ВестиРасчетЗарплаты

Константа не должна присутствовать в командном интерфейсе сама по себе (флажок "Использовать стандартные команды" должен быть снят).
В состав ФО должны входить зарплатные реквизиты справочника **Сотрудники**, а константа должна присутствовать флажком в общей форме **НастройкаПрограммы**.

### Общую форму и общую команду **НастройкаПрограммы**

Форма должна содержать основной реквизит типа **НаборКонстант** и поле флажка для константы **ВестиРасчетЗарплаты**.
Общая команда должна открывать общую форму, принадлежать подсистеме **Настройки** и присутствовать в командном интерфейсе раздела "Настройки".

### Параметр сеанса ТекущийСотрудник

Типа **СправочникСсылка.Сотрудники**. Должен заполняться элементом справочника **Сотрудники**, идентификатор пользователя ИБ которого совпадает с идентификатором текущего пользователя ИБ.

### Роли БазовыеПрава и ПолныеПрава

Роль **ПолныеПрава** должна давать права на все, кроме:
* интерактивного удаления элементов справочников;
* пометки на удаление и удаления помеченных предопределенных элементов справочников. 

Роль **БазовыеПрава** должна давать права на чтение, просмотр и ввод по строке всех данных. Редактирование, добавление и изменение разрешается только для справочника **Контрагенты**. Роль не должна давать права на открытие настроек программы.

### Подсистему "Настройки"

Содержащую все справочники и общую команду **НастройкаПрограммы**.

## Процесс выполнения

Старайтесь использовать наработки, выполненные ранее в домашних заданиях блока А.

### Подсистема Настройки

Создайте подсистему **Настройки**, куда будете включать все добавляемые далее объекты метаданных.

### Константа и ФО

Создайте константу **ВестиРасчетЗарплаты** типа **Булево** и соответствующую ФО.
Константу включите в подсистему **Настройки**. Не забудьте снять флажок "Использовать стандартные команды", чтобы константа не появилась в командном интерфейсе сама по себе.

### Настройка программы

Создайте общую форму **НастройкаПрограммы**.
Добавьте в нее основной реквизит типа **НаборКонстант**.
Выведите константу **ВестиРасчетЗарплаты** на форму полем флажка. Заголовок флажка по общим правилам разместите справа.
Создайте общую команду **НастройкаПрограммы**, в модуле которой реализуйте открытие общей формы.
Включите команду и форму в подсистему **Настройки**.

### Справочник Сотрудники

Создйте справочник и наполните его указанным в Требованиях набором реквизитов.
Включите зарплатные реквизиты в состав ФО **ВестиРасчетЗарплаты**.

Создайте две формы элемента, **ФормаАдминистратора** и **ФормаПользователя**. Форма администратора будет основной.

В модуле менеджера определите обработчик события **ОбработкаПолученияФормы()**. В нем, в зависимости от наличия права доступа **Администирование**, открывайте форму администратора или форму пользователя. Право администрирования проверяйте так:

    Если ПравоДоступа("Администрирование", Метаданные) Тогда
		
Выбранную форму лучше возвращать не по имени, а как объект метаданных, например:

    Метаданные.Справочники.Сотрудники.Формы.ФормаАдминистратора
    

#### ФормаПользователя

Выведите в нее реквизиты сотрудника в разумном порядке.

#### ФормаАдминистратора

Можно создать копированием формы **ФормаПользователя**.
Создайте в ней две группы, левую и правую.
В левую выведите реквизиты сотрудника в разумном порядке (как в форме пользователя).
Добавьте реквизиты формы **ВходРазрешен** (Булево), **ИмяДляВхода** и **Пароль** и выведите их в правую группу флажком и двумя полями ввода. Для поля ввода **Пароль** включите режим пароля, чтобы введенное забивалось звездочками.

##### Случайный пароль
Создайте команду и кнопку **СлучайныйПароль**.
В обработчике создайте случайный пароль из 5-6 букв и цифр, отключив у поля ввода **Пароль** режим пароля, чтобы пользователь увидел его и мог скопировать.

##### ПриЧтенииНаСервере
Найдите пользователя ИБ по идентификатору (**ТекущийОбъект.ИдентификаторПользователяИБ**).
Заполните по данным пользователя ИБ реквизиты формы **ИмяДляВхода** и **Пароль**. Реквизит **ВходРазрешен** заполните по реквизиту пользователя ИБ **АутентификацияСтандартная**.
Если идентификатор не заполнен, или поиск пользователя ИБ возвращает **Неопределено**, считайте, что вход не разрешен, а имя и пароль пусты.
    
##### ПередЗаписьюНаСервере
Если идентификатор пользователя ИБ заполнен - найдите пользователя ИБ и обновите его реквизиты значениями реквизитов  **ВходРазрешен**, **ИмяДляВхода** и **Пароль**.
Если идентификатор не заполнен, а флажок **ВходРазрешен** выставлен - создайте пользователя ИБ и добавьте ему роль **Метаданные.Роли.БазовыеПрава**. 

После создания пользователя ИБ присвойте его идентификатор реквизиту **ИдентификаторПользователяИБ** записываемого объекта, чтобы найти этого пользователя ИБ при следующем открытии формы.
Обратите внимание на то, что учебная версия платформы не позволяет задать непустой пароль. Логику задания пароля, однако, все равно нужно реализовать, а в тестах задавать пустой пароль.

#### Форма списка

Создайте форму списка, добавив в нее все существенные реквизиты в разумном порядке и расположении.

### Параметр сеанса

Создайте параметр сеанса **ТекущийСотрудник** типа **СправочникСсылка.Сотрудники**.
В модуле сеанса определите процедуру **УстановкаПараметровСеанса**.
В ней, для простоты не анализируя параметр **ТребуемыеПараметры**, найдите сотрудника по значению реквизита **ИдентификаторПользователяИБ**, используя функцию менеджера:

    Справочники.Сотрудники.НайтиПоРеквизиту(<...>);

Значение идентификатора получите, обратившись к функции **ТекущийПользователь()** менеджера **ПользователиИнформационнойБазы**.

### Справочник Контрагенты

Создайте справочник и наполните его указанным в Требованиях набором реквизитов.
Не забудьте поставить флажок "Заполнять из данных заполнения" хотя бы для реквизитов **Тип** и **Ответственный**.
В модуле объекта определите обработчик **ОбработкаЗаполнения**, в котором заполните реквизит **Тип** значением по умолчанию **ЮридическоеЛицо**, а реквизит **Ответственный** - значением параметра сеанса **ТекущийСотрудник**.

#### Форма элемента

Создайте форму элемента, поместив на нее все существенные реквизиты в разумном порядке и расположении.

Реализуйте управление доступностью поля ввода **КПП** в зависимости от типа контрагента, сделав это в обработчиках событий **ПриЧтенииНаСервере** формы и **ПриИзменении** поля ввода **Тип**.

Реализуйте функцию определения корректности ИНН, которая работала бы со всеми ИНН и возвращала бы, кроме значения Булево (Истина - корректен, Ложь - некорректен), текстовое описание ошибки как неявно возвращаемое значение. 
Алгоритмы расчета:

https://www.egrul.ru/test_inn.html
https://keysystems.ru/files/fo/arm_budjet/show_docum/BKS/onlinehelp/index.html?ro_kr_algor_klyuch_inn.htm

Реализуйте вызов этой функции в двух местах:
* В обработчике события **ПередЗаписью** формы - с отказом от записи при неверном ИНН и выводом предупреждения, содержащего текстовое описание ошибки.
* В обработчике события **ПриИзменении** поля ввода ИНН - с подсветкой текста поля ввода и/или с выводом рядом с полем ввода текстового описания ошибки в виде декорации или подсказки.

Считайте, что пустой ИНН корректен.

В обработчике события **ПриИзменении** поля ввода **Наименование** реализуйте поиск распространенных сокращений организационно-правовых форм в начале строки и заполнение полного наименования по краткому с заменой сокращения ОПФ на ее полное наименование. Не изменяйте полное наименование, если оно уже было изменено пользователем вручную. Чтобы это реализовать, при чтении формы получайте полное наименование из краткого и, если полученный заменой результат соответствует полному наименованию, запоминайте в невидимом булевском реквизите формы признак того, что полное наименование можно изменять автоматически.

#### Форма списка

Создайте форму списка, добавив в нее все существенные реквизиты в разумном порядке и расположении.

### Роли

Создайте роли **ПолныеПрава** и **БазовыеПрава**.

В состав роли **ПолныеПрава** включите все права, сняв только:
* интерактивное удаление элементов справочников.
* пометку на удаление и удаление помеченных предопределенных элементов.

В состав роли **БазовыеПрава** включите права на чтение, просмотр и ввод по строке всех данных, включая параметры сеанса. Разрешите также изменение, добавление, редактирование и интерактивную пометку удаления справочника **Контрагенты**.

### Проверка

1. Проверьте, что подсистема **Настройки** содержит все добавленные вами объекты метаданных.
2. Создайте пользователя ИБ с полными правами.
3.  Запустив программу под ним, удостоверьтесь в видимости раздела "Настройки", всех справочников и команды "Настройка программы".
4. Включите ФО "Вести расчет зарплаты".
5. В справочнике "Сотрудники" создайте одного или нескольких сотрудников, разрешив вход.
6. Запустив программу под ними, удостоверьтесь в видимости раздела "Настройки" и всех справочников.
7. Открыв какого-нибудь сотрудника, убедитесь, что открывается именно форма пользователя.
8. Проверьте создание и редактирование контрагентов, введя реальные данные.


В существующей подсистеме Настройки:

* Перечислением **ТипыНоменклатуры** (Товар, Услуга)

* Перечислением **СтавкиНДС** (БезНДС, НДС10, НДС20)

* Справочником **НоменклатурныеГруппы**
   * Без иерархии, с наименованием разумной длины

* Справочником **Номенклатура**, который:
   * Имеет наименование разумной длины и неограниченную иерархию групп и элементов
   * Содержит реквизиты Тип, СтавкаНДС и НоменклатурнаяГруппа, используемые для элементов и обязательные к заполнению

* Документом **УстановкаЦен**, который:
   * Содержит реквизит шапки Ответственный и табличную часть Цены с реквизитами Номенклатура и Цена
   * Имеет форму, в которой реализованы:
      * Выбор и подбор номенклатуры с автоматическим назначением цен согласно срезу последних регистра сведений Цены
   * Формирует движения по регистру сведений **Цены**
   
* Регистром сведений **Цены**, который:
   * Содержит измерение Номенклатура и ресурс Цена
   * Является периодическим с подчинением регистратору (документу **УстановкаЦен**)

* Документом **УстановкаСкидок**, который:
   * Содержит реквизит шапки Ответственный и табличную часть Скидки с реквизитами НоменклатураНоменклатурнаяГруппа и Скидка
   * Имеет форму, в которой реализованы:
      * Выбор номенклатуры и номенклатурных групп с автоматическим назначением скидок согласно срезу последних регистра сведений Скидки
   * Формирует движения по регистру сведений **Скидки**
   
* Регистром сведений **Скидки**, который:
   * Содержит измерение НоменклатураНоменклатурнаяГруппа и ресурс Скидка
   * Является периодическим с подчинением регистратору (документу **УстановкаСкидок**)
   
* Журналом документов **ЦеныИСкидки**, который:
   * Содержит документы **УстановкаЦен** и **УстановкаСкидок** и графу Ответственный

Подсистемой **Сделки**, а в ней:

* Документом **ПоступлениеТоваровИУслуг**, который:
   * Содержит реквизиты шапки Поставщик, Ответственный, Сумма и табличную часть ТоварыИУслуги с реквизитами Номенклатура, Количество, Цена, Сумма, СтавкаНДС, СуммаНДС
   * Имеет форму, в которой реализован выбор и подбор номенклатуры с автоматическим пересчетом числовых колонок по правилам:
      * При изменении реквизитов Количество и Цена пересчитывается Сумма и СуммаНДС (см. ниже "Правила расчета НДС")
      * При изменении реквизита Сумма пересчитывается Цена и СуммаНДС
      * При изменении реквизита СтавкаНДС пересчитывается СуммаНДС
   * Перед записью заполняет реквизит шапки Сумма итогом по одноименной колонке табличной части
   * Формирует движения:
      * Расход по регистру накопления **ВзаиморасчетыСКонтрагентами** с указанием поставщика в сумме общего итога по реквизиту ТЧ Сумма
      * Приход по регистру накопления **Товары** в разрезе номенклатуры типа Товар согласно реквизитам ТЧ Количество и Сумма
      * Движения по регистру накопления **Расходы** в разрезе номенклатуры типа Услуга согласно реквизиту ТЧ Сумма

* Документом **РеализацияТоваровИУслуг**, который:
   * Содержит реквизит шапки Покупатель, Ответственный, Сумма и табличную часть ТоварыИУслуги с реквизитами Номенклатура, Количество, Скидка, Цена, Сумма, СтавкаНДС, СуммаНДС
   * Имеет форму, в которой реализован выбор и подбор номенклатуры с автоматическим назначением цены и скидки, а также пересчетом числовых колонок по правилам:
      * При изменении реквизитов Количество и Цена пересчитывается Сумма (с учетом скидки) и СуммаНДС (см. ниже "Правила расчета НДС")
      * При изменении реквизита Скидка пересчитывается Сумма и СуммаНДС (см. ниже "Применение скидок")
      * При изменении реквизита Сумма пересчитывается СуммаНДС
      * При изменении реквизита СтавкаНДС пересчитывается СуммаНДС
   * Перед записью заполняет реквизит шапки Сумма итогом по одноименной колонке табличной части
   * Формирует движения:
      * Приход по регистру накопления **ВзаиморасчетыСКонтрагентами** с указанием покупателя в сумме общего итога по реквизиту ТЧ Сумма
      * Расход по регистру накопления **Товары** в разрезе номенклатуры типа Товар согласно реквизиту ТЧ Количество и сумме, определенной согласно средней стоимости остатков этого товара. В отсутствие достаточного остатка проведение не выполняется.
      * Движения по регистру накопления **Расходы** в разрезе номенклатуры типа Товар в сумме себестоимости продаж (сумме расхода по регистру Товары)
      * Движения по регистру накопления **Доходы** в разрезе номенклатуры всех типов согласно реквизиту ТЧ Сумма
      
* Журналом документов **Сделки**, который:
   * Содержит документы **ПоступлениеТоваровИУслуг** и **РеализацияТоваровИУслуг** с графами Контрагент, Ответственный и Сумма

Подсистемой **Деньги**, а в ней:

* Документом **ПоступлениеДенежныхСредств**, который:
  * Содержит реквизиты Плательщик и Сумма
  * Формирует движение: расход по регистру накопления **ВзаиморасчетыСКонтрагентами** с указанием плательщика и суммы

* Документом **СписаниеДенежныхСредств**, который:
  * Содержит реквизиты Получатель и Сумма
  * Формирует движение: приход по регистру накопления **ВзаиморасчетыСКонтрагентами** с указанием получателя и суммы
  
* Журналом документов **Деньги**, который:
   * Содержит документы **ПоступлениеДенежныхСредств** и **СписаниеДенежныхСредств** с графами Контрагент, Ответственный и Сумма

* Регистром накопления **ВзаиморасчетыСКонтрагентами**, который:
  * Имеет вид Остатки
  * Содержит измерение Контрагент и ресурс Сумма
  * Подчинен регистраторам **ПоступлениеТоваровИУслуг**, **РеализацияТоваровИУслуг**, **ПоступлениеДенежныхСредств**, **РасходованиеДенежныхСредств**
  * Положительные остатки по нему означают дебиторскую задолженность (нам должны), отрицательные - кредиторскую (мы должны)
      
* Регистром накопления **Товары**, который:
  * Имеет вид Остатки
  * Содержит измерение Номенклатура и ресурсы Количество, Сумма
  * Подчинен регистраторам **ПоступлениеТоваровИУслуг** и **РеализацияТоваровИУслуг**
  * Хранит текущие остатки товаров и их себестоимость с учетом НДС
  
* Регистром накопления **Доходы**, который:
  * Имеет вид Обороты
  * Содержит измерение Номенклатура и ресурсы Количество, Сумма
  * Подчинен регистратору **РеализацияТоваровИУслуг**
  * Хранит доходы (выручку) от реализации товаров и услуг с учетом НДС
  
* Регистром накопления **Расходы**, который:
  * Имеет вид Обороты
  * Содержит измерение Номенклатура и ресурсы Количество, Сумма
  * Подчинен регистраторам **ПоступлениеТоваровИУслуг**, **РеализацияТоваровИУслуг**
  * Хранит расходы по приобретенным услугам и себестоимость реализованных товаров с учетом НДС
  
* Отчетом **ДоходыИРасходы**, который:
  * Выводит, соединяя, данные регистров **Доходы** и **Расходы** в три колонки ("Доходы", "Расходы", "Прибыль")
  * Содержит группировку по номенклатуре с учетом иерархии и общие итоги
  
* Отчетом **ДвижениеТоваров**, который:
  * Выводит данные регистра **Товары**: остатки и обороты по количеству и сумме
  * Содержит группировку по номенклатуре с учетом иерархии и общие итоги
  * Не суммирует количества в общем итоге и по иерархии номенклатуры
  
* Отчетом **ВзаиморасчетыСКонтрагентами**, который:
  * Выводит данные регистра **ВзаиморасчетыСКонтрагентами**: остатки и обороты
  * Содержит группировку по контрагентам и общие итоги
  
Ценообразование должно быть доступно только роли **ПолныеПрава**.

### Применение скидок

Скидки определяются по срезу последних регистра сведений Скидки. Если скидка установлена и на конкретный элемент справочника **Номенклатура**, и на номенклатурную группу, приоритет имеет скидка для конкретного элемента.

Цена определяется по данным регистра сведений Цены и не пересчитывается при изменении скидки. Сумма определяется по цене с учетом скидки как:
   Сумма = Количество * Цена * (100 - Скидка) / 100
При изменении суммы изменяется скидка, но не цена, по обратной формуле:
   Скидка = 100 * (1 - Сумма / Количество / Цена)

### Правила расчета НДС

НДС рассчитывается по ставкам, определяемым по значению перечисления **СтавкиНДС** (БезНДС - 0%, НДС10 - 10%, НДС20 - 20%). Сумма НДС определяется умножением суммы на ставку (т.е. НДС рассчитывается по схеме "в том числе", например, для ставки 20% и суммы 120 р сумма НДС будет равна 120 * 0.2 / (1 + 0.2) = 20.

Подсистемой **Учет**, а в ней:

* Регистром бухгалтерии **Управленческий**, который:
  * Содержит ресурсы Количество и Сумма определяемых типов
  * Поддерживает корреспонденции
   
* Планом видов характеристик **ВидыСубконто**, который:
  * Включает значения типов СправочникСсылка.Номенклатура, СправочникСсылка.Контрагенты, СправочникСсылка.Сотрудники
  * Содержит предопределенные элементы Номенклатура, Контрагенты, Сотрудники

* Планом счетов **Управленческий**, который:
  * Используется для учета в регистре бухгалтерии **Управленческий**
  * В качестве видов субконто использует ПВХ **ВидыСубконто**
  * Содержит предопределенные счета:
    * Активный 41 **Товары** (виды субконто: Номенклатура, учет по количеству)
    * Активный 50 **ДенежныеСредства**
    * Пассивный 60 **РасчетыСПоставщиками** (виды субконто: Контрагенты)
    * Активный 62 **РасчетыСПокупателями** (виды субконто: Контрагенты)
    * Активно-пассивный 70 **РасчетыССотрудниками** (виды субконто: Сотрудники)
    * Активно-пассивный 90 **ПрибылиИУбытки** с субсчетами
      * 90.1 **Доходы**
      * 90.2 **Расходы**
      
* Отчетом **ОборотноСальдоваяВедомость**, который:
  * Построен на СКД
  * Выводит отстатки и обороты по счетам за выбранный период

* Документы, созданные в рамках диплома Б, должны быть доработаны для формирования проводок по регистру бухгалтерии **Управленческий**:
  * Документ **ПоступлениеТоваровИУслуг**:
    * Для товаров - Дт **Товары** с заполнением субконто Номенклатура - Кт **РасчетыСПоставщиками** с заполнением субконто Контрагенты на сумму закупки.
    * Для услуг - Дт **Расходы** - Кт **РасчетыСПоставщиками** с заполнением субконто Контрагенты на сумму закупки.
  * Документ **РеализацияТоваровИУслуг**:
    * Для всех строк - Дт **РасчетыСПокупателями** с заполнением субконто Контрагенты - Кт **Доходы** на сумму продажи.
    * Для товаров - Дт **Расходы** - Кт **Товары** с заполнением субконто Номенклатура на сумму себестоимости списанного товара. Себестоимость должна рассчитываться по данным регистра бухгалтерии, а не по данным регистра накопления **Товары**. Недопустимо списание товара в минус.
  * Документ **ПоступлениеДенежныхСредств** (после расширения типа реквизита **Плательщик** типом **СправочникСсылка.Сотрудники**):
    * Для контрагентов - Дт **ДенежныеСредства** - Кт **РасчетыСПокупателями** с заполнением субконто **Контрагенты** на сумму платежа.
    * Для сотрудников - Дт **ДенежныеСредства** - Кт **РасчетыССотрудниками** с заполнением субконто **Сотрудники** на сумму платежа.
  * Документом **СписаниеДенежныхСредств** (после расширения типа реквизита **Получатель** типом **СправочникСсылка.Сотрудники**):
    * Для контрагентов - Дт **РасчетыСПоставщиками** с заполнением субконто **Контрагенты** - Кт **ДенежныеСредства** на сумму платежа.
    * Для сотрудников - Дт **РасчетыССотрудниками** с заполнением субконто **Сотрудники** - Кт **ДенежныеСредства** на сумму платежа.

Подсистемой **Зарплата**, а в ней:

* Регистром сведений **Календарь**, который:
  * Содержит измерение **День** (Дата) и ресурс **Рабочий** (Число)

* Регистром расчета **Зарплата**, который:
  * Содержит измерение **Сотрудник** и ресурс **Сумма** определяемого типа
  * Поддерживает период действия и период регистрации
  * Привязан к регистру **Календарь** как к календарю
   
* Планом видов расчета **Начисления**, который:
  * Используется для учета в регистре расчета **Зарплата**
  * Использует период действия и зависит по базе по периоду действия от самого себя.
  * Содержит предопределенные виды расчетов:
    * Больничный, Отпуск, ФиксированнаяПремия
    * ОплатаПоОкладу, вытесняемый видами расчета Больничный и Отпуск
    * ПремияПроцентом, зависящая по базе от вида расчета ОплатаПоОкладу
      
* Документом **НачислениеСписком**, который:
  * Содержит реквизит **Начисление** (ПланВидовРасчетаСсылка.Начисления) и начало и конец периода действия
  * Содержит табличную часть **Сотрудники** с сотрудниками и суммами
  * При проведении формирует движения:
    * По регистру расчета **Зарплата** с указанием сотрудника, вида расчета, периодов и суммы
    * По регистру бухгалтерии **Управленческий** в Дт счета **Расходы** с Кт счета **РасчетыССотрудниками** с заполнением субконто **Сотрудники** на ту же сумму.

* Документом **НачислениеОплатыПоОкладу**, который:
  * Содержит реквизит **ЗаМесяц** (Дата), определяющий месяц периода действия
  * Содержит табличную часть **Сотрудники** с сотрудниками
  * При проведении формирует движения:
    * По регистру расчета **Зарплата** с указанием сотрудника, вида расчета, периодов и суммы. Сумма рассчитывается по данным графиков как оклад, умноженный на частное от деления фактически отработанного времени (с учетом вытеснения больничным и окладом) на норму времени.
    * По регистру бухгалтерии **Управленческий** в Дт счета **Расходы** с Кт счета **РасчетыССотрудниками** с заполнением субконто **Сотрудники** на ту же сумму.    
    
* Документом **НачислениеПремииПроцентом**, который:
  * Содержит реквизит **ЗаМесяц** (Дата), определяющий месяц периода действия, и **Процент** (Число), определяющий процент премии
  * Содержит табличную часть **Сотрудники** с сотрудниками
  * При проведении формирует движения:
    * По регистру расчета **Зарплата** с указанием сотрудника, вида расчета, периодов и суммы. Сумма рассчитывается по данным базы (оплаты по окладу за базовый период) умножением базы на процент.
    * По регистру бухгалтерии **Управленческий** в Дт счета **Расходы** с Кт счета **РасчетыССотрудниками** с заполнением субконто **Сотрудники** на ту же сумму.

Подсистемой **Взаимодействие**, а в ней:

* Справочником **Роли** с наименованием разумной длины

* Регистром сведений **ИсполнителиРолей**, который:
  * Содержит измерения **Роль** (СправочникСсылка.Роли) и **Исполнитель** (СправочникСсылка.Сотрудники)
  * Используется для адресации задач

* Задачей **Задача**, которая:
  * Содержит реквизиты адресации **Исполнитель** (основной, СправочникСсылка.Сотрудники) и **Роль** (СправочникСсылка.Роли), заполняемые по данным процесса
  * В качестве текущего исполнителя использует значение параметра сеанса **ТекущийСотрудник**
  * Содержит реквизит **ПодробноеОписание** (строка неограниченной длины), заполняемый по данным процесса
  * Содержит форму задачи с ее прикладными реквизитами, недоступными для редактирования
  * Содержит форму **ЗадачиМне**, которая:
    * Содержит задачи, адресованные текущему сотруднику по данным виртуальной таблицы **Задача.Задача.ЗадачиПоИсполнителю** с учетом ролевой адресации
    * Выведена на рабочую область начальной страницы

* Процессом **Поручение**, который:
  * Использует задачу **Задача**
  * Содержит реквизиты, достаточные для заполнения создаваемых задач, и реквизит **Предмет** (ОпределяемыйТип.ПредметПроцесса)
  * Имеет простую схему, состоящую из точки старта, точки действия и точки завершения
  * При создании задач заполняет их наименование, подробное описание, исполнителя и роль по собственным данным
