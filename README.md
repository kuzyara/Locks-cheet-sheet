# Шпаргалка по блокировкам 1С

*Задача: наложение блокировок на регистр сведений*

* в репозитории 1С (https://github.com/1CEnterprise/MSQL-for-1C)
* в картинках (https://infostart.ru/public/629017/)
![alt text](https://github.com/kuzyara/Locks-cheet-sheet/blob/master/Блокировки.jpg)
Блокировки на **НаборЗаписей.Прочитать()** в 8.2 и в 8.3  работают по-разному:
* Например, в 8.2 у вас любое чтение будет ставить S-блокировку. Причем, чтение – это не просто Запрос.Выполнить(), но и Ссылка.Реквизит, Ссылка.ПолучитьОбъект() и т.д. 
* А в 8.3 без режима совместимости уже блокировок на чтение не будет. Таким образом, 8.3 безусловно выигрывает в плане параллельности работы.
* Ну а дальше начинается самое интересное – например, для конструкции НаборЗаписей.Прочитать() в управляемом режиме 8.2 у вас будет S-блокировка на сервере СУБД (это естественно). Но кроме этого также будет разделяемая блокировка на сервере 1С, причем это проявляется и в 8.2, и в 8.3. И главная проблема в том, что эта разделяемая блокировка у вас будет длиться до конца транзакции – пока транзакция не закончится, данные будут блокированы.
Поэтому рекомендация номер один – если набор записей вам нужен только для чтения, лучше использовать запрос, а не объектную модель. Тогда вы ничего блокировать не будете, а если и будете, то ненадолго.

Блокировки платформы:
* Р - РежимБлокировкиДанных.Разделяемый
* И - РежимБлокировкиДанных.Исключительный

Блокировки СУБД:
* Sch-S = Блокировка стабильности схемы. Гарантирует, что элемент схемы, такой как таблица или индекс, не будет удален до тех пор, пока сеанс связи удерживает блокировку стабильности схемы на данный элемент схемы;
* Sch-М = Блокировка изменения схемы. Должен поддерживаться любым сеансом связи, во время которого предполагается изменить схему данного ресурса. Гарантирует, что другие сеансы не имеют ссылок на обозначенный объект;
* S = Коллективная блокировка. Удерживающему сеансу предоставлен коллективный доступ к ресурсу;
* U = Блокировка обновления. Указывает блокировку обновления, полученную на ресурсы, которые со временем могут быть обновлены. Используется для предотвращения общей формы взаимоблокировки, которая возникает, когда множество сеансов блокируют ресурсы для потенциального обновления в последующее время;
* X = Монопольная блокировка. Удерживающему сеансу предоставлен исключительный доступ к ресурсу;
* IS = Блокировка с намерением коллективного доступа. Указывает намерение поместить S блокировки на некоторые подчиненные ресурсы в иерархии блокировок;
* IU = Блокировка с намерением обновления. Указывает намерение поместить U блокировки на некоторые подчиненные ресурсы в иерархии блокировок;
* IX = Блокировка с намерением монопольного доступа. Указывает намерение поместить X блокировки на некоторые подчиненные ресурсы в иерархии блокировок;
* SIU = Коллективная блокировка с намерением обновления. Указывает коллективный доступ к ресурсу с намерением получения блокировок обновления на подчиненные ресурсы в иерархии блокировок;
* SIX = Коллективная блокировка с намерением монопольного доступа. Указывает коллективный доступ к ресурсу с намерением получения монопольных блокировок на подчиненные ресурсы в иерархии блокировок;
* UIX = Блокировка обновления с намерением монопольного доступа. Указывает блокировку обновления ресурса с намерением получения монопольных блокировок на подчиненные ресурсы в иерархии блокировок;
* BU = Блокировка массового обновления. Используется для массовых операций;
--------------------------------------------------
Какие бывают блокировки:
* Объектные блокировки: https://infostart.ru/public/543218/
    * Пессимистические (явные и неявные)
    * Оптимистические
      
Объектная оптимистическая блокировка
Оптимистическая блокировка представляет собой проверку, которая выполняется перед записью объекта в базу данных. У объекта есть свойство «ВерсияДанных», которая вместе с объектом считывается из базы данных. Оптимистическая блокировка производит перед записью производит сравнение значения свойства «ВерсияДанных» объекта, который находится в оперативной памяти с значением свойства «ВерсияДанных» объекта находящийся в базе данных. Если значения свойства «ВерсияДанных» у объектов отличается, то оптимистическая блокировка запрещает запись объекта в базу данных и выдает сообщение об ошибке.

Объектная пессимистическая блокировка (неявная, в форме)
Пессимистическая объектная блокировка предназначена для запрета изменений данных объекта, пока блокировка не будет снята. Система (с помощью соответствующих расширений формы объекта) автоматически устанавливает пессимистическую блокировку, в момент, когда пользователь пытается произвести изменение данных объекта. Если после этого другой пользователь, например, попытается выполнить редактирование того же объекта, ему будет выдано сообщение о том, что не удалось заблокировать объект.

Объектная пессимистическая блокировка (явная)
* Без формы (обычные формы)
Методы «Заблокировать()», «Разблокировать()» и «Заблокирован()», используются для объектов базы данных, существуют только на сервере. Метод «Заблокирован()» не может быть использован для проверки, заблокирован ли вообще объект базы данных, например, другими пользователями. Для этого следует использовать метод «Заблокировать()» в попытке.
* Управляемы формы
Для работы с блокировками из управляемой формы без вызова сервера можено использовать методы: «ЗаблокироватьДанныеФормыДляРедактирования()» и «РазблокироватьДанныеФормыДляРедактирования()». Данные методы используются для блокировки или разблокировки данных основного реквизита формы. Контролируются флагом «Сохраняемые данные». Данный флаг определяет будет ли при интерактивном редактировании блокироваться данные основного реквизита, или нет.

--------------------------------------------------
Какие бывают блокировки:
* Транзакционные:
    * управляемые
    * автоматические
--------------------------------------------------
 Какие бывают блокировки:
* Транзакционные:
    * исключительные
    * разделяемые
--------------------------------------------------
 Какие бывают блокировки:
* Транзакционные:
    * явные
    * неявные
