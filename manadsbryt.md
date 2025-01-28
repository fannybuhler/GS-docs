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

- Radera förra månadens prognossiffror.

##### 3. Uppdatera fliken &nbsp; [Stamkunder](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=258667635#gid=258667635)

- Kolla att sista dagen i månaden blir rätt

##### 4. Uppdatera fliken &nbsp; [Införsäljning](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=1823519833#gid=1823519833)

- Radera förra månadens inskrivna försäljning **STL Antal sålda (F)** och **Antal ÖVL (K)**.

##### 5. Gör &nbsp; [Månadsrapporteringen](manadsrapportering.md)

Fördelningen över månaden baseras på veckodagar enligt fliken [Leveransdagar fördelning](https://docs.google.com/spreadsheets/d/1pqKRh94o5V-bLYeRvWH9RxJO0B7omkVC2wWbL_vbZUs/edit?gid=975771408#gid=975771408).
Uppdateras förslagsvis några gånger per år eller om det varit stora ändringar.

## Månadsbryt i GS_utfall

> Rapportera sålda och levererade produkter efter månadsslut

Görs i dokumentet [GS_utfall sålda och levererade produkter](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=627471941#gid=627471941)

##### 1. Uppdatera fliken &nbsp; [STL Levererade](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=16986693#gid=16986693)

- Öppna Table Plus och kör först datumet, sedan resten av queryn.
- Klista in längst ner i fliken.

  _Vi kommer inom en snar framtid att hämta den här datan från Product Sales istället_

```
/* Utfall - Levererade STL-lådor */
SET
	@from_date = "2024-12-01";

SET
	@to_date = "2024-12-31";

SELECT
	boxes.datum,
	boxes.year,
	boxes.month,
	DATE_FORMAT(CONCAT(boxes.year - 1, '-', boxes.month, '-01'), '%Y-%m-%d') AS LY,
	boxes.product_id,
	boxes.name,
	boxes.sku,
	boxes.stl_interval,
	boxes.qty_delivered,
	new_s.new_customers AS new_stl_customers,
	"newnew" AS first_time_customers,
	if(new_s.new_customers > 0, round(boxes.qty_delivered - new_s.new_customers), boxes.qty_delivered) AS returning_customers,
	round(boxes.revenue) AS revenue,
	new_s.new_order_revenue AS revenue_new_stl_customers,
	if(new_s.new_customers > 0, round(boxes.revenue - new_s.new_order_revenue), round(boxes.revenue)) AS revenue_returning_stl_customers,
	boxes.box_revenue,
	boxes.aov,
	boxes.box_price
FROM
	(
		SELECT
			DATE_FORMAT(o.delivery_date, "%Y-%m-01") AS datum,
			year(o.delivery_date) AS YEAR,
			MONTH(o.delivery_date) AS MONTH,
			oi.product_id,
			oi.name,
			oi.sku,
			if(o.`interval` > 0, o.`interval`, 0) AS stl_interval,
			round(sum(oi.qty_ordered)) AS qty_delivered,
			sum(oi.qty_ordered) / count(o.entity_id) AS qty_per_order,
			sum(o.base_subtotal) AS revenue,
			round(sum(oi.qty_ordered * oi.price)) AS box_revenue,
			round(sum(o.base_subtotal) / sum(oi.qty_ordered)) AS aov,
			round(AVG(oi.price)) AS box_price
		FROM
			sales_order o
			JOIN sales_order_item oi ON o.entity_id = oi.order_id
		WHERE
			oi.sku LIKE "stamkundsladan%"
			AND oi.product_id NOT IN(315, 318, 248) /*nekade kortbetalningar */
			AND o.status IN ("complete", "processing", "pending")
			AND oi.price > 0
			AND o.`delivery_date` >= @from_date
			AND o.`delivery_date` < @to_date
		GROUP BY
			datum,
			YEAR,
			MONTH,
			oi.product_id,
			oi.sku,
			oi.name,
			stl_interval
	) boxes
	LEFT JOIN
	/* Antal kunder som fick sin första STL */
	(
		SELECT
			DATE_FORMAT(o.delivery_date, "%Y-%m-01") AS datum,
			YEAR(o.delivery_date) AS YEAR,
			MONTH(o.delivery_date) AS MONTH,
			i.product_id,
			i.name,
			i.sku,
			if(o.`interval` > 0, o.`interval`, 0) AS stl_interval,
			round(sum(i.qty_ordered)) AS new_customers,
			round(sum(o.subtotal)) AS new_order_revenue
		FROM
			sales_order o
			JOIN sales_order_item i ON o.entity_id = i.order_id
			AND i.sku LIKE "stamkundsladan%"
			AND o.entity_id IN (
				/* Inkludera endast kunders första levererade stl */
				SELECT
					min(o.entity_id)
				FROM
					sales_order o
					JOIN sales_order_item i ON o.entity_id = i.order_id
					AND o.status IN ("complete", "pending", "processing")
					AND o.`subtotal` > 0
					AND i.sku LIKE "stamkundsladan%"
					AND o.customer_id > 0
					AND o.delivery_date > 0
				GROUP BY
					o.customer_id
			)
		GROUP BY
			datum,
			YEAR,
			MONTH,
			i.product_id,
			i.name,
			i.sku,
			stl_interval
	) new_s ON boxes.datum = new_s.datum
	AND boxes.product_id = new_s.product_id
	AND boxes.name = new_s.name
	AND boxes.sku = new_s.sku
	AND boxes.stl_interval = new_s.stl_interval
ORDER BY
	boxes.datum,
	boxes.product_id,
	boxes.stl_interval;
```

##### 2. Uppdatera flikarna Levererade ÖVL & tillval samt Sålda ÖVL & Tillval

- Använd M2-rapporten Products Sales under M2 admin > GS Reports.
- Ställ in datum för månaden som ska rapporteras, sätt exporten till Delivery date och Månad (period type), lämna SKU tomt. Exportera som csv.
- Skapa en **ny flik** i LIVE och importera csv-filen, döp till **"levererat"**.
- Gör samma export men ändra inställning till “Order date” klistra in i **ny flik** i dokumentet, döp till **"sålt"**. Dessa kommer att användas till både ÖVL och Tillval.
- I både "levererat" och "sålt":

  - Radera sista fyra kolumnerna: Future Qty, Future Total ex vat, Tot. Qty samt Tot. Revenue ex vat.
  - Skapa filter på samtliga rubriker
  - Filtrera bort ev rader med produkt-id 17, 315, 318, 404, 405, 406 (Avser Stamkundslådan, betalning för tidigare leverans och nekade kortbetalningar).
  - Sätt filter på Sold As Type = “box” i respektive export
  - Kopiera de rader som återstår och klistra in i [Levererade ÖVL-lådor](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=1396535852#gid=1396535852) och [Sålda ÖVL-lådor](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=1241022428#gid=1241022428)
  - Dra ner de två sista kolumnerna för Marginal % och Marginalkronor då att de beräknas för de nya raderna.

- Byt filtret i "levererat" och "sålt" till Sold as Type = Addon.
- Kopiera de rader som återstår och klistra in i [Levererade Tillval](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=581555733#gid=581555733) och [Sålda Tillval](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=2142241328#gid=2142241328).
- Radera flikana "levererat" och "sålt" för att hålla dokumentet städat.

- I fliken [Levererat Översikt per månad](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=627471941#gid=627471941): Dra ner alla formler så att beräkningarna körs för senaste månaden.
- I fliken [Sålt Översikt per månad](https://docs.google.com/spreadsheets/d/1KwBj8qcOVttA0GvzNLp2zrXQ-8NTeE31_c4XP4en3Q8/edit?gid=1192122720#gid=1192122720): Dra ner alla formler så att beräkningarna körs för senaste månaden.

## Månadsbryt i GS_stamkunder

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

##### 3. Flyttade dagar

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

##### 4. Aktiverade kategori-STL:er

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

## Årsbryt

> Dashboard för året

Görs i dokumentetet [Dashboard år](https://docs.google.com/spreadsheets/d/19P2c9Srx0ldkn8ei1b9dqVwzTCLO5aUNchV7eVSjTsY/edit?gid=1254908849#gid=1254908849)

I Dashboard År följs viktiga KPI:er från Budget/Prognos månad för månad.
När all rapportering för månadsbrytet är klart fyller denna i sig automatiskt.

**KPI:er:** Omsättning, Antal lådor levererade, Snittpris, Stamkunder
