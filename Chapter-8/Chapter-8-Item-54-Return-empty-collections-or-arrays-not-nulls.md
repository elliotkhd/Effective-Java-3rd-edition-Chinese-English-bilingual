## Chapter 8. Methods（方法）

### Item 54: Return empty collections or arrays, not nulls（返回空集合或数组，而不是 null）

如下的方法很常见：

```java
// Returns null to indicate an empty collection. Don't do this!
private final List<Cheese> cheesesInStock = ...;
/**
* @return a list containing all of the cheeses in the shop,
* or null if no cheeses are available for purchase.
*/
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null: new ArrayList<>(cheesesInStock);
}
```

没有理由对没有奶酪可供购买的情况进行特殊处理。如果这样做，在客户端需要额外的代码处理可能为空的返回值，例如：

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
    System.out.println("Jolly good, just the thing.");
```

在几乎每次使用返回 null 来代替空集合或数组的方法时，都需要使用这种权宜之计。它很容易出错，因为编写客户端的程序员可能忘记编写特殊情况的代码来处理 null 返回。这样的错误可能会被忽略多年，因为这样的方法通常返回一个或多个对象。此外，在空容器中返回 null 会使返回容器的方法的实现复杂化。

有时有人认为，空返回值比空集合或数组更可取，因为它避免了分配空容器的开销。这个论点有两点是不成立的。首先，在这个级别上担心性能是不明智的，除非分析表明这个方法正是造成性能问题的真正源头（[Item-67](/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)）。第二，返回空集合和数组而不分配它们是可能的。下面是返回可能为空的集合的典型代码。通常，这就是你所需要的：

```java
//The right way to return a possibly empty collection
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

在不太可能的情况下，你有证据表明分配空集合会损害性能，你可以通过重复返回相同的不可变空集合来避免分配，因为不可变对象可以自由共享（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。下面是使用 `Collections.emptyList` 完成此任务的代码。如果你要返回一个 Set，你会使用 `Collections.emptySet`；如果要返回 Map，则使用 `Collections.emptyMap`。但是请记住，这是一个优化，很少真正需要它。如果你认为你需要它，测试一下前后的表现，确保它确实有帮助：

```java
// Optimization - avoids allocating empty collections
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList(): new ArrayList<>(cheesesInStock);
}
```

数组的情况与集合的情况相同。永远不要返回 null，而应该返回零长度的数组。通常，你应该简单地返回一个正确长度的数组，它可能是零长度。注意，我们将一个零长度的数组传递到 toArray 方法中，以指示所需的返回类型，即 Cheese[0]：

```java
//The right way to return a possibly empty array
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

如果你认为分配零长度数组会损害性能，你可以重复返回相同的零长度数组，因为所有的零长度数组都是不可变的：

```java
// Optimization - avoids allocating empty arrays
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

在优化版本中，我们将相同的空数组传递到每个 toArray 调用中，当 cheesesInStock 为空时，这个数组将从 getCheeses 返回。不要为了提高性能而预先分配传递给 toArray 的数组。研究表明，这样做会适得其反 [Shipilev16]:

```java
// Don’t do this - preallocating the array harms performance!
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

总之，永远不要用 null 来代替空数组或集合。它使你的 API 更难以使用，更容易出错，并且没有性能优势。

---