---
title: "Channels In Golang"
read_time: true
date: 2021-05-27T20:12:05+05:30
tags:
  - go
  - Do You Concur?
---

This is the final post in my "Do You Concur?" series. If you've not checked the other two out it is highly recommended you do that before reading this one. With that out of the way let's get started!

## What Are Channels?

A channel in Go can be best understood by comparing it with a box. It is a box where goroutines can put and take out the data. So essentially channels help your goroutines to share data. One goroutine can send a value into the channel and another goroutine can receive those values. A very simple code snippet demonstrating this is:

```
package main

import "fmt"

func main() {
	c := make(chan int)
	go func() {
		c <- 69
	}()
	fmt.Println(<-c)
}
```

## Channels Are Blockers!

"Channels block a goroutine if there is no buffer space."

A lot to explain in this one line. Let us first run some things. Trying running the following code:

```
package main

import "fmt"

func main() {
	c := make(chan int)
	c <- 69
	fmt.Println(<-c)
}
```

It will give you an error. Now try running:

```
package main

import "fmt"

func main() {
	c := make(chan int, 1)
	c <- 69
	fmt.Println(<-c)
}
```

This should work perfectly fine! So let us start understanding what is happening.



The 1 that we specified while initializing `c := make(chan int, 1)` is called the buffer space. In the code that didn't run there was no buffer space. A channel like that is called an unbuffered channel. Now read the following golden rule very carefully:

"Sending to unbuffered channels blocks the goroutine until something receives and vice-versa."

Quoting directly from [this](https://stackoverflow.com/a/56706591),

"Sending to a channel with no available buffer space blocks the sender until the send can complete; receiving from a channel with no available messages blocks the receiver until the receive can complete."

Looking at the code which failed, there was only one goroutine, the main goroutine. So when it reached the `c <- 69` line, it was blocked until some other goroutine received from c. But since there was no other goroutine it would remain blocked forever and that is why our code failed.

To fix this we could either launch a separate goroutine as shown in the initial code snippet or we could increase the buffer space to 1 as we did in the third code snippet. Increasing the buffer space to 1 basically told the channel that you can continue without blocking even if you have 1 value being sent to you with no one to receive it. So if we modify the third code snippet to as follows it will again not run as now we're sending two values:

```
package main

import "fmt"

func main() {
	c := make(chan int, 1)
	c <- 69
	c <- 69
	fmt.Println(<-c)
}
```

Well, this was it for this post and I hope you now have a basic understanding of channels and how they work. With this series, I tried to cover the basics around concurrency in Golang and I hope I was able to put the ideas across. 

Thank you for reading :)

