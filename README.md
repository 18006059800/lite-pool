Table of Contents
=================

   * [1. Install](#1-install)
      * [1.1. Brief introduction](#11-brief-introduction)
      * [1.2. Requirements](#12-requirements)
      * [1.3. Install from source code](#13-install-from-source-code)
   * [2. Simple usage](#2-simple-usage)
      * [2.1. PoolBuilder](#21-poolbuilder)
      * [2.2. Usage](#22-usage)
   * [3. Benchmark](#3-benchmark)


# 1. Install  
## 1.1. Brief introduction  

A lite fast object pool written by Java.  

## 1.2. Requirements  
jdk 1.8+  
maven-3.2.3+  

## 1.3. Install from source code  

``` 
    $mvn clean install -Dmaven.test.skip=true
```  

# 2. Simple usage  
## 2.1. PoolBuilder  

| **config** | **default value**  |  **details**                                                         |
| ---------- | ------------------ | ---------------------------------------------------------------------|
| minimum    | 0                  |  minimum allowed objects in pool                                     |
| maximum    | 16                 |  maximum allowed objects in pool                                     |
| tti        | 15 minutes         |  time to idle, optional maximum pool objects' idle time, unit ms     |
| ttl        | 60 minutes         |  time to live, optional maximum pool objects' life time, unit ms     |
| tenancy    | 1  minutes         |  optional leak detection timeout, unit ms, (**MUST** >= `interval`)  |
| timeout    | 8  seconds         |  default acquire timeout, unit ms                                    |
| interval   | 15 seconds         |  default house keeping scheduler's interval, unit ms                 |
| local      | true               |  use `ThreadAllocator` as L1 cache or not                            |
| verbose    | false              |  print log or not                                                    |
| fifo       | false              |  pool allocation order, `false` has better performance               |
| allocator  | DefaultAllocator   |  pool allocator, can be customized by extending `AbstractAllocator`  |
| supplier   |                    |  required callback for creating pool objects                         |
| consumer   |                    |  optional callback for destroying pool objects                       |
| validator  |                    |  optional callback for validating pool objects                       |
  

## 2.2. Usage  

```java  
    public class YourPoolObject {
    }
    
    Pool<YourPoolObject> pool = new PoolBuilder<YourPoolObject>()
                    .local(true) // using thread local
                    .supplier(() -> new YourPoolObject())
                    .interval(interval)
                    .minimum(minimum)
                    .maximum(maximum)
                    .timeout(timeout)
                    .ttl(ttl)
                    .tti(tti)
                    .verbose(true)
                    ...
                    .build("object pool");
    pool.start();
    try {
        for(int i = 0; i < 1000; i++) {
            YourPoolObject object = null;
            try {
                object = pool.acquire();
                if (object != null) {
                    // your code goes here. 
                }
            } finally {
                if (object != null) pool.release(object);
            }
        }
    } finally {
        pool.stop();
    }
```

# 3. Benchmark

Test env:  

```xml  
    OS : Windows 7 Home(64bit)
    CPU: Intel(R) Core(TM) i3-4710 CPU @ 3.70GHz  3.70GHz
    RAM: 8.00 GB
    JDK: java version "1.8.0_151"

```

Test case:  
  
```java  
    TestObject object = pool.acquire();
    if (object != null) pool.release(object);
```
  
(unit: ops/ms)  
  
Case 1: 10 minimum, 10 maximum, 1 thread polling from pool  
Result:  

```java  
Benchmark                  Mode  Cnt     Score     Error   Units
ObjectPoolBenchmark.test  thrpt   10  9491.206 ± 108.402  ops/ms
```
  
Case 2: 10 minimum, 10 maximum, 2 thread polling from pool  
Result:  

```java  
Benchmark                  Mode  Cnt     Score     Error   Units
ObjectPoolBenchmark.test  thrpt   10  8327.384 ± 368.566  ops/ms
```
  
Case 3: 10 minimum, 10 maximum, 5 thread polling from pool  
Result:  

```java  
Benchmark                  Mode  Cnt     Score    Error   Units
ObjectPoolBenchmark.test  thrpt   10  8150.062 ± 30.242  ops/ms
```
  
Case 4: 10 minimum, 10 maximum, 10 thread polling from pool  
Result:  

```java  
Benchmark                  Mode  Cnt     Score     Error   Units
ObjectPoolBenchmark.test  thrpt   10  9486.971 ± 200.483  ops/ms
```
  
Case 5: 10 minimum, 10 maximum, 20 thread polling from pool  
Result:  

```java  
Benchmark                  Mode  Cnt     Score     Error   Units
ObjectPoolBenchmark.test  thrpt   10  7691.710 ± 530.855  ops/ms
```
  
Case 6: 10 minimum, 10 maximum, 50 thread polling from pool  
Result:  

```java  
Benchmark                  Mode  Cnt     Score     Error   Units
ObjectPoolBenchmark.test  thrpt   10  7858.702 ± 584.944  ops/ms
```
