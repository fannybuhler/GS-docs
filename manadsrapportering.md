# LIVE Månadsrapportering

[LIVE Looker Studio](https://lookerstudio.google.com/u/1/reporting/c26d2b74-92b0-4e10-9d56-cae729b4bf9a/page/OcPuD)

[LIVE Google Spreadsheet](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=0#gid=0)

[Arbetsdokument Product Owner](https://docs.google.com/document/d/1pllOjWyltkUvYwzyBv6ONDSVjxbfYzdPQOwzbjWZaaU/edit?tab=t.0)

## Uppdatera månadsdashboarden - måndag och torsdag

LIVE i Looker studio hämtar data från LIVE spreadsheet. De flikar som importeras är Data, Stamkunder, Införsäljning och Införsäljning GA4.

### 1. Uppdatera fliken "Införsäljning"

Sätt datum för innevarande månad

```
set @from = DATE_FORMAT(CONCAT(YEAR(CURDATE()), '-', MONTH(CURDATE()), '-01'), '%Y-%m-%d');
set @to = DATE_FORMAT(DATE_ADD(CONCAT(YEAR(CURDATE()), '-', MONTH(CURDATE()), '-01'), INTERVAL 1 MONTH), '%Y-%m-%d');
```

Klista in antalet per dag i **STL antal sålda**

```
SELECT DATE(s.created_at), count(s.entity_id)
FROM gs_subscription s
WHERE s.state = 2 AND s.created_at >= @FROM AND s.created_at < @TO AND s.converted IS NULL
GROUP BY DATE(s.created_at)
ORDER BY DATE(s.created_at) ASC;
```

Klista in antalet per dag i **Antal ÖVL**

```
SELECT
	DATE_FORMAT(o.created_at, "%Y-%m-%d") AS datum,
	round(sum( IF(is_box.value = 1, 1, 0) * i.qty_ordered)) AS boxes
FROM
	sales_order o
	JOIN sales_order_item i ON o.entity_id = i.order_id
	LEFT JOIN catalog_product_entity_int is_box ON i.product_id = is_box.entity_id AND is_box.attribute_id = 264
WHERE
	o.created_at >= @from AND o.created_at < @to AND o.status IN("complete", "pending", "processing") AND o.base_subtotal > 0 AND i.price > 0 AND o.increment_id NOT IN(
	/* Generated subscription orders */
	SELECT
		o.increment_id FROM sales_order o
		JOIN sales_order_item i ON o.entity_id = i.order_id
	WHERE
		o.created_at BETWEEN @FROM AND @TO AND o.status IN("complete", "pending", "processing") AND(i.name LIKE 'stamkundsladan%') AND o.subtotal > 0)
GROUP BY
	datum
ORDER BY
	datum ASC;
```

### 2. Uppdatera fliken "Databas planerad omsättning"

Denna visar genererade ordrar och inplanerade leveranser.

```
/* Planerad omsättning innevarande månad */
WITH RECURSIVE DateList AS (
    SELECT DATE_FORMAT(NOW(), '%Y-%m-01') AS datum
    UNION ALL
    SELECT DATE_ADD(datum, INTERVAL 1 DAY)
    FROM DateList
    WHERE DATE_ADD(datum, INTERVAL 1 DAY) <= LAST_DAY(NOW())
),

orders_revenue as (
    SELECT days.datum, days.revenue + IF(stl.revenue is null, 0, stl.revenue) as prognosis_revenue, days.boxes + IF(stl.boxes is null,0,stl.boxes) as prognosis_boxes
    FROM (
        select orders.datum, count(distinct orders.entity_id) as no_of_orders, round(sum(boxes)) as boxes, round(sum(orders.order_subtotal)) as revenue
        FROM (
            select DATE_FORMAT(o.delivery_date, '%Y-%m-%d') as datum, o.entity_id, min(o.base_subtotal) as order_subtotal, sum(IF(is_box.value = 1,1,0) * i.qty_ordered) as boxes
            from sales_order o
            straight_join sales_order_item i on o.entity_id = i.order_id
            left join catalog_product_entity_int is_box on i.product_id = is_box.entity_id and is_box.attribute_id = 264
            WHERE
                o.status in ("complete", "pending", "processing") AND
                o.delivery_date >= @from AND
                o.delivery_date < @to AND
                o.base_subtotal > 0 AND
                i.product_id NOT IN (315,318,248) /*nekade kortbetalningar */ AND
                i.price > 0
            group BY datum, o.entity_id
            ) orders
        group by orders.datum
        ) days
        left join (
            /* Prognos omsättning innevarande månad dag för dag STL:er */
            select DATE_FORMAT(products.next_delivery_date, '%Y-%m-%d') as datum, sum(products.revenue) as revenue, sum(products.boxes) as boxes
            FROM (
                select p.next_delivery_date, p.product_id, sum(p.product_price) as revenue, sum(p.qty) as qty, sum(IF(is_box.value = 1,1,0) * p.qty) as boxes
                from gs_subscription s
                join gs_subscription_products p on s.entity_id = p.parent_id AND
                    s.state = 2 AND
                    s.status = 1 AND
                    p.is_active = 1 AND
                    p.product_price > 0 AND
                    p.next_delivery_date >= @FROM AND
                    p.next_delivery_date < @TO
                left join catalog_product_entity_int is_box on p.product_id = is_box.entity_id and is_box.attribute_id = 264
                GROUP BY p.next_delivery_date, p.product_id

                UNION ALL

                select p.next_delivery_date, p.product_id, sum(p.product_price) as revenue, sum(p.qty) as qty, sum(IF(is_box.value = 1,1,0) * p.qty) as boxes
                from endlesssubscription_subscription s
                join endlesssubscription_subscription_products p on s.entity_id = p.parent_id AND
                    s.state = 2 AND
                    s.status = 1 AND
                    p.is_active = 1 AND
                    p.product_price > 0 AND
                    p.next_delivery_date >= @FROM AND
                    p.next_delivery_date < @TO
                left join catalog_product_entity_int is_box on p.product_id = is_box.entity_id and is_box.attribute_id = 264
                GROUP BY p.next_delivery_date, p.product_id
                ) products

             GROUP BY datum
             ORDER BY datum, revenue desc

         ) stl on days.datum = stl.datum
)

SELECT DateList.datum, IFNULL(orders_revenue.prognosis_revenue,0) as prognosis_revenue, IFNULL(orders_revenue.prognosis_boxes,0) as prognosis_boxes
from DateList
left join orders_revenue on DateList.datum = orders_revenue.datum;
```

### 3. Uppdatera fliken "Omsättning M2 product Sales"

- Gå till M2 Admin > GS Reports > Product Sales.
- Ställ in månadens datum + Delivery date + Period Type (month). Lämna SKU tomt. Se till att siffrorna uppdateras efter sista rapporteringen, annars ladda om fliken så att datan hämtas på nytt.
- Exportera som csv, sedan importerar du och ersätter datan från ruta A1. Kolla att de nedersta raderna inte står med flera gånger.
- Uppdatera tidsstämpeln.
- Kolla samtliga flikar i LIVE:en och felsök värden som saknas.

### 4. Gå till Looker Studio

- Klicka på de tre prickarna uppe till höger och sen Refresh data. Uppdateringar hämtas också automatiskt var 4.e timme.
