---
image:
    path: assets/images/og_image.png
    width: 500
    height: 500
---

# Sync Package in Go
Go offers various tools to manage concurrent execution. While channels are the high-level means of communication and synchronization, the sync package provides with low-level primitives to handle synchronization effectively. It also includes atomic sub-package, which provides low-level atomic memory primitives useful for implementing synchronization algorithms. Mostly used components of the sync package include: WaitGroup, Mutex, RWMutex, Pool, Map, Cond, and Once. Values containing types defined in sync package should not be copied.

## WaitGroup
WaitGroup is used to wait for a collection of goroutines to finish. It tracks the number of goroutines and blocks until all goroutines have finished executing. Methods include:

**Add(d int):** Increments the waiting counter by `d`.

**Done():** Decrements the counter by one, indicating that a goroutine has completed its task.

**Wait():** Blocks until the counter becomes zero.

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

**Lock():** Acquires the lock. If lock already in use, calling goroutine is blocked till lock is released.

**TryLock():** Attempts to acquire the lock and returns success status as bool.

**Unlock():** Releases the lock.

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
An RWMutex is a reader/writer mutual exclusion lock. The lock can be held by an arbitrary number of readers or a single writer. RWMutex is useful in scenarios where you have multiple readers but only a few writers. Methods include:

**Lock():** Acquires the write lock. If the lock is already in use, the calling goroutine blocks until the lock is available.

**Unlock():** Releases the write lock.

**RLock():** Acquires a read lock. Multiple readers can hold the lock simultaneously.

**RUnlock():** Releases a read lock.

```go
var (
	rwMutex sync.RWMutex
	value   int
	wg      sync.WaitGroup
)

func reader(id int) {
	defer wg.Done()
	rwMutex.RLock()
	fmt.Printf("Reader %d: Value is %d\n", id, value)
	time.Sleep(100 * time.Millisecond)
	rwMutex.RUnlock()
}

func writer(id int) {
	defer wg.Done()
	rwMutex.Lock()
	value += 10
	fmt.Printf("Writer %d: Updated value to %d\n", id, value)
	time.Sleep(100 * time.Millisecond)
	rwMutex.Unlock()
}

func main() {
	wg.Add(4)
	go reader(1)
	go writer(1)
	go reader(2)
	go reader(3)
	wg.Wait()
}
```

Execution:
```
snwzt@cara ~> go run main.go
Reader 3: Value is 0
Reader 1: Value is 0
Writer 1: Updated value to 10
Reader 2: Value is 10
```

## Pool
It is a scalable pool of temporary, reusable objects. Its purpose is to cache allocated, but unused items for later reuse, relieving pressure on the garbage collector. Methods include:

**Get():** fetches an arbitrary item from the Pool, removes it from the Pool, and returns it to the caller.

**Put():** adds item to the Pool.

```go
func main() {
	var pool sync.Pool

	pool.Put("Data1")
	pool.Put("Data2")

	fmt.Println(pool.Get()) // Output: Data1
}
```

## Cond
Cond is used to create a condition variable, which allows goroutines to wait for or announce changes to a shared state. It's typically used with a Mutex. Methods include:

**Wait():** Waits for a notification. The associated Mutex must be locked before calling Wait.

**Signal():** Wakes up one goroutine waiting on the condition variable.

**Broadcast():** Wakes up all goroutines waiting on the condition variable.

```go
var (
	mutex sync.Mutex
	cond  = sync.NewCond(&mutex)
)

func waitCondition(id int) {
	mutex.Lock()
	cond.Wait()
	fmt.Printf("Goroutine %d resumed\n", id)
	mutex.Unlock()
}

func main() {
	for i := 1; i <= 3; i++ {
		go waitCondition(i)
	}

	time.Sleep(1 * time.Second)

	mutex.Lock()
	fmt.Println("Broadcasting signal to all goroutines")
	cond.Broadcast()
	mutex.Unlock()
}
```

## Once
Once ensures that a piece of code is executed only once, regardless of the number of goroutines executing it. This is useful for one-time initialization. Methods include:

**Do():** Executes the function passed as argument only once.

```go
var once sync.Once

func initialize() {
	fmt.Println("Initialization done")
}

func main() {
	for i := 1; i <= 3; i++ {
		go func(id int) {
			once.Do(initialize)
			fmt.Printf("Goroutine %d\n", id)
		}(i)
	}
	time.Sleep(1 * time.Second)
}
```

## Map
Map is a concurrent map safe for concurrent use by multiple goroutines without additional locking or coordination. Unlike the standard Go map, which requires explicit synchronization, sync.Map handles concurrency internally. This makes it suitable for scenarios where high read and write contention is expected. The zero value of a sync.Map is empty and ready for use. However, in most cases, using a plain Go map with explicit locking (via sync.Mutex or sync.RWMutex) is recommended due to better performance in low contention scenarios. Methods include:

**Store():** Sets the value for a key.

**Load():** Returns the value stored in the map for a key, and a boolean indicating whether the key was found.

**Delete():** Deletes the value for a key.

**LoadOrStore():** Returns the existing value for the key if present, otherwise stores and returns the given value.

```go
func main() {
	var m sync.Map

	m.Store("key1", "value1")
	m.Store("key2", "value2")

	value, ok := m.Load("key1")
	if ok {
		fmt.Printf("Key: key1, Value: %s\n", value)
	}

	m.Range(func(key, value interface{}) bool {
		fmt.Printf("Key: %s, Value: %s\n", key, value)
		return true
	})
}
```

