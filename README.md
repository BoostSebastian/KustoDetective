# KustoDetective
### Case 1

```
// 0cbc13e0aa7d487e8e797d3de3823161	De Revolutionibus Magnis Data	1613	Gustav Kustov	lat	256	1764
Books | where book_title == "De Revolutionibus Magnis Data"
```


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