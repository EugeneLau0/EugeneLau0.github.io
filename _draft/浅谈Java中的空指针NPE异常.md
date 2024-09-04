
在Java中，NullPointerException是一种非常常见的异常。常常出现在以下5种场景中：

1. 参数值是包装类如Integer，在使用时会出现自动拆箱导致空指针；
2. 字符串类型常常出现空指针异常；
3. 例如ConcurrentHashMap这类不支持key或value为null，强行put null的key或value则会出现空指针异常；
4. A对象包含B对象，在通过A对象的字段获取B之后，没有对字段进行判空就级联调用B的方法而出现空指针异常；
5. 方法或远程服务返回List不是空而是null，没有进行判空就直接调用List的方法而出现空指针异常。

<!--more-->

为了对以上5种场景进行理解，下面将进行举例说明。