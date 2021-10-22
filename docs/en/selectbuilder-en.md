# SelectBuilder

Using `SelectBuilder` you can build queries indicating the types involved in the call to each method. `SelectBuilder` does not depend on any type, the types are indicated in the method calls.

We create a `SelectBuilder` like this:

```` csharp
var builder = new SelectBuilder();
````

### Adding tables

Once we have the builder we can indicate the type that represents the main table:

```` csharp
builder.From<DeliveryNote>();
````

And we can add a `JOIN` indicating the union expression between the types, for example we can add the supplier table to our delivery note table:
```` csharp
builder.Join<DeliveryNote, Supplier>((dn, s) => dn.SupplierId == s.Id && !s.IsDeleted);
````

We can continue joining tables, for example now we add the delivery note lines:
```` csharp
builder.Join<DeliveryNote, DeliveryNoteDetail>((dn, dnd) => dn.Id == dnd.DeliveryNoteId);
````

At any time we can add one more table with `Join` indicating as the first generic argument, a table that already exists in the query and as the second one we want to add, now you could add, for example, a join with the user who created the delivery note:
```` csharp
builder.Join<DeliveryNote, User>((dn, u) => dn.CreatedBy == u.Id);
````

In exceptional cases we might want to add a table that requires a condition with more than one table, for example joining delivery notes with delivery note lines and products by the creation user, in this case we can do:
```` csharp
builder.Join<DeliveryNote, DeliveryNoteDetail, Product>((t1, t2, t3) 
    => t1.CreatedBy == t3.CreatedBy 
    && t2.CreatedBy == t3.CreatedBy);
````

When we make a `Join`, the last generic argument used is the table that we are going to join and the previous ones are tables that must already be in the query either as main ones or joined with` Join`.

### Selecting columns

We can select columns with the `Select` method, indicating with generics the types of the properties that we are going to use. The types that we indicate as generic arguments must have been added to the query with `From` or` Join`. We could add delivery note fields like this:
```` csharp
builder.Select<DeliveryNote>(dn => dn.Number, dn => dn.TotalAmount);
````

We can also use a projection to name the result columns:
```` csharp
builder.Select<DeliveryNote>(dn => 
    new {
         DeliveryNoteNumber = dn.Number, 
         Total = dn.TotalAmount
    });
````

Or use an existing type:
```` csharp
builder.Select<DeliveryNote>(dn => new DeliveryNoteDto 
    {
        DeliveryNoteNumber = dn.Number, 
        Total = dn.TotalAmount
    });
````

We can also add columns using various types:
```` csharp
builder.Select<DeliveryNote, Supplier>((dn, s) => new  
    {
        DeliveryNoteNumber = dn.Number, 
        Total = dn.TotalAmount,
        SupplierName = s.Name
    });
````

In the columns we can use functions ([Funciones SQL](./functions-en.md)):

```` csharp
builder.Select<Supplier>(s => new { SupplierName == s.Name.SubString(0, 10)});
builder.Select<DeliveryNoteDetail>(s => new { TotalLineas == DbFunctions.Count(s.Id)});
````

> DbFunctions is a static class with database functions, the number of functions can be extended ([Extensibilidad](./extensibility-en.md)).

### Filtering by fields

As with `Select`, with` Where` we must pass as generic arguments the types of the objects that we handle in the expressions, and these types must have been added to the query with `From` or` Join`
```` csharp
builder.Where<Supplier>(s => s.Name.Contains("CocaCola"));
builder.Where<Supplier>(s => s.Name.StartsWith("Coca"));
builder.Where<Supplier>(s => s.Type == "Corporation");
````

Constant parameters such as `"CocaCola"`, `"Coca"` and `"Corporation"` are added to the query as parameters, the dialect in use will mark how these parameters are named, in SqlServer `@SqlParamN` is used where `N` is the parameter number in the order they were entered.

These parameters can also be variables, ToleSql will resolve them and add them as parameters:
```` csharp
var supplierType = "Corporation";
builder.Where<Supplier>(s => s.Type == supplierType);

Assert.Equal(builder.Parameter[0], supplierType);
````

Of course we can also add conditions using different types:
```` csharp
builder.Where<Supplier, DeliveryNote>((s, dn) => 
    s.Type == "Corporation" 
    && dn.IsDeleted 
    && dn.Year == "2015");
````

## Sorting and grouping.

Once you know the dynamics of ToleSql with the `Join`, `Select` and `Where`, it is easy to sort and group:

```` csharp
builder.OrderBy<DeliveryNote>(dn => dn.date);
builder.OrderBy<Supplier>(s => s.Balance);

builder.GroupBy<DeliveryNote>(dn => dn.SupplierId);
builder.Having<DeliveryNote>(dn => 
    DbFunctions.Max(dn.TotalAmount) > 1000);
````

## Adding extreme SQL

It is possible at some point to need to add something that we cannot generate with expressions, either because ToleSql does not support it or because it cannot be implemented easily.

With ToleSql you can add SQL in text at any time, as a column, as a data source (FROM), as JOIN or wheres ...
Suppose we want to put a `CASE WHEN` in a column, we could do this:

```` csharp
builder.From<Invoice>()
  .Where<Invoice>(i => !i.IsDelted)
  .SelectSql(@"case when serie = 'a' then 'Legal invoice'
    case serie = 'b' then 'Invoice in black' 
    end as typeInvoice");
````

And stay so wide.