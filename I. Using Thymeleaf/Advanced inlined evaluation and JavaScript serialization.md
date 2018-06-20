### 高级内联表达式与JavaScript序列化

An important thing to note regarding JavaScript inlining is that this expression evaluation is intelligent and not limited to Strings. Thymeleaf will correctly write in JavaScript syntax the following kinds of objects:

- Strings
- Numbers
- Booleans
- Arrays
- Collections
- Maps
- Beans (objects with getter and setter methods)

For example, if we had the following code:
```html
<script th:inline="javascript">
    ...
    var user = /*[[${session.user}]]*/ null;
    ...
</script>
```
That `${session.user}` expression will evaluate to a `User` object, and Thymeleaf will correctly convert it to Javascript syntax:
```html
<script th:inline="javascript">
    ...
    var user = {"age":null,"firstName":"John","lastName":"Apricot",
                "name":"John Apricot","nationality":"Antarctica"};
    ...
</script>
```
The way this JavaScript serialization is done is by means of an implementation of the `org.thymeleaf.standard.serializer.IStandardJavaScriptSerializer` interface, which can be configured at the instance of the `StandardDialect` being used at the template engine.

The default implementation of this JS serialization mechanism will look for the [Jackson library](https://github.com/FasterXML/jackson) in the classpath and, if present, will use it. If not, it will apply a built-in serialization mechanism that covers the needs of most scenarios and produces similar results (but is less flexible).







之前提及的 JavaScript内联机制的智能，远不止应用JavaScript特定的转义和以有效的字面量输出表达式结果。

比如，我们可以在JavaScript注释里包装我们的（转义的）内联表达式：
```html
<script th:inline="javascript">
    ...
    var username = /*[[${session.user.name}]]*/ "Gertrud Kiwifruit";
    ...
</script>
```
Thymeleaf会忽略所有写在注释后面分号前面的东西（这里是`'Gertrud Kiwifruit'`），所以执行的结果会和没有使用包装的注释的情况一样：
```html
<script th:inline="javascript">
    ...
    var username = "Sebastian \"Fruity\" Applejuice";
    ...
</script>
```
但是，再仔细看一下原始的模版代码：
```html
<script th:inline="javascript">
    ...
    var username = /*[[${session.user.name}]]*/ "Gertrud Kiwifruit";
    ...
</script>
```
注意：这是`有效的JavaScript`代码。当你静态地打开你的模版文件（不在服务器上执行）时，它也将完美的执行。

所以，这儿讲的就是一种实现`JavaScript自然模版`的方式！