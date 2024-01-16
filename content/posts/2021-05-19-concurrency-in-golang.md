---
title: "Concurrency In Golang"
read_time: true
date: 2021-05-19T20:12:05+05:30
tags:
  - go
  - Do You Concur?
---

I recently started learning more about concurrency in golang and so as with other things I learn, I decided to write about it. This article will be a part of a short series I plan to write about concurrency in Go. So let's get started!

## Concurrency Vs Parallelism

Before we start looking at some code, I think it would be better to know what concurrency actually is. But even before we do that let us first talk about parallelism because it is a word that often comes up when talking about concurrency. 

In very simple terms parallelism can be understood as when two *instructions* are executed in parallel, that is, simultaneously instead of the traditional way of sequential execution. "Instructions" is a loosely used word in that sentence and could have different meanings in different contexts. More often than not it means functions or methods. For example,  if you have this:

```
func main(){
    woof()
    meow()
}
``` 
Then the functions get executed sequentially, that is, first the `woof` function returns and then after that, the `meow` function returns. If they were being executed parallelly then immediately after starting the execution of `woof` the execution of `meow` would start without waiting for `woof` to return.

So now that you have an idea of what parallelism is, let us come back to our original question: what is concurrency?

Concurrency is a design pattern. It is a way to build things. Concurrency enables parallelism but it doesn't guarantee it. 

Let me make it more clear by extending the above example. If you tell the code above that `woof` *can* be called in a separate CPU thread then that would be concurrency. Now this doesn't guarantee that `woof` will be called in a separate thread because what if you only have one thread? 

But in case circumstances allow it then the two functions will be executed parallelly. I hope this makes it more clear what I mean by the line that concurrency is a design pattern.

> Parallelism is not the goal of concurrency, concurrency's goal is a good structure. - [Rob Pike](https://youtu.be/oV9rvDllKEg?t=172)

## Concurrency in Go

So now that you have an idea of what concurrency is let us look at concurrency in Golang. I'm going to try and keep this as simple as possible. Let me expand on the code shown above. Let us say we have the following `main.go` file:

```
package main

import (
	"fmt"
)

func main() {
	woof()
	meow()
}

func woof() {
	for i := 1; i <= 5; i++ {
		fmt.Println("woof ", i)
	}
}

func meow() {
	for i := 1; i <= 5; i++ {
		fmt.Println("meow ", i)
	}
}
```

Right now the instructions in the main function would execute sequentially. So first `woof` will be called and when it returns then and only then will `meow` be called. To *enable* parallel execution we use the `go` keyword right before the instruction. So, replace the main function with:

```
func main() {
	go woof()
	meow()
}
```

Now let us see what happens. Run the program and you'll see that output from only the meow function. Strange, no?

When we talk about concurrency you can imagine parallel lanes and each instruction executing in its own lane parallel to the others. Each of these lanes is what we call a `goroutine`. So when we execute stuff sequentially, there is only one goroutine, the main goroutine.

We created another one when said `go woof()`. But what happened was that before this goroutine could print its output, the main goroutine executed the next instruction (which was calling meow) and after that, it saw that there was nothing to be executed so it ended.

And when the main goroutine ends all others end automatically. So the reason we didn't see any output from `woof` is because it's routine ended before it could print anything.

So how do we avoid this?

## Enter WaitGroup

WaitGroup provides us with the necessary synchronization needed to execute both functions. Replace the contents of your main.go with:

```
package main

import (
	"fmt"
	"runtime"
	"sync"
)

var wg sync.WaitGroup

func main() {
	// add one thing to wait for
	wg.Add(1)
	go woof()
	meow()

	// stop waiting when 0 things in waiting list
	wg.Wait()
}

func woof() {
	for i := 1; i <= 5; i++ {
		fmt.Println("woof ", i)
	}
	// remove one thing from waiting list
	wg.Done()
}

func bar() {
	for i := 1; i <= 5; i++ {
		fmt.Println("meow ", i)
	}
}
```

Run the program now and you should see output from both the functions. So what exactly happened?

I personally find this very easy to understand by imagining a waiting list. Before launching our `woof` function, we added `1` thing to the waiting list. Then the program launched a goroutine for the execution of `woof` and moved on to the next instruction which was to execute `meow`. But this time instead of ending it saw `wg.Wait()` which instructed it to wait until the waiting list was empty. And if you notice carefully, in our `woof` function we added `wg.Done()` which basically tells the program that this process is now done and you can take this out of the waiting list!

I hope that makes sense and you now have a much better idea of what concurrency is and how it works in golang. Over the next set of articles, I'll try to explain other concepts surrounding it with a lot more examples so keep an eye out for those if you found this one interesting :)

Thanks for reading!