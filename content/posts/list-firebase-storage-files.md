+++
title = 'How to list Firebase Storage files in Golang'
date = 2024-06-27T17:35:23-03:00
draft = false
+++

Hi, today I want to share with you how I list all the files in certain folder of firebase storage and return them as an array. I this case only the files that are on the top level of that folder, and not the files inside a second folder.

### Dependencies

The package is ["cloud.google.com/go/storage"](https://pkg.go.dev/firebase.google.com/go/storage). You can install it using `go get  cloud.google.com/go/storage`

### Why is this useful?

I myself had the necessity of cleaning the storage folder of old files left there, so listing them is the first step. And then I took the returning array to delete over some conditions.

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

That you can find on package [docs](https://pkg.go.dev/firebase.google.com/go/storage), although I found very little examples about it.

Since I wanted only the files that were on the first folder, and not inside a second folder, I will be splitting the name of the file and taking only the ones whose split name array length were equal to 2.

### Limitations

As we are using a query that uses a string as prefix for selecting the folder we want to list, we can have the issue of returning more than one folder, since various folder names may fit the filter we are using. 

For example, if we have a folder called “1” (that was my case since my folders names were clients IDs), and we use that as a filter, our function may return also the content of the “10” folder. So we have to be careful of filtering manually after.

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

And thats it! thanks for reaching the end.