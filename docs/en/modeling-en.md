# Modeling

### Global modeling

Sometimes the database tables are not called like the classes of our objects, the same with the column names and we can even use different schemes where we classify our tables, for this we have the `Modeling` class.

For this we can do this:

```` csharp
Modeling.Model<Supplier>().SetSchema("WH");
Modeling.Model<DeliveryNote>().SetSchema("WH");
Modeling.Model<DeliveryNoteDetail>().SetSchema("WH");
Modeling.Model<Product>().SetSchema("WH");
Modeling.Model<EquivalentProduct>().SetSchema("WH");
Modeling.Model<DeliveryNoteDetail>().SetColumnName(dnd => dnd.DeliveryNoteId, "DeliveryNote_Id");
Modeling.Model<User>()
    .SetSchema("LoB")
    .SetTable("SecurityProfile")
    .SetColumnName(u => u.CreatedBy, "CreatedBy_Id");
````

This would set the values globally. Whenever the `Supplier` type is resolved, the schema will be `WH`, whenever the `User` type is resolved, the `LoB` schema and the table name `SecurityProfile` will be used.

We can also establish a default schema for all tables that do not have an associated schema:

```` csharp
Modeling.DefaultSchema = "dbo";
````

### Modeled by builder

Alternatively we can set modeling settings at the level of the builder itself.
First, a type is translated into a table using the name of the type and without a schema, the next step is to set the default schema if it has been defined. Then the type is searched in the global `Modeling` and finally it is searched in the `Modeling` of the builder. Therefore the more specific configuration prevails.

Here are some tests that prove it.

Default scheme:
```` csharp
Modeling.DefaultSchema = "KKK";
var b = new SelectBuilder();
b.From<Supplier>();

var gen = b.GetSqlText();
var spec = "SELECT * FROM [KKK].[Supplier] AS [T0]";
Assert.Equal(spec, gen);
````

Default schema + global modeling:
```` csharp
Modeling.DefaultSchema = "KKK";
Modeling.Model<Supplier>().SetSchema("CUS").SetTable("CUSTOMER");
var b = new SelectBuilder();
b.From<Supplier>();

var gen = b.GetSqlText();
var spec = "SELECT * FROM [CUS].[CUSTOMER] AS [T0]";
Assert.Equal(spec, gen);
````

Default schema + global modeling + builder modeling:
```` csharp
Modeling.ResetModeling();
Modeling.DefaultSchema = "KKK";
Modeling.Model<Supplier>().SetSchema("CUS").SetTable("CUSTOMER");

var b = new SelectBuilder();
b.Modeling.Model<Supplier>().SetSchema("dbo").SetTable("MyTable");
b.From<Supplier>();

var gen = b.GetSqlText();
var spec = "SELECT * FROM [dbo].[MyTable] AS [T0]";

Assert.Equal(spec, gen);
````

In short, we have different ways to set the names of our tables to our types.