# C# Stuff

A repository containing cool C# code snippets and concepts.

## Better Memory Management

Some memory management improvements and concepts.

References

- [Memory Anti-Patterns in C#](https://medium.com/criteo-engineering/memory-anti-patterns-in-c-7bb613d55cf0)

### A better IDisposable pattern

The normal IDisposable pattern usually recommended is
unnecessarliy complicated and vague.

Here is a simpler more clear version:.

```csharp
class DisposableMe : IDisposable
{
    private bool _disposed = false;

    ~DisposableMe()
    {
        Cleanup(true);
    }

    public void Dispose()
    {
        Cleanup(false);
    }

    private void Cleanup(bool fromGC)
    {
        if (_disposed) return;

        try
        {
            if (fromGC)
            {
                // Clean unmanaged resources.
                return;
            }

            // Clean up managed resources ONLY if not called from GC.
        }
        finally
        {
            _disposed = true;

            if (!fromGC)
            {
                GC.SuppressFinalize(this);
            }
        }
    }
}
 ```

### Provide list capacity when possible

It is recommended to provide a capacity when creating a List or a collection instance.
The .NET implementation of such classes usually stores the values in an array
that need to be resized when new elements are added: it means that:

- A new array is allocated
- The former values are copied to the new array
- The former array is no more referenced

### Prefer StringBuilder to +/+= for string concatenation

Creating temporary objects will increase the number of garbage collections
and impact performances. Since the string class is immutable,
each time you need to get an updated version of a string of characters,
the .NET framework ends up creating a new string.

For string concatenation, avoid using Concat, **+** or **+=**.
This is especially important in loop or methods called very often.
For example in the following code, a ```StringBuilder``` should be used:

```csharp
var productIds = string.Empty;

while (match.Success)
{
   productIds += match.Groups[2].Value + "\n";
   match = match.NextMatch();
}
```

Again in loops, avoid creating temporary string such as in the following
code where ```SearchValue.ToUpper()``` do not change in the loop:

```csharp
if (SelectedColumn == Resources.Journaux.All && !String.IsNullOrEmpty(SearchValue))
    source = model.DataSource.Where(x => x.ItemId.Contains(SearchValue)
        || x.ItemName.ToUpper().Contains(SearchValue.ToUpper())
        || x.ItemGroupName.ToUpper().Contains(SearchValue.ToUpper())
        || x.CountingGroupName.ToUpper().Contains(SearchValue.ToUpper()));


if (SelectedColumn == Resources.Journaux.ItemNumber)
    source = model.DataSource.Where(x => x.ItemId.ToUpper().Contains(SearchValue.ToUpper()));


if (SelectedColumn == Resources.Journaux.ItemName)
    source = model.DataSource.Where(x => x.ItemName.ToUpper().Contains(SearchValue.ToUpper()));


if (SelectedColumn == Resources.Journaux.ItemGroup)
    source = model.DataSource.Where(x => x.ItemGroupName.ToUpper().Contains(SearchValue.ToUpper()));

if (SelectedColumn == Resources.Journaux.CountingGroup)
    source = model.DataSource.Where(x => x.CountingGroupName.ToUpper().Contains(SearchValue.ToUpper()));
```

The effect is even worse due to the Where() clause that create a new
temporary upper string for each element of the sequence!

This recommendation also applies to types that provides string-based direct
access to characters such as in the following code:

```csharp
if (!uriBuilder.ToString().EndsWith(".", true, invCulture))
```

where ```ToString()``` is not needed because it is possible to directly access the
last character:

```csharp
if (uriBuilder[uriBuilder.Length - 1] != '.')
```

### Caching strings and interning

s
