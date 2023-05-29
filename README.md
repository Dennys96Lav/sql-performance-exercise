# sql-performance-exercise


https://learn.microsoft.com/it-it/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms


Download and import AdventureWorksLT2014.bak


Esercizio 1 : OTTIMIZZARE LA SEGUENTE QUERY :


```
SELECT * FROM Sales.SalesOrderHeader
WHERE OrderDate BETWEEN '2019-01-01' AND '2019-12-31'
```

Svolgere sensitity e valutare se conviene creare un index.

```
-- Aggiungi un indice sulla colonna OrderDate
CREATE INDEX IX_SalesOrderHeader_OrderDate ON Sales.SalesOrderHeader (OrderDate)

-- Svolgi la query utilizzando l'indice creato
SELECT * FROM Sales.SalesOrderHeader WITH (INDEX(IX_SalesOrderHeader_OrderDate))
WHERE OrderDate BETWEEN '2019-01-01' AND '2019-12-31'
```

__________________________________________________________________________________


Esercizio 2 : OTTIMIZZARE LA SEGUENTE QUERY.


Ottimizzazione delle join.
Ottimizza la seguente query che coinvolge una join:


```
SELECT ProductID, Name, StandardCost, ListPrice
FROM Production.Product
INNER JOIN Sales.SalesOrderDetail ON Production.Product.ProductID = Sales.SalesOrderDetail.ProductID
WHERE Sales.SalesOrderDetail.UnitPrice > 100

```


Soluzione :


```

-- Aggiungi un indice sulla colonna UnitPrice della tabella SalesOrderDetail
CREATE INDEX IX_SalesOrderDetail_UnitPrice ON Sales.SalesOrderDetail (UnitPrice)

-- Svolgi la query utilizzando gli indici creati
SELECT ProductID, Name, StandardCost, ListPrice
FROM Production.Product
INNER JOIN Sales.SalesOrderDetail WITH (INDEX(IX_SalesOrderDetail_UnitPrice))
    ON Production.Product.ProductID = Sales.SalesOrderDetail.ProductID
WHERE Sales.SalesOrderDetail.UnitPrice > 100
```

_____________________________________________________________________________________


Esercizio 3 :

```
SELECT p.Name, sod.OrderQty, sod.UnitPrice, sod.LineTotal
FROM Production.Product p
INNER JOIN Sales.SalesOrderDetail sod ON p.ProductID = sod.ProductID
WHERE sod.OrderQty > 10
```


Soluzione:
```
-- Aggiungi un indice sulle colonne ProductID e OrderQty della tabella SalesOrderDetail
CREATE INDEX IX_SalesOrderDetail_ProductID_OrderQty ON Sales.SalesOrderDetail (ProductID, OrderQty)

-- Svolgi la query utilizzando l'indice creato
SELECT p.Name, sod.OrderQty, sod.UnitPrice, sod.LineTotal
FROM Production.Product p
INNER JOIN Sales.SalesOrderDetail sod WITH (INDEX(IX_SalesOrderDetail_ProductID_OrderQty))
    ON p.ProductID = sod.ProductID
WHERE sod.OrderQty > 10
```

_________________________


Esercizio 4 :

```
SELECT p.ProductID, p.Name
FROM Production.Product p
WHERE p.ProductID IN (SELECT ProductID FROM Sales.SalesOrderDetail)
```

Utilizzo di istruzioni EXISTS
Ottimizza la seguente query utilizzando l'istruzione EXISTS invece di IN:

```
SELECT p.ProductID, p.Name
FROM Production.Product p
EXISTS (SELECT 1 FROM Sales.SalesOrderDetail sod WHERE sod.ProductID = p.ProductID)
```

_____________________________________



ESERCIZIO 5 :

Si vuole ottimizzare la query :


```
SELECT ProductID, Name, StandardCost, ListPrice
FROM Production.Product
WHERE Color = 'Red'
```

In questo caso si usa l'indice.

Supponiamo di avere Red , blue , yellow , black


```
-- Aggiungi un indice filtrato sulla colonna Color della tabella Product
CREATE INDEX IX_Product_Color ON Production.Product (Color) WHERE Color = 'Red'

-- Svolgi la query utilizzando l'indice filtrato
SELECT ProductID, Name, StandardCost, ListPrice
FROM Production.Product WITH (INDEX(IX_Product_Color))
WHERE Color = 'Red'
```

_________________________

ESERCIZIO 6 :
Utilizzo di una stored procedure
Ottimizza la seguente query utilizzando una stored procedure per evitare il parsing del codice SQL ripetutamente:


```

DECLARE @StartDate DATE = '2022-01-01'
DECLARE @EndDate DATE = '2022-12-31'

SELECT ProductID, Name, StandardCost, ListPrice
FROM Production.Product
WHERE SellStartDate BETWEEN @StartDate AND @EndDate

```

Soluzione:
1. Crea una stored procedure:
```
CREATE PROCEDURE GetProductsBySellStartDate
    @StartDate DATE,
    @EndDate DATE
AS
BEGIN
    SELECT ProductID, Name, StandardCost, ListPrice
    FROM Production.Product
    WHERE SellStartDate BETWEEN @StartDate AND @EndDate
END
```

2. Esegui la stored procedure utilizzando i parametri desiderati:

```
EXEC GetProductsBySellStartDate @StartDate = '2022-01-01', @EndDate = '2022-12-31'
```

_________________________________________________________________


Esercizio 7 :


Utilizzo di indici inclusi
Ottimizza la seguente query utilizzando indici inclusi per coprire le colonne richieste dalla query .


```

SELECT p.ProductID, p.Name, sod.OrderQty, sod.UnitPrice, sod.LineTotal
FROM Production.Product p
INNER JOIN Sales.SalesOrderDetail sod ON p.ProductID = sod.ProductID


```

Soluzione utilizzando **include** :

```
-- Aggiungi un indice incluso sulla colonna ProductID della tabella SalesOrderDetail
CREATE INDEX IX_SalesOrderDetail_ProductID_IncludedColumns
ON Sales.SalesOrderDetail (ProductID)
INCLUDE (OrderQty, UnitPrice, LineTotal)

-- Svolgi la query utilizzando l'indice incluso
SELECT p.ProductID, p.Name, sod.OrderQty, sod.UnitPrice, sod.LineTotal
FROM Production.Product p
INNER JOIN Sales.SalesOrderDetail sod WITH (INDEX(IX_SalesOrderDetail_ProductID_IncludedColumns))
    ON p.ProductID = sod.ProductID
```

___________________________________________


Esercizio 8 : 

Ottimizza la seguente query utilizzando un indice filtrato con una clausola LIKE per ridurre il numero di righe coinvolte:


```

SELECT ProductID, Name
FROM Production.Product
WHERE Name LIKE '%Bike%'

```

```

ALTER TABLE SalesLT.Product ADD BikeType bit default 0

UPDATE SalesLT.Product
SET BikeType=1
WHERE Name LIKE '%Bike%'

SELECT ProductID, Name
FROM SalesLT.Product
where BikeType=1

CREATE NONCLUSTERED INDEX IX_Product_Name ON  SalesLT.Product (name) where BikeType=1


```

________________________



Esecizio 9 :



Utilizzo di istruzioni EXISTS per query correlate complesse
Ottimizza la seguente query utilizzando istruzioni EXISTS per semplificare una query correlata complessa:


```
SELECT p.ProductID, p.Name
FROM Production.Product p
WHERE p.ProductID IN (
    SELECT sod.ProductID
    FROM Sales.SalesOrderDetail sod
    INNER JOIN Sales.SalesOrderHeader soh ON sod.SalesOrderID = soh.SalesOrderID
    WHERE soh.OrderDate >= '2022-01-01'
)
```

Soluzione :


```
SELECT p.ProductID, p.Name
FROM Production.Product p
WHERE EXISTS (
    SELECT 1
    FROM Sales.SalesOrderDetail sod
    INNER JOIN Sales.SalesOrderHeader soh ON sod.SalesOrderID = soh.SalesOrderID
    WHERE sod.ProductID = p.ProductID
    AND soh.OrderDate >= '2022-01-01'
)

```

_____________________________________________________


Esecizio 10 :

Ottimizza la seguente query utilizzando un'istruzione CASE efficiente per evitare duplicati di codice:

```
SELECT
    CASE
        WHEN Color = 'Red' THEN 'Rosso'
        WHEN Color = 'Blue' THEN 'Blu'
        WHEN Color = 'Green' THEN 'Verde'
        ELSE 'Altro'
    END AS ColorTranslated
FROM Production.Product
```

Soluzione :


```
SELECT
    CASE Color
        WHEN 'Red' THEN 'Rosso'
        WHEN 'Blue' THEN 'Blu'
        WHEN 'Green' THEN 'Verde'
        ELSE 'Altro'
    END AS ColorTranslated
FROM Production.Product
```

________________________

Esecizio 11 :

Utilizzo di CTE (Common Table Expression)
Ottimizza la seguente query utilizzando una CTE per migliorare la leggibilità del codice e ottimizzare l'esecuzione:


```
SELECT p.ProductID, p.Name, sod.OrderQty
FROM Production.Product p
INNER JOIN Sales.SalesOrderDetail sod ON p.ProductID = sod.ProductID
WHERE p.ProductID IN (SELECT ProductID FROM Production.ProductCategory WHERE ParentProductCategoryID = 1)


```

SINTASSI CTE :

CTE -> definire temporameamente una tabella virtuale all'interno di una query

```
WITH nome_cte (colonna1, colonna2, ..., colonnaN) AS (
    -- Definizione della CTE
    SELECT ...
    FROM ...
    WHERE ...
)
```

Una volta definita la CTE, può essere utilizzata come una tabella temporanea all'interno della query principale. Ad esempio:

```
SELECT *
FROM nome_cte
WHERE colonna1 = valore;
```



```
-- Svolgi la query utilizzando una CTE
WITH ProductCTE AS (
    SELECT ProductID
    FROM Production.ProductCategory
    WHERE ParentProductCategoryID = 1
)
SELECT p.ProductID, p.Name, sod.OrderQty
FROM Production.Product p
INNER JOIN Sales.SalesOrderDetail sod ON p.ProductID = sod.ProductID
WHERE p.ProductID IN (SELECT ProductID FROM ProductCTE)


```



________________________


Esecizio 12 :


Utilizzo di parametri di query opzionali
Ottimizza la seguente query utilizzando parametri di query opzionali per gestire diversi scenari di filtro:


```

DECLARE @Color NVARCHAR(20) = NULL
DECLARE @Size NVARCHAR(20) = NULL

SELECT ProductID, Name, Color, Size
FROM Production.Product
WHERE (@Color IS NULL OR Color = @Color)
AND (@Size IS NULL OR Size = @Size)

```


Soluzione :


```
DECLARE @Color NVARCHAR(20) = NULL
DECLARE @Size NVARCHAR(20) = NULL

SELECT ProductID, Name, Color, Size
FROM Production.Product
WHERE (Color = @Color OR @Color IS NULL)
AND (Size = @Size OR @Size IS NULL)
OPTION (RECOMPILE) -- Aggiorna il piano di esecuzione in base ai valori dei parametri


```

_______________________



Esecizio 13 :

Utilizzo di operatori di aggregazione
Ottimizza la seguente query utilizzando operatori di aggregazione per calcolare il totale delle vendite per ogni colore dei prodotti:

```

SELECT Color, SUM(ListPrice) AS TotalSales
FROM Production.Product
GROUP BY Color

```

Soluzione :


```
SELECT Color, SUM(ListPrice) AS TotalSales
FROM Production.Product
GROUP BY Color
OPTION (HASH GROUP) -- Specifica l'uso dell'operatore di aggregazione hash

```

Group by : Indica che i risultati devono essere raggruppati in base ai valori della colonna "Color". 
Ciò significa che verranno restituiti i risultati separati per ogni colore presente nella tabella.

Option Hash group : Questa parte dell'istruzione specifica l'uso dell'operatore di aggregazione hash per l'aggregazione dei dati.
L'operatore di aggregazione hash è un algoritmo che viene utilizzato per eseguire l'aggregazione in modo efficiente.

L'operatore di aggregazione hash è un algoritmo utilizzato per eseguire l'aggregazione in modo efficiente. Quando viene applicato a una query che richiede l'aggregazione dei dati, l'operatore di aggregazione hash suddivide i dati in gruppi in base ai valori delle colonne specificate nell'istruzione GROUP BY. Successivamente, esegue l'aggregazione all'interno di ciascun gruppo e restituisce i risultati finali.

La ragione per cui viene utilizzato l'operatore di aggregazione hash è che può gestire grandi quantità di dati in modo efficiente. Sfrutta una tecnica di hashing per distribuire i dati tra le partizioni di memoria, consentendo una parallelizzazione efficace dell'elaborazione. Ciò si traduce in una migliore velocità di esecuzione delle query che coinvolgono l'aggregazione dei dati.


___________________________________________________


Eserzio 14 :

```
SELECT p.Name, sod.OrderQty, sod.UnitPrice, sod.LineTotal
FROM Production.Product p
INNER JOIN Sales.SalesOrderDetail sod ON p.ProductID = sod.ProductID
WHERE p.ProductID BETWEEN 100 AND 200
```


Soluzione :


```
-- Aggiungi un indice columnstore alla tabella SalesOrderDetail
CREATE CLUSTERED COLUMNSTORE INDEX IX_SalesOrderDetail_Columnstore
ON Sales.SalesOrderDetail

-- Svolgi la query utilizzando l'indice columnstore
SELECT p.Name, sod.OrderQty, sod.UnitPrice, sod.LineTotal
FROM Production.Product p
INNER JOIN Sales.SalesOrderDetail sod
    ON p.ProductID = sod.ProductID
WHERE p.ProductID BETWEEN 100 AND 200
```

**QUANDO LO UTILIZZO ?**


Gli indici CLUSTERED COLUMNSTORE possono migliorare significativamente le prestazioni delle query su grandi volumi di dati, consentendo di ottenere risultati più rapidi e di sfruttare al meglio le capacità di compressione dei dati.


Gli indici CLUSTERED COLUMNSTORE sono particolarmente adatti per le tabelle di grandi dimensioni che richiedono analisi e aggregazioni frequenti. Tuttavia, possono non essere adatti per tabelle con aggiornamenti frequenti o per query che richiedono l'accesso ai dati di tutte le colonne.



A differenza degli indici tradizionali, che memorizzano i dati in un formato di riga, gli indici **CLUSTERED COLUMNSTORE** organizzano i dati in un formato di colonna. Ciò significa che i dati di una colonna specifica vengono archiviati in modo contiguo sul disco, consentendo una compressione molto più efficiente e una scansione più rapida delle colonne interessate dalle query.


**FUNZIONAMENTO CLUSTERED COLUMNSTORE :** 


    1- Archiviazione dei dati: i dati vengono suddivisi in segmenti di colonna e compressi per ridurre lo spazio di archiviazione richiesto. La compressione viene eseguita sia tramite la compressione di dizionario che tramite la compressione dei segmenti di colonna.

    2-Elaborazione delle query: quando una query viene eseguita sulla tabella indicizzata, il motore di SQL Server può eseguire una scansione selettiva solo dei segmenti di colonna necessari per soddisfare la query, invece di dover leggere tutte le righe come nei tradizionali indici a riga.

    3-Elaborazione delle aggregazioni: gli indici CLUSTERED COLUMNSTORE sono particolarmente efficienti per le operazioni di aggregazione, come somme, medie o conteggi su grandi set di dati. Poiché i dati di una colonna sono archiviati in modo contiguo, le operazioni di aggregazione possono essere eseguite in modo molto rapido.

    4-Eliminazione di colonne non necessarie: gli indici CLUSTERED COLUMNSTORE consentono di eliminare le colonne non necessarie dalle operazioni di query, riducendo ulteriormente la quantità di dati da elaborare.

    5-Supporto per l'archiviazione columnstore in memoria: a partire da SQL Server 2014, è stato introdotto il supporto per gli indici CLUSTERED COLUMNSTORE in memoria. Questo consente di ottenere prestazioni ancora migliori per le query che coinvolgono tabelle ottimizzate per la memoria.


______________



