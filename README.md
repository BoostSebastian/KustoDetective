# KustoDetective

This is my codes from the website: https://detective.kusto.io/inbox

### Case 1

Igenst the data with the following:

```
.execute database script <|
// Create table for the books
.create-merge table Books(rf_id:string, book_title:string, publish_date:long, author:string, language:string, number_of_pages:long, weight_gram:long)
// Import data for books
// (Used data is utilzing catalogue from https://github.com/internetarchive/openlibrary )
.ingest into table Books ('https://kustodetectiveagency.blob.core.windows.net/digitown-books/books.csv.gz') with (ignoreFirstRecord=true)
// Create table for the shelves
.create-merge table Shelves (shelf:long, rf_ids:dynamic, total_weight:long) 
// Import data for shelves
.ingest into table Shelves ('https://kustodetectiveagency.blob.core.windows.net/digitown-books/shelves.csv.gz') with (ignoreFirstRecord=true)
```

Then start by look up the book.

```
// 0cbc13e0aa7d487e8e797d3de3823161	De Revolutionibus Magnis Data	1613	Gustav Kustov	lat	256	1764
Books | where book_title == "De Revolutionibus Magnis Data"
```

And here is the result how I found the book.

```
Shelves
// multi value expand = split array
| mv-expand rf_ids to typeof(string)
// join book in shelf rf_ids with book rf_id
| join  Books on $left.rf_ids == $right.rf_id
| summarize 
// summarize with sum for all books that are the same
    actual_total_weight   = take_any(total_weight), // take_any ? all are the same
    expected_total_weight = sum(weight_gram)
    by  shelf
//| sort by shelf asc 
| extend weight_diffrence = tolong(actual_total_weight - expected_total_weight)
| sort by weight_diffrence desc 
//$left.rf_ids == $right.rf_id
```