# LIVE Månadsrapportering

[LIVE Looker Studio](https://lookerstudio.google.com/u/1/reporting/c26d2b74-92b0-4e10-9d56-cae729b4bf9a/page/OcPuD)

[LIVE Google Spreadsheet](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=0#gid=0)

[Arbetsdokument Product Owner](https://docs.google.com/document/d/1pllOjWyltkUvYwzyBv6ONDSVjxbfYzdPQOwzbjWZaaU/edit?tab=t.0)

## Uppdatera månadsdashboarden - måndag och torsdag

LIVE i Looker studio hämtar data från LIVE spreadsheet vilket är det dokument där alla följande steg ska göras. De flikar som importeras är Data, Stamkunder, Införsäljning och Cohort nykund.

### Uppdatera Införsäljningen

1. Kör queryn **Datum innevarande månad**.
``` sql
set @from = DATE_FORMAT(CONCAT(YEAR(CURDATE()), '-', MONTH(CURDATE()), '-01'), '%Y-%m-%d');
set @to = DATE_FORMAT(DATE_ADD(CONCAT(YEAR(CURDATE()), '-', MONTH(CURDATE()), '-01'), INTERVAL 1 MONTH), '%Y-%m-%d');
```
2. Kör **Sålda STL nykund** och klistra in i kolumn H: **Antal sålda STL**.
``` sql
SELECT DATE(s.created_at), count(s.entity_id)
FROM gs_subscription s
WHERE s.state = 2 AND s.created_at >= @FROM AND s.created_at < @TO AND s.converted IS NULL
GROUP BY DATE(s.created_at)
ORDER BY DATE(s.created_at) ASC;
```
3. Kör **Sålda ÖVL** och klistra in i kolumn O: **Antal sålda ÖVL** i samma flik.
``` sql
SELECT
	DATE_FORMAT(o.created_at, "%Y-%m-%d") AS datum,
	round(sum(IF(is_box.value = 1, 1, 0) * i.qty_ordered)) AS boxes
FROM
	sales_order o
	JOIN sales_order_item i ON o.entity_id = i.order_id
	LEFT JOIN catalog_product_entity_int is_box ON i.product_id = is_box.entity_id
	AND is_box.attribute_id = 264
WHERE
	o.created_at >= @from
	AND o.created_at < @to
	AND o.status IN ("complete", "pending", "processing")
	AND o.base_subtotal > 0
	AND i.price > 0
	AND o.increment_id NOT IN(
		/* Generated subscription orders */
		SELECT
			o.increment_id
		FROM
			sales_order o
			JOIN sales_order_item i ON o.entity_id = i.order_id
		WHERE
			o.created_at BETWEEN @FROM AND @TO
			AND o.status IN ("complete", "pending", "processing")
			AND (i.name LIKE 'stamkundsladan%')
			AND o.subtotal > 0
	)
GROUP BY
	datum
ORDER BY
	datum ASC;
```

---

### Uppdatera fliken Omsättning M2 Product Sales Day

- Gå till **M2 Admin > GS Reports > Product Sales**.
- För att ta ut rapporten:
  - Ställ in månadens datum
  - Delivery date
  - Period Type: **day**
  - Lämna SKU tomt
- Rapporten kan ta lång tid att hämta. Ladda om sidan om siffrorna inte uppdateras.
- Exportera som `.csv`.
- Radera kolumnerna A–R i fliken.
- Importera från ruta A1.
- Uppdatera tidsstämpeln.

---

### Uppdatera fliken Cohort nykund

Denna flik följer hur lång tid det tar för nya kunder att ta minst två leveranser.

- Kör queryn **Cohort nykund**.
- Klistra in resultatet i cell A2.

``` sql
SET @start_date = DATE_FORMAT(DATE_SUB(CURDATE(), INTERVAL 6 MONTH), '%Y-%m-01');
SET @end_date = DATE_FORMAT(CURDATE(), '%Y-%m-01');

WITH new_customers AS (
    SELECT customer_id, created_at
    FROM gs_subscription
    WHERE created_at BETWEEN @start_date AND @end_date
      AND converted IS NULL
),
deliveries AS (
    SELECT 
        o.customer_id,
        COUNT(DISTINCT o.entity_id) AS delivery_count
    FROM sales_order o
    JOIN sales_order_item oi ON o.entity_id = oi.order_id
    WHERE o.status IN ("complete", "processing",

```

---

### Uppdatera Looker Studio

- Gå in i **LIVE Månadsrapportering** i Looker Studio.
- Klicka på de tre prickarna uppe till höger → *Refresh data*.
- Uppdateringar hämtas också automatiskt var fjärde timme.

---

### Kontrollera LIVE-flikarna

- Gå igenom samtliga flikar i **LIVE** och felsök värden som saknas.

---
