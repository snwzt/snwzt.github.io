# Sync Package in Go
Go offers various tools to manage concurrent execution. While channels are the high-level means of communication and synchronization, the sync package provides with low-level primitives to handle synchronization effectively. It also includes atomic sub-package, which provides low-level atomic memory primitives useful for implementing synchronization algorithms. Mostly used components of the sync package include: WaitGroup, Mutex, RWMutex, Pool, Map, Cond, and Once. Values containing types defined in sync package should not be copied.

## WaitGroup
WaitGroup is used to wait for a collection of goroutines to finish. It tracks the number of goroutines and blocks until all goroutines have finished executing. Methods include:
- **Add(d int):** Increments the waiting counter by `d`.
- **Done():** Decrements the counter by one, indicating that a goroutine has completed its task.
- **Wait():** Blocks until the counter becomes zero.

```go
func 

func main() {
	var wg sync.WaitGroup

    wg.Add(1)
    go worker(wg *sync.WaitGroup) {
        defer wg.Done()

        fmt.Println("Worker starting")
        time.Sleep(5 * time.Second)
        fmt.Printf("Worker done")
    }(&wg)

	wg.Wait()
}
```

## Mutex
A Mutex is used to provide mutual exclusion, ensuring that only one goroutine can access a critical section of code at a time. Critical section of code is the section of code which is being accessed by multiple goroutines, i.e. it is a shared resource and without mutual exclusion lock, it will result in race conditions. Mutex is a form of pessimistic locking. Methods include:
- **Lock():** Acquires the lock. If lock already in use, calling goroutine is blocked till lock is released.
- **TryLock():** Attempts to acquire the lock and returns success status as bool.
- **Unlock():** Releases the lock.

```go
var (
	value int
	mutex sync.Mutex
	wg    sync.WaitGroup
)

func task1() {
	defer wg.Done()

	for i := 0; i < 3; i++ {
		mutex.Lock()
		value++
		time.Sleep(100 * time.Millisecond)
		fmt.Println("Task 1 updated value to:", value)
		mutex.Unlock()
	}
}

func task2() {
	defer wg.Done()

	for i := 0; i < 3; i++ {
		for {
			if mutex.TryLock() {
				break
			} else {
				fmt.Println("Task 2 couldn't acquire lock, trying again...")
				time.Sleep(500 * time.Millisecond)
			}
		}
		value += 2
		time.Sleep(100 * time.Millisecond)
		fmt.Println("Task 2 updated value to:", value)
		mutex.Unlock()
	}
}

func main() {
	wg.Add(2)

	go task1()
	go task2()

	wg.Wait()

	fmt.Println("Final value:", value)
}
```

Execution:
```
snwzt@cara ~> go run main.go
Task 2 updated value to: 2
Task 2 updated value to: 4
Task 2 couldn't acquire lock, trying again...
Task 1 updated value to: 5
Task 1 updated value to: 6
Task 1 updated value to: 7
Task 2 updated value to: 9
Final value: 9
```

## RWMutex

## Pool
It is a scalable pool of temporary, reusable objects. Its purpose is to cache allocated, but unused items for later reuse, relieving pressure on the garbage collector. Methods include:
- **Get():** fetches an arbitrary item from the Pool, removes it from the Pool, and returns it to the caller.
- **Put():** adds item to the Pool.

```go
func main() {
	var pool sync.Pool

	pool.Put("Data1")
	pool.Put("Data2")

	fmt.Println(pool.Get()) // Output: Data1
}
```

## Cond 

## Once

## Map

