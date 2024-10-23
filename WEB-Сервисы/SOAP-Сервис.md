```bsl
// SOAP-сервис "Simple Object Access Protocol"

//========================================================================
// Создание Web-сервиса:

// 1. Создаем Web-сервис "Demo"
// 2. Прочее => Имя пространства имен:	https://demo.webservice
// 3. Прочее => Имя файла публикации:	demo.1cws
// 4. Прочее => Пакеты XDTO:			https://demo.webservice
// 5. Операции => создаем операцию "pricelist"
// 6. Операции => pricelist => Добавляем параметр "category"
// 7. Операции => Имя процедуры => жмем на Лупу и переходим в модуль:

Функция pricelist(category)
	
	ВидНоменклатуры = Неопределено;
	Если НЕ ПустаяСтрока(category) Тогда
		ВидНоменклатуры = Справочники.ВидыНоменклатуры.НайтиПоНаименованию(category, Истина);
	КонецЕсли;
	
	ДанныеXTDO = СформироватьПрайсЛистДляСайтаXML(ВидНоменклатуры);
	
	Возврат ДанныеXTDO;
	
КонецФункции


&НаСервере
Функция СформироватьПрайсЛистДляСайтаXML(ВидНоменклатуры)
	
	Запрос = Новый Запрос("
			|ВЫБРАТЬ
			|	Номенклатура.Наименование					КАК Наименование,
			|	Номенклатура.Артикул						КАК Артикул,
			|	Номенклатура.ВидНоменклатуры.Наименование	КАК ВидНоменклатуры
			|ИЗ
			|	Справочник.Номенклатура КАК Номенклатура");
	Если ВидНоменклатуры <> Неопределено Тогда
		ВидНоменклатуры = Справочники.ВидыНоменклатуры.НайтиПоНаименованию(ВидНоменклатуры, Истина);	
		Запрос.Текст = Запрос.Текст + " ГДЕ Номенклатура.ВидНоменклатуры = &ВидНоменклатуры";
		Запрос.УстановитьПараметр("ВидНоменклатуры", ВидНоменклатуры);
	КонецЕсли;

	Прайс = ФабрикаXDTO.Создать(ФабрикаXDTO.Тип("https://demo.webservice", "ПрайсЛист"));
	
	Выборка = Запрос.Выполнить().Выбрать();
	Пока Выборка.Следующий() Цикл
		СтрокаПрайса = ФабрикаXDTO.Создать(ФабрикаXDTO.Тип("https://demo.webservice", "СтрокаПрайса"));
		СтрокаПрайса.Наименование	= Выборка.Наименование;
		СтрокаПрайса.Категория		= Выборка.ВидНоменклатуры;
		СтрокаПрайса.Артикул		= Выборка.Артикул;
		Прайс.СтрокаПрайса.Добавить(СтрокаПрайса);
	КонецЦикла;
	
	Возврат Прайс;

КонецФункции


//========================================================================
// Описываем структуру XML-документа (ответа сервиса) с помощью XDTO-пакетама:

// 1. Создаем XDTO-пакет "ДемоСхема"
// 2. URI пространства имен: https://demo.webservice

// 3. Добавляем новый тип объекта: СтрокаПрайса
// - СтрокаПрайса => ПКМ => Добавляем свойства:
// - Имя: Наименование. Тип: string (http://www.w3.org/2001/XMLSchema)
// - Имя: Артикул.		Тип: string (http://www.w3.org/2001/XMLSchema)
// - Имя: Категория.	Тип: string (http://www.w3.org/2001/XMLSchema)
// - Имя: Цена.			Тип: float (http://www.w3.org/2001/XMLSchema)

// 4. Добавляем новый тип объекта: ПрайсЛист
// - ПрайсЛист => ПКМ => Добавляем свойство:
// - Имя: СтрокаПрайса
// - Тип: string СтрокаПрайса (https://demo.webservice)
// - Максимальное количество: -1 (это значит неограниченно)

// 5. Донастраиваем наш веб-сервис "Demo"
// - Web-сервисы => Demo => Пакеты XDTO => https://demo.webservice
// - Операции => pricelist => Тип возв. знач. ПрайсЛист (https://demo.webservice)
// - Администрирование => Публикация на веб сервисе => Публикуем веб-сервис "Demo"

// Итоговая схема:
<xs:schema xmlns:tns="https://demo.webservice" xmlns:xs="http://www.w3.org/2001/XMLSchema" targetNamespace="https://demo.webservice" attributeFormDefault="unqualified" elementFormDefault="qualified">
	<xs:complexType name="СтрокаПрайса">
		<xs:sequence>
			<xs:element name="Артикул" type="xs:string"/>
			<xs:element name="Наименование" type="xs:string"/>
			<xs:element name="НаименованиеПолное" type="xs:string"/>
			<xs:element name="Категория" type="xs:string"/>
			<xs:element name="Цена" type="xs:float"/>
		</xs:sequence>
	</xs:complexType>
	<xs:complexType name="ПрайсЛист">
		<xs:sequence>
			<xs:element name="СтрокиПрайса" type="tns:СтрокаПрайса" maxOccurs="unbounded"/>
		</xs:sequence>
	</xs:complexType>
</xs:schema>


//========================================================================
// Отправка запросов:

// 1. Устанавливаем "chrome-extension://eipdnjedkpcnlmmdfdkgfpljanehloah/workspace"
// 2. Boomerang => ADD SERVICE => SOAP [WSDL] => http://localhost/Education/ws/demo?wsdl
// 3. Слева появится сервис "Demo" => DemoSoap12Binding pricelist => Клик х2 => SEND
// 4. Обновление сервиса: Клик ПКМ на сервисе "Demo" => Reload

// 5. Отправляем POST запрос на адрес http://localhost/Education/ws/demo
<x:Envelope
    xmlns:x="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:dem="https://demo.webservice">
    <x:Header/>
    <x:Body>
        <dem:pricelist>
            <dem:category>Телевизоры</dem:category>
        </dem:pricelist>
    </x:Body>
</x:Envelope>

// 6. Получаем ответ (200 ОК):
<soap:Envelope
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <m:pricelistResponse
            xmlns:m="https://demo.webservice">
            <m:return
                xmlns:xs="http://www.w3.org/2001/XMLSchema"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
					xsi:type="m:ПрайсЛист">
                <m:СтрокиПрайса>
                    <m:Наименование>Телевизор Samsung UE55AU8000U</m:Наименование>
                    <m:Категория>Телевизоры</m:Категория>
                    <m:Артикул>UE55AU8000</m:Артикул>
                </m:СтрокиПрайса>
                <m:СтрокиПрайса>
                    <m:Наименование>Телевизор KIVI 65U710KB</m:Наименование>
                    <m:Категория>Телевизоры</m:Категория>
                    <m:Артикул>65U710KB65</m:Артикул>
                </m:СтрокиПрайса>
                <m:СтрокиПрайса>
                    <m:Наименование>Телевизор LG 50NANO766PA</m:Наименование>
                    <m:Категория>Телевизоры</m:Категория>
                    <m:Артикул>50NANO766P</m:Артикул>
                </m:СтрокиПрайса>
            </m:return>
        </m:pricelistResponse>
    </soap:Body>
</soap:Envelope>
```