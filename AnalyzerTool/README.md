---
description: 与MAT相关的一些说明,使用方法等...
---

# Eclipse Memory Analyzer

{% file src=".gitbook/assets/java_pid20176.hprof" %}
堆溢出dump文件
{% endfile %}

> 以上述文件进行说明

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption><p>概览图</p></figcaption></figure>

* 饼状图: 展示最大的几个对象所占内存的比例
* `Histogram`: 列出内存中每个对象的名字,数量和大小
* `Dominator Tree`: 将内存中所有对象按大小进行排序,并且可以分析对象之间的引用结构

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption><p>Dominator Tree图</p></figcaption></figure>

> 内存最大的对象也是最应该去怀疑的

* `Retained Heap`: 表示这个对象以及它所持有的引用(包括直接和间接)所占的总内存
* `Shallow Heap` 当前对象所占内存大小,不包含引用关系
* 每一行左边有一个**图标**,部分图标右下角带有一个橙色的点.带有橙点的对象表明该对象可以从`GC Roots`通过可达性分析算法访问到的.\
  \
  可以被访问的对象在GC时无法回收,但不一定是泄露的对象,因为部分对象系统需要一直使用,可以观察到部分带橙点的对象右边带有`System Class` 字样,说明这是一个由系统管理的对象.\
  \
  可以观察到的是,第一行的对象所占的内存最大,并且`@ 0xffda4218 main` 是由用户创建的,说明由于该对象能被`GC Roots` 访问到导致不能被回收,导致它持有的其它引用也无法被回收,至此,堆溢出的原因找到了\
  \
  当然,或许我们可能遇到这样一种情况,即内存占用较大的对象在图标上都没有橙点,即没有被`GC Roots` 引用,这可能是该对象持有的引用可以被`GC Roots` 访问到,此时我们可以在怀疑的对象上`右键 -> Path to GC Roots -> exclude weak references` 因为弱引用不会阻止对象被垃圾回收器回收,所以将其排除. 或许会通过层层引用找到被它引用的对象

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption><p>Histogram图</p></figcaption></figure>

上图将当前应用程序中的所有对象的名字、数量和大小列出,在该图中`OOMObject` 对象的`Shallow Heap` 最高,我们可以通过 `右键 -> List objects -> with incoming references` 来查看具体是谁在使用这些对象.



参考:

[Eclipse Memory Analyze](https://blog.csdn.net/a1510841693/article/details/104770912)











