#### Enable a way (handler, command argument) returning git hash and build time of an app

```go
package main

var gitCommitHash string
var buildTime string

func main() {
    // return gitCommitHash and buildTime variables in a handler or if specific command argument is provided
}
```
There is a linker flag `-X` to write information into the variable at link time. It might be used in `Dockerfile` like the following:

```Dockerfile
# ...
RUN GITCOMMITHASH=`git rev-parse --short HEAD` BUILDTIME=`date -u '+%Y-%m-%dT%H:%M:%SZ'` && GOFLAGS="-w -s -X main.gitCommitHash=`echo $GITCOMMITHASH` -X main.buildTime=`echo $BUILDTIME`" && CGO_ENABLED=0 GOOS=linux go build -ldflags="$GOFLAGS" -a -o /go/bin/app .
# ...
```

#### Do not return error text to a user in a response body

```go
w.WriteHeader(http.StatusInternalServerError)
json.NewEncoder(w).Encode(errs(H{"description": "failed to do something: " + err.Error()}))
return
```

In such case error must be logged and a client should be provided with a `Reference`:

```go
logger.Error("failed to do something",
    zap.Error(err),    
    zap.String("reference", ref),
)
w.WriteHeader(http.StatusInternalServerError)
w.Header().Set("Reference", ref)
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
