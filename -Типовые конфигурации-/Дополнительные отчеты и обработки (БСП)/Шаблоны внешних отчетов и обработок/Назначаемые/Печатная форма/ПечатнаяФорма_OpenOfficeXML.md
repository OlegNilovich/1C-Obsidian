
Модуль объекта обработки:
```bsl
#Область ПрограммныйИнтерфейс

Функция СведенияОВнешнейОбработке() Экспорт
	
	ИмяОбработки = ЭтотОбъект.Метаданные().Имя; 
    Синоним = ЭтотОбъект.Метаданные().Синоним; 
    Синоним = ?(ЗначениеЗаполнено(Синоним),Синоним, ИмяОбработки);

	Вид = ДополнительныеОтчетыИОбработкиКлиентСервер.ВидОбработкиПечатнаяФорма();
	ПараметрыРегистрации = ДополнительныеОтчетыИОбработки.СведенияОВнешнейОбработке();
	ПараметрыРегистрации.Вид = Вид;
	ПараметрыРегистрации.Версия = "1.0.0.1";
	ПараметрыРегистрации.Информация = "Внешняя печатная форма Open Office XML """ + Синоним + """";

	ПараметрыРегистрации.Назначение.Добавить("Документ.ЗаказПокупателя");

	Использование = ДополнительныеОтчетыИОбработкиКлиентСервер.ТипКомандыВызовСерверногоМетода();
	Команда = ПараметрыРегистрации.Команды.Добавить();
	Команда.Представление = Синоним;
	Команда.Идентификатор = ИмяОбработки;
	Команда.Использование = Использование;
	Команда.ПоказыватьОповещение = Истина;
	
	Возврат ПараметрыРегистрации;

КонецФункции

#КонецОбласти

#Область Печать

Процедура Печать(МассивОбъектов, КоллекцияПечатныхФорм, ОбъектыПечати, ПараметрыВывода) Экспорт

	ПечатнаяФорма = УправлениеПечатью.СведенияОПечатнойФорме(КоллекцияПечатныхФорм, ЭтотОбъект.Метаданные().Имя);
	ПечатнаяФорма.ОфисныеДокументы = ОфисныеДокументы(МассивОбъектов);
	ПечатнаяФорма.СинонимМакета = ЭтотОбъект.Метаданные().Синоним;
	
КонецПроцедуры

Функция ОфисныеДокументы(МассивОбъектов)
	
	ОфисныеДокументы = Новый Соответствие;
	
	Шаблон = "Заказ покупателя № [Номер] от [Дата] ([Контрагент)]";
	
	РеквизитыДокументов = ОбщегоНазначения.ЗначенияРеквизитовОбъектов(МассивОбъектов, "Номер, Дата, Контрагент");
	
	Для каждого Ссылка Из МассивОбъектов Цикл
		
		РеквизитыДокумента = РеквизитыДокументов[Ссылка];
		РеквизитыДокумента.Дата = Формат(РеквизитыДокумента.Дата, "ДЛФ=D");
		РеквизитыДокумента.Номер = ПрефиксацияОбъектовКлиентСервер.НомерНаПечать(РеквизитыДокумента.Номер, Истина, Истина);
		
		ИмяДокумента = СтроковыеФункцииКлиентСервер.ВставитьПараметрыВСтроку(Шаблон, РеквизитыДокумента);
		
		АдресДокумента = СформироватьОфисныйДокумент(Ссылка);

		ОфисныеДокументы.Вставить(АдресДокумента, ИмяДокумента);
	
	КонецЦикла;
	
	Возврат ОфисныеДокументы;
		
КонецФункции

Функция СформироватьОфисныйДокумент(Ссылка)
  
 	// Подготавливаем макет для формирования печатной формы OpenXML
    МакетДокумента = ПолучитьМакет("ПФ_DOC_ДемоОфисныйДокумент");
    Макет = УправлениеПечатью.ИнициализироватьМакетОфисногоДокумента(МакетДокумента, Неопределено);
	
	// Создаем структуру областей формируемой печатной формы OpenXМL
	ОписаниеОбластей = Новый Структура;
	УправлениеПечатью.ДобавитьОписаниеОбласти(ОписаниеОбластей, "Заголовок",	 "Общая");
	УправлениеПечатью.ДобавитьОписаниеОбласти(ОписаниеОбластей, "Шапка",		 "Общая");
	УправлениеПечатью.ДобавитьОписаниеОбласти(ОписаниеОбластей, "ШапкаТаблицы",	 "Общая");
	УправлениеПечатью.ДобавитьОписаниеОбласти(ОписаниеОбластей, "СтрокаТаблицы", "СтрокаТаблицы");
	УправлениеПечатью.ДобавитьОписаниеОбласти(ОписаниеОбластей, "Подвал",		 "Общая");
	
	// Подготавливаем печатную форму в формате офисного документа
	ПечатнаяФорма = УправлениеПечатью.ИнициализироватьПечатнуюФорму(Неопределено, Неопределено, Макет);

	//получаем данные документа
	Запрос = Новый Запрос;
	Запрос.Текст = 
		"ВЫБРАТЬ
		|	РеквизитыШапки.Номер КАК Номер,
		|	РеквизитыШапки.Дата КАК Дата,
		|	РеквизитыШапки.Контрагент КАК ПредставлениеЗаказчика,
		|	РеквизитыШапки.Договор КАК Договор,
		|	РеквизитыШапки.СуммаДокумента КАК СуммаДокумента,
		|	РеквизитыШапки.Ответственный КАК Менеджер,
		|	РеквизитыШапки.Комментарий КАК Комментарий,
		|	РеквизитыШапки.Организация КАК ПредставлениеОрганизации
		|ИЗ
		|	Документ.ЗаказПокупателя КАК РеквизитыШапки
		|ГДЕ
		|	РеквизитыШапки.Ссылка = &Ссылка
		|;
		|
		|////////////////////////////////////////////////////////////////////////////////
		|ВЫБРАТЬ
		|	ТабличнаяЧасть.Ссылка КАК Ссылка,
		|	ТабличнаяЧасть.НомерСтроки КАК НС,
		|	ТабличнаяЧасть.Номенклатура КАК Номенклатура,
		|	ТабличнаяЧасть.Количество КАК Количество,
		|	ТабличнаяЧасть.ЕдиницаИзмерения КАК ЕдИзм,
		|	ТабличнаяЧасть.Цена КАК Цена,
		|	ТабличнаяЧасть.СтавкаНДС КАК СтавкаНДС,
		|	ТабличнаяЧасть.Сумма КАК Сумма,
		|	ТабличнаяЧасть.СуммаНДС КАК СуммаНДС,
		|	ТабличнаяЧасть.Всего КАК СуммаВсего
		|ИЗ
		|	Документ.ЗаказПокупателя.Запасы КАК ТабличнаяЧасть
		|ГДЕ
		|	ТабличнаяЧасть.Ссылка = &Ссылка";
	
	Запрос.УстановитьПараметр("Ссылка", Ссылка);
	
	ДанныеДляПечати = Запрос.ВыполнитьПакет();

	Шапка = ДанныеДляПечати[0].Выгрузить();
	Товары = ДанныеДляПечати[1].Выгрузить();
	
	ДанныеШапка = ОбщегоНазначения.СтрокаТаблицыЗначенийВСтруктуру(Шапка[0]);
	ДанныеШапка["Номер"] = ПрефиксацияОбъектовКлиентСервер.НомерНаПечать(ДанныеШапка["Номер"], Истина, Истина);
	ДанныеШапка["Дата"] = Формат(ДанныеШапка["Дата"], "ДФ=dd.MM.yyyy");
	
	ДанныеТовары = ОбщегоНазначения.ТаблицаЗначенийВМассив(Товары);
	
	// Вывод заголовка
	Область = УправлениеПечатью.ОбластьМакета(Макет, ОписаниеОбластей["Заголовок"]);
	УправлениеПечатью.ПрисоединитьОбластьИЗаполнитьПараметры(ПечатнаяФорма, Область, ДанныеШапка);
		
	// Вывод шапки документа
	Область = УправлениеПечатью.ОбластьМакета(Макет, ОписаниеОбластей["Шапка"]);
	УправлениеПечатью.ПрисоединитьОбластьИЗаполнитьПараметры(ПечатнаяФорма, Область, ДанныеШапка);
	
	// Вывод таблицы
	Если ДанныеТовары.Количество() > 0 Тогда
		Область = УправлениеПечатью.ОбластьМакета(Макет, ОписаниеОбластей["ШапкаТаблицы"]);
		УправлениеПечатью.ПрисоединитьОбласть(ПечатнаяФорма, Область, Ложь);
		Область = УправлениеПечатью.ОбластьМакета(Макет, ОписаниеОбластей["СтрокаТаблицы"]);
		УправлениеПечатью.ПрисоединитьИЗаполнитьКоллекцию(ПечатнаяФорма, Область, ДанныеТовары);
	КонецЕсли;

	// Вывод подвала
	Область = УправлениеПечатью.ОбластьМакета(Макет, ОписаниеОбластей["Подвал"]);
	УправлениеПечатью.ПрисоединитьОбластьИЗаполнитьПараметры(ПечатнаяФорма, Область, ДанныеШапка);

	// Помещаем сформированную печатную форму в соответствие ОфисныеДокументы
	АдресХранилищаПечатнойФормы = УправлениеПечатью.СформироватьДокумент(ПечатнаяФорма);
	
	//удаление временных файлов
	УправлениеПечатью.ОчиститьСсылки(ПечатнаяФорма);	
	УправлениеПечатью.ОчиститьСсылки(Макет);

	Возврат АдресХранилищаПечатнойФормы;
	
КонецФункции // СформироватьОфисныйДокумент()

#КонецОбласти
```

Макет(двоичные данные) ПФ_DOC_ДемоОфисныйДокумент:
![[Pasted image 20241112060602.png]]