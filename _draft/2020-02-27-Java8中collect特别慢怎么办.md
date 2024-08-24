
## Stream的Collect操作

> 在Java8中，普遍会使用lambda表达式，因为其丰富的API让你的代码看起来确实很爽，尤其在对集合操作的时候特别方便，但是也会有注意不到的坑。

```java
List<Object> objects = new ArrayList<>(10000);
// 伪代码，初始化集合......
List<Object> filterObj = objects
							.stream()
							.filter(Objects::nonNull)
							.parallel() // 并行处理
							.collect(Collectors.toList());
```


如上代码所示，是一个过滤后再组成一个list集合的操作。中间有一段标注的**parallel**方法，在未使用这个并行方法时，788条数据花了4S，使用该方法之后用时缩短到0.6S。那么是不是用parallel方法，总是并行处理就对了呢？其实这用法是需要分场景的，下面简单看下parallelStream的实现机制。

- 了解parallelStream前先看下**ForkJoin**和**ForkJoinPool**，众所周知ForkJoin框架是Java7提供并行处理的框架，使用了分而治之策略。总的说来就是把一个大任务拆分成很多子任务，子任务执行完成后，再将结果汇总输出。细节这里不再赘述，网上大把解析的文章。

当有了并行的思路，便明白其中线程需要切换上下文也是有时间开销的，而且因为是多线程所以输出是无序的，需要说这里需要分场景来决定是否使用parallelStream。

> **使用建议**：对于大数据量和易于切割的数据类型如ArrayList使用集合收集，且是多核服务器时，最好使用并行（parallel）的方式。但是这种方式是以磁盘、CPU换效率的处理，可能会导致CPU突然的上升。而对于小数据量和单核服务器则不建议使用！

## lambda的使用局限
> 首先，我们一般认为lambda提供了与传统方式接近相等的性能，但在一些特定场景下依旧存在较大性能差异。
1. **初始化开销**，在首次调用时，JVM需要为其构建[CallSite实例](https://docs.oracle.com/javase/8/docs/api/java/lang/invoke/CallSite.html)，也就意味着在初始化使用lambda可能导致启动程序变慢。因此初始化建议使用传统方式。
2. 对于lambda这种**fluent风格**编码对调试不友好，程序栈更复杂，对异常检查也存在着一些局限。

