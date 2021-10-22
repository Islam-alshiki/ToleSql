# SQL special functions

* `string.Contains(parameter)` - `LIKE %parameter%`  
To search by content: `builder.Where(p => p.Name.Contains("Javi"));` Or otherwise `builder.Where(p => "Javier Ros".Contains(p.Name));`
* `string.StartsWith(parameter)` - `LIKE parameter%`  
To find text that begins with something: `builder.Where(p => p.Name.StartsWith("Javi"));` Or otherwise `builder.Where(p => "Javier Ros".StartsWith(p.Name));`
* `string.EndsWith(parameter)` - `LIKE %parameter`  
To find text that ends with another text: `builder.Where(p => p.Name.EndsWith("Javi"));` Or otherwise `builder.Where(p => "Javier Ros".EndsWith(p.Name));`
* `string.SubString(from, count)` - `SUBSTRING(parameter, from, count)`  
To extract a substring of length `count` from the index `from`.
`builder.Select(p => pName.SubString(0, 3));`
* `DbFunctions.Sum(param)` - `SUM(param)`  
To show sums in the groupings: `builder.Select(p => DbFunctions.Sum(p.Amount))`
* `DbFunctions.Sum(param)` - `COUNT(param)`
* `DbFunctions.Max(param)` - `MAX(param)`
* `DbFunctions.Min(param)` - `MIN(param)`


To support new functions see the section [Extensibilidad](./extensibility-es.md)