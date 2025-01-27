## Chapter 4. Classes and Interfaces（类和接口）

### Item 15: Minimize the accessibility of classes and members（尽量减少类和成员的可访问性）

要将设计良好的组件与设计糟糕的组件区别开来，最重要的因素是：隐藏内部数据和其他实现细节的程度。设计良好的组件隐藏了所有实现细节，将 API 与实现完全分离。组件之间只通过它们的 API 进行通信，而不知道彼此的内部工作方式。这个概念被称为信息隐藏或封装，是软件设计的基本原则 [Parnas72]。

由于许多原因，信息隐藏是重要的，其中大部分原因源于这样一个事实：它解耦了组成系统的组件，允许它们被独立开发、测试、优化、使用、理解和修改。这加快了系统开发进度，因为组件可以并行开发。也减轻了维护的负担，因为组件可以被更快地理解、调试或替换，而不必担心会损害其他组件。虽然信息隐藏本身不会获得良好的性能，但它可以实现有效的性能调优：一旦系统完成，概要分析确定了哪些组件会导致性能问题（[Item-67](/Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)），就可以在不影响其他组件正确性的情况下对这些组件进行优化。信息隐藏增加了软件的复用性，因为没有紧密耦合的组件在其他场景中通常被证明是有用的，除了开发它们时所在的场景之外。最后，信息隐藏降低了构建大型系统的风险，因为即使系统没有成功，单个组件也可能被证明是成功的。

Java 有许多工具来帮助隐藏信息。访问控制机制 [JLS, 6.6] 指定了类、接口和成员的可访问性。实体的可访问性由其声明的位置以及声明中出现的访问修饰符（private、protected 和 public）决定。正确使用这些修饰符是信息隐藏的关键。

经验法则很简单：让每个类或成员尽可能不可访问。换句话说，在不影响软件正常功能时，使用尽可能低的访问级别。

对于顶级（非嵌套）类和接口，只有两个可能的访问级别：包私有和公共。如果用 public 修饰符声明一个顶级类或接口，它将是公共的；否则，它将是包私有的。如果顶级类或接口可以设置为包私有，那么就应该这么做。通过将其设置为包私有，可以使其成为实现的一部分，而不是导出的 API 的一部分，并且可以在后续版本中修改、替换或删除它，而不必担心损害现有的客户端。如果将其公开，就有义务永远提供支持，以保持兼容性。

如果包级私有顶级类或接口只被一个类使用，那么可以考虑：在使用它的这个类中，将顶级类设置为私有静态嵌套类（[Item-24](/Chapter-4/Chapter-4-Item-24-Favor-static-member-classes-over-nonstatic.md)）。对于包中的所有类以及使用它的类来说，这降低了它的可访问性。但是，降低公共类的可访问性比减少包级私有顶级类的可访问性重要得多：公共类是包 API 的一部分，而包级私有顶级类已经是实现的一部分。

对于成员（字段、方法、嵌套类和嵌套接口），有四个可能的访问级别，这里按可访问性依次递增的顺序列出：

- **私有**，成员只能从声明它的顶级类内部访问。

- **包级私有**，成员可以从包中声明它的任何类访问。技术上称为默认访问，即如果没有指定访问修饰符（接口成员除外，默认情况下，接口成员是公共的），就会得到这个访问级别。

- **保护**，成员可以通过声明它的类的子类（会受一些限制 [JLS, 6.6.2]）和声明它的包中的任何类访问。

- **公共**，该成员可以从任何地方访问。

在仔细设计了类的公共 API 之后，你应该本能的使所有成员都是私有的。只有当同一包中的另一个类确实需要访问一个成员时，你才应该删除 private 修饰符，使成员变为包级私有。如果你发现自己经常这样做，那么你应该重新确认系统的设计，看看是否有其他方式能产生更好地相互解耦的类。也就是说，私有成员和包级私有成员都是类实现的一部分，通常不会影响其导出的 API。但是，如果类实现了 Serializable（[Item-86](/Chapter-12/Chapter-12-Item-86-Implement-Serializable-with-great-caution.md) 和 [Item-87](/Chapter-12/Chapter-12-Item-87-Consider-using-a-custom-serialized-form.md)），这些字段可能会「泄漏」到导出的 API 中。

对于公共类的成员来说，当访问级别从包级私有变为保护时，可访问性会有很大的提高。保护成员是类导出 API 的一部分，必须永远支持。此外，导出类的保护成员表示对实现细节的公开承诺（[Item-19](/Chapter-4/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)）。需要保护成员的场景应该相对少见。

有一个关键规则限制了你降低方法的可访问性。如果一个方法覆盖了超类方法，那么它在子类中的访问级别就不能比超类 [JLS, 8.4.8.3] 更严格。这对于确保子类的实例在超类的实例可用的任何地方都同样可用是必要的（Liskov 替换原则，请参阅 [Item-15](/Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)）。如果违反此规则，编译器将在尝试编译子类时生成错误消息。这个规则的一个特例是，如果一个类实现了一个接口，那么该接口中的所有类方法都必须在类中声明为 public。

为了便于测试代码，你可能会试图使类、接口或成员更容易访问。这在一定程度上是好的。为了测试，将公共类成员由私有变为包私有是可以接受的，但是进一步提高可访问性是不可接受的。换句话说，将类、接口或成员作为包导出 API 的一部分以方便测试是不可接受的。幸运的是，也没有必要这样做，因为测试可以作为包的一部分运行，从而获得对包私有元素的访问权。

**公共类的实例字段很少采用 public 修饰**（[Item-16](/Chapter-4/Chapter-4-Item-16-In-public-classes-use-accessor-methods-not-public-fields.md)）。如果实例字段不是 final 的，或者是对可变对象的引用，那么将其公开，你就放弃了限制字段中可以存储的值的能力。这意味着你放弃了强制包含字段的不可变的能力。此外，你还放弃了在修改字段时采取任何操作的能力，因此 **带有公共可变字段的类通常不是线程安全的。** 即使一个字段是 final 的，并且引用了一个不可变的对象，通过将其公开，你放弃了切换到一个新的内部数据表示的灵活性，而该字段并不存在。

同样的建议也适用于静态字段，只有一个例外。你可以通过公共静态 final 字段公开常量，假设这些常量是类提供的抽象的组成部分。按照惯例，这些字段的名称由大写字母组成，单词以下划线分隔（[Item-68](/Chapter-9/Chapter-9-Item-68-Adhere-to-generally-accepted-naming-conventions.md)）。重要的是，这些字段要么包含基本数据类型，要么包含对不可变对象的引用（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）。包含对可变对象的引用的字段具有非 final 字段的所有缺点。虽然引用不能被修改，但是引用的对象可以被修改会导致灾难性的后果。

请注意，非零长度的数组总是可变的，因此对于类来说，拥有一个公共静态 final 数组字段或返回该字段的访问器是错误的。如果一个类具有这样的字段或访问器，客户端将能够修改数组的内容。这是一个常见的安全漏洞来源：

```java
// Potential security hole!
public static final Thing[] VALUES = { ... };
```

要注意的是，一些 IDE 生成了返回私有数组字段引用的访问器，这恰恰会导致这个问题。有两种方法可以解决这个问题。你可以将公共数组设置为私有，并添加一个公共不可变 List：

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

或者，你可以将数组设置为私有，并添加一个返回私有数组副本的公共方法：

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

如何在这些备选方案中进行选择，请考虑客户可能会如何处理结果。哪种返回类型更方便？哪种表现会更好？

对于 Java 9，作为模块系统的一部分，还引入了另外两个隐式访问级别。模块是包的分组单位，就像包是类的分组单位一样。模块可以通过模块声明中的导出声明显式地导出它的一些包（按照约定包含在名为 `module-info.java` 的源文件中）。模块中未导出包的公共成员和保护成员在模块外不可访问；在模块中，可访问性不受导出声明的影响。通过使用模块系统，你可以在模块内的包之间共享类，而不会让整个世界看到它们。未导出包中的公共类和保护成员产生了两个隐式访问级别，它们是正常公共级别和保护级别的类似物。这种共享的需求相对较少，通常可以通过重新安排包中的类来解决。

与四个主要的访问级别不同，这两个基于模块的级别在很大程度上是建议级别。如果将模块的 JAR 文件放在应用程序的类路径上，而不是模块路径上，模块中的包将恢复它们的非模块行为：包的公共类的所有公共成员和保护成员都具有正常的可访问性，而不管模块是否导出包 [Reinhold,1.2]。严格执行新引入的访问级别的一个地方是 JDK 本身：Java 库中未导出的包在其模块之外确实不可访问。

对于典型的 Java 程序员来说，访问保护不仅是有限实用的模块所提供的，而且本质上是建议性的；为了利用它，你必须将包以模块分组，在模块声明中显式地声明它们的所有依赖项，重新安排源代码树，并采取特殊操作以适应从模块中对非模块化包的任何访问 [Reinhold, 3]。现在说模块能否在 JDK 之外得到广泛使用还为时过早。与此同时，除非你有迫切的需求，否则最好还是不使用它们。

总之，你应该尽可能减少程序元素的可访问性（在合理的范围内）。在仔细设计了一个最小的公共 API 之后，你应该防止任何游离的类、接口或成员成为 API 的一部分。除了作为常量的公共静态 final 字段外，public 类应该没有公共字段。确保公共静态 final 字段引用的对象是不可变的。

---