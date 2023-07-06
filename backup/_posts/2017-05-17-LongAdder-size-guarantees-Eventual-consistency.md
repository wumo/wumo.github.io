---
title: "LongAdder sum() last update thread will get correct sum"
tags: ["concurrency"]
---

The javadoc for `LongAdder.sum()` is:
```java
   /** Returns the current sum.  The returned value is <em>NOT</em> an
     * atomic snapshot; invocation in the absence of concurrent
     * updates returns an accurate result, but concurrent updates that
     * occur while the sum is being calculated might not be
     * incorporated.
     *
     * @return the sum
     */
     public long sum() {
        Cell[] as = cells; Cell a;
        long sum = base;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }
```
如果限制应用场景，仅有有限个线程进行有限次更新，那么最后一次更新（全局时间）的线程调用`LongAdder.sum()`将获得正确的sum。

这个例子用来熟悉`volatile`和`happen-before`比较好。
<!--more-->