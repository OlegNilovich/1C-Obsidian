#ЗАПРОСЫ 

Оператор **ЕСТЬ NULL** позволяет проверить значение выражения слева от него на [NULL](v8help://SyntaxHelperQueries/NULL). Если значение равно [NULL](v8help://SyntaxHelperQueries/NULL) – результатом оператора будет [ИСТИНА](v8help://SyntaxHelperQueries/TRUE), иначе – [ЛОЖЬ](v8help://SyntaxHelperQueries/FALSE). Применение **НЕ** изменяет действие оператора на обратное.

> **см. также:** [Логические выражения](v8help://SyntaxHelperQueries/condition_expressions.html)

#### Пример:

> ВЫБРАТЬ  
>    Справочник.Номенклатура.Наименование,  
>    Справочник.Номенклатура.ЗакупочнаяЦена  
> ГДЕ   
>    Справочник.Номенклатура.ЗакупочнаяЦена Есть NULL