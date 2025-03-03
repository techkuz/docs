# Настройки датасета

Датасет работает с источниками в режиме прямого доступа: все запросы к данным {{ datalens-short-name }} выполняет на стороне источника. Настройки датасета определяют, каким образом датасет будет работать с данными источника.


## Подключение нескольких таблиц {#multi-table}

Если в источнике доступно несколько таблиц, вы можете объединять их с помощью оператора JOIN.
Объединение происходит через создание связи. В связи вы указываете поля исходной таблицы и поля таблицы, с которой происходит объединение.

Доступны следующие операторы JOIN:

* [INNER](https://en.wikipedia.org/wiki/Join_(SQL)#Inner_join)
* [LEFT](https://en.wikipedia.org/wiki/Join_(SQL)#Left_outer_join)
* [RIGHT](https://en.wikipedia.org/wiki/Join_(SQL)#Right_outer_join)
* [FULL](https://en.wikipedia.org/wiki/Join_(SQL)#Full_outer_join)

## Фильтрация по умолчанию для новых чартов {#default-filters}

В датасете можно [создать](../../operations/dataset/create-filter.md) фильтр по умолчанию. Он будет применен к любому новому чарту, созданному на основе данных из текущего датасета.

{% note info %}

- Фильтр для отдельного чарта задается в настройках чарта.
- Фильтры по умолчанию не применяются к данным в области предпросмотра датасета.

{% endnote %}

Фильтрация по умолчанию для новых чартов позволяет:
* Уменьшить объем данных, запрашиваемых из источника при построении чарта.
* Не добавлять одинаковые фильтры в новые чарты, созданные на основе данных из одного датасета.


## Управление доступом {#access-management}

Вы можете настроить права доступа ко всему датасету.  Подробнее в разделе [{#T}](../../security/manage-access.md). 

Также можно разграничить доступ к данным на уровне строк (_Row-level security_ или _RLS_). Подробнее в разделе [{#T}](../../security/row-level-security.md).


## Выполнение SQL-запросов в датасетах {#sql-request-in-datatset}

Источник данных датасета можно определять произвольными SQL-запросами над подключениями к БД. Текст запроса при обращении к источнику данных исполняется в виде подзапроса. Подробнее о том, как использовать SQL-запросы в датасете, читайте в разделе [{#T}](../../operations/dataset/add-data.md).
При использовании SQL-запросов в датасетах рекомендуется:
* Ограничить права пользователя, прописанного в подключении, до `read-only`.
* Пользователям, которые не должны иметь права на выполнение произвольного запроса, назначить право доступа `Исполнение` на подключение и связанные с ним датасеты.

Включить или отключить использование подзапросов в качестве источника можно при [создании подключения](../connection.md) и при его редактировании.

#### См. также {#see-also}
- [{#T}](../../operations/dataset/create.md)
- [{#T}](../calculations/index.md)
- [{#T}](../calculations/index.md#how-to-create-calculated-field)
