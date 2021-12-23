# Map

java为数据结构中的映射定义了一个接口java.util.Map,他实现了四个类，分别是：HashMap，HashTable，LinkedHashMapTreeMap

==Map不允许键重复，但允许值重复==



## HashMap

最常用的Map，根据键的hashcode值来存储数据，根据键可以直接获得他的值（==因为相同的键hashcode值相同，在地址为hashcode值的地方存储的就是值，所以根据键可以直接获得值==），具有很快的访问速度，遍历时，取得数据的顺序完全是随机的，HashMap最多只允许一条记录的键为null，允许多条记录的值为null，HashMap不支持线程同步，即任意时刻可以有多个线程同时写HashMap，这样对导致数据不一致，如果需要同步，可以使用synchronziedMap的方法使得HashMap具有同步的能力或者使用concurrentHashMap



> ConcurrentHashMap与HashMap相比，有以下不同点

- ConcurrentHashMap线程安全，而HashMap非线程安全
- HashMap允许Key和Value为null，而ConcurrentHashMap不允许
- HashMap迭代器是强一致性，ConcurrentHashMap迭代器是弱一致性，HashMap不允许通过Iterator遍历的同时通过HashMap修改，而ConcurrentHashMap允许该行为，并且该更新对后续的遍历可见。

面试:https://blog.csdn.net/qq_39455116/article/details/82830102





## HashTable

与HashMap类似，不同的是，==它不允许记录的键或值为空，支持线程同步==，即任意时刻只能有一个线程写HashTable，因此也导致HashTable在写入时比较慢!



## LinkedHashMap

是HahsMap的一个子类，但它==保持了记录的插入顺序==，遍历时先得到的肯定是先插入的，也可以在构造时带参数，按照应用次数排序，在遍历时会比HahsMap慢，不过有个例外，当HashMap的容量很大，实际数据少时，遍历起来会比LinkedHashMap慢（因为它是链啊），因为HashMap的遍历速度和它容量有关，LinkedHashMap遍历速度只与数据多少有关



## TreeMap

实现了sortMap接口，==能够把保存的记录按照键排序（默认升序）==，也可以指定排序比较器，遍历时得到的数据是排过序的



> 什么情况用什么类型的Map：
>
> 在Map中插入，删除，定位元素：HashMap
>
> 要按照自定义顺序或自然顺序遍历：TreeMap
>
> 要求输入顺序和输出顺序相同：LinkedHashMap