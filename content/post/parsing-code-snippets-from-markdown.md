---
title: "Parsing code snippets from markdown with Go"
description: "Extracting code snippets from github README file using Go"
date:   2018-03-15T09:00:00+02:00
author: Sébastien Le Gall
categories: [Go, Markdown, Github]
---

Those time, I've been thinking that choosing the right vendor that fit your needs or will solve your problem has became a pain.

With the growth of platforms like Github and dependencies management tools (whatever the language) more and more people share their work and open source the peace of code they have created to solve a specific problem. That's great !

But at the end... you find out that for a given problem you want to solve, their are tones of libs that claim to do the same thing but each in a different way.

When I need to choose a third-party component, I usually start by reading code example (hopefully in the README file).

The code bellow is a peace of Go code that parse Github readme file of a given repository and look for code example to show in the terminal.

It uses the markdown parser [golang-commonmark/markdown](https://github.com/golang-commonmark/markdown). Again, it took me time to find out the right vendor to do the job. You know... the one that don't do too much magic and stay simple. But not to simple because I don't want to waste time. The one that is also well documented....

Anyway, the code bellow can be run easily in command line using the `--url` flag within you pass the Github url of the repository you want to extract code snippets. It get the content of the README file, parse the markdown and print the extracted peaces of code.

If you combine this with the Go import system, it becomes really easy to quickly know how a third party library may be use :

```sh
go run main.go --url https://github.com/{the go lib you want to use}
```

Here is the code :

```go
package main

import (
	"github.com/golang-commonmark/markdown"
	"fmt"
	"net/http"
	"io/ioutil"
	"log"
	"flag"
	"net/url"
)

//snippet represents the snippet we will output.
type snippet struct {
	content string
	lang string
}

//getSnippet extract only code snippet from markdown object.
func getSnippet(tok markdown.Token) snippet {
	switch tok := tok.(type) {
	case *markdown.CodeBlock:
		return snippet{
			tok.Content,
			"code",
		}
	case *markdown.CodeInline:
		return snippet{
			tok.Content,
			"code inline",
		}
	case *markdown.Fence:
		return snippet{
			tok.Content,
			tok.Params,
		}
	}
	return snippet{}
}

//readFromWeb call the given url and return the content of the readme.
func readFromWeb(url string) ([]byte, error) {
	resp, err := http.Get(url)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	return ioutil.ReadAll(resp.Body)
}


func main() {

	//Flag management
	var urlString string
	flag.StringVar(&urlString, "url", "", `The url of the github repository`)
	flag.Parse()
	if urlString == "" {
		log.Fatalln("Please, provide an url for the readme to parse.")
	}

	//Parse URL
	u, err := url.Parse(urlString)
	if err != nil {
		log.Fatalln("Impossible to parse the URL")
	}

	//Read the readme file
	readMe, err := readFromWeb(
		fmt.Sprintf("https://raw.githubusercontent.com%s/master/README.md",
			u.Path)
		)
	if err != nil {
		log.Fatalf(err.Error())
	}

	//Parse the markdown
	md := markdown.New(markdown.XHTMLOutput(true), markdown.Nofollow(true))
	tokens := md.Parse(readMe)

	//Print the result
	for _, t := range tokens {
		snippet := getSnippet(t)

		if snippet.content != "" {
			fmt.Printf("##### Lang : %s ###### \n", snippet.lang)
			fmt.Println(snippet.content)
		}
	}

}
```

This source code [can be found on Github](https://github.com/seblegall/snippetizer).