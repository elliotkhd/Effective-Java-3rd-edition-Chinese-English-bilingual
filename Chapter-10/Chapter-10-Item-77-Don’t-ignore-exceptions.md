## Chapter 10. Exceptions（异常）

### Item 77: Don’t ignore exceptions（不要忽略异常）

虽然这一建议似乎显而易见，但它经常被违反，因此值得强调。当 API 的设计人员声明一个抛出异常的方法时，他们试图告诉你一些事情。不要忽略它！如果在方法调用的周围加上一条 try 语句，其 catch 块为空，可以很容易忽略异常：

```java
// Empty catch block ignores exception - Highly suspect!
try {
    ...
}
catch (SomeException e) {
}
```

**空 catch 块违背了异常的目的，** 它的存在是为了强制你处理异常情况。忽略异常类似于忽略火灾警报一样，关掉它之后，其他人就没有机会看到是否真的发生了火灾。你可能侥幸逃脱，但结果可能是灾难性的。每当你看到一个空的 catch 块，你的脑海中应该响起警报。

在某些情况下，忽略异常是合适的。例如，在关闭 FileInputStream 时，忽略异常可能是合适的。你没有更改文件的状态，因此不需要执行任何恢复操作，并且已经从文件中读取了所需的信息，因此没有理由中止正在进行的操作。记录异常可能是明智的，这样如果这些异常经常发生，你应该研究起因。**如果你选择忽略异常，catch 块应该包含一条注释，解释为什么这样做是合适的，并且应该将变量命名为 ignore：**

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // Default; guaranteed sufficient for any map
try {
    numColors = f.get(1L, TimeUnit.SECONDS);
}
catch (TimeoutException | ExecutionException ignored) {
    // Use default: minimal coloring is desirable, not required
}
```

本条目中的建议同样适用于 checked 异常和 unchecked异常。不管异常是表示可预测的异常条件还是编程错误，用空 catch 块忽略它将导致程序在错误面前保持静默。然后，程序可能会在未来的任意时间点，在与问题源没有明显关系的代码中失败。正确处理异常可以完全避免失败。仅仅让异常向外传播，可能会导致程序走向失败，保留信息有利于调试。

---