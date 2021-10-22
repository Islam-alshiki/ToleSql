# Extensibility of SQL functions

ToleSql can be extended to support new functions, it has a `simple` system to be able to incorporate new SQL functions and generate SQL according to methods used in expressions.

ToleSql uses an [ExpressionVisitor](https://msdn.microsoft.com/en-us/library/bb882521(v=vs.90).aspx) to loop through expressions passed, when it detects a function call it calls a list of * interceptors * to try to process said call, if it is not intercepted by any interceptor it tries to solve it by evaluating the expression and generating a constant, so if a member of a parameter is used in the call, the process peta.

The following expression used in the call to the `Select` method causes the interceptors to be called:

``` csharp
builder.Select<Invoice>(i => new { 
    Supplier = i.SupplierId, 
    MaxAmount = DbFunctions.Max(i.TotalAmount) 
});
```

And from that, something like:

````sql
SELECT [T0].[SupplierId] as Supplier, MAX([T0].[TotalAmount]) AS MaxAmount From [Invoice]
````


Ultimately it is to translate `DbFunctions.Max (i.TotalAmount)` into `MAX ([T0]. [TotalAmount])`

Here is the interceptor used for the SQL `MAX` function. When ToleSql encounters the call to the `DbFunctions.Max` method, it calls the interceptors until one of them returns other than null.

```` csharp
public class DbFunctionsMax : MethodCallInterceptorBase
{
    public override bool Intercept(MethodCallExpression m, 
        StringBuilder sql, Func<Expression, Expression> visit) // (1) input parameters
    {
        if (m.Method.DeclaringType == typeof(DbFunctions) 
            && m.Method.Name == nameof(DbFunctions.Max)) // (2) method check.
        {
            var identifier = m.Arguments[0]; // (3) Access to identifier
            sql.Append($"{Dialect.Keyword(SqlKeyword.Max)}"); // (4) Writing the literal 'MAX'
            sql.Append($"{Dialect.Symbol(SqlSymbols.StartGroup)}"); // (4a)
            visit(identifier); // (5) Procesado del identificador.
            sql.Append($"{Dialect.Symbol(SqlSymbols.EndGroup)}"); // (4b)
            return true; // (6) We indicate that we have captured the expression.
        }
        // If the check does not pass, we return false to indicate that we have not done anything.
        return false; 
    }
}
````

1. Interceptor input parameters.
* Receive the method call expression. This expression contains the method that is called and the arguments that will be passed to it. From here we can access the type where the method is declared and its name.
* The string builder where the SQL query is being added. When we have to write SQL we will do it with `sql.Append()`.
* A method to process expressions, the arguments used in the method call expression can be constants, parameters, function calls, ...
2. Checking the method.
The first thing we do is check that the method that `DbFunctions.Max()` is being called, for that we check the type where the method `Method.DeclaringType` is declared and the name of the same `Method.Name`, if they match `DbFunctions` and `Max` then we process, otherwise we return false to tell ToleSql to keep looking for interceptors for this expression.
3. Access to the identifier.
We know that our `Max` function only accepts one parameter, so we access it through the expression. Being a `MethodCallExpression` it has a property called `Arguments` which is the list of arguments of the function, in this case we take the first and only one.
4. Writing literals.
Now what we do is write the literal `MAX` in SQL, in this case what is done is request it from the active dialect, we use `Dialect.Keyword (SqlKeyword.Max)` , `Keyword` is a dialect method that given a keyword returns a literal, in this case it will return `MAX` unless we use a dialect for a database where it is not called like that.
Then we write an opening of parentheses `(`, then we will put the identifier and finally (4c) we will close the parentheses. The opening of parentheses is also done through the dialect, in this case we ask for the symbol `StartGroup` or`EndGroup` to close: `Dialect.Symbol (SqlSymbols.StartGroup)` or `Dialect.Symbol (SqlSymbols.EndGroup)`.
5. Identifier processing.
The identifier can be another complex expression, if we were sure that it is a member of a parameter we could take its name, but since it can be anything we process it with the visitor. In this example the expression is `DbFunctions.Max(i.TotalAmount)`, but it could be `DbFunctions.Max(i.Count * i.Price * 0.20)`, as ToleSql already knows how to process expressions better we invoke it with `visit(identifier)`.

In [/ToleSql/Expressions/Visitors/Interceptors](../../src/ToleSql/Expressions/Visitors/Interceptors) we can see the list of available interceptors.

Once we have built our interceptor we must add it to the ToleSql configuration so that it takes it into account:

```` csharp
Configuration.RegisterInterceptor(new DbFunctionsMax());
````

Currently the interceptors are added in the static constructor of the class [`Configuration`](../../src/ToleSql/Configuration.cs).
