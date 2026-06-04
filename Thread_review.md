# Java Thread Review

---

## 1. How to Create Threads — All Approaches

### Approach 1: Extending `Thread` Class
```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread name: " + Thread.currentThread().getName());
        System.out.println("Hello from MyThread!");
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start(); // IMPORTANT: call start(), NOT run()
        System.out.println("Hello from Main Thread!");
    }
}
```

> ⚠️ Problem: Java only allows ONE parent class. If you extend Thread, you CANNOT extend anything else!

---

### Approach 2: Implementing `Runnable` Interface ✅
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Hello from Runnable!");
    }
}

public class Main {
    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable);
        thread.start();
    }
}
```

> ✅ Better — your class can still extend another parent class!

---

### Approach 3: Lambda (Modern, Clean) ✅
```java
public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("Hello from Lambda Thread!");
        });
        thread.start();
    }
}
```

> Lambda is just a SHORT way of writing Runnable — they are exactly the same thing!

---

### Approach 4: `Callable` + `Future` (When you need a Return Value)
```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) throws Exception {

        Callable<String> task = () -> {
            Thread.sleep(2000);
            return "Result from thread!";
        };

        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<String> future = executor.submit(task);

        System.out.println("Doing other work while thread runs...");

        String result = future.get(); // waits until thread finishes
        System.out.println("Got result: " + result);

        executor.shutdown();
    }
}
```

| | `Runnable` | `Callable<T>` |
|---|---|---|
| Return value | ❌ void | ✅ returns T |
| Throws exception | ❌ No | ✅ Yes |
| Used with | `Thread` | `ExecutorService` |

---

### Approach 5: `ExecutorService` — Thread Pool ✅✅
```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        for (int i = 1; i <= 5; i++) {
            int taskNumber = i;
            executor.submit(() -> {
                System.out.println("Task " + taskNumber +
                    " running on " + Thread.currentThread().getName());
            });
        }

        executor.shutdown();
    }
}
```

#### Types of Thread Pools:
```java
Executors.newFixedThreadPool(4);       // Fixed number of threads
Executors.newSingleThreadExecutor();   // Only 1 thread
Executors.newCachedThreadPool();       // Creates threads as needed
Executors.newScheduledThreadPool(2);   // For scheduling tasks
```

---

### Approach 6: `CompletableFuture` ✅✅ (Most Modern — Java 8+)
```java
import java.util.concurrent.CompletableFuture;

public class Main {
    public static void main(String[] args) throws Exception {

        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            return "Hello from async thread!";
        });

        future
            .thenApply(result -> result.toUpperCase())
            .thenAccept(result -> System.out.println(result))
            .join();
    }
}
```

---

### All Approaches Summary

| # | Approach | Return Value? | Best For |
|---|----------|---------------|----------|
| 1 | `extends Thread` | ❌ | Learning only |
| 2 | `implements Runnable` | ❌ | When need to extend other class |
| 3 | Lambda | ❌ | Short simple tasks |
| 4 | `Callable + Future` | ✅ | Need result back |
| 5 | `ExecutorService` | ✅ | Production, many tasks |
| 6 | `CompletableFuture` | ✅ | Modern async, chaining tasks |

---

## 2. Runnable vs Extends Thread

```
Runnable        =  The TASK (what to do)
Thread          =  The WORKER (who does it)

extends Thread  =  Thread IS the task       (merged together)
Runnable        =  Thread RECEIVES the task (separated)
```

> The ONLY real reason to use Runnable is so your class can still extend another parent class.
> You still always need `new Thread(runnable)` to actually launch it.

---

## 3. Thread Information

```java
Thread t = Thread.currentThread();

t.getName()      // name of thread
t.getId()        // unique ID number
t.getPriority()  // priority (1-10)
t.getState()     // current state (RUNNING, WAITING, etc.)
t.isDaemon()     // is it a background thread?
```

### Thread States:
```
NEW           →  Thread created but not started yet
RUNNABLE      →  Thread is running or ready to run
BLOCKED       →  Waiting to get a lock
WAITING       →  Waiting for another thread (no timeout)
TIMED_WAITING →  Waiting with a timeout (ex: Thread.sleep(1000))
TERMINATED    →  Thread finished its task
```

---

## 4. Callable — Minimum Required Code

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {

        // 1. Define the task
        Callable<String> task = () -> "Hello from Callable!";

        // 2. Create thread pool
        ExecutorService executor = Executors.newSingleThreadExecutor();

        // 3. Submit task → thread starts running
        Future<String> future = executor.submit(task);

        // 4. Get result
        try {
            String result = future.get();
            System.out.println(result);
        } catch (Exception e) {
            System.out.println("Something went wrong: " + e.getMessage());
        }

        // 5. Shutdown
        executor.shutdown();
    }
}
```

### The 5 Necessary Components:
```
1. Callable<T> task    →  define the task (with return type)
2. ExecutorService     →  create the thread pool
3. executor.submit()   →  submit task, thread starts running
4. future.get()        →  get the returned result
5. executor.shutdown() →  close the pool when done
```

---

## 5. Without Lambda — Anonymous Class

```java
Callable<String> cl = new Callable<String>() {
    @Override
    public String call() throws Exception {
        return "This is callable!";
    }
};
```

> Lambda is just a shortcut for writing Anonymous Class — both do exactly the same thing!

---

## 6. `throw` vs `throws`

| | `throw` | `throws` |
|---|---------|---------|
| What it does | **Actually fires** the exception | **Warns** caller that exception might happen |
| Execution | **STOPS immediately** | Code runs normally |
| Where used | Inside method body | On method signature |
| Example | `throw new Exception("error")` | `public void method() throws Exception` |

```
throws  =  Warning sign on a road — "be careful!"
           You can still drive, just be aware

throw   =  Road BLOCKED — "STOP! Cannot go further!"
           You MUST stop, no way through
```

---

## 7. `Future` Interface

```java
public interface Future<V> {
    V get();                  // pick up result (waits until ready)
    V get(timeout, unit);     // pick up result but only wait limited time
    boolean isDone();         // is the task finished yet?
    boolean isCancelled();    // was the task cancelled?
    boolean cancel(...);      // cancel the task
}
```

```
executor.submit(task)   →  Starts thread, returns Future immediately
Future<String>          →  Holds the promise of a future result
future.get()            →  YOU decide when to collect the result
```

---

## 8. `synchronized` and `Lock`

### Problem Without `synchronized`:
```java
// Two threads withdrawing at same time → NEGATIVE BALANCE!
Thread-A is withdrawing 800
Thread-B is withdrawing 800   ← both passed the if check!
Thread-A done! Balance: 200
Thread-B done! Balance: -600  ← WRONG!
```

### Fix With `synchronized`:
```java
public class BankAccount {
    int balance = 1000;

    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            balance -= amount;
            System.out.println(Thread.currentThread().getName() +
                " done! Balance: " + balance);
        } else {
            System.out.println(Thread.currentThread().getName() +
                " failed! Not enough balance: " + balance);
        }
    }
}
```

### Fix With `Lock`:
```java
import java.util.concurrent.locks.*;

public class BankAccount {
    int balance = 1000;
    Lock lock = new ReentrantLock();

    public void withdraw(int amount) {
        lock.lock(); // manually lock
        try {
            if (balance >= amount) {
                balance -= amount;
                System.out.println("Done! Balance: " + balance);
            }
        } finally {
            lock.unlock(); // always unlock in finally!
        }
    }
}
```

### `tryLock()` — Don't Wait Forever:
```java
boolean acquiredLock = lock.tryLock();

if (acquiredLock) {
    try {
        // do the work
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("Could not get lock! Try again later.");
}
```

```
synchronized  →  wait FOREVER for lock, no choice
tryLock()     →  try to get lock, if busy → move on!
```

---

## 9. `wait()`, `notify()`, `notifyAll()`, `join()`, `sleep()`

### `wait()` and `notify()` — Producer / Consumer
```java
public class Kitchen {
    boolean foodReady = false;

    public synchronized void consume() throws Exception {
        while (!foodReady) {
            System.out.println("Consumer: No food yet... waiting!");
            wait(); // releases lock, pauses here
        }
        System.out.println("Consumer: Food ready! Eating now!");
    }

    public synchronized void produce() throws Exception {
        System.out.println("Producer: Cooking food...");
        Thread.sleep(2000);
        foodReady = true;
        System.out.println("Producer: Food is ready!");
        notify(); // wake up the waiting consumer!
    }
}
```

### `notifyAll()` — Wake Up All Waiting Threads
```java
public synchronized void produce() throws Exception {
    Thread.sleep(2000);
    foodReady = true;
    System.out.println("Producer: Food ready for everyone!");
    notifyAll(); // wake up ALL waiting threads!
}
```

### `join()` — Wait For Thread To Finish
```java
Thread cookThread = new Thread(() -> {
    System.out.println("Chef: Cooking...");
    try { Thread.sleep(3000); } catch (Exception e) {}
    System.out.println("Chef: Food is ready!");
});

cookThread.start();
cookThread.join(); // main thread WAITS here until cookThread finishes!
System.out.println("Waiter: Serving food!"); // runs AFTER chef done
```

### `sleep()` — Keeps The Lock!
```java
synchronized (lock) {
    System.out.println("T1: Sleeping but keeping the lock!");
    Thread.sleep(3000); // sleeps but KEEPS the lock!
}
```

---

## 10. Full Summary Table

| Method | Releases Lock? | Who Wakes It? | Where Used |
|--------|---------------|---------------|------------|
| `wait()` | ✅ Yes | `notify()` / `notifyAll()` | inside `synchronized` |
| `notify()` | ❌ No | — | inside `synchronized` |
| `notifyAll()` | ❌ No | — | inside `synchronized` |
| `sleep()` | ❌ No | automatically after time | anywhere |
| `join()` | ❌ No | target thread finishing | anywhere |
| `tryLock()` | ❌ No | — | with `Lock` interface |

---

*Generated on 2026-06-04*
