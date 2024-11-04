The Admin APIs facilitate administrative tasks related to IAM user management, bucket ownership modifications, and bucket listings with their respective owners. These APIs ensure robust and secure interactions by requiring requests to be authenticated using `AWS Signature Version 4 (SigV4)`. Only **admin** or **root** users, possessing the necessary AWS access key ID and secret access key, can execute these operations.

The Admin APIs use XML encoding for all request and response bodies. This XML-based approach aligns with standard AWS API practices, providing a structured and consistent format for data exchange. Ensure that requests adhere to the specified XML structure to facilitate proper parsing and processing by the API.

Error responses follow a structure similar to AWS S3 errors, with error codes prefixed by `XAdmin` to distinguish them as admin-specific errors. This convention aids in differentiating standard AWS errors from `VersityGW` admin-specific issues. For detailed information on the error structure, refer to the [AWS S3 Error Responses documentation](https://docs.aws.amazon.com/AmazonS3/latest/API/ErrorResponses.html).

## Create user

The endpoint allows to create a new IAM user account.

### Request

* Method: PATCH
* Endpoint: /create-user
* Content-Type: application/xml

### Request Body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Account>
    <Access> string </Access>
    <Secret> string </Secret>
    <Role> role </Role>
    <UserID> number </UserID>
    <GroupID> number </GroupID>
</Account>
```

| Property | Type | Description | Required |
|:------:  | :---: | :---------:| :------: |
| Account | Account | the root level tag | yes |
| Access | string | user access key id | yes |
| Secret | string | user secret access key | yes |
| Role | string(user/userplus/admin) | user role | yes |
| UserID | number | user id | no |
| GroupID | number | user group id | no |

### Response Syntax

```
HTTP/1.1 201
```

### Error Responses

* `XAdminAccessDenied` - Only admin users have access to this resource.
* `XAdminUserExists` - A user with the provided access key ID already exists.
* `XAdminInvalidArgument` - User role has to be one of the following: 'user', 'admin', 'userplus'.
* `MalformedXML` - The XML you provided was not well-formed or did not validate against our published schema.
* `InternalError` - We encountered an internal error, please try again.

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
	"encoding/xml"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
	"github.com/aws/smithy-go"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

type Account struct {
	Access  string `xml:"Access"`
	Secret  string `xml:"Secret"`
	Role    string `xml:"Role"`
	UserID  int    `xml:"UserID"`
	GroupID int    `xml:"GroupID"`
}

func main() {
	acc := Account{
		Access:  "test_user_1",
		Secret:  "test_user_1_secret",
		Role:    "user",
		UserID:  1,
		GroupID: 3,
	}

	accxml, err := xml.Marshal(acc)
	if err != nil {
		log.Fatalf("failed to serialize user data: %v\n", err)
	}

	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/create-user", adminEndpoint), bytes.NewBuffer(accxml))
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256(accxml)
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %v\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}

	if resp.StatusCode >= 400 {
		defer resp.Body.Close()

		body, err := io.ReadAll(resp.Body)
		if err != nil {
			log.Fatalf("failed to read the request body: %v\n", err)
		}

		log.Fatalln(parseApiError(body))
	}
}

func parseApiError(body []byte) error {
	var apiErr smithy.GenericAPIError
	err := xml.Unmarshal(body, &apiErr)
	if err != nil {
		apiErr.Code = "InternalServerError"
		apiErr.Message = err.Error()
	}

	return &apiErr
}

```

## Delete User

The endpoint allows to delete an IAM user account:

### Request

* Method: PATCH
* Endpoint: /delete-user?access=<user_access_key_id>

### Response Syntax

```
HTTP/1.1 204
```

### Error Responses

* `XAdminAccessDenied` - Only admin users have access to this resource.
* `InternalError` - We encountered an internal error, please try again.

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
	"encoding/xml"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
	"github.com/aws/smithy-go"
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
		log.Fatalf("failed to send the request: %v\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256([]byte{})
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %v\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}

	if resp.StatusCode >= 400 {
		defer resp.Body.Close()

		body, err := io.ReadAll(resp.Body)
		if err != nil {
			log.Fatalf("failed to read the request body: %v\n", err)
		}

		log.Fatalln(parseApiError(body))
	}
}

func parseApiError(body []byte) error {
	var apiErr smithy.GenericAPIError
	err := xml.Unmarshal(body, &apiErr)
	if err != nil {
		apiErr.Code = "InternalServerError"
		apiErr.Message = err.Error()
	}

	return &apiErr
}

```

## Update User

The endpoint allows to update `secret`, `userID`, `groupID` properties in an IAM user account.

### Request

* Method: PATCH
* Endpoint: /update-user
* Content-Type: application/xml

### Request Body

```xml
<?xml version="1.0" encoding="UTF-8"?>
<MutableProps>
    <Secret> string </Secret>
    <UserID> number </UserID>
    <GroupID> number </GroupID>
</MutableProps>
```

| Property | Type | Description | Required |
|:------:  | :---: | :---------:| :------: |
| MutableProps | MutableProps | the root level tag | yes |
| Secret | string | user secret access key | no |
| UserID | number | user id | no |
| GroupID | number | user group id | no |

### Response Syntax

```
HTTP/1.1 200
```

### Error Responses

* `XAdminAccessDenied` - Only admin users have access to this resource.
* `XAdminInvalidArgument` - User access key ID is missing.
* `XAdminUserNotFound` - No user exists with the provided access key ID.
* `MalformedXML` - The XML you provided was not well-formed or did not validate against our published schema.
* `InternalError` - We encountered an internal error, please try again.

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
	"encoding/xml"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
	"github.com/aws/smithy-go"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

type MutableProps struct {
	Secret  *string `xml:"Secret"`
	UserID  *int    `xml:"UserID"`
	GroupID *int    `xml:"GroupID"`
}

func main() {
	userAccess := "del_user_access_key"
	secret := "user_secret"
	userId := 5
	groupId := 12

	p := MutableProps{
		Secret:  &secret,
		UserID:  &userId,
		GroupID: &groupId,
	}

	propsxml, err := xml.Marshal(p)
	if err != nil {
		log.Fatalf("failed to parse user attributes: %v\n", err)
	}

	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/update-user?access=%v", adminEndpoint, userAccess), bytes.NewBuffer(propsxml))
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256(propsxml)
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %v\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}
	if resp.StatusCode >= 400 {
		defer resp.Body.Close()

		body, err := io.ReadAll(resp.Body)
		if err != nil {
			log.Fatalf("failed to read the request body: %v\n", err)
		}

		log.Fatalln(parseApiError(body))
	}
}

func parseApiError(body []byte) error {
	var apiErr smithy.GenericAPIError
	err := xml.Unmarshal(body, &apiErr)
	if err != nil {
		apiErr.Code = "InternalServerError"
		apiErr.Message = err.Error()
	}

	return &apiErr
}

```

## List Users

The endpoint allows to list all the gateway users.

### Request

* Method: PATCH
* Endpoint: /list-users

### Response Syntax

```xml
HTTP/1.1 200
<?xml version="1.0" encoding="UTF-8"?>
<ListUserAccountsResult>
    <Account>
        <Access> string </Access>
        <Secret> string </Secret>
        <Role> role </Role>
        <UserID> number </UserID>
        <GroupID> number </GroupID>
    </Account>
</ListUserAccountsResult>
```

### Error Responses

* `XAdminAccessDenied` - Only admin users have access to this resource.
* `InternalError` - We encountered an internal error, please try again.

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
	"encoding/xml"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
	"github.com/aws/smithy-go"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

type Account struct {
	Access  string `xml:"Access"`
	Secret  string `xml:"Secret"`
	Role    string `xml:"Role"`
	UserID  int    `xml:"UserID"`
	GroupID int    `xml:"GroupID"`
}

type ListUserAccountsResult struct {
	Accounts []Account
}

func main() {
	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/list-users", adminEndpoint), nil)
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256([]byte{})
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %v\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("failed to read the response body: %v\n", err)
	}

	if resp.StatusCode >= 400 {
		log.Fatalln(parseApiError(body))
	}

	var result ListUserAccountsResult
	if err := xml.Unmarshal(body, &result); err != nil {
		log.Fatalf("failed to parse response body: %v\n", err)
	}

	fmt.Println(result)
}

func parseApiError(body []byte) error {
	var apiErr smithy.GenericAPIError
	err := xml.Unmarshal(body, &apiErr)
	if err != nil {
		apiErr.Code = "InternalServerError"
		apiErr.Message = err.Error()
	}

	return &apiErr
}

```

## Change Bucket Owner

The endpoint allows to change a bucket owner.

### Request

* Method: PATCH
* Endpoint: /change-bucket-owner?bucket=<bucket_name>&owner=<new_owner_access_key_id>

### URI Request Parameters

| Parameter | Description | Required |
|:---------:| :---------: | :------: |
| bucket | the bucket name | yes |
| owner | the new owner access key ID | yes |

### Response Syntax

```
HTTP/1.1 204
```

### Error Responses

* `XAdminAccessDenied` - Only admin users have access to this resource.
* `XAdminUserNotFound` - No user exists with the provided access key ID.
* `NoSuchBucket` - The specified bucket does not exist.
* `InternalError` - We encountered an internal error, please try again.

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
	"encoding/xml"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
	"github.com/aws/smithy-go"
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
		log.Fatalf("failed to send the request: %v\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256([]byte{})
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %v\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}
	if resp.StatusCode >= 400 {
		defer resp.Body.Close()

		body, err := io.ReadAll(resp.Body)
		if err != nil {
			log.Fatalf("failed to read the request body: %v\n", err)
		}

		log.Fatalln(parseApiError(body))
	}
}

func parseApiError(body []byte) error {
	var apiErr smithy.GenericAPIError
	err := xml.Unmarshal(body, &apiErr)
	if err != nil {
		apiErr.Code = "InternalServerError"
		apiErr.Message = err.Error()
	}

	return &apiErr
}

```

## List Buckets and Owners

The endpoint allows to list all the buckets and their owners.

### Request

* Method: PATCH
* Endpoint: /list-buckets

### Response Syntax

```xml
HTTP/1.1 200
<?xml version="1.0" encoding="UTF-8"?>
<ListBucketsResult>
    <Bucket>
        <Name> string </Name>
        <Owner> string </Owner>
    </Bucket>
</ListBucketsResult>
```

### Error Responses

* `XAdminAccessDenied` - Only admin users have access to this resource.
* `InternalError` - We encountered an internal error, please try again.

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
	"encoding/xml"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/aws/aws-sdk-go-v2/aws"
	v4 "github.com/aws/aws-sdk-go-v2/aws/signer/v4"
	"github.com/aws/smithy-go"
)

const (
	adminEndpoint = "http://127.0.0.1:7070"
	adminAccess   = "admin_access_key_id"
	adminSecret   = "admin_secret_access_key"
	serverRegion  = "us-east-1"
)

type Bucket struct {
	Name  string `xml:"Name"`
	Owner string `xml:"Owner"`
}

type ListBucketsResult struct {
	Buckets []Bucket `xml:"Bucket"`
}

func main() {
	req, err := http.NewRequest(http.MethodPatch, fmt.Sprintf("%v/list-buckets", adminEndpoint), nil)
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}

	signer := v4.NewSigner()

	hashedPayload := sha256.Sum256([]byte{})
	hexPayload := hex.EncodeToString(hashedPayload[:])

	req.Header.Set("X-Amz-Content-Sha256", hexPayload)

	err = signer.SignHTTP(req.Context(), aws.Credentials{AccessKeyID: adminAccess, SecretAccessKey: adminSecret}, req, hexPayload, "s3", serverRegion, time.Now())
	if err != nil {
		log.Fatalf("failed to sign the request: %v\n", err)
	}

	client := &http.Client{}

	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("failed to send the request: %v\n", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("failed to read the response body: %v\n", err)
	}

	if resp.StatusCode >= 400 {
		log.Fatalln(parseApiError(body))
	}

	var result ListBucketsResult
	if err := xml.Unmarshal(body, &result); err != nil {
		log.Fatalf("failed to parse the response body: %v\n", err)
	}

	fmt.Println(result)
}

func parseApiError(body []byte) error {
	var apiErr smithy.GenericAPIError
	err := xml.Unmarshal(body, &apiErr)
	if err != nil {
		apiErr.Code = "InternalServerError"
		apiErr.Message = err.Error()
	}

	return &apiErr
}

```