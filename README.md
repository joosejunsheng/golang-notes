# golang-notes


- [Channels](#channels)
- [Efficiently Resizing Slice](#efficiently-resizing-slice)
 

## Channels
```go
func main(){
    ch1 := make(chan int)
    go func(){
        for i := 0; i < 10; i++ {
            ch1 <- i
        }
    }()
    for n := range ch1 {
        fmt.Printf("n = %d\n", n)
    }
}
```

`range` behaves differently with slice and with channel.

When it comes to slice, `range` actually runs only once to iterate the entire slice once.

But when it comes to channel, `range` behaves in a way that it continues to loop to wait for and receive values from channel UNTIL channel is closed.

In the case above, the code will result in `fatal error: all goroutines are asleep - deadlock!`

To fix it, just add a `close` after iterating the loop in the goroutine
```go
func main(){
    ch1 := make(chan int)
    go func(){
        for i := 0; i < 10; i++ {
            ch1 <- i
        }
	close(ch1) // <------ This line to fix close the channel
    }()
    for n := range ch1 {
        fmt.Printf("n = %d\n", n)
    }
}
```
Result : 
```go
n = 0
n = 1
n = 2
n = 3
n = 4
n = 5
n = 6
n = 7
n = 8
n = 9
```

### select
Using `select` in Go is like listening for available data on channel, or even multiple channels!

One Channel

```go
func sendData(ch chan string, message string, delay time.Duration) {
    time.Sleep(delay)
    ch <- message
}

func main() {
    ch1 := make(chan string)

    go sendData(ch1, "Message from Channel 1", 3*time.Second)

    for {
        select {
        case msg := <-ch1:
            fmt.Println("Received from Channel 1:", msg)
        default:
            fmt.Println("INFINITE LOOP")
        }
    }
}
```

The `for` loop above will be an infinite loop to `select`, repeating to consume data whenever data is ready in channels

Result :
```go
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
...
Received from Channel 1: Message from Channel 1 <----- After 3 seconds
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
...
```

Multiple Channels

```go
func sendData(ch chan string, message string, delay time.Duration) {
    time.Sleep(delay)
    ch <- message
}

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go sendData(ch1, "Message from Channel 1", 3*time.Second)
    go sendData(ch2, "Message from Channel 2", 1*time.Second)

    for {
        select {
        case msg := <-ch1:
            fmt.Println("Received from Channel 1:", msg)
        case msg := <-ch2:
            fmt.Println("Received from Channel 2:", msg)
        default:
            fmt.Println("INFINITE LOOP")
        }
    }
}
```
Result :
```go
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
...
Received from Channel 2: Message from Channel 2 <----- After 1 seconds
...
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
...
Received from Channel 1: Message from Channel 1 <----- After 3 seconds
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
INFINITE LOOP
...
```
## Efficiently Resizing Slice

```go
a := []int{1, 2, 3, 4, 5}
numFree := len(a)
fmt.Println(a) // Output: [1 2 3 4 5]

copy(a, a[1:])
fmt.Println(a) // Output: [2 3 4 5 5]

res = a[:numFree-1]
fmt.Println(res) // Output: [2 3 4 5]
```

Using `a = a[1:]` will end up printing the same output as `res = a[:numFree-]` but why the write more codes?

Technically when you re-assign a slice, eg. `b = a[1:]`, `res = a[:numFree-]`, it creates a new slice by pointing to the original slice / shares the same underlying array. The memory is still allocated even if you do `b = a[1:]`, the only difference is the new slice `b = a[1:]` is an entire new slice referencing the original slice where it starts from the position 1 instead of 0, `a[0]` is still in the memory.

By using `copy(a, a[1:])`, it moves the entire slice 1 position to the left. `res = a[:numFree-1]` is then executed to shorten the slice by 1, end up with a size of 4, remaining the capacity of 5. This will not leave `a[0]` "dangling" in the background like what we did using `a = a[1:]`.

