# Decision C07 - lazy singleton in Java

A dev implements a lazy singleton with double-checked locking:

```java
private static Config instance;
Config get() {
  if (instance == null) {
    synchronized (Config.class) {
      if (instance == null) instance = new Config(); // heavy init
    }
  }
  return instance;
}
```

`instance` is a plain (non-`volatile`) static field. **Question:** is this
thread-safe on the JVM? Commit and name the single biggest risk.
