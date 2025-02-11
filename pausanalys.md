# Pausanalys

## Urval

Alla stamkunder (pausad som aktiv), som har pausat någon gång under 2023-2024. OBS, vi kan bara se kundens senaste pausdatum. Det betyder att kunder som pausar/aktiverar flera gånger bara finns med under sitt senaste pausdatum, de kunder vi ser från t.ex. 2023 är därför bara de kunder som pausade då och inte har pausat sedan dess.

## Tillvägagångssätt

1. Exportera pausenkäten från Dot > Content > Surveys, pages and forms > Pausad STL- enkät v3 > Form report > Export to csv

2. Kör följande query mot databasen för att ta fram rådata

```
SELECT
	customer_id,
    created_at,
    `status`,
    email,
    paused_at,
    last_delivery_date,
    next_delivery_date,
    product_id, -- Determines Stamkundslåda type (17, 404, 405, 406)
    `interval`, -- Escaped to avoid conflict with MySQL reserved keyword
    delivery_count,
    source
FROM (
    -- Endless Subscription: Include only inactive subscriptions
    SELECT
    	s.customer_id,
        s.created_at,
        s.status,
        c.email,
        s.paused_at,
        sp.last_delivery_date,
        sp.next_delivery_date,
        sp.product_id,
        sp.delivery_count,
        sp.`interval`, -- Escaped delivery interval
        'kategori' AS source
    FROM customer_entity c
    JOIN endlesssubscription_subscription s
        ON c.entity_id = s.customer_id
    JOIN endlesssubscription_subscription_products sp
        ON s.entity_id = sp.parent_id
    WHERE s.state = 2 -- Valid order
      AND s.status IN (0, 1) -- Inactive
      AND s.paused_at >= "2023-01-01 00:00:00"
      -- Uncomment to add date range
      -- AND s.paused_at <= "2024-12-30 23:59:59"

    UNION ALL

    -- GS Subscription: Include only inactive subscriptions
    SELECT
    	customer_id,
        s.created_at,
        s.status,
        c.email,
        s.paused_at,
        sp.last_delivery_date,
        sp.next_delivery_date,
        sp.product_id,
        sp.delivery_count,
        sp.`interval`, -- Escaped delivery interval
        'poäng' AS source
    FROM customer_entity c
    JOIN gs_subscription s
        ON c.entity_id = s.customer_id
    JOIN gs_subscription_products sp
        ON s.entity_id = sp.parent_id
    WHERE s.state = 2 -- Valid order
      AND s.status IN (0, 1) -- 0 = pausad, 1 = aktiv
      AND s.paused_at >= "2023-01-01 00:00:00"
      AND s.paused_at < "2025-01-01 23:59:59"
) a
WHERE product_id IN (17, 404, 405, 406);
```

3. Städa upp data allt eftersom behov uppstår. T.ex. skriv om status från siffror till ord, dela upp datum så att du kan använda det i formler. Skriv om intervall från dagar till veckor. Ta fram customer_id_occurences för att se hur många gånger varje kund är med i listan

`=COUNTIF(B$2:B, B2)`

4. Om du behöver kan du också ta ut historiken för att customer_id'n du tidigare hämtat ut. Här är ett exempel när vi hämtar historik för poängkunder, här behöver du se till så att du använder samma avgränsningar som i ovan query.

```
SELECT h.*
FROM gs_subscription_history h
JOIN (
    SELECT DISTINCT s.customer_id
    FROM gs_subscription s
    WHERE s.state = 2
      AND s.status IN (0, 1) -- Inactive subscriptions
      AND s.paused_at >= "2024-01-01 00:00:00"
      AND s.paused_at < "2025-01-01 00:00:00"
) f ON h.customer_id = f.customer_id
WHERE h.updated_at >= "2024-01-01"
	AND h.updated_at < "2025-01-01"
	AND h.history LIKE "%Change the status%";
```
