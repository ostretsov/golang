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
w.WriteHeader(http.StatusBadRequest)
json.NewEncoder(w).Encode(map[string]string{
    "code": "1",
    "description": "failed to decode json body",
})
return
```
