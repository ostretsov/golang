#### Enable a way (handler, command argument) returning git hash and build time of an app

```go
package main

var gitCommitHash string
var buildTime string

func main() {
    // Return values of gitCommitHash and buildTime in a handler, or print them in stdout if specific command argument is provided.
}
```
There is a linker flag `-X` to write information into the variable at link time. It might be used in `Dockerfile` like the following:

```Dockerfile
# ...
RUN GITCOMMITHASH=`git rev-parse --short HEAD` BUILDTIME=`date -u '+%Y-%m-%dT%H:%M:%SZ'` && GOFLAGS="-w -s -X main.gitCommitHash=`echo $GITCOMMITHASH` -X main.buildTime=`echo $BUILDTIME`" && GOOS=linux go build -ldflags="$GOFLAGS" -a -o /go/bin/app .
# ...
```

#### TODO: Add probe endpoint for k8s

#### Generate short UUID reference ID or use `Reference-ID` of an incoming HTTP-request if it's specified

```go
package middleware

import (
	"context"
	"github.com/lithammer/shortuuid/v3"
	"net/http"
)

type referenceID string
var referenceIDKey = referenceID("reference_id")

func ReferenceID(handler http.Handler) http.HandlerFunc {
	return func(w http.ResponseWriter, req *http.Request) {
		refID := req.Header.Get("Reference-ID")
		if refID == "" {
			refID = shortuuid.New()
		}
		ctx := context.WithValue(req.Context(), referenceIDKey, refID)
		req = req.WithContext(ctx)

		w.Header().Set("Reference-ID", refID)

		handler.ServeHTTP(w, req)
	}
}

func GetReferenceID(r *http.Request) string {
	val := r.Context().Value(referenceIDKey)
	if val == nil {
		return ""
	}
	return val.(string)
}
```

#### Do not return error text to a user in a response body

```go
w.WriteHeader(http.StatusInternalServerError)
json.NewEncoder(w).Encode(errs(H{"description": "failed to do something: " + err.Error()}))
return
```

In such a case error must be logged and a client should be provided with the `Reference-ID` header:

```go
logger.Error("failed to do something",
    zap.Error(err),    
    zap.String("reference_id", ref),
)
w.WriteHeader(http.StatusInternalServerError)
w.Header().Set("Reference-ID", ref)
json.NewEncoder(w).Encode(errs(H{"description": "failed to do something"}))
return
```

#### Always return an error code on 400 HTTP status code (http.StatusBadRequest)

An error code is easily machinable by a client of an API.

```go
err := json.NewDecoder(r.Body).Decode(&reqBody)
if err != nil {
    w.WriteHeader(http.StatusBadRequest)
    json.NewEncoder(w).Encode(errs(H{
        "code": "1",
        "description": "failed to decode json body",
    }))
    return
}
```
