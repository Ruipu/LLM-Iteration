# Spring Boot Learning Notes

---

## 1. RESTful API

REST (Representational State Transfer) is an architectural style for designing networked APIs. A RESTful API follows these 6 constraints:

1. **Client-Server** — UI and data storage are separated
2. **Stateless** — Each request contains all information needed; the server stores no session state
3. **Cacheable** — Responses must define whether they can be cached
4. **Uniform Interface** — Consistent conventions using URLs and HTTP methods
5. **Layered System** — Client doesn't need to know what's between it and the final server
6. **Code on Demand** *(optional)* — Server can send executable code to the client

### Cacheable
Caching means storing a copy of a response so you don't have to ask for it again.

```http
Cache-Control: max-age=3600        # cache for 1 hour
Cache-Control: no-store            # never cache
ETag: "abc123"                     # fingerprint for conditional requests
```

---

## 2. HTTP Requests in Spring Boot

### Two Sides of HTTP in Spring Boot

**Side 1 — RECEIVING HTTP Requests (@RestController)**
```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        return userService.update(id, user);
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

**Side 2 — SENDING HTTP Requests (Calling Other APIs)**
```java
// FeignClient (most popular in Microservices)
@FeignClient(name = "payment-service", url = "https://payment-api.com")
public interface PaymentClient {
    @PostMapping("/charge")
    PaymentResponse charge(@RequestBody PaymentRequest request);
}
```

### Project Structure
```
src/main/java/com/yourapp/
├── controller/     ← RECEIVE requests
├── service/        ← Business logic
├── client/         ← SEND requests to other APIs
└── repository/     ← Database access
```

---

## 3. Threads — How to Create

### Approach 1: Extending `Thread` Class
```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Hello from MyThread!");
    }
}
new MyThread().start();
```
> ⚠️ Cannot extend any other class if you extend Thread!

### Approach 2: Implementing `Runnable` Interface ✅
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Hello from Runnable!");
    }
}
new Thread(new MyRunnable()).start();
```
> ✅ Your class can still extend another parent class!

### Approach 3: Lambda ✅
```java
new Thread(() -> System.out.println("Hello from Lambda!")).start();
```

### Approach 4: `Callable` + `Future`
```java
Callable<String> task = () -> "Result from thread!";
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<String> future = executor.submit(task);
String result = future.get();
executor.shutdown();
```

### Approach 5: ExecutorService — Thread Pool ✅✅
```java
ExecutorService executor = Executors.newFixedThreadPool(3);
for (int i = 1; i <= 5; i++) {
    int taskNum = i;
    executor.submit(() -> System.out.println("Task " + taskNum));
}
executor.shutdown();
```

### Approach 6: CompletableFuture ✅✅
```java
CompletableFuture.supplyAsync(() -> "Hello!")
    .thenApply(String::toUpperCase)
    .thenAccept(System.out::println)
    .join();
```

### Summary Table
| Approach | Return Value? | Best For |
|----------|---------------|----------|
| `extends Thread` | ❌ | Learning only |
| `implements Runnable` | ❌ | When need to extend other class |
| Lambda | ❌ | Short simple tasks |
| `Callable + Future` | ✅ | Need result back |
| `ExecutorService` | ✅ | Production, many tasks |
| `CompletableFuture` | ✅ | Modern async, chaining tasks |

---

## 4. Runnable vs Extends Thread

```
Runnable  =  The TASK (what to do)
Thread    =  The WORKER (who does it)

extends Thread   =  Thread IS the task (merged together)
implements Runnable =  Thread RECEIVES the task (separated)
```

> The ONLY real reason to use Runnable — so your class can still extend another parent class. You still always need `new Thread(runnable)` to launch it.

---

## 5. Thread Information

```java
Thread t = Thread.currentThread();
t.getName()       // name of thread
t.getId()         // unique ID number
t.getPriority()   // priority (1-10)
t.getState()      // current state
t.isDaemon()      // is it a background thread?
```

### Thread States
```
NEW           →  Thread created but not started
RUNNABLE      →  Thread is running or ready to run
BLOCKED       →  Waiting to get a lock
WAITING       →  Waiting indefinitely for another thread
TIMED_WAITING →  Waiting with a timeout
TERMINATED    →  Thread finished
```

---

## 6. Callable — Minimum Required Code

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

### Without Lambda — Anonymous Class
```java
Callable<String> cl = new Callable<String>() {
    @Override
    public String call() throws Exception {
        return "This is callable!";
    }
};
```

---

## 7. `throw` vs `throws`

| | `throw` | `throws` |
|---|---------|---------|
| What it does | **Actually fires** the exception | **Warns** caller exception might happen |
| Execution | **STOPS immediately** | Code runs normally |
| Where used | Inside method body | On method signature |

```
throws  =  Warning sign — "be careful, this might fail!"
throw   =  Road BLOCKED — "STOP! Cannot go further!"
```

---

## 8. `Future` Interface

```java
public interface Future<V> {
    V get();               // pick up result (waits until ready)
    boolean isDone();      // is the task finished yet?
    boolean cancel(...);   // cancel the task
}
```

```
executor.submit(task)  →  Starts thread, returns Future immediately
future.get()           →  YOU decide when to collect the result
```

---

## 9. `synchronized` and `Lock`

### synchronized
```java
public synchronized void withdraw(int amount) {
    if (balance >= amount) {
        balance -= amount;
    }
}
```

### ReentrantLock
```java
Lock lock = new ReentrantLock();

public void withdraw(int amount) {
    lock.lock();
    try {
        if (balance >= amount) {
            balance -= amount;
        }
    } finally {
        lock.unlock(); // always in finally!
    }
}
```

### tryLock()
```java
boolean acquiredLock = lock.tryLock();
if (acquiredLock) {
    try { /* do work */ }
    finally { lock.unlock(); }
} else {
    System.out.println("Could not get lock! Try again later.");
}
```

```
synchronized  →  wait FOREVER for lock
tryLock()     →  try to get lock, if busy → move on!
```

---

## 10. `wait()`, `notify()`, `notifyAll()`, `join()`, `sleep()`

### wait() and notify() — Producer/Consumer
```java
public synchronized void consume() throws Exception {
    while (!foodReady) {
        wait(); // releases lock, pauses here
    }
    System.out.println("Eating!");
}

public synchronized void produce() throws Exception {
    Thread.sleep(2000);
    foodReady = true;
    notify(); // wake up one waiting thread
}
```

### join() — Wait For Thread To Finish
```java
cookThread.start();
cookThread.join(); // main waits here until cookThread finishes!
System.out.println("Serving food!"); // runs AFTER cook done
```

### sleep() vs wait()
```
sleep()  →  pauses thread but KEEPS the lock!
wait()   →  pauses thread and RELEASES the lock!
```

### Summary Table
| Method | Releases Lock? | Who Wakes It? | Where Used |
|--------|---------------|---------------|------------|
| `wait()` | ✅ Yes | `notify()` / `notifyAll()` | inside `synchronized` |
| `notify()` | ❌ No | — | inside `synchronized` |
| `notifyAll()` | ❌ No | — | inside `synchronized` |
| `sleep()` | ❌ No | automatically after time | anywhere |
| `join()` | ❌ No | target thread finishing | anywhere |

---

## 11. Thread Lifecycle — Interview Answer

A thread in Java goes through six states during its lifecycle. It starts in the **New** state when a `Thread` object is created but `start()` has not been called yet. Once `start()` is called, the thread moves to the **Runnable** state, meaning it is either running or ready to run and waiting for the CPU to schedule it. When a thread calls `sleep()` or `join()` with a timeout, it enters the **Timed Waiting** state. When a thread calls `wait()` without a timeout, it enters the **Waiting** state until another thread calls `notify()` or `notifyAll()`. If a thread tries to enter a `synchronized` block already locked by another thread, it enters the **Blocked** state. Finally, once the thread finishes executing its `run()` method, it moves to the **Terminated** state and cannot be restarted.

---

## 12. Race Condition — How to Handle

### Solution 1: `synchronized`
```java
public synchronized void withdraw(int amount) {
    if (balance >= amount) balance -= amount;
}
```

### Solution 2: `ReentrantLock`
```java
lock.lock();
try { if (balance >= amount) balance -= amount; }
finally { lock.unlock(); }
```

### Solution 3: Atomic Classes
```java
AtomicInteger balance = new AtomicInteger(1000);
balance.addAndGet(-amount); // atomic, no lock needed!
```

### Solution 4: `volatile` + Double-Checked Locking Singleton
```java
private static volatile Singleton instance;

public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton();
            }
        }
    }
    return instance;
}
```

### Interview Answer
`volatile` is combined with the **Double-Checked Locking Singleton** pattern, where we check if the instance is null twice — once without locking for performance, and once inside a `synchronized` block to prevent duplicate creation. `volatile` is necessary here because without it, the CPU can reorder object creation steps and cause another thread to receive an uninitialized object. Together they ensure only one instance is created safely across all threads.

---

## 13. `equals()`, `hashCode()`, and HashMap

### `==` vs `equals()`
```java
String a = new String("Hello");
String b = new String("Hello");

System.out.println(a == b);       // false — different memory address
System.out.println(a.equals(b));  // true  — same content
```

| | `==` | `equals()` |
|---|---|---|
| Compares | Memory address | Actual content |
| Use for | Primitives | Strings, Objects |

### Why Override Both `hashCode()` and `equals()`?

```java
class Dog {
    String name;

    @Override
    public int hashCode() {
        return name.hashCode(); // same name → same hashCode!
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Dog)) return false;
        Dog other = (Dog) obj;
        return this.name.equals(other.name);
    }
}
```

```
hashCode()  →  finds the RIGHT bucket   (fast lookup)
equals()    →  finds the RIGHT key      (correct match)

Always override BOTH together — never just one!
```

### Interview Answer
In Java, `hashCode()` returns an integer used by `HashMap` to determine which bucket to store an object in. However, two different objects can sometimes produce the same hash code, which is called a **hash collision**. When a collision occurs, multiple keys end up in the same bucket, so `HashMap` then calls `equals()` to distinguish between them. This is why overriding `equals()` is critical — by default it compares memory addresses, so even when `HashMap` reaches the correct bucket after a collision, it will fail to match the key and `get()` returns null. By overriding `equals()` to compare actual content, `HashMap` can correctly identify the right key within the bucket.

---

## 14. Optional Class

```java
public Optional<User> findUser(int id) {
    if (id == 1) return Optional.of(new User("John"));
    return Optional.empty();
}

// Common methods
result.isPresent();                          // check if value exists
result.ifPresent(u -> System.out.println(u)); // run only if present
result.orElse(new User("Guest"));            // default if empty
result.orElseThrow(() -> new RuntimeException("Not found!")); // throw if empty
```

### In Spring Boot
```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

public User getUser(String email) {
    return userRepository.findByEmail(email)
        .orElseThrow(() -> new RuntimeException("User not found!"));
}
```

---

## 15. Reflection in Java

Reflection is the ability of a program to inspect and manipulate its own structure at runtime — reading classes, fields, methods, and annotations without knowing them at compile time.

```java
// Load class at runtime
Class<?> clazz = Class.forName("Dog");
Object obj      = clazz.getDeclaredConstructor().newInstance();
Method method   = clazz.getMethod("bark");
method.invoke(obj);

// Access private field!
Field field = clazz.getDeclaredField("name");
field.setAccessible(true);
field.set(dogObject, "Rex");
```

### How Spring Uses Reflection
```java
// Developer writes:
@Autowired
UserService userService;

// Spring internally does via Reflection:
Field field = UserController.class.getDeclaredField("userService");
field.setAccessible(true);
field.set(controllerInstance, new UserService()); // inject!
```

```
Spring is built BEFORE you write your code
Spring has NO IDEA what classes you wrote
Spring uses Reflection to SCAN and DISCOVER at runtime
```

### Who Uses Reflection
```
Spring      →  reads @Autowired, @RestController
Hibernate   →  reads @Entity, @Column
JUnit       →  discovers @Test methods
Jackson     →  reads fields to convert object ↔ JSON
```

---

## 16. HTTP

HTTP (HyperText Transfer Protocol) is a request-response protocol defining how client and server communicate. Key components:

### HTTP Methods
| Method | Action |
|--------|--------|
| `GET` | Read |
| `POST` | Create |
| `PUT` | Replace |
| `PATCH` | Partial update |
| `DELETE` | Delete |

### Status Codes
```
200 OK              → success
201 Created         → resource created
400 Bad Request     → invalid input
401 Unauthorized    → not logged in
403 Forbidden       → no permission
404 Not Found       → resource missing
500 Internal Error  → server crashed
```

---

## 17. TCP 3-Way Handshake

```
CLIENT                     SERVER
  │                           │
  │  ── SYN ───────────────►  │   "I want to connect!"
  │                           │
  │  ◄─ SYN-ACK ───────────   │   "OK! I am ready, are you?"
  │                           │
  │  ── ACK ───────────────►  │   "Yes! Let's start!"
  │                           │
  │  ════ CONNECTION ESTABLISHED ════
```

```
TCP  =  Layer 4 (Transport Layer)  — reliable delivery
HTTP =  Layer 7 (Application Layer) — request/response format
```

### TCP vs UDP
| | TCP | UDP |
|---|---|---|
| Reliable | ✅ | ❌ |
| Handshake | ✅ | ❌ |
| Speed | Slower | Faster |
| Used for | HTTP, Email, DB | Video streaming, Gaming |

---

## 18. OSI Model Layers

```
Layer 7  →  Application   →  HTTP, HTTPS        ← Developer works here!
Layer 6  →  Presentation  →  SSL/TLS
Layer 5  →  Session       →  Session management
Layer 4  →  Transport     →  TCP, UDP
Layer 3  →  Network       →  IP, routing
Layer 2  →  Data Link     →  MAC, Ethernet
Layer 1  →  Physical      →  cables, wifi
```

> As a Junior SDE, you work 99% at **Layer 7 — Application Layer!**

---

## 19. Three-Tier Architecture

```
Three-tier architecture = how you organize your Spring Boot code!

┌─────────────────────────────┐
│      SPRING BOOT APP        │
│                             │
│  [ Controller ]  Layer 1    │  ← Presentation Layer
│       │                     │
│  [ Service   ]  Layer 2     │  ← Business Layer
│       │                     │
│  [ Repository]  Layer 3     │  ← Data Layer
│                             │
└─────────────────────────────┘
          │
          ▼
    [ MySQL Database ]          ← outside the 3 layers!
```

### Project Structure
```
net.javaguides.ems/
├── controller/
│   └── EmployeeController.java   ← Layer 1
├── service/
│   └── EmployeeServiceImpl.java  ← Layer 2
└── repository/
    └── EmployeeRepository.java   ← Layer 3
```

### Interview Answer
Three-tier architecture separates a Spring Boot application into Controller, Service, and Repository. The Controller receives HTTP requests and delegates to the Service. The Service contains all business logic, validation, and data transformation. The Repository handles all database communication. Each layer can evolve independently — changing the database only affects the Repository, changing business rules only affects the Service, and changing the API format only affects the Controller.

---

## 20. Annotations

```
Annotation     =  a sticky note on your code — does nothing by itself!
Compiler       =  reads @Override, @SuppressWarnings
Spring Boot    =  reads @RestController, @Autowired via Reflection
IntelliJ       =  reads annotations for syntax highlighting and warnings
```

### Meta Annotation
```java
// Meta Annotation = annotation ABOUT another annotation
@Target(ElementType.TYPE)           // ← meta annotation
@Retention(RetentionPolicy.RUNTIME) // ← meta annotation
@Controller                         // ← meta annotation
@ResponseBody                       // ← meta annotation
public @interface RestController { }
```

> Meta = Greek word meaning "about" — nothing to do with Meta (Facebook) company!

---

*Generated on 2026-06-08*
