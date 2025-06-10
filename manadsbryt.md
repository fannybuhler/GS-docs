# LIVE Månadsbryt

## Relaterade dokument

[LIVE Looker Studio](https://lookerstudio.google.com/u/1/reporting/c26d2b74-92b0-4e10-9d56-cae729b4bf9a/page/OcPuD)

[LIVE Månadsrapportering data](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=0#gid=0)

[Arbetsdokument Product Owner](https://docs.google.com/document/d/1pllOjWyltkUvYwzyBv6ONDSVjxbfYzdPQOwzbjWZaaU/edit?tab=t.0)

[GS_utfall sålda och levererade produkter](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=627471941#gid=627471941)

[GS_utfall stamkunder](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=0#gid=0)

## Månadsbryt i LIVE

> Uppdatera grafer i looker studio som visar hur vi taktar på i månaden

Görs i dokumentet [LIVE Månadsrapportering data](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=0#gid=0)

##### 1. Uppdatera fliken &nbsp; [Data](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=0#gid=0)

- Denna fyller i månadens datum själv men man behöver manuellt fixa raden i slutet så det är rätt antal dagar för innevarande månad.
- Radera förra månadens inskrivna **Omsättning (E)** och **Lådor antal levererade (J)**.

##### 2. Uppdatera fliken &nbsp; [Databas planerad omsättning](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=1971546303#gid=1971546303)

- Uppdatera datumen.

##### 3. Uppdatera fliken &nbsp; [Stamkunder](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=258667635#gid=258667635)

- Kolla att sista dagen i månaden blir rätt

##### 4. Uppdatera fliken &nbsp; [Införsäljning](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=1823519833#gid=1823519833)

- Radera förra månadens inskrivna försäljning **STL Antal sålda (F)** och **Antal ÖVL (K)**.

##### 5. Gör &nbsp; [Månadsrapporteringen](manadsrapportering.md)

Fördelningen över månaden baseras på veckodagar enligt fliken [Leveransdagar fördelning](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=975771408#gid=975771408).
Uppdateras förslagsvis några gånger per år eller om det varit stora ändringar.

## Månadsbryt i GS_utfall sålda och levererade produkter

> Rapportera sålda och levererade produkter efter månadsslut

Görs i dokumentet [GS_utfall sålda och levererade produkter](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=627471941#gid=627471941)

### Magento: Levererade produkter

1. Gå till **Magento > GS Reports > Product Sales**
2. Ställ in:
   - **Datumintervall**: föregående månad
   - **Date type**: Delivery date
   - **Period type**: Month
3. Exportera som CSV.
4. Öppna fliken **ps_delivered**:
   - Klistra in CSV:en längst ner i **kolumn A**.
   - Radera header-raden från importen.
   - Kolumner **T (Product Category)**, **U och V (Tillval omsättning)** fylls i automatiskt. Dra ner rader vid behov.
   - Om cell i kolumn T är röd med texten “Other”:
     - Kopiera lådans **ID**, **namn** och **SKU**.
     - Gå till fliken **products_lookup** och fyll i **Category** med STL, ÖVL eller TVL (tillvalslåda). Detta behöver bara göras när vi sålt en ny låda som inte redan finns där.

### Magento: Sålda produkter

1. Kör samma rapport, men med:
   - **Date type**: _Order date_
2. Uppdatera webbläsaren för att säkerställa färska siffror.
3. Exportera som CSV.
4. Klistra in i fliken **ps_ordered**:
   - Klistra in längst ner i **kolumn A**.
   - Radera header-raden.
   - Kontrollera **kolumn T (Product Category)** som ovan.

### Databas: Levererade STL-lådor

För de levererade stamkundslådorna använder vi en separat query. Detta görs för att få mer detaljerad information om ordervärde och snittpriser som inte går att få ur Magento-rapporten.

#### Steg:

1. Ändra datumen högst upp i queryn nedan.
2. Kör queryn.
3. Klistra in resultatet längst ner i **kolumn A** i fliken [STL Levererade](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=418109271#gid=418109271).
4. Kompletterande data till höger i fliken hämtas från **Product Sales** och uppdateras automatiskt – dra ner kolumnerna vid behov.


```sql
/* Utfall - Levererade STL-lådor */
SET @from_date = "2025-07-01";
SET @to_date = "2025-07-31";

SELECT
    boxes.datum,
    boxes.year,
    boxes.month,
    boxes.product_id,
    boxes.name,
    boxes.sku,
    boxes.qty_delivered,
    COALESCE(new_s.new_customers, 0) AS first_time_stl_customers,
    IF(new_s.new_customers > 0, ROUND(boxes.qty_delivered - new_s.new_customers), boxes.qty_delivered) AS repeat_stl_customers,
    ROUND(boxes.revenue) AS total_order_revenue,
    COALESCE(new_s.new_order_revenue, 0) AS first_time_stl_customer_revenue,
    IF(new_s.new_customers > 0, ROUND(boxes.revenue - new_s.new_order_revenue), ROUND(boxes.revenue)) AS repeat_customer_revenue,
    boxes.box_revenue AS total_stl_revenue,
    boxes.aov
FROM
(
    SELECT
        DATE_FORMAT(o.delivery_date, "%Y-%m-01") AS datum,
        YEAR(o.delivery_date) AS YEAR,
        MONTH(o.delivery_date) AS MONTH,
        oi.product_id,
        oi.name,
        oi.sku,
        ROUND(SUM(oi.qty_ordered)) AS qty_delivered,
        SUM(oi.qty_ordered) / COUNT(o.entity_id) AS qty_per_order,
        SUM(o.base_subtotal) AS revenue,
        ROUND(SUM(CASE WHEN oi.price > 0 THEN oi.qty_ordered * oi.price ELSE 0 END)) AS box_revenue,
        ROUND(SUM(CASE WHEN o.base_subtotal > 0 THEN o.base_subtotal ELSE 0 END) / 
              COUNT(CASE WHEN o.base_subtotal > 0 THEN 1 END)) AS aov,
        ROUND(AVG(oi.price)) AS box_price
    FROM
        sales_order o
        JOIN sales_order_item oi ON o.entity_id = oi.order_id
    WHERE
        oi.product_id IN (17, 404, 405, 406)
        AND o.status IN ("complete", "processing")
        AND o.is_cron_order = 1
        AND o.delivery_date >= @from_date
        AND o.delivery_date <= @to_date
    GROUP BY
        datum, YEAR, MONTH, oi.product_id, oi.sku, oi.name
) boxes
LEFT JOIN
(
    SELECT
        DATE_FORMAT(o.delivery_date, "%Y-%m-01") AS datum,
        YEAR(o.delivery_date) AS YEAR,
        MONTH(o.delivery_date) AS MONTH,
        i.product_id,
        i.name,
        i.sku,
        ROUND(SUM(i.qty_ordered)) AS new_customers,
        ROUND(SUM(o.subtotal)) AS new_order_revenue
    FROM
        sales_order o
        JOIN sales_order_item i ON o.entity_id = i.order_id
        JOIN (
            SELECT MIN(oj.entity_id) AS first_order_id
            FROM sales_order oj
            JOIN sales_order_item ij ON oj.entity_id = ij.order_id
            WHERE
                oj.status IN ("complete", "processing")
                AND oj.is_cron_order = 1
                AND ij.product_id IN (17, 404, 405, 406)
                AND oj.customer_id > 0
                AND oj.delivery_date >= DATE_SUB(@from_date, INTERVAL 2 YEAR)
            GROUP BY oj.customer_id
        ) AS first_orders ON o.entity_id = first_orders.first_order_id
    WHERE
        i.product_id IN (17, 404, 405, 406)
    GROUP BY
        datum, YEAR, MONTH, i.product_id, i.name, i.sku
) new_s ON boxes.datum = new_s.datum
        AND boxes.product_id = new_s.product_id
        AND boxes.name = new_s.name
ORDER BY boxes.datum, boxes.product_id;
```
---

## Månadsbryt i GS_utfall stamkunder

> Rapportera stamkunder

Görs i dokumentet [GS_utfall stamkunder](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=0#gid=0)

##### 1. Nya stamkunder per låda och intervall

- Skriv in rätt datum och kör följande query

```
SELECT DATE_FORMAT(o.created_at,"%Y-%m-01") as datum, year(o.created_at) as year, month(o.created_at) as month, "LY" as LY, i.product_id, i.`name` as product_name, i.sku, o.`interval`, count(o.subscription_id)
FROM sales_order o JOIN sales_order_item i on o.entity_id = i.order_id and i.product_id in (404,405,406,17)
join gs_subscription s on o.subscription_id = s.entity_id and s.`state` = 2 /* Kolla att prenumerationen skapades upp ok */
WHERE
    o.created_at >= "2024-12-01 00:00:00" AND o.created_at < "2024-12-31 23:59:59" AND
    o.status = "subscription_start"
GROUP BY datum, year, month, LY, i.product_id, product_name, sku,  o.`interval`
ORDER BY datum, year, month, i.product_id,  o.`interval`;
```

- Klistra in resultatet i fliken [Stamkundslådan införsäljning](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=517552797#gid=517552797)
- Dra ner raderna i kolumn K till O.

##### 2. Befintliga stamkunder dag för dag

- Exportera data om kategorikunder för månaden från M2 > Sales > Endless Subscription > Subscription Statistics
- Klistra in i fliken [Stamkunder Kategori](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=1220565309#gid=1220565309)

- Exportera data om poängkunder för månaden från M2 > Sales > GS Subscription > Subscription Statistics
- Klistra in i fliken [Stamkunder Poäng](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=931244722#gid=931244722)

##### 3. Fördelning av stamkundslådan per storlek

- Kör följande query

```
SELECT product_id, COUNT(*) AS active_customers
FROM (
    -- Count from gs_subscription
    SELECT sp.product_id
    FROM gs_subscription s
    JOIN gs_subscription_products sp ON s.entity_id = sp.parent_id
    WHERE s.state = 2
      AND s.status = 1
      AND sp.product_id IN (17, 404, 405, 406)

    UNION ALL

    -- Count from endlesssubscription_subscription
    SELECT sp.product_id
    FROM endlesssubscription_subscription s
    JOIN endlesssubscription_subscription_products sp ON s.entity_id = sp.parent_id
    WHERE s.state = 2
      AND s.status = 1
      AND sp.product_id IN (17, 404, 405, 406)
) AS combined_data
GROUP BY product_id
ORDER BY product_id;
```

- Klistra in i fliken [Stamkunder](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=0#gid=0) i kolumn AV, AZ, BD, BH

##### 4. Flyttade dagar

- Skriv in rätt datum och kör följande query för flyttade dagar, första är för kategori och andra är för poäng.

```
/* Flyttade dagar per storlek och interval kategori-STL */
/* OBS intervall och produkt är från prenumerationen vid tillfället för frågan, inte ändringen */
SELECT YEAR(h.updated_at) AS YEAR, MONTH(h.updated_at) AS MONTH, p.product_id, p.interval, round(sum(h.delivery_change_day)) AS total_days_moved
FROM endlesssubscription_history h
JOIN endlesssubscription_subscription_products p ON p.parent_id = h.subscription_id AND p.`product_id`=17
WHERE h.updated_at > "2023-06-01"
AND h.`delivery_change_day` IS NOT NULL
AND h.history NOT LIKE "%Change the status%"
GROUP BY YEAR(h.updated_at), MONTH(h.updated_at), p.product_id, p.interval
ORDER BY YEAR(h.updated_at), MONTH(h.updated_at), p.product_id, p.interval;


/* Flyttade dagar per storlek och interval poäng-STL */
/* OBS intervall och produkt är från prenumerationen vid tillfället för frågan, inte ändringen */
SELECT YEAR(h.updated_at) AS YEAR, MONTH(h.updated_at) AS MONTH, p.product_id, p.interval, round(sum(h.delivery_change_day)) AS total_days_moved
FROM gs_subscription_history h
JOIN gs_subscription_products p ON p.parent_id = h.subscription_id AND p.`product_id` IN (404,405,406)
WHERE h.updated_at > "2023-06-01"
AND h.`delivery_change_day` IS NOT NULL
AND h.history NOT LIKE "%Change the status%"
GROUP BY YEAR(h.updated_at), MONTH(h.updated_at), p.product_id, p.interval
ORDER BY YEAR(h.updated_at), MONTH(h.updated_at), p.product_id, p.interval;
```

- Klistra in resultatet i flikarna [Flyttade dagar Kategori](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=1162912392#gid=1162912392) och [Flyttade dagar Poäng](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=652096899#gid=652096899)

##### 5. Aktiverade kategori-STL:er

- Kör följande query

```
/* Aktiveringar av kategoristl */
/* ta ut startade poängstl:er som konverterat från kategori
hämta det subscription:idt från kategoris historiktabell
kolla på dens näst sista historikinlägg om det var att den bytte status till 0
 */

select DATE_FORMAT(changes.created_at, "%Y-%m-01") AS datum, year(changes.created_at) as year, month(changes.created_at) as month, count(entity_id) as aktiveringar
from (
     select
         gs.entity_id, gs.customer_id, gs.status as gs_status, gs.created_at,
         h.updated_at,
         h.status as h_status,
         h.history,
         ROW_NUMBER() OVER (PARTITION BY h.subscription_id ORDER BY h.updated_at DESC) AS change_number
     from gs_subscription gs
     join endlesssubscription_history h on gs.converted = h.subscription_id
     where
         gs.converted > 0 AND
         gs.state = 2 AND
         gs.created_at > "2024-03-01" /* Endast aktivering till poänglåda startar */
) changes
where
    changes.change_number = 2 AND /* Näst sista ändringen */
    changes.h_status = 0 /* Bara de som var pausade innan cancelleringen */
group BY datum, year, month
order by datum;
```

- Klistra in resultatet i [Aktiveringar från pausade kategori-STL](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=605473185#gid=605473185)
- Dra ner raderna i [Stamkunder](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=0#gid=0)
- Dra ner raderna i [Stamkundslådan Omsättning](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=729871671#gid=729871671)
- Dra ner raderna i [Ta din låda-metrics](https://docs.google.com/spreadsheets/d/1VZagqGnJ5WWwV9c2fB4ryoaD4XNgyBj-LitrDlYXDnE/edit?gid=154047510#gid=154047510)

---

## Årsbryt

> Dashboard för året

Görs i dokumentetet [Dashboard år](https://docs.google.com/spreadsheets/d/19P2c9Srx0ldkn8ei1b9dqVwzTCLO5aUNchV7eVSjTsY/edit?gid=1254908849#gid=1254908849)

I Dashboard År följs viktiga KPI:er från Budget/Prognos månad för månad.
När all rapportering för månadsbrytet är klart fyller denna i sig automatiskt.

**KPI:er:** Omsättning, Antal lådor levererade, Snittpris, Stamkunder
