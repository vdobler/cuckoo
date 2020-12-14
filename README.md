# Cuckoo Filter

Cuckoo Filter in Go

This is maintained fork of from github.com/mtchavez/cuckoo.
It differs from the original by:

- Strict use of Go modules
- Up to date dependencies
- No makefile for build and test


## Usage

- [New Filter](#new-filter)
- [Configuring](#configuration)
- [Insert](#insert)
- [Insert Unique](#insert-unique)
- [Lookup](#lookup)
- [Delete](#delete)
- [Item Count](#item-count)
- [Save](#save)
- [Load](#load)

### New Filter

Create a new filter with default configuration

    package main
 
    import "github.com/mtchavez/cuckoo"
 
    func main() {
        cuckoo.New()
    }

### Configuration

You can configure a filter via a `ConfigOption` type and the composed config option
functions provided.

	package main

	import "github.com/mtchavez/cuckoo"

	func main() {
		options := []cuckoo.ConfigOption{
			cuckoo.BucketEntries(uint(24)),
			cuckoo.BucketTotal(uint(1 << 16)),
			cuckoo.FingerprintLength(uint(1)),
			cuckoo.Kicks(uint(250)),
		}
		cuckoo.New(options...)
	}

### Insert

Inserting items into a filter

	package main

	import "github.com/mtchavez/cuckoo"

	func main() {
	    filter := cuckoo.New()
	    filter.Insert([]byte("special-items"))
	}

### Insert Unique

Inserting items into a filter only if they do not already exist

	package main

	import (
	    "fmt"
	    "github.com/mtchavez/cuckoo"
	)

	func main() {
	  filter := cuckoo.New()
	  filter.InsertUnique([]byte("special-items"))
	  filter.InsertUnique([]byte("special-items"))
	  if filter.ItemCount() != 1 {
	      fmt.Println("Expected only 1 item")
	  }
	}

### Lookup

Check if items exist in the filter using Lookup

	package main

	import (
	    "fmt"
	    "github.com/mtchavez/cuckoo"
	)

	func main() {
	  filter := cuckoo.New()
	  filter.Insert([]byte("special-items"))
	  found := filter.Lookup([]byte("special-items"))
	  if !found {
	      fmt.Println("Expected to find item in filter")
	  }
	}

### Delete

Deleting an item if it exists in the filter

	package main

	import (
	    "fmt"
	    "github.com/mtchavez/cuckoo"
	)

	func main() {
	  filter := cuckoo.New()
	  filter.Insert([]byte("special-items"))
	  deleted := filter.Delete([]byte("special-items"))
	  if !deleted {
	      fmt.Println("Expected to delete item from filter")
	  }
	}

### Item Count

Getting the item count of filter. **Using Insert with duplicates will cause the
item count to be more like a total items inserted count**. Using InsertUnique
and checking the ItemCount will be more of a *real* item count.

	package main

	import (
        "fmt"
	    "github.com/mtchavez/cuckoo"
	)

	func main() {
	  filter := cuckoo.New()
	  filter.InsertUnique([]byte("special-items"))
	  filter.InsertUnique([]byte("special-items"))
	  if filter.ItemCount() != 1 {
	      fmt.Println("Expected only 1 item")
	  }
	}

### Save

Encode and save a filter to be able to re-build or re-load back into memory.

	package main

	import (
	    "fmt"
	    "github.com/mtchavez/cuckoo"
	)

	func main() {
		filter := New()
		item := []byte("Largo")
		filter.InsertUnique(item)
		filter.Save("./tmp/example_save.gob")
	}

### Load

Load a filter back into memory from an encoded filter saved to a file.

package main

	import (
	    "fmt"
	    "github.com/mtchavez/cuckoo"
	)

	func main() {
		filter := New()
		item := []byte("Largo")
		filter.InsertUnique(item)
		filter.Save("./tmp/example_save.gob")

		loadedFilter, _ := Load("./tmp/example_save.gob")
		fmt.Printf("Loaded filter has same item? %v\n\n", loadedFilter.Lookup(item))
	}

## Benchmarks

There are benchmark tests to check performance of the filter. The following results
were ran on a 2.3 GHz Intel Core i7

```
# Updated: 2017-08-15

BenchmarkCuckooNew-8                  20          69606308 ns/op
BenchmarkInsert-8                2000000              1000 ns/op
BenchmarkInsertUnique-8          5000000               405 ns/op
BenchmarkLookup-8                5000000               399 ns/op
```

## Tests

Run tests via `go test` or with provided `Makefile`

`go test -v -cover` or `make test`
