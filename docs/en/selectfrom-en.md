# SelectFrom and LinQ

With `SelectFrom` you can build queries starting from a specific type and including other types using the `Join` method. `SelectFrom` also accepts LinQ query syntax.

`SelectFrom` makes internal use of [`SelectBuilder`](./selectbuilder-en.md) so it is good that you know how it works.

### Select from a table

```` csharp
var customers = new SelectFrom<Customer>();
````

Once we have this object we can call its methods to select columns:

```` csharp
customers.Select(c => new { c.Name, c.Balance } );
````

Order:
```` csharp
customers.OrderBy(c => c.Balance).ThenByDescending(c => c.Date);
````

Group:
```` csharp
customers.GroupBy(c => c.ContactCustomerId);
````

Filter:
```` csharp
customers.Where(c => c.Balance > 1000);
````

### Adding types to queries

We can add other tables by doing `Join` with other types:

```` csharp
var query = new SelectFrom<DeliveryNote>();
query
    .Select(dn => new { dn.TotalAmount, dn.SupplierId } );
    .Join<Supplier>((dn, s) => dn.SupplierId == s.Id);
````

Our query object is of type `SelectFrom<TEntity>`, but when we do a `Join` the result is a `SelectFrom<TEntity,TJoinedEntity>`. The second inherits from the first, so once we do the join we can invoke methods `Select`, `OrderBy`, `GroupBy`, ... with two parameters, like this:

```` csharp
var query = new SelectFrom<DeliveryNote>();
query
    .Select(dn => new { dn.TotalAmount, dn.SupplierId } );
    .Join<Supplier>((dn, s) => dn.SupplierId == s.Id)
    .Select((dn, s) => new { Supplier = dn.SupplierId + s.Name });    
````

If we do `Join` again the result is again `SelectFrom<TEntity,TJoinedEntity>` therefore if we do a new `Join` with `User` the result is a `SelectFrom <Supplier, User>` , it is chained the new table with the last one you joined.

```` csharp
var query = new SelectFrom<DeliveryNote>();
query
    .Select(dn => new { dn.TotalAmount, dn.SupplierId } );
    .Join<Supplier>((dn, s) => dn.SupplierId == s.Id)
    .Join<User>((s, u) => s.CreatedBy == u.Id)
    .Select((s, u) => new { 
        UserName = u.Name, 
        SupplierName = s.Name 
    });
````

### Using LinQ syntax

If you like you can use the LinQ syntax, for example:

```` csharp
var b = from d in new SelectFrom<DeliveryNote>()
        select d;
````

Use `where`:
```` csharp
var b = from d in new SelectFrom<DeliveryNote>()
        where d.TotalAmount > 100 && d.Year == "2017"
        select d;
````

Projections with clusters:

```` csharp
var b = from d in new SelectFrom<DeliveryNote>()
        group d by new { d.SupplierId, d.Year } into g
        select new { 
            Supplier = g.SupplierId, 
            Year = g.Year, 
            TotalAmount = DbFunctions.Sum(g.TotalAmount) 
        };
````

Link types with `join`:
```` csharp
var b = from d in new SelectFrom<DeliveryNote>()
        join s in new SelectFrom<Supplier>() on d.SupplierId equals s.Id
        select new { d, s };
````

Multi-key joins are not yet supported.

`SelectFrom` can come in handy for quick queries as it is very convenient to bind method calls (fluently).