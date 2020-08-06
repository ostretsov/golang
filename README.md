#### Do not return error text to a user in a response body

```go
w.WriteHeader(http.StatusInternalServerError)
json.NewEncoder(w).Encode(errs(H{"description": "failed to do something: " + err.Error()}))
return
```

In such a case error must be logged and a client should be provided with a `Reference`:

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
