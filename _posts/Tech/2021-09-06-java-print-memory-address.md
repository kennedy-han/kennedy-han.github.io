---
layout: post
title: "java打印对象内存地址"
description: "java"
category: Tech
tags: [java]
---




## java打印对象内存地址



打印一个对象，默认会调用toString()方法

```
public class Main {

    public static void main(String[] args) {
        Main m1 = new Main();
        Main m2 = new Main();

        System.out.println(m1);
        System.out.println(m2);
    }
}
```

会输出:

```
testing.Main@1b6d3586
testing.Main@4554617c
```



如果不自定义toString方法，会默认调用Object类toString()方法

Object类toString()方法：

```
public String toString() {
	return this.getClass().getName() + "@" + Integer.toHexString(this.hashCode());
}
```

而hashCode是Object类中的native方法：

```
public native int hashCode();
```



### JDK 8 Hotspot中，hashCode实现

https://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/a3f01f9da231/src/share/vm/runtime/synchronizer.cpp

关键代码get_next_hash()方法定义：

```
// hashCode() generation :
//
// Possibilities:
// * MD5Digest of {obj,stwRandom}
// * CRC32 of {obj,stwRandom} or any linear-feedback shift register function.
// * A DES- or AES-style SBox[] mechanism
// * One of the Phi-based schemes, such as:
//   2654435761 = 2^32 * Phi (golden ratio)
//   HashCodeValue = ((uintptr_t(obj) >> 3) * 2654435761) ^ GVars.stwRandom ;
// * A variation of Marsaglia's shift-xor RNG scheme.
// * (obj ^ stwRandom) is appealing, but can result
//   in undesirable regularity in the hashCode values of adjacent objects
//   (objects allocated back-to-back, in particular).  This could potentially
//   result in hashtable collisions and reduced hashtable efficiency.
//   There are simple ways to "diffuse" the middle address bits over the
//   generated hashCode values:
//

static inline intptr_t get_next_hash(Thread * Self, oop obj) {
  intptr_t value = 0 ;
  if (hashCode == 0) {
     // This form uses an unguarded global Park-Miller RNG,
     // so it's possible for two threads to race and generate the same RNG.
     // On MP system we'll have lots of RW access to a global, so the
     // mechanism induces lots of coherency traffic.
     value = os::random() ;
  } else
  if (hashCode == 1) {
     // This variation has the property of being stable (idempotent)
     // between STW operations.  This can be useful in some of the 1-0
     // synchronization schemes.
     intptr_t addrBits = cast_from_oop<intptr_t>(obj) >> 3 ;
     value = addrBits ^ (addrBits >> 5) ^ GVars.stwRandom ;
  } else
  if (hashCode == 2) {
     value = 1 ;            // for sensitivity testing
  } else
  if (hashCode == 3) {
     value = ++GVars.hcSequence ;
  } else
  if (hashCode == 4) {
     value = cast_from_oop<intptr_t>(obj) ;
  } else {
     // Marsaglia's xor-shift scheme with thread-specific state
     // This is probably the best overall implementation -- we'll
     // likely make this the default in future releases.
     unsigned t = Self->_hashStateX ;
     t ^= (t << 11) ;
     Self->_hashStateX = Self->_hashStateY ;
     Self->_hashStateY = Self->_hashStateZ ;
     Self->_hashStateZ = Self->_hashStateW ;
     unsigned v = Self->_hashStateW ;
     v = (v ^ (v >> 19)) ^ (t ^ (t >> 8)) ;
     Self->_hashStateW = v ;
     value = v ;
  }

  value &= markOopDesc::hash_mask;
  if (value == 0) value = 0xBAD ;
  assert (value != markOopDesc::no_hash, "invariant") ;
  TEVENT (hashCode: GENERATE) ;
  return value;
}
```



由全局变量 `hashCode`取值，对应不同的生成策略

它定义在：

https://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/a3f01f9da231/src/share/vm/runtime/globals.hpp

```
product(intx, hashCode, 5,                                                \
          "(Unstable) select hashCode generation algorithm") 
```

默认为5，非稳定参数（Unstable)可以通过-XX启动参数指定。

查看系统JVM参数设定：

```
$ java -XX:+PrintFlagsFinal -version | grep hashCode
     intx hashCode                                  = 5                                   {product}
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
```



尝试设置VM参数：

```
-XX:hashCode=2
```

再次运行得到结果：

```
testing.Main@1
testing.Main@1
```



### 当重写了对象的hashCode方法时

如果重写了hashCode()方法，再打印对象内存地址，使用`System.identityHashCode()`方法：

```
public class Main {

    public static void main(String[] args) {
        Main m1 = new Main();
        Main m2 = new Main();

        System.out.println(m1);	// 5
        System.out.println(m2); // 5

        System.out.println(System.identityHashCode(m1)); // 对象内存地址 460141958
        System.out.println(System.identityHashCode(m2)); // 对象内存地址 1163157884
    }

    @Override
    public int hashCode() {
        return 5;
    }
}
```

当然，如果指定 `-XX:hashCode=2`输出结果还是为：

```
testing.Main@5
testing.Main@5
1
1
```



### 总结：

是否指定了VM参数

如果没有重写hashCode()方法，直接打印对象

重写了hashCode()方法，使用System.identityHashCode()打印内存地址



---

VM参数hashCode取值，参考：

https://juejin.cn/post/6971946031764209678
