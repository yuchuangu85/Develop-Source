# Volatile Vs Atomic Vs Synchronized

## What is the difference between atomic / volatile / synchronized?

You are specifically asking about how they internally work, so here you are:

**No synchronization**

```java
private int counter;

public int getNextUniqueIndex() {
  return counter++; 
}
```

It basically reads value from memory, increments it and puts back to memory. This works in single thread but nowadays, in the era of multi-core, multi-CPU, multi-level caches it won't work correctly. First of all it introduces race condition (several threads can read the value at the same time), but also visibility problems. The value might only be stored in "local" CPU memory (some cache) and not be visible for other CPUs/cores (and thus - threads). This is why many refer to local copy of a variable in a thread. It is very unsafe. Consider this popular but broken thread-stopping code:

```java
private boolean stopped;

public void run() {
    while(!stopped) {
        //do some work
    }
}

public void pleaseStop() {
    stopped = true;
}
```

Add volatile to stopped variable and it works fine - if any other thread modifies stopped variable via pleaseStop() method, you are guaranteed to see that change immediately in working thread's while(!stopped) loop. BTW this is not a good way to interrupt a thread either, see: How to stop a thread that is running forever without any use and Stopping a specific java thread.

**AtomicInteger**

```java
private AtomicInteger counter = new AtomicInteger();

public int getNextUniqueIndex() {
  return counter.getAndIncrement();
}
```

The AtomicInteger class uses CAS (compare-and-swap) low-level CPU operations (no synchronization needed!) They allow you to modify a particular variable only if the present value is equal to something else (and is returned successfully). So when you execute getAndIncrement() it actually runs in a loop (simplified real implementation):

```java
int current;
do {
  current = get();
} while(!compareAndSet(current, current + 1));
```

So basically: read; try to store incremented value; if not successful (the value is no longer equal to current), read and try again. The compareAndSet() is implemented in native code (assembly).

**volatile without synchronization**

```java
private volatile int counter;

public int getNextUniqueIndex() {
  return counter++; 
}
```

This code is not correct. It fixes the visibility issue (volatile makes sure other threads can see change made to counter) but still has a race condition. This has been explained multiple times: pre/post-incrementation is not atomic.

The only side effect of volatile is "flushing" caches so that all other parties see the freshest version of the data. This is too strict in most situations; that is why volatile is not default.

```java
volatile without synchronization (2)
volatile int i = 0;
void incIBy5() {
  i += 5;
}
```

The same problem as above, but even worse because i is not private. The race condition is still present. Why is it a problem? If, say, two threads run this code simultaneously, the output might be + 5 or + 10. However, you are guaranteed to see the change.

**Multiple independent synchronized**

```java
void incIBy5() {
  int temp;
  synchronized(i) { temp = i }
  synchronized(i) { i = temp + 5 }
}
```

Surprise, this code is incorrect as well. In fact, it is completely wrong. First of all you are synchronizing on i, which is about to be changed (moreover, i is a primitive, so I guess you are synchronizing on a temporary Integer created via autoboxing...) Completely flawed. You could also write:

```java
synchronized(new Object()) {
  //thread-safe, SRSLy?
}
```

No two threads can enter the same synchronized block with the same lock. In this case (and similarly in your code) the lock object changes upon every execution, so synchronized effectively has no effect.

Even if you have used a final variable (or this) for synchronization, the code is still incorrect. Two threads can first read i to temp synchronously (having the same value locally in temp), then the first assigns a new value to i (say, from 1 to 6) and the other one does the same thing (from 1 to 6).

The synchronization must span from reading to assigning a value. Your first synchronization has no effect (reading an int is atomic) and the second as well. In my opinion, these are the correct forms:

```java
void synchronized incIBy5() {
  i += 5 
}

void incIBy5() {
  synchronized(this) {
    i += 5 
  }
}

void incIBy5() {
  synchronized(this) {
    int temp = i;
    i = temp + 5;
  }
}
```

## Volatile Vs Atomic [duplicate]

The effect of the volatile keyword is approximately（大约） that each individual read or write operation on that variable is atomic.

Notably（值得注意的地方）, however, an operation that requires more than one read/write -- such as i++, which is equivalent to i = i + 1, which does one read and one write -- is not atomic, since another thread may write to i between the read and the write.

The Atomic classes, like AtomicInteger and AtomicReference, provide a wider variety（变种） of operations atomically, specifically including increment for AtomicInteger.