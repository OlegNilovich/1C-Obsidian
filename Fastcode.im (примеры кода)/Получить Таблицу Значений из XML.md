Производит *десериализацию* таблицы значений из строки XML
###### Первый способ
```bsl
Функция ЗначениеИзСтрокиXML(СтрокаXML)
	
	ЧтениеXML = Новый ЧтениеXML;
	ЧтениеXML.УстановитьСтроку(СтрокаXML);
	
	Возврат СериализаторXDTO.ПрочитатьXML(ЧтениеXML);
	
КонецФункции
```
###### Второй способ
```bsl
Функция ПрочитатьТаблицуИзXML(СтрокаXML)
	
	ЧтениеXML = Новый ЧтениеXML;
	ЧтениеXML.УстановитьСтроку(СтрокаXML);
	ТипОбъектаXDTO = ФабрикаXDTO.Тип("http://v8.1c.ru/8.1/data/core","ValueTable");
	ОбъектXDTO = ФабрикаXDTO.ПрочитатьXML(ЧтениеXML,ТипОбъектаXDTO); 
	ОбъектXDTO.Проверить();
	
	ЧтениеXML.Закрыть();
	
	Возврат СериализаторXDTO.ПрочитатьXDTO(ОбъектXDTO);
	
КонецФункции
```