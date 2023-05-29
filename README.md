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





















