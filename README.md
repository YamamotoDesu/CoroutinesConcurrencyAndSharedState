# CoroutinesConcurrencyAndSharedState

SharedStateProblem
```kt
fun main() {
    runBlocking {
        var counter = 0
        withContext(Dispatchers.Default) {
            massiveRun { counter++ }
        }
        println("Counter = $counter")
    }
}

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100
    val k = 1000
    val time = measureTimeMillis {
        coroutineScope {
            repeat(n) {
                launch {
                    repeat(k) {
                        action()
                    }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")
    /*
    Completed 100000 actions in 29 ms
    Counter = 56504
     */
}
```

## AtomicVariables

```kt
fun main() {
    runBlocking {
        var counter = AtomicInteger(0)
        withContext(Dispatchers.Default) {
            massiveRunAtomic { counter.incrementAndGet() }
        }
        println("Counter = $counter")
    }
}

suspend fun massiveRunAtomic(action: suspend () -> Unit) {
    val n = 100
    val k = 1000
    val time = measureTimeMillis {
        coroutineScope {
            repeat(n) {
                launch {
                    repeat(k) {
                        action()
                    }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")
    /*
    Completed 100000 actions in 51 ms
    Counter = 100000
     */
}
```

## ThreadConfinement

```kt
fun main() {
    runBlocking {
        val counterContext = newSingleThreadContext("CounterContext")
        var counter = 0
//        withContext(Dispatchers.Default) {
//            massiveRunThreads {
//                withContext(counterContext) {
//                    counter++
//                }
//            }
//            /*
//            Completed 100000 actions in 2774 ms
//            Counter = 100000
//             */
//        }

        withContext(counterContext) {
            massiveRunThreads {
                counter++
            }
            /*
            Completed 100000 actions in 65 ms
            Counter = 100000
             */
        }
        println("Counter = $counter")
    }
}

suspend fun massiveRunThreads(action: suspend () -> Unit) {
    val n = 100
    val k = 1000
    val time = measureTimeMillis {
        coroutineScope {
            repeat(n) {
                launch {
                    repeat(k) {
                        action()
                    }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")
}
```

## MutualExclusionLocks
```kt
fun main() {
    runBlocking {
        val mutex = Mutex()
        var counter = 0
        withContext(Dispatchers.Default) {
            massiveRunMutex {
                mutex.withLock {
                    counter++
                }
            }
        }
        println("Counter = $counter")
    }
}

```
