cache2go
========

Concurrency-safe golang caching library with expiration capabilities.

## Update Info
基于cache2go，对其的自动续期功能进行了修改。
让用户设置cache之时，可以选择缓存是否有自动延期功能。

## Installation

Make sure you have a working Go environment (Go 1.2 or higher is required).
See the [install instructions](http://golang.org/doc/install.html).

To install cache2go, simply run:

    go get github.com/muesli/cache2go

To compile it from source:

    cd $GOPATH/src/github.com/muesli/cache2go
    go get -u -v
    go build && go test -v

## Example
```go
package main

import (
	"github.com/muesli/cache2go"
	"fmt"
	"time"
)

// Keys & values in cache2go can be of arbitrary types, e.g. a struct.
type myStruct struct {
	text     string
	moreData []byte
}

func main() {
	// Accessing a new cache table for the first time will create it.
	cache := cache2go.Cache("myCache")

	// We will put a new item in the cache. It will expire after
	// not being accessed via Value(key) for more than 5 seconds.
	val := myStruct{"This is a test!", []byte{}}
	cache.Add("someKey", 5*time.Second, &val)

	// Let's retrieve the item from the cache.
	res, err := cache.Value("someKey")
	if err == nil {
		fmt.Println("Found value in cache:", res.Data().(*myStruct).text)
	} else {
		fmt.Println("Error retrieving value from cache:", err)
	}

	// Wait for the item to expire in cache.
	time.Sleep(6 * time.Second)
	res, err = cache.Value("someKey")
	if err != nil {
		fmt.Println("Item is not cached (anymore).")
	}

	// Add another item that never expires.
	cache.Add("someKey", 0, &val)

	// cache2go supports a few handy callbacks and loading mechanisms.
	cache.SetAboutToDeleteItemCallback(func(e *cache2go.CacheItem) {
		fmt.Println("Deleting:", e.Key(), e.Data().(*myStruct).text, e.CreatedOn())
	})

	// Remove the item from the cache.
	cache.Delete("someKey")

	// And wipe the entire cache table.
	cache.Flush()
}
```

To run this example, go to examples/mycachedapp/ and run:

    go run mycachedapp.go

You can find a [few more examples here](https://github.com/muesli/cache2go/tree/master/examples).
Also see our test-cases in cache_test.go for further working examples.

## Development

[![GoDoc](https://godoc.org/github.com/golang/gddo?status.svg)](https://godoc.org/github.com/muesli/cache2go)
[![Build Status](https://travis-ci.org/muesli/cache2go.svg?branch=master)](https://travis-ci.org/muesli/cache2go)
[![Coverage Status](https://coveralls.io/repos/github/muesli/cache2go/badge.svg?branch=master)](https://coveralls.io/github/muesli/cache2go?branch=master)
[![Go ReportCard](http://goreportcard.com/badge/muesli/cache2go)](http://goreportcard.com/report/muesli/cache2go)
