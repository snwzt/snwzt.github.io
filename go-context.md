# Context in Go
The context package in Go is designed to carry deadlines, cancellation signals, and other request-scoped values across API boundaries and between processes. This package is essential for managing the lifecycle of requests in concurrent programming. Essentially, the context in Go is the same as the **context** in the English language. So, when we pass a context to a goroutine, we want it to know the **context** for the execution. For example, in the case of a web backend written in Go, if it is trying to connect to the database, we don't want it to block the operation by retrying forever. We can use a timeout context of 10 seconds, and after 10 seconds it will stop trying to establish the connection.

There are two types of context-

**Background:** Returns an empty context that has methods like Deadline, Done, Err, and Value.

**TODO:** Similar to Background but used as a placeholder for contexts that will be implemented later.

## Methods in Context
**Deadline:** Returns the time when work is supposed to be done on behalf of the context and after that it should be cancelled.

**Done:** It is basically a channel which tells you if goroutine has been finished or not.

**Err:** If done is closed, it will return error. If not, it will return nil.

**Value:** Returns value associated with the context

## Functions in Context
**Cause:** Returns the "cause" of a context cancellation. If cause is not explicitly defined, it will return same value as Err() method.

**WithCancel:** Creates a new context that can be canceled. It returns the new context and a cancel function.

**WithCancelCause:** Works like WithCancel but lets you provide a reason or "cause" for cancellation.

**WithDeadline:** Creates a new context that will automatically cancel at a specific time. It returns the new context and a cancel function.

**WithDeadlineCause:** Similar to WithDeadline, but lets you prove a "cause" for cancellation.

**WithTimeout:** Creates a new context that will automatically cancel after a specific duration. It returns the new context and a cancel function.

**WithTimeoutCause:** Similar to WithTimeout, but lets you prove a "cause" for cancellation.

**WithValue:** Creates a new New context with a specific value associated with a key. It returns the new context.

**WithoutCancel:** Creates a context from an existing context that can't be canceled.

## Example
Examples for using functions in context discussed above:
```go
func worker(ctx context.Context, name string, wg *sync.WaitGroup) {
	defer wg.Done()

	for {
		select {
		case <-ctx.Done():
			fmt.Printf("%s: Context Done with error: %v, Cause: %v \n", name, ctx.Err(), context.Cause(ctx))
			return
		default:
		}
	}
}

func main() {
	baseCtx := context.Background()
	var wg sync.WaitGroup

	// New context with cancellation
	ctxWithCancel, cancel := context.WithCancel(baseCtx)
	wg.Add(1)
	go worker(ctxWithCancel, "WithCancel", &wg)
	cancel()

	// New context with cancellation and cause
	ctxWithCancelCause, cancelCause := context.WithCancelCause(baseCtx)
	wg.Add(1)
	go worker(ctxWithCancelCause, "WithCancelCause", &wg)
	cancelCause(fmt.Errorf("WithCancelCause: canceled for cringe reasons"))

	// New context with deadline
	ctxWithDeadline, cancel := context.WithDeadline(baseCtx, time.Now().Add(1*time.Second))
	t, _ := ctxWithDeadline.Deadline()
	fmt.Printf("Deadline time: %v\n", t) // Prints deadline time
	wg.Add(1)
	go worker(ctxWithDeadline, "WithDeadline", &wg)

	// New context with deadline and cause
	ctxWithDeadlineCause, _ := context.WithDeadlineCause(baseCtx, time.Now().Add(1*time.Second),
		fmt.Errorf("WithDeadlineCause: reached deadline"))
	wg.Add(1)
	go worker(ctxWithDeadlineCause, "WithDeadlineCause", &wg)

	// New context with timeout
	ctxWithTimeout, cancel := context.WithTimeout(baseCtx, 1*time.Second)
	wg.Add(1)
	go worker(ctxWithTimeout, "WithTimeout", &wg)

	// New context with timeout and cause
	ctxWithTimeoutCause, _ := context.WithTimeoutCause(baseCtx, 1*time.Second,
		fmt.Errorf("WithTimeoutCause: timeout occurred"))
	wg.Add(1)
	go worker(ctxWithTimeoutCause, "WithTimeoutCause", &wg)

	// New context with value
	ctxWithValue := context.WithValue(baseCtx, "fruit", "apple")
	wg.Add(1)
	go func(ctx context.Context, wg *sync.WaitGroup) {
		defer wg.Done()

		if val := ctx.Value("fruit"); val != nil {
			fmt.Printf("WithValue: Got value: %v\n", val)
		} else {
			fmt.Println("WithValue: No value found")
		}
	}(ctxWithValue, &wg)

	// Wait for all goroutines to complete
	wg.Wait()
}
```

Output:
```
snwzt@cara ~> go run main.go
WithCancel: Context Done with error: context canceled, Cause: context canceled 
Deadline time: 2024-06-09 17:34:51.116821795 +0530 IST m=+1.000071125
WithValue: Got value: apple
WithCancelCause: Context Done with error: context canceled, Cause: WithCancelCause: canceled for cringe reasons 
WithDeadlineCause: Context Done with error: context deadline exceeded, Cause: WithDeadlineCause: reached deadline 
WithTimeout: Context Done with error: context deadline exceeded, Cause: context deadline exceeded 
WithDeadline: Context Done with error: context deadline exceeded, Cause: context deadline exceeded 
WithTimeoutCause: Context Done with error: context deadline exceeded, Cause: WithTimeoutCause: timeout occurred 
```