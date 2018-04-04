### 第11条：覆盖equals方法时总要覆盖hashCode方法

**You must override hashCode in every class that overrides equals.** If you fail to do so, your class will violate the general contract for hashCode, which will prevent it from functioning properly in collections such as HashMap and HashSet. Here is the contract, adapted from the Object specification :

* When the hashCode method is invoked on an object repeatedly during an execution of an application, it must consistently return the same value, provided no information used in equals comparisons is modified. This value need not remain consistent from one execution of an application to another.

* If two objects are equal according to the equals\(Object\) method, then calling hashCode on the two objects must produce the same integer result.

* If two objects are unequal according to the equals\(Object\) method, it is not required that calling hashCode on each of the objects must produce distinct results. However, the programmer should be aware that producing distinct results for unequal objects may improve the performance of hash tables.

**在每个覆盖了equals方法的类里，我们也必须覆盖hashCode方法。**如果我们不这么做，我们的类将会违反hashCode的通用约定，而这也将导致我们的类无法在诸如HashMap和HashSet这样的集合中正常运作。下面是约定的内容，依据于Object规范：

* 只要equals方法里用到的信息没有被更改，那么在一个应用的执行期间，不管调用hashCode方法多少次，hashCode方法都必须持续返回相同的值。但在一个应用程序的多次执行过程中，这个值可以不一致。
* 若两个对象根据equals\(Object\)方法比较是相等的，那么分别调用这两个对象的hashCode方法时必须产生相同的整型数值。
* 若两个对象根据equals\(Object\)方法比较是不等的，那么分别调用这两个对象的hashCode方法时不一定要产生不同的数值。只是程序员应该知道，对于不等的对象若能产生不同的哈希值，有助于提高哈希表的性能。

The key provision that is violated when you fail to override hashCode is the second one: equal objects must have equal hash codes. Two distinct instances may be logically equal according to a class’s equals method, but to Object’s hashCode method, they’re just two objects with nothing much in common. Therefore, Object’s hashCode method returns two seemingly random numbers instead of two equal numbers as required by the contract.

因没有覆盖hashCode方法而违反约定的关键条约是第二条：相等的对象必须有着相同的哈希码（hash code）。根据类的equals方法，两个截然不同的实例在逻辑上可以是相等的，但对于Object的hashCode方法来说，它们也只是两个没有共同之处的对象。因此，Object的hashCode方法会根据两个对象分别返回两个看起来随机的数值，而不是像约定要求的那样返回两个相等的数值。

For example, suppose you attempt to use instances of the PhoneNumber class from Item 10 as keys in a HashMap:

例如，假设我们想将条目10里的PhoneNumber类的实例作为一个HashMap的键：

```
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
```

At this point, you might expect m.get\(new PhoneNumber\(707, 867, 5309\)\) to return "Jenny", but instead, it returns null. Notice that two PhoneNumber instances are involved: one is used for insertion into the HashMap, and a second, equal instance is used for \(attempted\) retrieval. The PhoneNumber class’s failure to override hashCode causes the two equal instances to have unequal hash codes, in violation of the hashCode contract. Therefore, the get method is likely to look for the phone number in a different hash bucket from the one in which it was stored by the put method. Even if the two instances happen to hash to the same bucket, the get method will almost certainly return null, because HashMap has an optimization that caches the hash code associated with each entry and doesn’t bother checking for object equality if the hash codes don’t match.

在这个时候，你可能会期望m.get\(new PhoneNumber\(707, 867, 5309\)\)会返回“Jenny”，然而，它返回了null。我们注意到，这里涉及到两个PhoneNumber实例：一个用来插入HashMap，另一个相等的实例用来（试图用来）获取值。PhoneNumber没有覆盖hashCode导致两个相等的实例没有相等的哈希码，这违反了hashCode约定。因此，put方法将电话号码放入了一个哈希桶（hash bucket）里，而get方法却去一个不同的哈希桶里寻找。即使这两个对象恰好都哈希进了同一个桶，get方法还是很有可能会返回null，因为HashMap有一项优化，它会将每个项关联的哈希码缓存起来，假如传进来的哈希码不匹配，则不校验对象是否相等。

Fixing this problem is as simple as writing a

proper hashCode method for PhoneNumber. So what should

a hashCode method look like? It’s trivial to write a bad one. This one,

for example, is always legal but should never be used:
