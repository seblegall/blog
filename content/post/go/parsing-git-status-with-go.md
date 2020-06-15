---
title: "Parsing git status with Go"
description: "Useful tips to get the status of a git repository using Go."
date:   2020-06-15T14:00:00+02:00
author: Sebastien Le Gall
categories: [Go, git]
---

At work I often have to switch from a repo to another, making some change here and not there, starting a new branch on repo A and then, fixing an issue on repo B. Ideally, a big fat monorepo should help me to deal with the ~40 apps and libs I work on. But monorepos have drawbacks when you're not Facebook or Google. By the time, they become heavier and soon a simple `git status` takes more than 2 seconds.

Instead of a monorepo I have built a few script helping me to deal with all those repos. One of them helps me knowing the status of each repository, just to know where I am and what I was doing before being interrupted.

So here we are... building a Go script to parse a git repository status.
<!--more-->

### Read the status from git command output

There are some Go libs [available around the web](https://github.com/go-git/go-git) that re-implement git in full Go. But most of the time, it requires to do a lot of setup to get the git status (such as creating an in-memory git repo). For simple use case, I like to use the original git. After all, all we need is to execute a shell command, right?

Let's start with that :

```golang
package main

import (
	"bytes"
	"fmt"
	"io"
	"os/exec"
)

//GetShortStatus read the git status of the repository located at path
func GetShortStatus(path string) (io.Reader, error) {
	return execOutput(fmt.Sprintf("git -C %s status -s -b --porcelain", path))
}

//It is useful to declare a var instead of a function for testing purpose
var execOutput = func(c string) (io.Reader, error) {
	out, err := exec.Command("/bin/sh", "-c", c).Output()

	return bytes.NewReader(out), err
}
```

Behind the scene, the `GetShortStatus(string)` method execute a simple git status command : `git status -s -b`

* `-b` is an option to output the current git branch
* `-s` is an option to output the short version of the git status output
* `--porcelain` is a dedicated git option to output an easy to parse string

Here is an output example :

```bash
## master...origin/master
?? content/post/go/parsing-git-status-with-go.md
```

### Parsing the short git status output using Go

After getting the `git status` command output we still need to parse it. I chose to use a `struct` to store the current branch name and the files status. I could also have returned both as 2 returned parameters.

```golang
package main

import (
	"bufio"
	"io"
	"strings"
)

type Output struct {
	Branch      string
	FilesStatus []string
}

//Parse parses a git status output command
//It is compatible with the short version of the git status command
func ParseShort(r io.Reader) Output {

	s := bufio.NewScanner(r)

	var branch string
	//Extract branch name
	for s.Scan() {
		//Skip any empty line
		if len(s.Text()) < 1 {
			continue
		}

		branch = parseBranch(s.Text())
		break
	}

	var fs []string
	for s.Scan() {
		if len(s.Text()) < 1 {
			continue
		}
		fs = append(fs, s.Text())
	}

	return Output{
		Branch:      branch,
		FilesStatus: fs,
	}
}

func parseBranch(input string) string {
	s := bufio.NewScanner(strings.NewReader(input))
	s.Split(bufio.ScanWords)

	//check if input is a status branch line output
	s.Scan()
	if s.Text() != "##" {
		return ""
	}

	//read next word and return the branch name
	s.Scan()
	b := strings.Split(s.Text(), "...")
	return b[0]
}
```

The `parseBranch(string)` method extract the branch name from the pattern : `## {branch}...{remote branch}`.

Concerning the file status, I chose not to parse the file status line because I didn't need it, but following the same logic as for the branch, it's quite easy to do, [knowing the meaning of the 2 first characters](https://git-scm.com/docs/git-status).

```bash
X          Y     Meaning
-------------------------------------------------
	 [AMD]   not updated
M        [ MD]   updated in index
A        [ MD]   added to index
D                deleted from index
R        [ MD]   renamed in index
C        [ MD]   copied in index
[MARC]           index and work tree matches
[ MARC]     M    work tree changed since index
[ MARC]     D    deleted in work tree
[ D]        R    renamed in work tree
[ D]        C    copied in work tree
-------------------------------------------------
D           D    unmerged, both deleted
A           U    unmerged, added by us
U           D    unmerged, deleted by them
U           A    unmerged, added by them
D           U    unmerged, deleted by us
A           A    unmerged, both added
U           U    unmerged, both modified
-------------------------------------------------
?           ?    untracked
!           !    ignored
-------------------------------------------------
```

### Putting all together

Finally, all we still need to do is putting all together :

```golang
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	args := os.Args[1:]

	if len(args) < 1 {
		log.Fatalf("please, provide a git repository as command argument")
	}

	out, err := GetShortStatus(args[0])
	if err != nil {
		log.Fatalf("unable to read git repository status : %s", err.Error())
	}

	status := ParseShort(out)

	fmt.Println("Branch is ", status.Branch)
	for _, fs := range status.FilesStatus {
		fmt.Printf("    %s\n", fs)
	}

}
```

🥳 Tada !