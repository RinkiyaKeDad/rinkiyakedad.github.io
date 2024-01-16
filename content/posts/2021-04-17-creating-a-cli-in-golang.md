---
title: "Creating A CLI In Golang"
read_time: true
date: 2021-04-17T20:12:05+05:30
tags:
 - go
 - cli
---

This is *technically* the first post on my blog so I'm pretty excited to write this one 😄 

For the last few months, I've been working on [depstat](https://github.com/kubernetes-sigs/depstat), which is a command-line tool to analyze dependencies, as part of my CNCF Internship. I've had a blast working on this especially because of the welcoming community. [depstat](https://github.com/kubernetes-sigs/depstat) is going to be used to evaluate dependency updates to Kubernetes and has been written in Go so I thought about sharing some stuff I learned :D

## The Tools

We're going to use [Cobra](https://github.com/spf13/cobra) to create our CLI. If you want to create a CLI in Golang, trust me, Cobra is the way to *GO*. The fact that it is used in the [Kubernetes codebase](https://github.com/kubernetes/kubernetes/blob/master/go.mod#L83) should be enough proof of its awesomeness. 

The simplest way to get started with Cobra is using the [Cobra Generator](https://github.com/spf13/cobra/blob/master/cobra/README.md#cobra-generator). You can simply run 
```
go get github.com/spf13/cobra/cobra
```
to get the cobra binary in your `$GOPATH/bin` directory.

Run `cobra help` from your terminal to verify that it was installed correctly. A common error you might get is:
```
zsh: command not found: cobra
```
This might be happening because zsh is not picking up the cobra binary from `$GOPATH/bin`. To fix that open your `.zshrc` file and add the following line:
```
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```
> Note if you're using bash instead of zsh you might need to edit the `.bash_profile` file instead

You should now be good to go!

For the sake of this tutorial, we're going to create a very simple `sound` CLI. This will take either "dog" or "cat" as an argument and print the sound they make as the result. 

Pretty useless but extremely simple for learning the basics. So let's get started!


## Cobra Basics

We can start our cobra project using:

```
mkdir sound
cd sound
cobra init --pkg-name=sound
```

This will set up a cobra project for us. The two important things which this sets up for us are:

1. `main.go`: There isn't a lot going on in this file. All it does is call the `execute` method in the `cmd` package.
2. `cmd/root.go`: For the sake of simplification replace the contents of your `root.go` file with these:

```
package cmd

import (
 "github.com/spf13/cobra"
)

// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{
 Use: "sound",
 Short: "A brief description of your application",
 Long: `A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
 // Run: func(cmd *cobra.Command, args []string) { },
}

func Execute() {
 cobra.CheckErr(rootCmd.Execute())
}
```

Now, this is much easier to explain :)

In `Use` we specify the name of the command/sub-command. Currently, we only have our base command, `sound` without any subcommands. The `Short` and `Long` fields are used by cobra to generate `help` for our commands. 

Let's now try this out!

## Running The CLI

Before installing, run `go mod init sound` to initialize go modules for the project. Then run `go mod tidy` to generate the `go.sum` file. Once this is done simply run,
```
go install
```
to install your CLI.

To test it out, run
`sound` 
and you should see the following output:

```
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.
```

If you haven't already noticed, this is the same text present by default in the `Long` field. Currently, the base command does nothing so running it just prints the help for it. Let's change this behavior now.

## Adding Some Logic

We now want our command to take an argument from the user and then print something based on that. For that functionality, the code is pretty simple:

```
// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{
	Use:   "sound",
	Short: "A brief description of your application",
	Long: `A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
	Run: func(cmd *cobra.Command, args []string) {
		if len(args) == 0 {
			log.Fatal("please provide a pet")
		}
		if args[0] == "dog" {
			fmt.Println("wooooof!")
		} else if args[0] == "cat" {
			fmt.Println("meoooow!")
		} else {
			fmt.Println("sorry your pet is currently not supported :(")
		}
	},
}
```

What we have done should be pretty straightforward to understand. Basically, Cobra gives us access to anything the user entered after the command (or subcommand) in form of the `args` slice. We can then check the animal type and print our result accordingly. Pretty simple! :)

To see each change you make take place in effect, you'll have to run `go install` again to update your binary. Do that and test out your sound CLI!

Well, that was it folks! I know there is a lot to cover when it comes to Cobra but I just wanted to show you how easy it is to create a simple command-line tool with it. If you're interested in learning more about it, I highly recommend you go check out the [official docs](https://cobra.dev).