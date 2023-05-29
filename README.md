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





































