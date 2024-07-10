The Admin APIs facilitate administrative tasks related to IAM user management, bucket ownership modifications, and bucket listings with their respective owners. These APIs ensure robust and secure interactions by requiring requests to be authenticated using `AWS Signature Version 4 (SigV4)`. Only **admin** or **root** users, possessing the necessary AWS access key ID and secret access key, can execute these operations.

## Create user

The endpoint allows to create a new IAM user account.

### Request

* Method: PATCH
* Endpoint: /create-user
* Content-Type: application/json

### Request Body

| Property | Type | Description | Required |
|:------:  | :---: | :---------:| :------: |
| access | string | user access key id | yes |
| secret | string | user secret access key | yes |
| role | string(user/userplus/admin) | user role | yes |
| userID | number | user id | no |
| groupID | number | user group id | no |

### Responses

**Success(201 Created)**
```
The user has been created successfully
```

**Error(403 Forbidden)**
```
access denied: only admin users have access to this resource
```

**Error(400 Bad Request)**
```
invalid parameters: user role have to be one of the followings: 'user', 'admin', 'userplus'
```
```
failed to parse request body
```

**Error(409 Conflict)**
```
failed to create user: update iam data: user already exists
```

**Error(500 Internal Server Error)**
```
<any unexpected internal server error>
```

### Example Usage

**versitygw admin CLI**
```
versitygw admin -a <admin_access_key_id> -s <admin_secret_access_key> -er http://127.0.0.1:7070 create-user -a <user_access_key_id> -s <user_secret_access_key> -r <user_role> -ui <user_userID> -gi <user_groupID>
```

**Programmatically with Go**
```go
package main

import (
	"bytes"
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

type user struct {
	Access  string `json:"access"`
	Secret  string `json:"secret"`
	Role    string `json:"role"`
	UserID  int    `json:"userID"`
	GroupID int    `json:"groupID"`
}

func main() {
	u := user{
		Access:  "test_user_1",
		Secret:  "test_user_1_secret",
		Role:    "user",
		UserID:  1,
		GroupID: 3,
	}

	userJSON, err := json.Marshal(u)
	if err != nil {
		log.Fatalf("failed to serialize user data: %w\n", err)
	}

	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/create-user", adminEndpoint), bytes.NewBuffer(userJSON))
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256(userJSON)
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %w\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("failed to read the response body: %w\n", err)
	}

	if resp.StatusCode >= 400 {
		log.Fatalf("request failed with message %s\n", body)
	}

	fmt.Printf("%s\n", body)
}

```

## Delete User

The endpoint allows to delete an IAM user account:

### Request

* Method: PATCH
* Endpoint: /delete-user?access=<user_access_key_id>

### Responses

**Success(200 OK)**
```
The user has been deleted successfully
```

**Error(403 Forbidden)**
```
access denied: only admin users have access to this resource
```

**Error(500 Internal Server Error)**
```
<any unexpected internal server error>
```

### Example Usage

**versitygw admin CLI**
```
versitygw admin -a <admin_access_key_id> -s <admin_secret_access_key> -er http://127.0.0.1:7070 delete-user -a <user_access_key_id>
```

**Programmatically with Go**
```go
package main

import (
	"crypto/sha256"
	"encoding/hex"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

func main() {
	userAccess := "del_user_access_key"

	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/delete-user?access=%v", adminEndpoint, userAccess), nil)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256([]byte{})
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %w\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("failed to read the response body: %w\n", err)
	}

	if resp.StatusCode >= 400 {
		log.Fatalf("request failed with message %s\n", body)
	}

	fmt.Printf("%s\n", body)
}
```

## Update User

The endpoint allows to update `secret`, `userID`, `groupID` properties in an IAM user account.

### Request

* Method: PATCH
* Endpoint: /update-user
* Content-Type: application/json

### Request Body

| Property | Type | Description | Required |
|:------:  | :---: | :---------:| :------: |
| secret | string | user secret access key | no |
| userID | number | user id | no |
| groupID | number | user group id | no |

### Responses

**Success(200 OK)**
```
the user has been updated successfully
```

**Error(403 Forbidden)**
```
access denied: only admin users have access to this resource
```

**Error(400 Bad Request)**
```
missing user access parameter
```
```
invalid request body
```

**Error(404 Not Found)**
```
failed to update user account: user not found
```

**Error(500 Internal Server Error)**
```
<any unexpected internal server error>
```

### Example Usage

**versitygw admin CLI**
```
versitygw admin -a <admin_access_key_id> -s <admin_secret_access_key> -er http://127.0.0.1:7070 update-user -a <user_access_key_id> -s <user_secret_access_key> -ui <user_userID> -gi <user_groupID>
```

**Programmatically with Go**
```go
package main

import (
	"bytes"
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

type props struct {
	Secret  *string `json:"secret"`
	UserID  *int    `json:"userID"`
	GroupID *int    `json:"groupID"`
}

func main() {
	userAccess := "del_user_access_key"
	secret := "user_secret"
	userId := 5
	groupId := 12

	p := props{
		Secret:  &secret,
		UserID:  &userId,
		GroupID: &groupId,
	}

	propsJSON, err := json.Marshal(p)
	if err != nil {
		log.Fatalf("failed to parse user attributes: %w\n", err)
	}

	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/update-user?access=%v", adminEndpoint, userAccess), bytes.NewBuffer(propsJSON))
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256(propsJSON)
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %w\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("failed to read the response body: %w\n", err)
	}

	if resp.StatusCode >= 400 {
		log.Fatalf("request failed with message %s\n", body)
	}

	fmt.Printf("%s\n", body)
}
```

## List Users

The endpoint allows to list all the gateway users.

### Request

* Method: PATCH
* Endpoint: /list-users

### Responses

**Success(200 OK)**
```json
[
    {
        "access": "string",
        "secret": "string",
        "role": "user/userplus/admin",
        "userID": "number",
        "groupID": "number"
    },
    ...
]
```

**Error(403 Forbidden)**
```
access denied: only admin users have access to this resource
```

**Error(500 Internal Server Error)**
```
<any unexpected internal server error>
```

### Example Usage

**versitygw admin CLI**
```
versitygw admin -a <admin_access_key_id> -s <admin_secret_access_key> -er http://127.0.0.1:7070 list-users
```

**Programmatically with Go**
```go
package main

import (
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

type user struct {
	Access  string `json:"access"`
	Secret  string `json:"secret"`
	Role    string `json:"role"`
	UserID  int    `json:"userID"`
	GroupID int    `json:"groupID"`
}

func main() {
	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/list-users", adminEndpoint), nil)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256([]byte{})
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %w\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("failed to read the response body: %w\n", err)
	}

	if resp.StatusCode >= 400 {
		log.Fatalf("request failed with message %s\n", body)
	}

	var accs []user
	if err := json.Unmarshal(body, &accs); err != nil {
		log.Fatalf("failed to parse response body: %w\n", err)
	}

	fmt.Println(accs)
}
```

## Change Bucket Owner

The endpoint allows to change a bucket owner.

### Request

* Method: PATCH
* Endpoint: /change-bucket-owner?bucket=<bucket_name>&owner=<new_owner_access_key_id>

### Responses

**Success(200 OK)**
```
Bucket owner has been updated successfully
```

**Error(403 Forbidden)**
```
access denied: only admin users have access to this resource
```

**Error(404 Not Found)**
```
user specified as the new bucket owner does not exist
```

**Error(500 Internal Server Error)**
```
<any unexpected internal server error>
```

### Example Usage

**versitygw admin CLI**
```
versitygw admin -a <admin_access_key_id> -s <admin_secret_access_key> -er http://127.0.0.1:7070 change-bucket-owner -b <bucket_name> -o <new_owner_access>
```

**Programmatically with Go**
```go
package main

import (
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

func main() {
	bucket := "bucket_name"
	newOwner := "new_owner_access"

	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/change-bucket-owner/?bucket=%v&owner=%v", adminEndpoint, bucket, newOwner), nil)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256([]byte{})
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %w\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("failed to read the response body: %w\n", err)
	}

	if resp.StatusCode >= 400 {
		log.Fatalf("request failed with message %s\n", body)
	}

	fmt.Println(string(body))
}
```

## List Buckets and Owners

The endpoint allows to list all the buckets and their owners.

### Request

* Method: PATCH
* Endpoint: /list-buckets

### Responses

**Success(200 OK)**
```json
[
    {
        "name": "string",
        "owner": "string"
    }
    ...
]

```

**Error(403 Forbidden)**
```
access denied: only admin users have access to this resource
```

**Error(500 Internal Server Error)**
```
<any unexpected internal server error>
```

### Example Usage

**versitygw admin CLI**
```
versitygw admin -a <admin_access_key_id> -s <admin_secret_access_key> -er http://127.0.0.1:7070 list-buckets
```

**Programmatically with Go**
```go
package main

import (
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

type bucket struct {
	Name  string `json:"name"`
	Owner string `json:"owner"`
}

func main() {
	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/list-buckets", adminEndpoint), nil)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256([]byte{})
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %w\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %w\n", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("failed to read the response body: %w\n", err)
	}

	if resp.StatusCode >= 400 {
		log.Fatalf("request failed with message %s\n", body)
	}

	var buckets bucket
	if err := json.Unmarshal(body, &buckets); err != nil {
		log.Fatalf("failed to parse the response body: %w\n", err)
	}

	fmt.Println(buckets)
}
```