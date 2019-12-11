---
title: "Integrating logrus with cobra"
description: "Setting up the log level before any command execution"
date:   2019-01-17T14:00:00+02:00
author: Sébastien Le Gall
categories: [Go, logrus, cobra]
---
> You can find a working example on my GitHub: [https://github.com/seblegall/blog-scripts/tree/master/logrus-cobra](https://github.com/seblegall/blog-scripts/tree/master/logrus-cobra)

I recently had to refactor the way one of my app print logs. My idea was to use `logrus` as it is a very well known lib to produce logs with Go.

Logrus let you print nicely color-coded and structured logs and is completely API compatible with the standard library logger.

![logrus](/img/article/go/logrus.png)

My first question was about log level and how to setup it once across all sub-directory/package of my project.

Under the hood, logrus instantiate a new variable `log`. Once done, calling the `logrus.SetLevel()` function will store the level directly in this newly instantiated struct.

Thus, we have a global struct, available across all sub-package making the level automatically shared without having to use dependency injection.

If you need to set a different level in a sub-package (for example), you just need to instantiate a new logger :

```golang
var log = logrus.New()

//...

log.Debug("debug log")
```
<!--more-->

> More about logrus here : [https://github.com/sirupsen/logrus](https://github.com/sirupsen/logrus)

Then I thought about adding a flag to my binary, letting the user set its own log level.

My application is using `cobra`, so I had to decided where to read the flag and where to init the logger by setting the level, depending on the flag value.

My first thought was to create a `SetLevel()` func which could be called in each cobra sub-command run function. But I found a better approach by looking at the source code of [Skaffold](https://github.com/GoogleContainerTools/skaffold/blob/69776b15674898a87ac61b9431f93ee68cffa6fd/cmd/skaffold/app/cmd/cmd.go#L51-L53)

Cobra has a cool feature called pre/post run. It's basically a function that will be executed right before the command start, or right after.

The same way you can defined "persistent" flags in cobra commands (which is a way to create inheritance between commands and sub-commands), the framework also offers a `PersistentPreRun` function.

That was the perfect place to put my log init as it is :

* Defined once for all sub-commands (which is a behaviour you probably want when creating a `verbose` flag.)
* A place where flags are already defined and read. So I could make a good use of the flag value set by the user.


Here is how it goes :

```golang
package cmd

import (
	"io"
	"os"

	"github.com/sirupsen/logrus"
	"github.com/spf13/cobra"
)

//The verbose flag value
var v string

var rootCmd = &cobra.Command{
	Use:   "mycmd",
	Short: "mycmd is a test project to integrate logrus",
}

//NewMyCmd return the root cobra command
func NewMyCmd() *cobra.Command {
    //Here is where we define the PreRun func, using the verbose flag value
    //We use the standard output for logs.
	rootCmd.PersistentPreRunE = func(cmd *cobra.Command, args []string) error {
		if err := setUpLogs(os.Stdout, v); err != nil {
			return err
		}
		return nil
	}

    //Here is where we bind the verbose flag
    //Default value is the warn level
    rootCmd.PersistentFlags().StringVarP(&v, "verbosity", "v", logrus.WarnLevel.String(), "Log level (debug, info, warn, error, fatal, panic")
    
	return rootCmd
}

//setUpLogs set the log output ans the log level
func setUpLogs(out io.Writer, level string) error {
	logrus.SetOutput(out)
	lvl, err := logrus.ParseLevel(level)
	if err != nil {
		return err
	}
	logrus.SetLevel(lvl)
	return nil
}
```

Once the `PersistentPreRun` func defined, you call easily create a sub-command which will use the logger.

```golang
var subCmd = &cobra.Command{
	Use:   "subcmd",
	Short: "subcmd is a subcommand of the main cmd",
	Run: func(cmd *cobra.Command, args []string) {
		runSubCmd()
	},
}

//...
//Inside the NewMyCmd() func
rootCmd.AddCommand(NewSubCmd())
//...

//NewSubCmd return the a sub cobra command
func NewSubCmd() *cobra.Command {
	return subCmd
}

func runSubCmd() {
	logrus.Debug("debug log")
}
```

This will produce :

```sh
$ go run main.go subcmd -v debug
DEBU[0000] debug log
```

By default, logrus is configured on Info level and the default value for my flag is Warn. Son the result proves it works ;-)

You are done !