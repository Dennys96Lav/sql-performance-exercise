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
















