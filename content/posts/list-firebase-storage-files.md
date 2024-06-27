+++
title = 'How to list Firebase Storage files in Golang'
date = 2024-06-27T17:35:23-03:00
draft = false
+++

Hi, today I want to show you how to list all the files in a particular folder of a Firebase storage and return them as an array. In this case only the files that are on the top level of that folder, and not the files inside a second folder.

### Dependencies

The package is ["cloud.google.com/go/storage"](https://pkg.go.dev/firebase.google.com/go/storage). You can install it using `go get  cloud.google.com/go/storage`

### Why is this useful?

I myself had the necessity of cleaning the storage folder of old files left there, so listing them is the first step, then I took the returned array to delete the files listed over some conditions.

But there are lots of situations where you read all the files in a foder, and having the list function on the toolbox may be useful for some of you.

### Code Breakdown

First we need to initiate a Bucket object with our Firebase credentials. I called this function `GetStorageBucket`, and takes from the env variables the keys of my firebase account and returns a Bucket instance.

```go
package storage

import (
	"context"

	"cloud.google.com/go/storage"
	firebase "firebase.google.com/go"
	"github.com/dirsacorp/scripts/modules/shared"
	"google.golang.org/api/option"
)

func GetStorageBucket() *storage.BucketHandle {
	err := shared.LoadEnv()
	if err != nil {
		panic(err)
	}
	
	storageBucket := shared.GetEnvWithKey("STORAGE_BUCKET")
	accountKey := shared.GetEnvWithKey("ACCOUNT_KEY")

	config := &firebase.Config{
		StorageBucket: storageBucket,
	}
	
	opt := option.WithCredentialsJSON([]byte(accountKey))
	app, err := firebase.NewApp(context.Background(), config, opt)
	
	if err != nil {
		panic(err.Error())
	}

	client, err := app.Storage(context.Background())
	if err != nil {
		panic(err.Error())
	}

	bucket, err := client.DefaultBucket()
	if err != nil {
		panic(err.Error())
	}

	return bucket
}

```

For listing files we are going to use:

```go
&storage.Query({prefix: "prefix"})
```

This can be found in the [docs](https://pkg.go.dev/firebase.google.com/go/storage) package, although I found very few examples.

Since I only wanted the files that were in the top folder, and not in a second folder, I will split the name of the file and take only those whose split name array length was equal to 2.

### Limitations

Since we are using a query that uses a string as a prefix to select the folder we want to list, we may have the problem of returning more than one folder, as different folder names may match the filter we are using. 

For example, if we have a folder called "1" (this was my case, since my folder names were client IDs) and we use that as a filter, our function may also return the contents of folder "10". So we have to be careful to filter manually afterwards.

### The listing function

```go
package storage

import (
	"context"
	"log"
	"strings"

	"cloud.google.com/go/storage"
	"google.golang.org/api/iterator"
)

func ListFiles(clientId string) []string {
	ctx := context.Background()

	bucket := GetStorageBucket()

	var files []string

	query := &storage.Query{Prefix: clientId}
	iter := bucket.Objects(ctx, query)

	for {
		file, err := iter.Next()
		if err == iterator.Done {
			break
		}
		if err != nil {
			log.Fatalf("error listing files: %s\n", err)
		}

		splitName := strings.Split(file.Name, "/")

		if len(splitName) == 2 {
			files = append(files, file.Name)
		}
	}

	return files
}

```

And that was it! Hope you found it useful.