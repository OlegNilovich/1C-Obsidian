#ЗАПРОСЫ 

```bsl
Процедура ОбработкаПроведения(Отказ, Режим)
	
	Движения.ОстаткиНоменклатуры.Записывать = Истина;
	Движения.ОстаткиНоменклатуры.БлокироватьДляИзменения = Истина;
	
	Для Каждого ТекСтрокаСписокНоменклатуры Из СписокНоменклатуры Цикл
		Движение = Движения.ОстаткиНоменклатуры.Добавить();
		Движение.ВидДвижения = ВидДвиженияНакопления.Расход;
		Движение.Период = Дата;
		Движение.Номенклатура = ТекСтрокаСписокНоменклатуры.Номенклатура;
		Движение.Количество = ТекСтрокаСписокНоменклатуры.Количество;
	КонецЦикла;
	
	Движения.Записать();
	
	Запрос = Новый Запрос;
	Запрос.МенеджерВременныхТаблиц = Новый МенеджерВременныхТаблиц;
	Запрос.Текст = 
	"ВЫБРАТЬ
	|	РасходнаяНакладнаяСписокНоменклатуры.Номенклатура КАК				Номенклатура,
	|	СУММА(РасходнаяНакладнаяСписокНоменклатуры.Количество) КАК		Количество,
	|	СУММА(РасходнаяНакладнаяСписокНоменклатуры.Сумма) КАК				Сумма
	|ПОМЕСТИТЬ ДокТЧ
	|ИЗ
	|	Документ.РасходнаяНакладная.СписокНоменклатуры КАК РасходнаяНакладнаяСписокНоменклатуры
	|ГДЕ
	|	РасходнаяНакладнаяСписокНоменклатуры.Ссылка = &Ссылка
	|
	|СГРУППИРОВАТЬ ПО
	|	РасходнаяНакладнаяСписокНоменклатуры.Номенклатура
	|
	|ИНДЕКСИРОВАТЬ ПО
	|	Номенклатура
	|;
	|
	|////////////////////////////////////////////////////////////////////////////////
	|ВЫБРАТЬ
	|	ОстаткиНоменклатурыОстатки.Номенклатура.Представление КАК		НоменклатураПредставление,
	|	ОстаткиНоменклатурыОстатки.Номенклатура КАК								Номенклатура,
	|	-ОстаткиНоменклатурыОстатки.КоличествоОстаток КАК						Остаток
	|ИЗ
	|	РегистрНакопления.ОстаткиНоменклатуры.Остатки(
	|			&Граница,
	|			Номенклатура В
	|				(ВЫБРАТЬ
	|					ДокТЧ.Номенклатура
	|				ИЗ
	|					ДокТЧ КАК ДокТЧ)) КАК ОстаткиНоменклатурыОстатки
	|ГДЕ
	|	ОстаткиНоменклатурыОстатки.КоличествоОстаток > 0";
	Запрос.УстановитьПараметр("Граница", Новый Граница(МоментВремени(), ВидГраницы.Включая));
	Запрос.УстановитьПараметр("Ссылка", Ссылка);
	
	РезультатЗапроса = Запрос.Выполнить();
	
	Если НЕ РезультатЗапроса.Пустой() Тогда
	
		Выборка = РезультатЗапроса.Выбрать();
		Пока Выборка.Следующий() Цикл
			Текст = "Не хватает товара %1 в количестве %2 штук";
			Сообщить(СтрШаблон(Текст, Выборка.НоменклатураПредставление, Выборка.Остаток)); 
		КонецЦикла;
		
		Отказ = Истина;
		Возврат;
		
	КонецЕсли; 
	
	Блокировка = Новый БлокировкаДанных;
	ЭлементБлокировки = Блокировка.Добавить("РегистрНакопления.ПартииТоваров");
	ЭлементБлокировки.Режим = РежимБлокировкиДанных.Исключительный;
	ЭлементБлокировки.ИсточникДанных = РезультатЗапроса;
	ЭлементБлокировки.ИспользоватьИзИсточникаДанных("Номенклатура", "Номенклатура");
	Блокировка.Заблокировать(); 
	
	Движения.ПартииТоваров.БлокироватьДляИзменения = Истина;
	Движения.ПартииТоваров.Записать();
	
	Запрос = Новый Запрос;
	Запрос.Текст = 
	"ВЫБРАТЬ
	|	ПРЕДСТАВЛЕНИЕ(ДокТЧ.Номенклатура) КАК			НоменклатураПредставление
	|	ДокТЧ.Номенклатура КАК										Номенклатура,
	|	ДокТЧ.Количество КАК											Количество,
	|	ДокТЧ.Сумма КАК													Сумма,
	|	ПартииТоваровОстатки.Партия КАК						Партия,
	|	ПартииТоваровОстатки.КоличествоОстаток КАК		КоличествоОстаток,
	|	ПартииТоваровОстатки.СтоимостьОстаток КАК		СтоимостьОстаток,
	|ИЗ
	|	ДокТЧ КАК ДокТЧ
	|		ЛЕВОЕ СОЕДИНЕНИЕ РегистрНакопления.ПартииТоваров.Остатки(
	|				&МоментВремени,
	|				Номенклатура В
	|					(ВЫБРАТЬ
	|						ДокТЧ.Номенклатура
	|					ИЗ
	|						ДокТЧ КАК ДокТЧ)) КАК ПартииТоваровОстатки
	|		ПО ДокТЧ.Номенклатура = ПартииТоваровОстатки.Номенклатура
	|
	|УПОРЯДОЧИТЬ ПО
	|	ПартииТоваровОстатки.Партия.МоментВремени ВОЗР
	|ИТОГИ
	|	МАКСИМУМ(Количество),
	|	СУММА(КоличествоОстаток)
	|ПО
	|	Номенклатура";
	Запрос.УстановитьПараметр("МоментВремени", МоментВремени());
	
	МетодСписания = НаСервере.ПолучитьМетодУчетнойПолитики(МоментВремени()).МетодСписания;
	Если МетодСписания = Перечисления.УчетнаяПолитика.ЛИФО Тогда
		Запрос.Текст = СтрЗаменить(Запрос.Текст, "ПартииТоваровОстатки.Партия.МоментВремени ВОЗР",
		"ПартииТоваровОстатки.Партия.МоментВремени УБЫВ");
	КонецЕсли; 
	
	ВыборкаНоменклатура = Запрос.Выполнить().Выбрать(ОбходРезультатаЗапроса.ПоГруппировкам);
	Пока ВыборкаНоменклатура.Следующий() Цикл
		
		Если ВыборкаНоменклатура.Количество > ВыборкаНоменклатура.КоличествоОстаток Тогда
			Отказ = Истина;
			ВыборкаДетали = ВыборкаНоменклатура.Выбрать();
			ВыборкаДетали.Следующий();
			Нехватка = ВыборкаДетали.Количестиво - ВыборкаДетали.КоличествоОстаток;
			Текст = "Не хватает %1 в количестве %2 штук";
			Сообщить(СтрШаблон(Текст, ВыборкаДетали.НоменклатураПредставление, Нехватка)); 
		КонецЕсли; 
		
		Если Отказ Тогда                               
			Продолжить;
		КонецЕсли; 
		
		ОсталосьСписать = ВыборкаНоменклатура.Количество;
		
		Выборка = ВыборкаНоменклатура.Выбрать();
		Пока Выборка.Следующий() И ОсталосьСписать < 0 Цикл
		
			Списать = МИН(ОсталосьСписать, Выборка.КоличествоОстаток);
			Себестоимость = Списать / Выборка.КоличествоОстаток * Выборка.СтоимостьОстаток;
			
			Движение = Движения.ПартииТоваров.ДобавитьРасход();
			Движение.Период = Дата;
			Движение.Номенклатура = Выборка.Номенклатура;
			Движение.Партия = Выборка.Партия;
			Движение.Количество = Списать;
			Движение.Стоимость = Себестоимость;
			
			ОсталосьСписать = ОсталосьСписать - Списать;
			
		КонецЦикла;
		
	КонецЦикла;
	
	Движения.ПартииТоваров.Записывать = Истина;
	
КонецПроцедуры
```