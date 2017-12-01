# Safe Programming

* Use static typing wherever possible.

## Be Functional

* Make data immutable wherever possible.
* Make methods pure wherever possible.

## Avoid `null` references

* Make references non-nullable wherever possible.
* Use the `[NotNull]` attribute for parameters, fields and results. Enforce it with a runtime check.
* Always test parameters, local variables and fields that can be `null`.

Instead of allowing `null` for a parameter, avoid null-checks with a null implementation.

```csharp
interface ILogger
{
  bool Log(string message);
}

class NullLogger : ILogger
{
  void Log(string message)
  {
    // NOP
  }
}
```

## Local variables

* Do not re-use local variable names, even though the scoping rules are well-defined and allow it. This prevents surprising effects when the variable in the inner scope is removed and the code continues to compile because the variable in the outer scope is still valid.
* Do not modify a variable with a prefix or suffix operator more than once in an expression. The following statement is not allowed:
  ```csharp
  items[propIndex++] = ++propIndex;
  ```

## Side Effects

A side effect is a change in an object as a result of reading a property or calling a method that causes the result of the property or method to be different when called again.

* Prefer pure methods.
* Void methods have side effects by definition.
* Writing a property must cause a side effect.
* Reading a property should not cause a side effect. An exception is lazy-initialization to cache the result.
* Avoid writing methods that return results _and_ cause side effects. An exception is lazy-initialization to cache the result.

## ”Access to Modified Closure”

`IEnumerable<T>` sequences are evaluated lazily. ReSharper will warn of multiple enumeration.

You can accidentally change the value of a captured variable before the sequence is evaluated. Since _ReSharper_ will complain about this behavior even when it does not cause unwanted side-effects, it is important to understand which cases are actually problematic.

```csharp
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = new List<string>();

foreach (var d in data)
{
  if (otherData.Where(od => od == d).Any())
  {
    overlapData.Add(d);
  }
}

Assert.That(overlapData.Count, Is.EqualTo(1)); // "bla"
```

The reference to the variable `d` will be flagged by _ReSharper_ and marked as an _“access to a modified closure”_. This indicates that a variable referenced—or “captured”—by the lambda expression—closure—will have the last value assigned to it rather than the value that was assigned to it when the lambda was created.

In the example above, the lambda is created with the first value in the sequence, but since we only use the lambda once, and then always before the variable has been changed, we don’t have to worry about side-effects. _ReSharper_ can only detect that a variable referenced in a closure is being changed within its scope.

Even though there isn’t a problem in this case, rewrite the `foreach`-statement above as follows to eliminate the _access to modified closure_ warning.

```csharp
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = data.Where(d => otherData.Where(od => od == d).Any()).ToList();

Assert.That(overlapData.Count, Is.EqualTo(1)); // "bla"
```

Finally, use library functionality wherever possible. In this case, we should use `Intersect` to calculate the overlap (intersection).

```csharp
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = data.Intersect(otherData).ToList();

Assert.That(overlapData.Count, Is.EqualTo(1)); // "bla"
```

Remember to be aware of how items are compared. The `Intersects` method above compares using `Equals`, not reference-equality.

The following example does not yield the expected result:

```csharp
var data = new[] { "foo", "bar", "bla" };

var threshold = 2;
var twoLetterWords = data.Where(d => d.Length == threshold);

threshold = 3;
var threeLetterWords = data.Where(d => d.Length == threshold);

Assert.That(twoLetterWords.Count(), Is.EqualTo(0));
Assert.That(threeLetterWords.Count(), Is.EqualTo(3));
```

The lambda in `twoLetterWords` _references_ `threshold`, which is then changed before the lambda is evaluated with `Count()`. There is nothing wrong with this code, but the results can be surprising. Use `ToList()` to evaluate the lambda in `twoLetterWords` _before_ the threshold is changed.

```csharp
var data = new[] { "foo", "bar", "bla" };
var threshold = 2;
var twoLetterWords = data.Where(d => d.Length == threshold).ToList();

threshold = 3;
var threeLetterWords = data.Where(d => d.Length == threshold);

Assert.That(twoLetterWords.Count(), Is.EqualTo(0));
Assert.That(threeLetterWords.Count(), Is.EqualTo(3));
```

## "Collection was modified; enumeration operation may not execute."

Changing a sequence during enumeration causes a runtime error.

The following code will fail whenever `data` contains an element for which `IsEmpty` returns `true`.

```csharp
foreach (var d in data.Where(d => d.IsEmpty))
{
  data.Remove(d);
}
```

To avoid this problem, use an in-memory copy of the sequence instead. A good practice is to use `ToList()` to create the copy and to call it in the `foreach` statement so that it's clear why it's being used.

```csharp
foreach (var d in data.Where(d => d.IsEmpty).ToList())
{
  data.Remove(d);
}
```

## "Possible multiple enumeration of IEnumerable"

Suppose, in the example above, that we also want to know how many elements were empty. Let's start by extracting `emptyElements` to a variable.

```csharp
var emptyElements = data.Where(d => d.IsEmpty);
foreach (var d in emptyElements.ToList())
{
  data.Remove(d);
}

return emptyElements.Count();
```

Since `emptyElements` is evaluated lazily, the call to `Count()` to return the result will evaluate the iterator again, producing a sequence that is now empty—because the `foreach`-statement removed them all from data. The code above will always return zero.

A more critical look at the code above would discover that the `emptyElements` iterator is triggered twice: by the call to `ToList()` and `Count()` (ReSharper will helpfully indicate this with an inspection). Both `ToList()` and `Count()` logically iterate the entire sequence.

To fix the problem, we lift the call to `ToList()` out of the `foreach` statement and into the variable.

```csharp
var emptyElements = data.Where(d => d.IsEmpty).ToList();
foreach (var d in emptyElements)
{
  data.Remove(d);
}

return emptyElements.Count;
```

We can eliminate the `foreach` by directly re-assigning `data`, as shown below.

```csharp
var dataCount = data.Count;
var data = data.Where(d => !d.IsEmpty).ToList();

return dataCount – data.Count;
```

The first algorithm is more efficient when the majority of item in `data` are empty. The second algorithm is more efficient when the majority is non-empty.

## Controlling Class Instantiation

Careless instatiation of a class can lead to unwanted side effects in transactions that are compcarried out downstream of the object's creation. Take the following class as an example:

```csharp
  public class Human
  {
    public string Name {get; set;}

    public int Age {get; set;}

    public bool HasHair {get; set;}
  }
```

An object can be created from this class as simply as follows:

```csharp
  var human = new Human();
```

Great - nice and easy with minimal code. But there are two problems to this:

1. Modern day software development practices such as Domain Driven Design have it that you should ensure your code reflects its real-world abstraction. In the example above, we have created a new `Human`. This new `Human` however is only a "shell" so to speak; having no attibutes or properties giving it any purpose. In this abstraction, it does not make sense to create a new `Human` that is missing vital information such as its Name and Age. It's not possible in real-life so why should this be allowed to happen in your code?

2. With a brand new `Human` object created, it will most likely either be passed to another method or saved to a database. This new instantiation, `human`, carries problems with it. Take the following example:

```csharp
  public int GetNumberOfLettersInName(Human human)
  {
      return human.Name.Length;
  }
```

If we were to pass our newly created `human` object into this method, we'd get a `NullReferenceException` thrown as the `Name` property was never set.

NULL reference checks could be introduced to the method but doing this often causes uneccessary background noise and impacts the readability of the code. It is advised that the following changes are made to the class:

```csharp
  public class Human
  {
    public string Name {get; set;}

    public int Age {get; set;}

    public bool HasHair {get; set;}

    private Human()
    {

    }

    public static Human Create(string name, int Age, bool hasHair)
    {
      return new Human()
      {
        Name = name,
        Age = age,
        HasHair = hasHair
      };
    }
  }
```

The benefits of this approach are:

* The default Constructor for the class has been made `private`. This means that only the class itself has the ability to create a new instance of itself. This stops the client from instantiating empty instances of the class that could cause errors in code consuming it.

* The `Create` method is the only publicly exposed way in which the client can create a new instance of the `Human` class. Due to this, you can control the minimum arguments required to create a valid instance of the class. In this example above, you can only create a new `Human` if you provide its `Name`, `Age` and whether or not it `HasHair`. By knowing that it's not possible to create a `Human` without a `Name`, it removes the need for NULL reference checks downstream of class instantiation.