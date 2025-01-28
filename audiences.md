# Audiences till Google & Meta

> Uppdateras en gång i månaden för riktning av annonser

## Ta ut nya listor

Kan tas fram ur Dot eller databasen, nedan beskrvning gäller databasen.

Görs i dokumentet [Målgrupper](https://docs.google.com/spreadsheets/d/1PxqcM2oQ6hhuroHQq60xwbsHY0NHnImj9z4nNFxISao/edit?gid=502918183#gid=502918183)

##### 1. Stamkunder status "aktiv"

Får annonser som “Ta din låda” och varumärkesannonser.

- Kör följande query

```
SELECT *
  FROM (
    SELECT c.email
    FROM customer_entity c
    JOIN endlesssubscription_subscription s
    ON c.entity_id = s.customer_id
    WHERE s.state = 2 AND s.status = 1

    UNION

    SELECT c.email
    FROM customer_entity c
    JOIN gs_subscription s
    ON c.entity_id = s.customer_id
    WHERE s.state = 2 AND s.status = 1
  ) a
  GROUP BY email;
```

- Klistra in in resultetet i fliken [STL Aktiv](https://docs.google.com/spreadsheets/d/1PxqcM2oQ6hhuroHQq60xwbsHY0NHnImj9z4nNFxISao/edit?gid=502918183#gid=502918183)
- Uppdatera datumet.

##### 2. Stamkunder status "pausad"

Stamkunder med pausad status som tagit en leverans de senaste 2 åren. Får annonser som "Ta din låda", "Vi saknar dig", högtidslådor, varumärke.

- För följande query

```
SELECT a.email
  FROM (
    SELECT c.email, s.customer_id
    FROM customer_entity c
    JOIN endlesssubscription_subscription s
    ON c.entity_id = s.customer_id
    WHERE s.state = 2 AND s.status = 0

    UNION

    SELECT c.email, c.entity_id
    FROM customer_entity c
    JOIN gs_subscription s
    ON c.entity_id = s.customer_id
    WHERE s.state = 2 AND s.status = 0
  ) a
  WHERE a.customer_id IN (
    SELECT DISTINCT o.customer_id FROM sales_order o
    JOIN sales_order_item i ON o.entity_id = i.order_id AND i.sku LIKE "stamkundsladan%"
    WHERE
      o.status IN ("complete", "pending", "processing") AND
      o.delivery_date > SUBDATE(curdate(), INTERVAL 24 MONTH) AND
      o.subtotal > 0 AND
      o.customer_id > 0
  ) AND
  a.email NOT IN (
    /* Aktiv status */
    SELECT *
    FROM (
      SELECT c.email
      FROM customer_entity c
      JOIN endlesssubscription_subscription s
      ON c.entity_id = s.customer_id
      WHERE s.state = 2 AND s.status = 1

      UNION

      SELECT c.email
      FROM customer_entity c
      JOIN gs_subscription s
      ON c.entity_id = s.customer_id
      WHERE s.state = 2 AND s.status = 1
    ) a
    GROUP BY email
  );
```

- Klistra in i fliken [STL pausad m leverans <2år](https://docs.google.com/spreadsheets/d/1PxqcM2oQ6hhuroHQq60xwbsHY0NHnImj9z4nNFxISao/edit?gid=414391793#gid=414391793)
- Uppdatera datumet
- Kontakter i den här kateogrin ska inte finnas med bland "STL aktiv". Formeln i kolumn C hjälper dig kolla efter dubletter. Filtrera för att se om det finns några, radera dom i så fall.

##### 3. Övrig låda-kunder

Kunder som köpt en engångslåda/övrig låda inom 2 år. Får annonser som “Köp STL”, övriga lådor, högtidslådor, varumärke.

- Kör följande query

```
SELECT c.email
FROM customer_entity c
JOIN sales_order o ON c.entity_id = o.customer_id AND o.delivery_date >= SUBDATE(curdate(), INTERVAL 24 MONTH)
JOIN sales_order_item i ON o.entity_id = i.order_id
WHERE o.status IN ("complete", "pending", "processing")
	AND i.sku LIKE "%lada%"
    AND i.sku NOT LIKE "stamkundslada%"
	AND i.price > 0
	AND o.subtotal > 0
	AND c.email NOT IN (
    /* Aktiv s  tatus */
    SELECT *
    FROM (
      SELECT c.email
      FROM customer_entity c
      JOIN endlesssubscription_subscription s
      ON c.entity_id = s.customer_id
      WHERE s.state = 2 AND s.status = 1

      UNION

      SELECT c.email
      FROM customer_entity c
      JOIN gs_subscription s
      ON c.entity_id = s.customer_id
      WHERE s.state = 2 AND s.status = 1
    ) a
    GROUP BY email
	)
	AND c.email NOT IN (
	  /* Pausad status */
    SELECT *
    FROM (
      SELECT c.email
      FROM customer_entity c
      JOIN endlesssubscription_subscription s
      ON c.entity_id = s.customer_id
      WHERE s.state = 2 AND s.status = 0

      UNION

      SELECT c.email
      FROM customer_entity c
      JOIN gs_subscription s
      ON c.entity_id = s.customer_id
      WHERE s.state = 2 AND s.status = 0
    ) a
    WHERE email NOT IN (
      /* Aktiv status */
      SELECT *
      FROM (
        SELECT c.email
        FROM customer_entity c
        JOIN endlesssubscription_subscription s
        ON c.entity_id = s.customer_id
        WHERE s.state = 2 AND s.status = 1

        UNION

        SELECT c.email
        FROM customer_entity c
        JOIN gs_subscription s
        ON c.entity_id = s.customer_id
        WHERE s.state = 2 AND s.status = 1
      ) a
      GROUP BY email
    )
	)
GROUP BY c.email;
```

- Klistra in i fliken [ÖVL 24 mån](https://docs.google.com/spreadsheets/d/1PxqcM2oQ6hhuroHQq60xwbsHY0NHnImj9z4nNFxISao/edit?gid=816248353#gid=816248353)
- Uppdatera datumet
- Kontakter i den här kateogrin ska inte finnas med bland "STL aktiv" eller "STL Pausad". Formeln i kolumn C hjälper dig kolla efter dubletter. Filtrera för att se om det finns några, radera dom i så fall.

##### 4. Kontaker

De sin prenumerar på vårt nyhetsbrev men inte handlat de senaste två åren. Får alla annonser utom de riktade mot stamkunder.

- Kör följande query

```
SELECT n.subscriber_email
FROM newsletter_subscriber n
WHERE n.created_at >= SUBDATE(curdate(), INTERVAL 2 YEAR)
AND n.subscriber_email NOT IN (
	/* Nånsin varit STL-kund */
    SELECT *
    FROM (
      SELECT c.email
      FROM customer_entity c
      JOIN endlesssubscription_subscription s
      ON c.entity_id = s.customer_id
      WHERE s.state = 2

      UNION

      SELECT c.email
      FROM customer_entity c
      JOIN gs_subscription s
      ON c.entity_id = s.customer_id
      WHERE s.state = 2
    ) a
    GROUP BY email
)
AND n.subscriber_email NOT IN (
  /* Order senaste X månaderna */
  SELECT c.email
  FROM customer_entity c
  JOIN sales_order o
  ON c.entity_id = o.customer_id AND o.delivery_date >= SUBDATE(curdate(), INTERVAL 24 MONTH)
  WHERE o.status IN ("complete", "pending", "processing")
	AND o.subtotal > 0
);
```

- Klistra in i fliken [Kontakter 2 år](https://docs.google.com/spreadsheets/d/1PxqcM2oQ6hhuroHQq60xwbsHY0NHnImj9z4nNFxISao/edit?gid=798121104#gid=798121104)
- Uppdatera datumet
- Kontakter i den här kateogrin ska inte finnas med bland "STL aktiv", "STL Pausad" eller "ÖVL". Formeln i kolumn C hjälper dig kolla efter dubletter. Filtrera för att se om det finns några, radera dom i så fall.

## Uppdatera Facebook Ads

[Officiell dokumentation](https://www.facebook.com/business/help/170456843145568?id=2469097953376494)

##### Uppdatera befintlig lista

- Gå till [Ads Manager](https://adsmanager.facebook.com/adsmanager/audiences?act=322569095189157&business_id=2131894573773722&tool=AUDIENCES&nav_source=business_manager&breakdown_regrouping=1&date=2023-01-01_2023-02-01).
- Du loggar in med ditt privata faceboook-konto (konstigt, jag vet), och google authenticator.
- Hitta de fyra segment du nyss tagit ut listor till, det är lättare om du filtrerar på Type: Custom audience och Source: Customer list.
- Klicka på segmentet och edit.
- Klicka på “Replace the list with new customers”.
- Ladda upp csv-fil.
- Under "Does your list include a column for customer value?" Fyll i No.
- Klicka på Next.
- Kolla kolumnmatchningen, den brukar förstå att epost är epost. Klicka på Import and create. OK att inte alla kolumner är med, enbart email ska med.
- Klicka på Done.
- Fortsätt med de andra segmenten.

##### Skapa ny lista

- Följ ovan steg men istället för att uppdatera befintliga segment klickar du på Create audience > Custom audience > Customer list > Next.
- Under "How to prepare your customer list", klicka bara på Next om din lista är klar, annars läs igenom Metas dokumentation.

## Uppdatera Google Ads

##### Uppdatera befintlig lista

- Gå till [Google Ads](https://ads.google.com/aw/overview?ocid=86743980&euid=1283354309&__u=5290053341&uscid=86743980&__c=5319539020&authuser=1&workspaceId=0)
- Gå till Tools > Shared library > Audience manager.
- Hitta de fyra segment du nyss tagit ut listor till. Hovra över namnet och vänta på popup, välj edit.
- Klicka på List members > Modify list.
- Klicka på Data source > Upload file manually.
- Klicka på List members > Edit your list based on customer contact information > Replace existing list members with a new customer list.
- Ladda upp csv-fil genom att klicka på Browse.
- Godkännn villkoren.
- Klicka på Save and continue.
- Får du upp ett felmeddalande om "unrecognized column headers" så klickar du på "Fix and upload".

##### Skapa ny lista

- Gå till [Google Ads](https://ads.google.com/aw/overview?ocid=86743980&euid=1283354309&__u=5290053341&uscid=86743980&__c=5319539020&authuser=1&workspaceId=0)
- Gå till Tools > Shared library > Audience manager.
- Klicka på plusset välj Customer list .
- Klicka på upload file manually.
- Segment name: Välj något kort och beskrivande.
- Data type: Upload Emails, Phones, and/or Mailing Addresses.
- Data to upload: ladda upp din csv-fil.
- Godkänn villkoren och spara.
