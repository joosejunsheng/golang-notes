# golang-notes

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

