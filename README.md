Buckit Go Client SDK for Amazon S3 Compatible Cloud Storage [![Sourcegraph](https://sourcegraph.com/github.com/buckit-io/buckit-go/-/badge.svg)](https://sourcegraph.com/github.com/buckit-io/buckit-go?badge) [![Apache V2 License](https://img.shields.io/badge/license-Apache%20V2-blue.svg)](https://github.com/buckit-io/buckit-go/blob/master/LICENSE)
==================================================================================================================================================================================================================================================================================================================================================================================================================

The Buckit Go Client SDK provides straightforward APIs to access any Amazon S3 compatible object storage.

This project is derived from the open source [MinIO Go Client SDK](https://github.com/minio/minio-go). It is maintained by Buckit for connecting to [Buckit](http://github.com/buckit-io/buckit) object storage and has no affiliation with MinIO, Inc.

This Quickstart Guide covers how to install the Buckit client SDK, connect to an S3-compatible object store, and create a sample file uploader. For a complete list of APIs and examples, see the [godoc documentation](https://pkg.go.dev/github.com/buckit-io/buckit-go/v7).

These examples presume a working [Go development environment](https://golang.org/doc/install) and the [`bm` command line tool](https://github.com/buckit-io/bm).

Download from Github
--------------------

From your project directory:

```sh
go get github.com/buckit-io/buckit-go/v7
```

Initialize a Client Object
--------------------------------

The client requires the following parameters to connect to an Amazon S3 compatible object storage:

| Parameter         | Description                                                |
|-------------------|------------------------------------------------------------|
| `endpoint`        | URL to object storage service.                             |
| `_buckit.Options_` | All the options such as credentials, custom transport etc. |

```go
package main

import (
	"log"

	buckit "github.com/buckit-io/buckit-go/v7"
	"github.com/buckit-io/buckit-go/v7/pkg/credentials"
)

func main() {
	endpoint := "buckit.example.net"
	accessKeyID := "YOUR-ACCESSKEYID"
	secretAccessKey := "YOUR-SECRETACCESSKEY"
	useSSL := true

	// Initialize client object.
	client, err := buckit.New(endpoint, &buckit.Options{
		Creds:  credentials.NewStaticV4(accessKeyID, secretAccessKey, ""),
		Secure: useSSL,
	})
	if err != nil {
		log.Fatalln(err)
	}

	log.Printf("%#v\n", client) // client is now set up
}
```

Example - File Uploader
-----------------------

This sample code connects to an object storage server, creates a bucket, and uploads a file to the bucket.

Use a sample Buckit server and replace the endpoint and credentials with values for your environment.

### FileUploader.go

This example does the following:

-	Connects to a sample Buckit server using the provided credentials.
-	Creates a bucket named `testbucket`.
-	Uploads a file named `testdata` from `/tmp`.
-	Verifies the file was created using `bm ls`.

	```go
	// FileUploader.go example
	package main

	import (
		"context"
		"log"

		buckit "github.com/buckit-io/buckit-go/v7"
		"github.com/buckit-io/buckit-go/v7/pkg/credentials"
	)

	func main() {
		ctx := context.Background()
		endpoint := "buckit.example.net"
		accessKeyID := "YOUR-ACCESSKEYID"
		secretAccessKey := "YOUR-SECRETACCESSKEY"
		useSSL := true

		// Initialize client object.
		client, err := buckit.New(endpoint, &buckit.Options{
			Creds:  credentials.NewStaticV4(accessKeyID, secretAccessKey, ""),
			Secure: useSSL,
		})
		if err != nil {
			log.Fatalln(err)
		}

		// Make a new bucket called testbucket.
		bucketName := "testbucket"
		location := "us-east-1"

		err = client.MakeBucket(ctx, bucketName, buckit.MakeBucketOptions{Region: location})
		if err != nil {
			// Check to see if we already own this bucket (which happens if you run this twice)
			exists, errBucketExists := client.BucketExists(ctx, bucketName)
			if errBucketExists == nil && exists {
				log.Printf("We already own %s\n", bucketName)
			} else {
				log.Fatalln(err)
			}
		} else {
			log.Printf("Successfully created %s\n", bucketName)
		}

		// Upload the test file
		// Change the value of filePath if the file is in another location
		objectName := "testdata"
		filePath := "/tmp/testdata"
		contentType := "application/octet-stream"

		// Upload the test file with FPutObject
		info, err := client.FPutObject(ctx, bucketName, objectName, filePath, buckit.PutObjectOptions{ContentType: contentType})
		if err != nil {
			log.Fatalln(err)
		}

		log.Printf("Successfully uploaded %s of size %d\n", objectName, info.Size)
	}
	```

**1. Create a test file containing data:**

You can do this with `dd` on Linux or macOS systems:

```sh
dd if=/dev/urandom of=/tmp/testdata bs=2048 count=10
```

or `fsutil` on Windows:

```sh
fsutil file createnew "C:\Users\<username>\Desktop\sample.txt" 20480
```

**2. Run FileUploader with the following commands:**

```sh
go mod init example/FileUploader
go get github.com/buckit-io/buckit-go/v7
go get github.com/buckit-io/buckit-go/v7/pkg/credentials
go run FileUploader.go
```

The output resembles the following:

```sh
2023/11/01 14:27:55 Successfully created testbucket
2023/11/01 14:27:55 Successfully uploaded testdata of size 20480
```

**3. Verify the Uploaded File With `bm ls`:**

```sh
bm ls sample/testbucket
[2023-11-01 14:27:55 UTC]  20KiB STANDARD TestDataFile
```

Full Examples
-------------

### Full Examples : Bucket Operations

-	[makebucket.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/makebucket.go)
-	[listbuckets.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/listbuckets.go)
-	[bucketexists.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/bucketexists.go)
-	[removebucket.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/removebucket.go)
-	[listobjects.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/listobjects.go)
-	[listobjectsV2.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/listobjectsV2.go)
-	[listincompleteuploads.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/listincompleteuploads.go)

### Full Examples : Bucket policy Operations

-	[setbucketpolicy.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/setbucketpolicy.go)
-	[getbucketpolicy.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/getbucketpolicy.go)
-	[listbucketpolicies.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/listbucketpolicies.go)

### Full Examples : Bucket lifecycle Operations

-	[setbucketlifecycle.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/setbucketlifecycle.go)
-	[getbucketlifecycle.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/getbucketlifecycle.go)

### Full Examples : Bucket encryption Operations

-	[setbucketencryption.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/setbucketencryption.go)
-	[getbucketencryption.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/getbucketencryption.go)
-	[removebucketencryption.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/removebucketencryption.go)

### Full Examples : Bucket replication Operations

-	[setbucketreplication.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/setbucketreplication.go)
-	[getbucketreplication.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/getbucketreplication.go)
-	[removebucketreplication.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/removebucketreplication.go)

### Full Examples : Bucket notification Operations

-	[setbucketnotification.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/setbucketnotification.go)
-	[getbucketnotification.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/getbucketnotification.go)
-	[removeallbucketnotification.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/removeallbucketnotification.go)


### Full Examples : File Object Operations

-	[fputobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/fputobject.go)
-	[fgetobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/fgetobject.go)

### Full Examples : Object Operations

-	[putobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/putobject.go)
-	[getobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/getobject.go)
-	[statobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/statobject.go)
-	[copyobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/copyobject.go)
-	[removeobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/removeobject.go)
-	[removeincompleteupload.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/removeincompleteupload.go)
-	[removeobjects.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/removeobjects.go)

### Full Examples : Encrypted Object Operations

-	[put-encrypted-object.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/put-encrypted-object.go)
-	[get-encrypted-object.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/get-encrypted-object.go)
-	[fput-encrypted-object.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/fputencrypted-object.go)

### Full Examples : Presigned Operations

-	[presignedgetobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/presignedgetobject.go)
-	[presignedputobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/presignedputobject.go)
-	[presignedheadobject.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/presignedheadobject.go)
-	[presignedpostpolicy.go](https://github.com/buckit-io/buckit-go/blob/master/examples/s3/presignedpostpolicy.go)

Explore Further
---------------

-	[Godoc Documentation](https://pkg.go.dev/github.com/buckit-io/buckit-go/v7)

Contribute
----------

[Contributors Guide](https://github.com/buckit-io/buckit-go/blob/master/CONTRIBUTING.md)

License
-------

This SDK is distributed under the [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0), see [LICENSE](https://github.com/buckit-io/buckit-go/blob/master/LICENSE) and [NOTICE](https://github.com/buckit-io/buckit-go/blob/master/NOTICE) for more information.
