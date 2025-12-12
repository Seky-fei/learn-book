# 1.基础知识

## 1.快速失败 和安全失败 的区别

​	HashMap、ArrayList 这些集合类，这些在 java.util 包的集合类就都是快速失败的；而 java.util.concurrent 包下的类都是安全失败，比如：ConcurrentHashMap。快速失败 (fail-fast) 和安全失败 (fail-safe) 的区别是什么？

**(1)快速失败（fail-fast）**

​	在使用迭代器对集合对象进行遍历的时候，如果 A 线程正在对集合进行遍历，此时 B 线程对集合进行修改（增加、删除、修改），或者 A 线程在遍历过程中对集合进行修改，都会导致 A 线程抛出 ConcurrentModificationException 异常。

```json
HashMap hashMap = new HashMap();
hashMap.put("不只Java-1", 1);
hashMap.put("不只Java-2", 2);
hashMap.put("不只Java-3", 3);

Set set = hashMap.entrySet();
Iterator iterator = set.iterator();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
	hashMap.put("下次循环会抛异常", 4);  //会抛出ConcurrentModificationException异常
	System.out.println("此时 hashMap 长度为" + hashMap.size());
}
```

为什么在用迭代器遍历时，修改集合就会抛异常时？

​	原因是迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个modCount 变量。集合在被遍历期间如果内容发生变化，就会改变 modCount 的值。

​	每当迭代器使用 hashNext()/next() 遍历下一个元素之前，都会检测 modCount 变量是否为expectedModCount值，是的话就返回遍历；否则抛出异常，终止遍历。

```json
#1.迭代器变量过程中，不能用集合本身进行修改（增加、删除、修改），可以用Iterator.remove()操作。
	迭代器可以在迭代的过程中删除底层集合的元素, 但是不可以直接调用集合的 remove(Object Obj) 删除，可以通过迭代器的 remove() 方法删除。（可以使用ListIterator进行修改元素，ListIterator的父接口是Iterator，是List接口中特有的迭代器）
```

**(2)安全失败（fail-safe）**

​	采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，故不会抛 ConcurrentModificationException 异常。

```json
 #java.util.concurrent 包下的类都是安全失败，比如：ConcurrentHashMap
```

## 2.Arrays.asList()的坑

Arrays.asList谨慎使用，有三个坑：

**(1)用Array.asList转换基础类型数组**

```json
public static void main() {
	int[] arr = {1, 2, 3};
	List list = Arrays.asList(arr);
	log.info("list:{} size:{} class:{}", list, list.size(), list.get(0).getClass());
}
#输出：
list:[[I@78cb5849] size:1 class:class [I
```

**(2)Arrays.asList 返回的 List 不支持增删操作**

​	Arrays.asList返回的ArrayList是Arrays内部的ArrayList，其继承AbstractList，AbstractList内部的add和remove没有实现，Arrays内部的ArrayList，没有重写add、remove等方法，调用会直接抛异常。

**(3)对原始数组的修改会直接影响得到的list**

Arrays.asList()生成的那个Arrays内部的ArrayList直接使用了原始的array，因此原始数组的修改会直接影响到生成的ArrayList。

```json
public static void main() {
	String[] arr = {"1", "2", "3"};
	List<String> list = Arrays.asList(arr);
	log.info("list 0 before modify: "+list.get(0));
	arr[0]="aaaaa";
	log.info("list 0 after modify: "+list.get(0));
}

#输出结果：
list 0 before modify: 1
list 0 after modify: aaaaa
```





