Отлично! Ниже приведён **шаблон Excel → CommerceML**, адаптированный под **строительный магазин** с возможностью экспорта данных о товарах, ценах, остатках и поставщиках в формате **CommerceML 2.1**.

---

## 📥 Шаблон Excel: "Каталог строительных товаров"

### 📄 Лист 1: `Товары`

| ИдТовара | Артикул | Наименование | Группа | ЕдиницаИзмерения | КодОКЕИ | ПоставщикИд | ПоставщикНаименование | ПоставщикИНН | Вес | Объём | Цвет | Материал | Плотность | ДругоеСвойство |
|----------|--------|--------------|--------|------------------|---------|-------------|------------------------|---------------|-----|--------|-------|-----------|------------|----------------|
| ITEM-PAINT-001 | PAINT-INT-3L-BLUE | Краска интерьерная акриловая, 3л, синяя | GROUP-PAINT-INTERIOR | шт | 796 | SUPPLIER-ACRILUX | ООО "АкрилСтрой" | 7712345678 | 3.2 | 3 | Синий | Акрил | 1200 | Водостойкая |
| ITEM-FIX-001 | DUBEL-8x40 | Дюбель-гвоздь 8x40 мм, 50 шт | GROUP-FIXING | шт | 796 | SUPPLIER-METIZ | ООО "Метиз-Сервис" | 7788990011 | 0.1 | 50 | — | Сталь | — | — |

> ⚠️ Примечание:  
> - `Группа` — Идентификатор из классификатора (должен соответствовать в листе `Группы`).  
> - `КодОКЕИ` — код единицы измерения по ОКЕИ (например: 796 = штука, 116 = метр, 166 = м², 113 = кг, 112 = литр).  
> - Дополнительные свойства (Цвет, Материал и т.д.) будут преобразованы в `ХарактеристикиТовара`.

---

### 📄 Лист 2: `Группы`

| ИдГруппы | НаименованиеГруппы | РодительскаяГруппа |
|----------|--------------------|---------------------|
| GROUP-PAINT | Лакокрасочные материалы | |
| GROUP-PAINT-INTERIOR | Краски интерьерные | GROUP-PAINT |
| GROUP-PAINT-EXTERIOR | Краски фасадные | GROUP-PAINT |
| GROUP-FIXING | Крепёж | |
| GROUP-INSULATION | Теплоизоляция | |

> 📌 `РодительскаяГруппа` — оставьте пустым, если группа верхнего уровня.

---

### 📄 Лист 3: `Цены`

| ИдТовара | ТипЦены | НаименованиеЦены | ЦенаЗаЕдиницу | Валюта | МинимальныйОбъём (опт) |
|----------|--------|------------------|---------------|--------|-------------------------|
| ITEM-PAINT-001 | PRICE-RETAIL | Розничная цена | 850.00 | руб | 1 |
| ITEM-PAINT-001 | PRICE-WHOLESALE | Оптовая цена | 720.00 | руб | 10 |
| ITEM-FIX-001 | PRICE-RETAIL | Розничная цена | 120.00 | руб | 1 |

> 💡 Можно указать разные цены в зависимости от объёма закупки.

---

### 📄 Лист 4: `Остатки`

| ИдТовара | СкладИд | СкладНаименование | Количество |
|----------|--------|-------------------|------------|
| ITEM-PAINT-001 | WAREHOUSE-MSK | Склад Москва | 45 |
| ITEM-FIX-001 | WAREHOUSE-MSK | Склад Москва | 200 |
| ITEM-FIX-001 | WAREHOUSE-SPB | Склад Санкт-Петербург | 75 |

---

### 📄 Лист 5: `Поставщики` (необязательно, если дублируется в `Товары`)

| ИдПоставщика | НаименованиеПоставщика | ИНН | Email | Телефон |
|--------------|------------------------|------|-------|---------|
| SUPPLIER-ACRILUX | ООО "АкрилСтрой" | 7712345678 | info@acrilux.ru | +7 (495) 123-45-67 |
| SUPPLIER-METIZ | ООО "Метиз-Сервис" | 7788990011 | sales@metiz.ru | +7 (495) 987-65-43 |

---

### 📄 Лист 6: `Свойства` (характеристики, которые будут в CommerceML)

| КодСвойства | НаименованиеСвойства | ТипЗначения | ЕдиницаИзмерения |
|-------------|------------------------|-------------|------------------|
| PROP-COLOR | Цвет | Справочник | |
| PROP-VOLUME | Объём | Число | л |
| PROP-MATERIAL | Материал | Справочник | |
| PROP-DENSITY | Плотность | Число | кг/м³ |
| PROP-WEIGHT | Вес | Число | кг |

> ⚠️ Эти свойства будут объявлены в `<Свойства>` внутри `<Классификатор>`.

---

## 🧩 Как преобразовать Excel → CommerceML?

### Варианты реализации:

### ✅ 1. **Через Python (рекомендуется для автоматизации)**

```python
import pandas as pd
from xml.etree.ElementTree import Element, SubElement, tostring
from xml.dom import minidom

# Загрузка данных
df_goods = pd.read_excel("catalog.xlsx", sheet_name="Товары")
df_groups = pd.read_excel("catalog.xlsx", sheet_name="Группы")
df_prices = pd.read_excel("catalog.xlsx", sheet_name="Цены")
df_stocks = pd.read_excel("catalog.xlsx", sheet_name="Остатки")
df_props = pd.read_excel("catalog.xlsx", sheet_name="Свойства")

# Создание корневого элемента
root = Element("КоммерческаяИнформация", {
    "ВерсияСхемы": "2.1",
    "ДатаФормирования": "2025-04-05T10:00:00"
})

# === Классификатор ===
classifier = SubElement(root, "Классификатор")
SubElement(classifier, "Ид").text = "CLASSIFIER-001"
SubElement(classifier, "Наименование").text = "Строительные товары"

# Группы
groups_elem = SubElement(classifier, "Группы")
for _, row in df_groups.iterrows():
    group = SubElement(groups_elem, "Группа")
    SubElement(group, "Ид").text = row["ИдГруппы"]
    SubElement(group, "Наименование").text = row["НаименованиеГруппы"]
    if pd.notna(row["РодительскаяГруппа"]):
        parent = SubElement(group, "Группы")
        SubElement(parent, "Группа")
        # (упрощённо — можно реализовать дерево)

# Свойства
props_elem = SubElement(classifier, "Свойства")
for _, row in df_props.iterrows():
    prop = SubElement(props_elem, "СвойствоНоменклатуры")
    SubElement(prop, "Ид").text = row["КодСвойства"]
    SubElement(prop, "Наименование").text = row["НаименованиеСвойства"]
    SubElement(prop, "ТипЗначений").text = row["ТипЗначения"]
    if pd.notna(row["ЕдиницаИзмерения"]):
        SubElement(prop, "Единица").text = row["ЕдиницаИзмерения"]

# === Каталог ===
catalog = SubElement(root, "Каталог")
SubElement(catalog, "Ид").text = "CATALOG-001"
products = SubElement(catalog, "Товары")

for _, good in df_goods.iterrows():
    product = SubElement(products, "Товар")
    SubElement(product, "Ид").text = good["ИдТовара"]
    SubElement(product, "Артикул").text = good["Артикул"]
    SubElement(product, "Наименование").text = good["Наименование"]

    unit = SubElement(product, "БазоваяЕдиница", {
        "Код": str(good["КодОКЕИ"]),
        "МеждународноеСокращение": "PCE"
    })
    unit.text = good["ЕдиницаИзмерения"]

    groups = SubElement(product, "Группы")
    SubElement(groups, "Ид").text = good["Группа"]

    # Поставщик
    supplier = SubElement(product, "Поставщик")
    SubElement(supplier, "Ид").text = good["ПоставщикИд"]
    SubElement(supplier, "Наименование").text = good["ПоставщикНаименование"]
    SubElement(supplier, "ИНН").text = str(good["ПоставщикИНН"])

    # Характеристики
    props = SubElement(product, "ХарактеристикиТовара")
    for prop_col in ["Цвет", "Материал", "Объём", "Плотность", "Вес", "ДругоеСвойство"]:
        if prop_col in df_props["НаименованиеСвойства"].values:
            code = df_props[df_props["НаименованиеСвойства"] == prop_col]["КодСвойства"].iloc[0]
            value = good[prop_col]
            if pd.notna(value):
                p = SubElement(props, "ХарактеристикаТовара")
                SubElement(p, "Ид").text = code
                SubElement(p, "Значение").text = str(value)

    # Цены
    prices = SubElement(product, "Цены")
    good_prices = df_prices[df_prices["ИдТовара"] == good["ИдТовара"]]
    for _, price_row in good_prices.iterrows():
        price = SubElement(prices, "Цена")
        SubElement(price, "Ид").text = price_row["ТипЦены"]
        SubElement(price, "Наименование").text = price_row["НаименованиеЦены"]
        SubElement(price, "ЦенаЗаЕдиницу").text = str(price_row["ЦенаЗаЕдиницу"])
        SubElement(price, "Валюта").text = price_row["Валюта"]
        SubElement(price, "Единица").text = good["ЕдиницаИзмерения"]

# === Остатки ===
stocks = SubElement(root, "Остатки")
SubElement(stocks, "Ид").text = "STOCK-001"
warehouses = SubElement(stocks, "Склады")
# (можно добавить уникальные склады из df_stocks)

stock_items = SubElement(stocks, "ОстаткиТоваров")
for _, stock in df_stocks.iterrows():
    item = SubElement(stock_items, "ОстатокТовара")
    nomenclature = SubElement(item, "Номенклатура")
    SubElement(nomenclature, "Ид").text = stock["ИдТовара"]
    warehouse = SubElement(item, "Склад")
    SubElement(warehouse, "Ид").text = stock["СкладИд"]
    SubElement(item, "Количество").text = str(stock["Количество"])

# Форматирование XML
def prettify(elem):
    rough = tostring(elem, 'utf-8')
    reparsed = minidom.parseString(rough)
    return reparsed.toprettyxml(indent="  ")

with open("catalog.xml", "w", encoding="utf-8") as f:
    f.write(prettify(root))

print("Файл catalog.xml успешно создан!")
```

> ✅ Сохраните как `export_to_commerceml.py` и запустите:  
> `pip install pandas openpyxl`  
> `python export_to_commerceml.py`

---

### ✅ 2. **Через Excel + VBA (для пользователей без Python)**

---

### ✅ 3. **Через Google Sheets + Apps Script**

---

## 📥 Скачать шаблон

---

## 🔚 Вывод

Этот шаблон:
- Удобен для ручного заполнения менеджерами.
- Легко автоматизируется.
- Соответствует стандарту CommerceML 2.1.
- Поддерживает категории, свойства, цены, остатки, поставщиков.
— могу адаптировать структуру под их требования.

Готов помочь с интеграцией!
