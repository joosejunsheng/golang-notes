# golang-notes

## Channels
```mjs
func main() {
	ch1 := make(chan int)

	go func() {
		for i := 0; i < 100; i++ {
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

```mjs
func main() {

	ch1 := make(chan int)

	go func() {
    		for i := 0; i < 100; i++ {
			ch1 <- i
		}
    		close(ch1) // <------ This line to fix close the channel
	}()
	
	for n := range ch1 {
		fmt.Printf("n = %d\n", n)
	}
}
```
