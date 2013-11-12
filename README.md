cache2go
========

Simple golang object caching library with expiration capabilities.

## Installation

Make sure you have a working Go environment. See the [install instructions](http://golang.org/doc/install.html).

To install cache2go, simply run:

    go get github.com/rif/cache2go

To compile it from source:

    git clone git://github.com/rif/cache2go.git
    cd cache2go && go build && go test -v

## Example
```go
package main

import (
	"github.com/rif/cache2go"
	"fmt"
	"time"
)

// Keys & values in cache2go can be off arbitrary types, e.g. a struct
type myStruct struct {
	text     string
	moreData []byte
}

func main() {
	// Accessing a new cache table for the first time will create it
	cache := cache2go.Cache("myCache")

	// We will put a new item in the cache. It will expire in 5 seconds
	val := myStruct{"This is a test!", []byte{}}
	cache.Cache("someKey", 5*time.Second, &val)

	// Let's retrieve the item from the cache
	res, err := cache.Value("someKey")
	if err == nil {
		fmt.Println("Found value in cache:", res.Data().(*myStruct).text)
	} else {
		fmt.Println("Error retrieving value from cache:", err)
	}

	// Wait for the item to expire in cache
	time.Sleep(6 * time.Second)
	res, err = cache.Value("someKey")
	if err != nil {
		fmt.Println("Item is not cached (anymore).")
	}

	// Add another item that never expires
	cache.Cache("someKey", 0, &val)

	// cache2go supports a few handy callbacks and loading mechanisms
	cache.SetAboutToDeleteItemCallback(func(e *cache2go.CacheItem) {
		fmt.Println("Deleting:", e.Key(), e.Data().(*myStruct).text, e.CreatedOn())
	})

	// Remove the item from the cache
	cache.Delete("someKey")

	// And wipe the entire cache table
	cache.Flush()
}
```

To run the application, go to examples/mycachedapp/ and run:

    go run mycachedapp.go

You can find a few more examples [few more examples here](https://github.com/rif/cache2go/tree/master/examples).
Also see our test-cases in cache_test.go for further working examples.

## Development
API docs can be found [here](http://go.pkgdoc.org/github.com/rif/cache2go).

Continous integration: [![Build Status](https://secure.travis-ci.org/rif/cache2go.png)](http://travis-ci.org/rif/cache2go)
