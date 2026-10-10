# Go Web Server Notes

Notes mapped to sections of [main.go](main.go).

| Section | Code | Topic |
|---|---|---|
| 1 | `main()` lines 34-37 | `Handle` vs `HandleFunc` |
| 2 | `main()` line 40 | `ListenAndServe` and the handler (`nil`) |
| 3 | `hellohandler` lines 21-31 | Path check and 404 |
| 4 | `hellohandler` params / `formhandler` | `w` (ResponseWriter) and `r` (Request) |
| 5 | Handler signatures | Why handlers have no return value |
| 6 | `formhandler` lines 9-19 | *(no notes yet)* |

---

## 1. Registering routes: `http.Handle` vs `http.HandleFunc`

**Code:** [main.go:34-37](main.go#L34-L37)

```go
fileServer := http.FileServer(http.Dir("./static"))
http.Handle("/", fileServer)
http.HandleFunc("/form", formhandler)
http.HandleFunc("/hello", hellohandler)
```

### `http.HandleFunc`: "just call this function"

Use it when you point at a regular, standalone function you wrote, and tell Go to run it when someone visits a path.

- **Analogy:** a buzzer button at a reception desk. You program it: "when pressed, run the `ringBell` function."
- **In code:** `http.HandleFunc("/form", formhandler)` means "if a user goes to `yourwebsite.com/form`, run my function `formhandler`."

### `http.Handle`: "use this special object"

Use it when you have a pre-built object (data plus code) that already knows how to handle web requests.

- **Analogy:** instead of writing the logic yourself, you hire a professional who brings their own toolbox and rules. You only tell them which door to guard.
- **In code:** `http.Handle("/", fileServer)`. `fileServer` is not a simple function. It is a built-in Go system that handles files on disk, security checks, and folder structures. Because it is an object, you register it with `http.Handle`.

---

## 2. Starting the server: `ListenAndServe` and the handler

**Code:** [main.go:40](main.go#L40)

```go
if err := http.ListenAndServe(":8080", nil); err != nil {
    log.Fatal(err)
}
```

### What is the handler?

The handler is the "brain" or master routing controller of the server. When a visitor arrives, `ListenAndServe` opens the door but doesn't know where to send them. It hands the visitor to the handler: "someone is here, you figure out what they want."

Its only job is to look at the URL (like `/hello` or `/form`) and decide which code or file to serve.

### Why pass `nil`?

The docs say: *"The handler is typically nil, in which case DefaultServeMux is used."*

- `DefaultServeMux` is Go's built-in default router.
- The earlier `http.Handle` / `http.HandleFunc` calls (lines 35-37) secretly registered the `/`, `/form` and `/hello` routes inside it.
- Passing `nil` means "don't build a custom router, use the default one where I already saved my routes."

**Analogy:** `ListenAndServe` is the main entrance of an office building.
- Pass a custom handler: you hire your own receptionist.
- Pass `nil`: you use the building's default receptionist (`DefaultServeMux`), who already has the directory of where `/hello` and `/form` are.

---

## 3. The path check in `hellohandler`

**Code:** [main.go:21-25](main.go#L21-L25)

```go
func hellohandler(w http.ResponseWriter, r *http.Request) {
    if r.URL.Path != "/hello" {
        http.Error(w, "404 not found", http.StatusNotFound)
        return
    }
    ...
}
```

This ensures the user only gets the response if they visit exactly `/hello`.

| Part | Meaning |
|---|---|
| `hellohandler` | The function name |
| `w http.ResponseWriter` | Builds and sends the response back to the client (e.g. a browser) |
| `r *http.Request` | The incoming request: URL, headers, form data, etc. |
| `r.URL.Path != "/hello"` | A guardrail. Go's default router can route paths like `/hello/world` to this handler, so we check for an exact match |
| `http.Error(...)` | Helper that sets the status to 404 (`http.StatusNotFound`) and writes the message `"404 not found"` |
| `return` | Exits immediately so the logic below doesn't run for the wrong URL |

**In plain English:** "If the URL path is anything other than exactly `/hello`, send a 404 and stop right there."

The next check ([main.go:26-29](main.go#L26-L29)) works the same way for the HTTP method: anything other than `GET` is rejected.

---

## 4. `w` (ResponseWriter) and `r` (Request)

**Code:** [main.go:9](main.go#L9) and [main.go:21](main.go#L21) (handler parameters), with usages at lines 11, 14, 17-18, 23, 27 and 30.

`w` is the `http.ResponseWriter` interface: the channel used to send data back to the client (browser, mobile app, etc.).

**Analogy:** `r` is an incoming letter from the user. `w` is a blank sheet of paper where you write your reply.

`w` is used in two ways:

1. **Writing error responses**
   ```go
   http.Error(w, "404 not found", http.StatusNotFound)
   ```
   Writes the error message and status code into the response.

2. **Writing the page content**
   ```go
   fmt.Fprintf(w, "Hello there!")
   ```
   `fmt.Fprintf` takes an output destination as its first argument. Passing `w` formats the string and writes it into the response body, which the user sees on screen.

---

## 5. Why handlers don't `return` a value

**Code:** handler signatures at [main.go:9](main.go#L9) and [main.go:21](main.go#L21), registered at [main.go:36-37](main.go#L36-L37).

### The output goes through `w`, not a return statement

A normal function like `add` calculates a number and hands it back to other code. A web handler's output is meant for a human across the internet, not for your code, so you stream it out through `w`.

### The signature is a strict "job description"

Go's web server requires handlers to match this exact blueprint:

```go
func(http.ResponseWriter, *http.Request)
```

Because `hellohandler` matches it, you can pass it to `http.HandleFunc("/hello", hellohandler)`. If you added a return type (e.g. `func hellohandler(...) string`), Go would refuse to compile, because it wouldn't know what to do with the returned value.

### Summary

- **Standard functions (`add`):** data in, data out, used elsewhere in your code.
- **Web handlers:** the request comes in, and you use `w` as a megaphone to shout the response out to the internet.

---

## 6. `formhandler`: no notes yet

**Code:** [main.go:9-19](main.go#L9-L19)

This handler calls `r.ParseForm()`, then reads `name` and `address` with `r.FormValue(...)` and writes them back via `w`. Add notes here when you've covered it.
