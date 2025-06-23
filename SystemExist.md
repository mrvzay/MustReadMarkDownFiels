# **What Happens Internally When `System.exit()` is Called in Java?**

When `System.exit()` is invoked, the Java Virtual Machine (JVM) initiates a **controlled shutdown sequence**. Here's a detailed breakdown of what happens internally:

---

## **🔹 1. `System.exit()` Execution Flow**
### **Method Signature**
```java
public static void exit(int status)
```
- **`status`**: 
  - `0` → Normal termination.
  - Non-zero → Abnormal termination (e.g., error).

### **Internal Steps**
1. **Security Check**  
   - The `SecurityManager` (if installed) verifies if the caller has permission to shut down the JVM (`checkExit(status)`).  
   - Throws `SecurityException` if denied.

2. **Invokes `Runtime.getRuntime().exit(status)`**  
   - Delegates to the `Runtime` class, which handles JVM lifecycle operations.

3. **JVM Shutdown Sequence Begins**  
   - The JVM starts its **shutdown hooks** (if registered).
   - Runs **finalizers** (if enabled, though deprecated since Java 9).
   - Terminates all **active threads** (non-daemon threads are stopped forcibly).

4. **Final JVM Termination**  
   - Calls native OS-level cleanup.
   - Returns the exit status to the operating system.

---

## **🔹 2. Key Components Involved**
### **A. Shutdown Hooks**
- Registered via `Runtime.addShutdownHook(Thread hook)`.  
- Executed **after** `System.exit()` but **before** JVM termination.  
- Used for cleanup (e.g., closing DB connections, temp files).  
- Example:
  ```java
  Runtime.getRuntime().addShutdownHook(new Thread(() -> {
      System.out.println("Cleanup before JVM exits!");
  }));
  ```

### **B. Finalizers (Deprecated in Java 9+)**
- Historically, objects with `finalize()` methods were processed during shutdown.  
- **Avoid using** (replaced by `Cleaner` API in modern Java).

### **C. Daemon vs. Non-Daemon Threads**
- **Non-daemon threads** keep the JVM alive.  
- `System.exit()` **forcibly terminates** them if they don’t exit gracefully.

---

## **🔹 3. OS-Level Actions**
- The JVM process sends an **exit signal** (e.g., `SIGTERM` on Unix/Linux).  
- OS releases allocated resources (memory, file handles).  
- Exit status is propagated to parent processes (e.g., shell scripts can check `$?`).

---

## **🔹 4. Edge Cases & Pitfalls**
### **A. `System.exit()` in `try-finally`**
```java
try {
    System.exit(0); // Finally block WON'T execute!
} finally {
    System.out.println("This never runs!");
}
```
- **Finally blocks are skipped** because `exit()` halts the JVM immediately after hooks.

### **B. SecurityManager Restrictions**
- If a `SecurityManager` denies `exit()`, the JVM throws:
  ```java
  throw new SecurityException("Exit denied!");
  ```

### **C. Exit in Shutdown Hooks**
- Calling `System.exit()` inside a shutdown hook can cause **deadlocks**.

---

## **🔹 5. Alternatives to `System.exit()`**
| Approach | Use Case |
|----------|----------|
| **Return from `main()`** | Graceful exit (preferred). |
| **Throw an exception** | Let uncaught exceptions terminate the program. |
| **`Runtime.halt(status)`** | Force-kill JVM (skips shutdown hooks). |

---

## **🔹 6. Example: JVM Shutdown Sequence**
```java
public class ExitDemo {
    public static void main(String[] args) {
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("Shutdown Hook: Cleaning up...");
        }));

        System.out.println("Starting...");
        System.exit(0); // Triggers shutdown sequence
        System.out.println("This line won't print!");
    }
}
```
**Output:**
```
Starting...
Shutdown Hook: Cleaning up...
```

---

## **🔹 7. Key Takeaways**
1. **`System.exit()` is a JVM killer** – Avoid unless necessary.  
2. **Shutdown hooks** allow cleanup but must be non-blocking.  
3. **Finally blocks don’t run** after `exit()`.  
4. **Modern Java favors `Runtime.halt()`** for emergencies (bypasses hooks).  

---

### **Final Thought**
Use `System.exit()` sparingly—most applications should terminate naturally by returning from `main()` or throwing exceptions. Need a deep dive into **JVM process lifecycle**? 🚀

--

Yes! When `System.exit(int status)` is called in Java, several **internal steps** take place that lead to the **termination of the JVM process**. Here's what happens under the hood:

---

## 🔧 **Internal Flow of `System.exit()`**

### 🔹 1. **Calls `Runtime.getRuntime().exit(status)`**

```java
System.exit(0); 
// Internally calls:
Runtime.getRuntime().exit(0);
```

### 🔹 2. **`Runtime.exit(int status)` Delegates to `Shutdown` Class**

* This method does some preparation for the shutdown process.
* Internally, it calls `Shutdown.exit(status)` (a private class in the JDK).

### 🔹 3. **Shutdown Hooks Are Run**

Before exiting, the JVM checks for any **registered shutdown hooks** (via `Runtime.addShutdownHook()`):

* These hooks are executed in an **unspecified order**
* Useful for cleanup: closing resources, saving state, etc.

```java
Runtime.getRuntime().addShutdownHook(new Thread(() -> {
    System.out.println("Shutdown hook executed");
}));
```

### 🔹 4. **Finalization**

* If `System.runFinalizersOnExit(true)` was called (not recommended), the JVM will run finalizers.
* This is generally skipped due to unpredictability.

### 🔹 5. **JVM Terminates**

* After hooks and finalizers, the JVM calls a native method:

  ```cpp
  JavaCalls::call_exit()
  ```
* JVM exits with the provided status code:

  * `0` = Normal termination
  * Non-zero = Abnormal termination (often used to signal error)

---

## ⚠️ **Important Notes**

* **All running threads are stopped abruptly**, except daemon threads.
* **Resources like sockets, files, DB connections** are not closed automatically unless managed via shutdown hooks.
* You should avoid calling `System.exit()` in long-running or web applications (like Spring Boot), as it kills the whole JVM.

---

## 🧪 Example:

```java
public class ExitDemo {
    public static void main(String[] args) {
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("Cleanup done before shutdown");
        }));

        System.out.println("Exiting now...");
        System.exit(0);
    }
}
```

### Output:

```
Exiting now...
Cleanup done before shutdown
```

---

## 📌 Summary

| Step | What Happens                       |
| ---- | ---------------------------------- |
| 1    | `System.exit()` → `Runtime.exit()` |
| 2    | Shutdown hooks executed            |
| 3    | Finalization (optional)            |
| 4    | JVM process terminates             |

---

Let me know if you want to explore **custom shutdown hooks** or **JVM termination in a Spring Boot app**.
