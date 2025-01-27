## Chapter 4. Classes and Interfaces（类和接口）

### Item 19: Design and document for inheritance or else prohibit it（继承要设计良好并且具有文档，否则禁止使用）

[Item-18](/Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md) 提醒你注意：将不是为继承设计并且缺少文档的「外部」类进行子类化的危险。那么，为继承而设计并且具备文档的类意味着什么呢？

首先，类必须精确地在文档中记录覆盖任何方法的效果。换句话说，类必须在文档中记录它对可覆盖方法的自用性。对于每个公共或受保护的方法，文档必须指出方法调用的可覆盖方法、调用顺序以及每次调用的结果如何影响后续处理过程。（可覆盖的意思是非 final 的，公共的或受保护的。）更一般地说，类必须记录它可能调用可覆盖方法的所有情况。例如，可能调用来自后台线程或静态初始化器的方法。

调用可覆盖方法的方法在其文档注释末尾应包含这些调用的描述。描述在规范的一个特殊部分中，标记为「Implementation Requirements（实现需求）」，它由 Javadoc 标签 `@implSpec` 生成。本节描述该方法的内部工作方式。下面是一个示例，复制自 `java.util.AbstractCollection` 规范：

> public boolean remove(Object o)

> 从此集合中移除指定元素的单个实例，如果存在（可选操作）。更正式地说，如果此集合包含一个或多个这样的元素，则删除元素 e，使得 `Objects.equals(o, e)`，如果此 collection 包含指定的元素，则返回 true（或等效地，如果此集合因调用而更改）。

> 实现需求：这个实现遍历集合，寻找指定的元素。如果找到元素，则使用迭代器的 remove 方法从集合中删除元素。注意，如果这个集合的迭代器方法返回的迭代器没有实现 remove 方法，并且这个集合包含指定的对象，那么这个实现将抛出 UnsupportedOperationException。

这篇文档无疑说明了重写迭代器方法将影响 remove 方法的行为。它还准确地描述了迭代器方法返回的迭代器的行为将如何影响 remove 方法的行为。与 [Item-18](/Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md) 中的情况相反，在 [Item-18](/Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md) 中，程序员子类化 HashSet 不能简单地说覆盖 add 方法是否会影响 addAll 方法的行为。

但是，这是否违背了一个格言：好的 API 文档应该描述一个给定的方法做什么，而不是如何做？是的，它确实违背了！这是继承违反封装这一事实的不幸结果。要为一个类编制文档，使其能够安全地子类化，你必须描述实现细节，否则这些细节应该是未指定的。

`@implSpec` 标记在 Java 8 中添加，在 Java 9 中大量使用。默认情况下应该启用这个标记，但是在 Java 9 中，Javadoc 实用程序仍然忽略它，除非传递命令行开关 `-tag "apiNote: a :API Note:"`。

为继承而设计不仅仅是记录自用性模式。为了允许程序员编写高效的子类而不受不必要的痛苦，类可能必须以明智地选择受保护的方法或（在很少的情况下）受保护的字段的形式为其内部工作提供挂钩。例如，考虑来自 `java.util.AbstractList` 的 removeRange 方法：

> protected void removeRange(int fromIndex, int toIndex)

> 从这个列表中删除所有索引位于 fromIndex（包含索引）和 toIndex（独占索引）之间的元素。将任何后续元素移到左边（减少其索引）。这个调用使用 `(toIndex - fromIndex)` 元素缩短列表。（如果 toIndex == fromIndex，此操作无效。）

> 此方法由此列表及其子列表上的 clear 操作调用。重写此方法以利用列表实现的内部特性，可以显著提高对该列表及其子列表的 clear 操作的性能。

> 实现需求：该实现获取位于 fromIndex 之前的列表迭代器，并依次重复调用 `ListIterator.next` 和 `ListIterator.remove`，直到删除整个范围的内容。注意：如果 `ListIterator.remove` 需要线性时间，这个实现需要平方级的时间。

> 参数

> 要删除的第一个元素的 fromIndex 索引。

> 要删除的最后一个元素后的索引。

此方法对列表实现的最终用户没有任何兴趣。它的提供只是为了让子类更容易在子列表上提供快速清晰的方法。在没有 removeRange 方法的情况下，当在子列表上调用 clear 方法或从头重写整个子列表机制时，子类将不得不处理二次性能——这不是一项简单的任务!

那么，在为继承设计类时，如何决定要公开哪些受保护的成员呢？不幸的是，没有灵丹妙药。你能做的最好的事情就是认真思考，做出最好的猜测，然后通过编写子类来测试它。你应该尽可能少地公开受保护的成员，因为每个成员都表示对实现细节的承诺。另一方面，你不能公开太多，因为缺少受保护的成员会导致类实际上无法用于继承。

**测试为继承而设计的类的唯一方法是编写子类。** 如果你忽略了一个关键的受保护成员，那么尝试编写子类将使遗漏变得非常明显。相反，如果编写了几个子类，而没有一个子类使用受保护的成员，则应该将其设置为私有。经验表明，三个子类通常足以测试一个可扩展类。这些子类中的一个或多个应该由超类作者以外的其他人编写。

当你为继承设计一个可能获得广泛使用的类时，请意识到你将永远致力于你所记录的自使用模式，以及在其受保护的方法和字段中隐含的实现决策。这些承诺会使在后续版本中改进类的性能或功能变得困难或不可能。因此，**你必须在发布类之前通过编写子类来测试类。**

另外，请注意，继承所需的特殊文档会使普通文档变得混乱，这种文档是为那些创建类实例并在其上调用方法的程序员设计的。在撰写本文时，很少有工具能够将普通 API 文档与只对实现子类的程序员感兴趣的信息分离开来。

为了允许继承，类必须遵守更多的限制。**构造函数不能直接或间接调用可重写的方法。** 如果你违反了这个规则，程序就会失败。超类构造函数在子类构造函数之前运行，因此在子类构造函数运行之前将调用子类中的覆盖方法。如果重写方法依赖于子类构造函数执行的任何初始化，则该方法的行为将不像预期的那样。为了使其具体化，下面是一个违反此规则的类：

```java
public class Super {
    // Broken - constructor invokes an overridable method
    public Super() {
        overrideMe();
    }
    public void overrideMe() {
    }
}
```

下面是覆盖 overrideMe 方法的子类，Super 的唯一构造函数错误地调用了 overrideMe 方法：

```java
public final class Sub extends Super {
    // Blank final, set by constructor
    private final Instant instant;
    Sub() {
        instant = Instant.now();
    }
    // Overriding method invoked by superclass constructor
    @Override
    public void overrideMe() {
        System.out.println(instant);
    }
    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

你可能希望这个程序打印两次 instant，但是它第一次打印 null，因为在子构造函数有机会初始化 instant 字段之前，超级构造函数调用了 overrideMe。注意，这个程序观察了两个不同状态的最后一个字段！还要注意，如果 overrideMe 立即调用了任何方法，那么当超级构造函数调用 overrideMe 时，它会抛出一个 NullPointerException。这个程序不抛出 NullPointerException 的唯一原因是 println 方法允许空参数。

注意，从构造函数调用私有方法、最终方法和静态方法是安全的，它们都是不可覆盖的。

可克隆和可序列化的接口在设计继承时存在特殊的困难。对于为继承而设计的类来说，实现这两种接口都不是一个好主意，因为它们给扩展类的程序员带来了沉重的负担。但是，你可以采取一些特殊的操作来允许子类实现这些接口，而无需强制它们这样做。[Item-13](/Chapter-3/Chapter-3-Item-13-Override-clone-judiciously.md) 和 [Item-86](/Chapter-12/Chapter-12-Item-86-Implement-Serializable-with-great-caution.md) 叙述了这些行动。

如果你确实决定在为继承而设计的类中实现 Cloneable 或 Serializable，那么你应该知道，由于 clone 和 readObject 方法的行为与构造函数非常相似，因此存在类似的限制：clone 和 readObject 都不能直接或间接调用可覆盖的方法。对于 readObject，覆盖方法将在子类的状态反序列化之前运行。在 clone 的情况下，覆盖方法将在子类的 clone 方法有机会修复 clone 的状态之前运行。在任何一种情况下，程序失败都可能随之而来。在 clone 的情况下，失败可以破坏原始对象和 clone。例如，如果覆盖方法假设它正在修改对象的深层结构的 clone 副本，但是复制还没有完成，那么就会发生这种情况。

最后，如果你决定在一个为继承而设计的类中实现 Serializable，并且这个类有一个 readResolve 或 writeReplace 方法，那么你必须使 readResolve 或 writeReplace 方法为 protected，而不是 private。如果这些方法是 private 的，它们将被子类静静地忽略。这是实现细节成为类 API 允许继承的一部分的又一种情况。

到目前为止，显然为继承而设计一个类需要付出很大的努力，并且对类有很大的限制。这不是一个可以轻易作出的决定。在某些情况下，这样做显然是正确的，例如抽象类，包括接口的骨架实现（[Item-20](/Chapter-4/Chapter-4-Item-20-Prefer-interfaces-to-abstract-classes.md)）。还有一些情况显然是错误的，比如不可变类（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。

但是普通的具体类呢？传统上，它们既不是最终的，也不是为子类化而设计和记录的，但这种状态是危险的。每当在这样的类中进行更改时，扩展类的子类就有可能中断。这不仅仅是一个理论问题。在修改未为继承而设计和记录文档的非最终具体类的内部结构后，接收与子类相关的 bug 报告并不罕见。

这个问题的最佳解决方案是禁止在没有设计和文档记录的类中进行子类化。有两种方法可以禁止子类化。两者中比较容易的是声明类 final。另一种方法是将所有构造函数变为私有或包私有，并在构造函数的位置添加公共静态工厂。这个替代方案提供了内部使用子类的灵活性，在 [Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md) 中进行了讨论。两种方法都可以接受。

这个建议可能有点争议，因为许多程序员已经习惯了子类化普通的具体类，以添加工具、通知和同步等功能或限制功能。如果一个类实现了某个接口，该接口捕获了它的本质，例如 Set、List 或 Map，那么你不应该对禁止子类化感到内疚。在 [Item-18](/Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md) 中描述的包装器类模式提供了一种优于继承的方法来增强功能。

如果一个具体的类没有实现一个标准的接口，那么你可能会因为禁止继承而给一些程序员带来不便。如果你认为必须允许继承此类类，那么一种合理的方法是确保该类永远不会调用其任何可重写的方法，并记录这一事实。换句话说，完全消除类对可重写方法的自使用。这样，您将创建一个对子类而言相当安全的类。重写一个方法不会影响任何其他方法的行为。

你可以在不改变类行为的情况下，机械地消除类对可重写方法的自使用。将每个可覆盖方法的主体移动到一个私有的「助手方法」，并让每个可覆盖方法调用它的私有助手方法。然后，用可覆盖方法的私有助手方法的直接调用替换可覆盖方法的每个自使用。

总之，为继承设计一个类是一项艰苦的工作。你必须记录所有的自用模式，并且一旦你记录了它们，你就必须在整个类的生命周期中都遵守它们。如果没有这样做，子类可能会依赖于超类的实现细节，如果超类的实现发生变化，子类可能会崩溃。为了允许其他人编写高效的子类，你可能还需要导出一个或多个受保护的方法。除非你知道确实需要子类，否则最好通过声明类为 final 或确保没有可访问的构造函数的方式来禁止继承。

---