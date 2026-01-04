# Golang Learning Notes

## Fundamental Language Characteristics

### Strongly Typed Language

Go is a **strongly typed** language, which means:

- **Type safety is enforced**: Once a variable is declared with a specific type, you cannot assign a value of a different type to it without explicit conversion
- **No implicit type coercion**: Go will not automatically convert between types (e.g., you can't add an integer to a string without converting one of them first)
- **Catches type errors at compile time**: Most type-related errors are caught before your program runs, preventing many common bugs

**Example:**
```go
var age int = 25
var name string = "Marko"

// This will cause a compile error:
// age = name  // cannot use name (type string) as type int

// You must explicitly convert:
// age = int(name)  // This would still fail because you can't convert string "Marko" to int
```

**Benefits:**
- Fewer runtime errors
- Better code reliability
- Easier to refactor code with confidence
- IDE and tooling can provide better autocomplete and error detection

---

### Statically Typed Language

Go is a **statically typed** language, which means:

- **Types are determined at compile time**: Variable types must be known when the code is compiled, not when it runs
- **Type checking happens before execution**: The compiler verifies all type information before generating the executable
- **Explicit or inferred type declarations**: You can declare types explicitly (`var x int = 5`) or let the compiler infer them (`x := 5`)

**Example:**
```go
// Explicit type declaration
var count int = 10

// Type inference (compiler determines type from the value)
message := "Hello, Go!"  // message is inferred as string
isActive := true         // isActive is inferred as bool

// Once declared, the type cannot change
count = 20        // OK - still an int
// count = "20"   // ERROR - cannot assign string to int variable
```

**Note:** Static typing (Go, Java, C++) checks types at compile time, while dynamic typing (Python, JavaScript, PHP) checks at runtime.

---

### Cross-Platform

Go is **cross-platform**, which means:

- **Write once, compile anywhere**: The same Go source code can be compiled to run on different operating systems and architectures
- **Native support for multiple targets**: Go can compile binaries for Windows, Linux, macOS, BSD, and more
- **Easy cross-compilation**: You can build executables for different platforms from a single development machine

**Example - Cross-compilation:**
```bash
# On Linux, compile for Windows
GOOS=windows GOARCH=amd64 go build -o myapp.exe

# On macOS, compile for Linux
GOOS=linux GOARCH=amd64 go build -o myapp

# On any platform, compile for ARM (e.g., Raspberry Pi)
GOOS=linux GOARCH=arm go build -o myapp-arm
```

**Supported platforms include:**
- **Operating Systems**: Linux, Windows, macOS, FreeBSD, OpenBSD, Plan 9, and more
- **Architectures**: amd64 (x86-64), 386 (x86), arm, arm64, and others

**Benefits:**
- Deploy the same codebase to different environments
- No need to maintain platform-specific code (in most cases)
- Single binary with no dependencies makes deployment simple

---

### Compiled Language

Go is a **compiled language**, which means:

- **Source code is translated to machine code**: The Go compiler converts your `.go` files directly into executable binaries
- **No interpreter needed**: The resulting binary runs directly on the target machine without requiring Go to be installed
- **Fast execution**: Compiled code runs much faster than interpreted languages because it's already in machine code
- **Standalone binaries**: Go produces single, self-contained executable files with no external dependencies (by default)

**Compilation process:**
```bash
# Compile a Go program
go build main.go

# This creates an executable binary (e.g., 'main' or 'main.exe')
# You can run it directly:
./main  # On Linux/macOS
main.exe  # On Windows
```

**Note:** Compiled languages (Go, C, Rust) translate to machine code before execution, while interpreted languages (Python, JavaScript, PHP) execute code line by line at runtime.

**Go's compilation benefits:**
- **Very fast compilation**: Despite being compiled, Go compiles incredibly quickly (feels almost like an interpreted language during development)
- **Static linking by default**: All dependencies are included in the binary (no "dependency hell")
- **Easy deployment**: Just copy the single binary file to your server
- **Performance**: Execution speed comparable to C/C++

**Example workflow:**
```go
// main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go!")
}
```

```bash
# Compile
go build main.go

# Run the compiled binary
./main
# Output: Hello, Go!
```

---

### Visibility Through Naming Convention

Go uses a **unique approach to visibility** (public/private access control) based on the **capitalization of the first letter** of identifiers:

- **Capitalized** (starts with uppercase letter) → **Exported** (public, visible outside the package)
- **Lowercase** (starts with lowercase letter) → **Unexported** (private, only visible within the package)

This applies to:
- Functions
- Variables
- Constants
- Types (structs, interfaces)
- Struct fields
- Methods

**Example:**
```go
package mypackage

// Exported (Public) - can be used by other packages
func ProcessData() {
    // ...
}

var MaxConnections int = 100

type User struct {
    Name  string  // Exported field - accessible from other packages
    Email string  // Exported field
    age   int     // Unexported field - only accessible within mypackage
}

// Unexported (Private) - only usable within mypackage
func validateEmail(email string) bool {
    // ...
    return true
}

var internalCounter int = 0

type config struct {
    apiKey string
    timeout int
}
```

**Using the exported items from another package:**
```go
package main

import "myproject/mypackage"

func main() {
    // ✓ Can use exported function
    mypackage.ProcessData()
    
    // ✓ Can access exported variable
    maxConn := mypackage.MaxConnections
    
    // ✓ Can create exported type
    user := mypackage.User{
        Name:  "Marko",
        Email: "marko@example.com",
        // age: 30,  // ✗ ERROR - cannot access unexported field
    }
    
    // ✓ Can access exported fields
    println(user.Name)
    
    // ✗ ERROR - cannot use unexported function
    // mypackage.validateEmail("test@example.com")
    
    // ✗ ERROR - cannot access unexported variable
    // mypackage.internalCounter
}
```

**Key Points:**

1. **Package-level visibility**: Unlike many languages (Java, C++, PHP), Go doesn't have `public`, `private`, or `protected` keywords. Visibility is controlled purely by naming.

2. **No class-level private**: In Go, there are no classes. Unexported items are private to the *package*, not to a specific type or struct.

3. **Enforced by the compiler**: The Go compiler will produce errors if you try to access unexported identifiers from outside their package.

4. **Package = directory**: A package consists of all `.go` files in the same directory. All files in that directory can access each other's unexported items.

**Note:** Unlike languages with `public`, `private`, `protected` keywords (PHP, Java, C++), Go uses capitalization for visibility control.

**Benefits of Go's approach:**
- **Simple and consistent**: One rule to remember - capital = public, lowercase = private
- **No keyword clutter**: Cleaner, more readable code
- **Visible at a glance**: You can immediately see what's public just by looking at the name
- **Package-based encapsulation**: Encourages good package design and clear boundaries

**Best practices:**
- Export only what needs to be used by other packages (minimize public API surface)
- Use descriptive names for exported items since they form your package's public interface
- Keep implementation details unexported to allow internal refactoring without breaking consumers

---

### Defer Statement

Go's **defer** statement schedules a function call to be executed **after the surrounding function returns**, regardless of how it returns (normal return, early return, or panic).

**Key characteristics:**
- **Always executes**: Deferred functions run even if the function returns early or panics
- **LIFO order**: Multiple defers execute in Last-In-First-Out (stack) order
- **Arguments evaluated immediately**: Function arguments are evaluated when defer is called, not when it executes

**Basic usage:**
```go
func processFile() error {
    file, err := os.Open("data.txt")
    if err != nil {
        return err
    }
    defer file.Close()  // Ensures file is closed when function exits
    
    // Do work with file...
    data, err := io.ReadAll(file)
    if err != nil {
        return err  // file.Close() still called before return
    }
    
    // More processing...
    return nil  // file.Close() called here too
}
```

**Handles early returns:**
```go
func validateAndProcess(id int) error {
    defer fmt.Println("Cleanup completed")
    
    if id < 0 {
        return errors.New("invalid id")  // defer still executes
    }
    
    if id > 1000 {
        return errors.New("id too large")  // defer still executes
    }
    
    // Normal processing...
    return nil  // defer executes here too
}
```

**Handles panics:**
```go
func safeOperation() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
        fmt.Println("Cleanup always runs")
    }()
    
    // This will panic
    panic("something went wrong")
    
    // This line never executes, but defer still runs
    fmt.Println("This won't print")
}
```

**Multiple defers (LIFO order):**
```go
func demonstrateLIFO() {
    defer fmt.Println("First defer")
    defer fmt.Println("Second defer")
    defer fmt.Println("Third defer")
    
    fmt.Println("Function body")
}

// Output:
// Function body
// Third defer
// Second defer
// First defer
```

**Common use cases:**

1. **Resource cleanup:**
```go
func databaseOperation() error {
    db, err := sql.Open("postgres", connString)
    if err != nil {
        return err
    }
    defer db.Close()
    
    // Use database...
    return nil
}
```

2. **Unlocking mutexes:**
```go
func updateCounter() {
    mu.Lock()
    defer mu.Unlock()
    
    counter++
    // Mutex automatically unlocked even if panic occurs
}
```

3. **Timing functions:**
```go
func expensiveOperation() {
    start := time.Now()
    defer func() {
        fmt.Printf("Took %v\n", time.Since(start))
    }()
    
    // Do work...
}
```

4. **Transaction handling:**
```go
func transferMoney() error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    
    defer func() {
        if err != nil {
            tx.Rollback()
        } else {
            tx.Commit()
        }
    }()
    
    // Perform operations...
    return nil
}
```

**Important notes:**
- Deferred function arguments are evaluated immediately when defer is called
- Defer has a small performance cost, but it's usually negligible
- Defer runs before return values are passed to caller (can modify named return values)

**Defer in loops - CRITICAL WARNING:**

Defer inside a loop defers the execution **for all iterations**, not per iteration. All deferred calls accumulate and execute when the function returns.

```go
// PROBLEMATIC - defers accumulate
func processFiles(filenames []string) {
    for _, filename := range filenames {
        file, err := os.Open(filename)
        if err != nil {
            continue
        }
        defer file.Close()  // ALL files stay open until function returns!
        
        // Process file...
    }
    // Only here do ALL defers execute
}
```

**Problem:** With 1000 files, all 1000 will remain open until the function exits, potentially causing resource exhaustion.

**Solution 1 - Use anonymous function:**
```go
func processFiles(filenames []string) {
    for _, filename := range filenames {
        func() {
            file, err := os.Open(filename)
            if err != nil {
                return
            }
            defer file.Close()  // Executes at end of THIS iteration
            
            // Process file...
        }()
    }
}
```

**Solution 2 - Explicit cleanup:**
```go
func processFiles(filenames []string) {
    for _, filename := range filenames {
        file, err := os.Open(filename)
        if err != nil {
            continue
        }
        
        // Process file...
        
        file.Close()  // Clean up immediately
    }
}
```

**Solution 3 - Extract to separate function:**
```go
func processFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()  // Executes when processFile returns
    
    // Process file...
    return nil
}

func processFiles(filenames []string) {
    for _, filename := range filenames {
        processFile(filename)  // Defer executes per file
    }
}
```

**Note:** Similar to `finally` blocks in languages like Java and PHP, but more flexible and integrated into the language.

---

### Runes - Lower Level Than Strings

Go has **runes**, which represent individual Unicode code points and are more fundamental than strings.

**Key concepts:**
- **Rune type**: `rune` is an alias for `int32` and represents a single Unicode character
- **Strings are sequences of bytes**: A string in Go is a read-only slice of bytes (UTF-8 encoded)
- **Runes handle Unicode properly**: Unlike byte indexing, runes correctly handle multi-byte Unicode characters

**Basic usage:**
```go
var char rune = 'A'        // Single quotes for rune literals
var emoji rune = '😀'
var cyrillic rune = 'Ж'

fmt.Printf("%c\n", char)   // Prints: A
fmt.Printf("%d\n", char)   // Prints: 65 (Unicode code point)
```

**Strings vs Bytes vs Runes:**
```go
s := "Hello, 世界"

// Length in bytes
fmt.Println(len(s))              // 13 bytes (not 9 characters!)

// Iterating as bytes (wrong for Unicode)
for i := 0; i < len(s); i++ {
    fmt.Printf("%c ", s[i])      // Breaks on multi-byte characters
}

// Iterating as runes (correct for Unicode)
for i, r := range s {
    fmt.Printf("Index: %d, Rune: %c, Value: %d\n", i, r, r)
}

// Converting to rune slice
runes := []rune(s)
fmt.Println(len(runes))          // 9 characters
fmt.Printf("%c\n", runes[7])     // 世
```

**Why runes matter:**
```go
text := "café"

// Byte length vs character count
fmt.Println(len(text))                    // 5 bytes (é is 2 bytes in UTF-8)
fmt.Println(len([]rune(text)))            // 4 characters

// Indexing strings gives bytes, not characters
fmt.Printf("%c\n", text[3])               // � (invalid, mid-character)

// Convert to runes for proper character access
runes := []rune(text)
fmt.Printf("%c\n", runes[3])              // é (correct)
```

**Common operations:**
```go
// String to runes
str := "Hello"
runeSlice := []rune(str)

// Runes to string
runes := []rune{'H', 'e', 'l', 'l', 'o'}
str = string(runes)

// Single rune to string
r := 'A'
str = string(r)

// Check if character is in range
var r rune = 'Ж'
if r >= 'А' && r <= 'я' {
    fmt.Println("Cyrillic character")
}
```

**Practical examples:**
```go
// Reverse a Unicode string correctly
func reverseString(s string) string {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}

// Count actual characters (not bytes)
func countChars(s string) int {
    return len([]rune(s))
}

// Get substring by character position
func substring(s string, start, length int) string {
    runes := []rune(s)
    if start >= len(runes) {
        return ""
    }
    end := start + length
    if end > len(runes) {
        end = len(runes)
    }
    return string(runes[start:end])
}
```

**Important notes:**
- Strings are immutable in Go (cannot modify individual bytes)
- Rune conversion creates a new slice (has memory cost)
- Use `range` over strings to iterate by runes automatically
- For ASCII-only strings, bytes and runes are equivalent
- For international text, always think in runes, not bytes

**Note:** Unlike languages where strings are character arrays, Go strings are byte slices requiring rune conversion for proper Unicode handling.

---

### Composition Over OOP

Go is **not an object-oriented programming (OOP) language** - instead, it uses **composition** to build complex types and behaviors.

**What Go doesn't have:**
- No classes
- No inheritance
- No class hierarchies
- No constructors (uses factory functions instead)
- No `extends` or `implements` keywords

**What Go uses instead:**
- **Structs** for data structures
- **Interfaces** for polymorphism
- **Embedding** for composition
- **Methods** on types (not just structs)

**Composition through struct embedding:**
```go
// Instead of inheritance, embed types
type Engine struct {
    Horsepower int
    Type       string
}

func (e Engine) Start() {
    fmt.Println("Engine starting...")
}

type Car struct {
    Engine  // Embedded - Car "has-a" Engine, not "is-a" Engine
    Brand   string
    Model   string
}

// Car automatically gets Engine's methods
car := Car{
    Engine: Engine{Horsepower: 200, Type: "V6"},
    Brand:  "Toyota",
    Model:  "Camry",
}

car.Start()                    // Can call Engine's method directly
fmt.Println(car.Horsepower)    // Can access Engine's fields directly
```

**Multiple composition:**
```go
type GPS struct {
    Location string
}

func (g GPS) Navigate() {
    fmt.Println("Navigating to", g.Location)
}

type Radio struct {
    Station string
}

func (r Radio) Play() {
    fmt.Println("Playing", r.Station)
}

type ModernCar struct {
    Engine
    GPS
    Radio
    Brand string
}

car := ModernCar{
    Engine: Engine{Horsepower: 250, Type: "V8"},
    GPS:    GPS{Location: "Home"},
    Radio:  Radio{Station: "FM 101"},
    Brand:  "Tesla",
}

car.Start()      // From Engine
car.Navigate()   // From GPS
car.Play()       // From Radio
```

**Interfaces for polymorphism:**
```go
// Define behavior without implementation
type Speaker interface {
    Speak() string
}

// Different types implement the interface
type Dog struct {
    Name string
}

func (d Dog) Speak() string {
    return "Woof!"
}

type Cat struct {
    Name string
}

func (c Cat) Speak() string {
    return "Meow!"
}

// Use interface for polymorphic behavior
func MakeSound(s Speaker) {
    fmt.Println(s.Speak())
}

dog := Dog{Name: "Buddy"}
cat := Cat{Name: "Whiskers"}

MakeSound(dog)  // Woof!
MakeSound(cat)  // Meow!
```

**No explicit "implements":**
```go
// A type satisfies an interface automatically
// if it has the required methods (implicit implementation)

type Writer interface {
    Write([]byte) (int, error)
}

type FileLogger struct {
    filename string
}

// FileLogger implements Writer automatically
// just by having the Write method with matching signature
func (f FileLogger) Write(data []byte) (int, error) {
    // Implementation...
    return len(data), nil
}

// No need to declare "implements Writer"
var w Writer = FileLogger{filename: "log.txt"}
```

**To adhere to an interface:**
Simply write receiver functions on your struct that match the interface method signatures. That's it - no keywords, no explicit declaration.

```go
// Define interface
type Speaker interface {
    Speak() string
    Listen() string
}

// Any type with these methods automatically satisfies Speaker
type Person struct {
    name string
}

func (p Person) Speak() string {
    return "Hello from " + p.name
}

func (p Person) Listen() string {
    return p.name + " is listening"
}

// Person now implements Speaker - automatically!
var s Speaker = Person{name: "Marko"}
```

**The compiler checks at compile time:**
```go
type Greeter interface {
    Greet() string
}

type Robot struct {
    id int
}

func (r Robot) Greet() string {
    return "Beep boop"
}

// This works - Robot has Greet() method
var g Greeter = Robot{id: 1}

// This would fail at compile time if Robot didn't have Greet()
// "cannot use Robot literal (type Robot) as type Greeter in assignment:
//  Robot does not implement Greeter (missing Greet method)"
```

**Multiple interfaces, same type:**
```go
type Reader interface {
    Read() string
}

type Writer interface {
    Write(string)
}

type File struct {
    content string
}

func (f *File) Read() string {
    return f.content
}

func (f *File) Write(data string) {
    f.content = data
}

// File implements BOTH Reader and Writer automatically
var r Reader = &File{}
var w Writer = &File{}
```

**Factory functions instead of constructors:**
```go
type User struct {
    id    int
    name  string
    email string
}

// Convention: New* functions act as constructors
func NewUser(name, email string) *User {
    return &User{
        id:    generateID(),
        name:  name,
        email: email,
    }
}

// Usage
user := NewUser("Marko", "marko@example.com")
```

**Benefits of composition:**
- **Flexibility**: Compose types from smaller, reusable components
- **No fragile base class problem**: Changes to embedded types don't break hierarchy
- **Clear dependencies**: You see exactly what's included in a type
- **Multiple "inheritance"**: Embed multiple types without diamond problem
- **Simpler mental model**: "has-a" relationships are clearer than "is-a"

**Important notes:**
- Embedding is not inheritance - it's composition
- Embedded fields can be accessed directly, but they're still separate
- Name conflicts in embedded types must be resolved explicitly
- Interfaces are satisfied implicitly (no explicit declaration needed)

**Note:** Unlike class-based OOP languages (Java, C++, PHP), Go favors composition and interfaces over inheritance hierarchies.

---

### Maps Are Unordered - Don't Sort Them

**Important principle: No one should be sorting a map in Go.**

Maps in Go are **intentionally unordered** data structures. The iteration order is randomized by design.

**Key characteristics:**
- **No guaranteed order**: Maps do not maintain insertion order or any other order
- **Randomized iteration**: Each iteration over a map may produce a different order
- **Intentional design**: This prevents developers from relying on map order

**Why maps are unordered:**
```go
m := map[string]int{
    "apple":  1,
    "banana": 2,
    "cherry": 3,
}

// Order is unpredictable and may change between runs
for key, value := range m {
    fmt.Println(key, value)
}
// Output could be: banana, apple, cherry
// Or: cherry, banana, apple
// Or any other order
```

**Go maps vs PHP associative arrays:**

Go maps are pure hash maps with NO order guaranteed:
```go
// Pure hash map - NO order guaranteed
m := map[string]int{"b": 2, "a": 1, "c": 3}
// Iteration order: RANDOM and changes between runs
```

PHP arrays are ordered hash tables - insertion order ALWAYS preserved:
```php
// Ordered hash table - insertion order ALWAYS preserved
$arr = ["b" => 2, "a" => 1, "c" => 3];
foreach ($arr as $k => $v) {
    // Always iterates: b, a, c (insertion order)
}
```

This is a fundamental difference - PHP arrays maintain insertion order automatically, while Go maps intentionally do not.

**If you need order, use the right data structure:**

**Option 1: Slice of keys (most common)**
```go
m := map[string]int{
    "banana": 2,
    "apple":  1,
    "cherry": 3,
}

// Extract and sort keys
keys := make([]string, 0, len(m))
for key := range m {
    keys = append(keys, key)
}
sort.Strings(keys)

// Iterate in sorted order
for _, key := range keys {
    fmt.Println(key, m[key])
}
```

**Option 2: Slice of structs**
```go
type Item struct {
    Key   string
    Value int
}

items := []Item{
    {"banana", 2},
    {"apple", 1},
    {"cherry", 3},
}

// Sort the slice
sort.Slice(items, func(i, j int) bool {
    return items[i].Key < items[j].Key
})

for _, item := range items {
    fmt.Println(item.Key, item.Value)
}
```

**Option 3: Separate ordered keys list**
```go
type OrderedMap struct {
    keys   []string
    values map[string]int
}

func (om *OrderedMap) Set(key string, value int) {
    if _, exists := om.values[key]; !exists {
        om.keys = append(om.keys, key)
    }
    om.values[key] = value
}

func (om *OrderedMap) Iterate(fn func(string, int)) {
    for _, key := range om.keys {
        fn(key, om.values[key])
    }
}
```

**Why "sorting maps" is wrong:**
- Maps are hash tables optimized for fast lookups, not ordering
- Trying to "sort a map" shows a misunderstanding of the data structure
- The correct approach is to extract data and sort it separately
- If you need order, you probably don't need a map

**When to use maps vs slices:**
```go
// Use map when:
// - Fast lookups by key are needed
// - Order doesn't matter
// - Checking for existence
usersByID := map[int]User{}

// Use slice when:
// - Order matters
// - Need to iterate in sequence
// - Need sorting
sortedUsers := []User{}

// Use both when:
// - Need fast lookup AND order
userMap := map[int]User{}
orderedIDs := []int{}  // Maintain order separately
```

**Important notes:**
- Map iteration order is deliberately randomized to prevent bugs from order assumptions
- Never rely on map order in production code
- If you're tempted to sort a map, reconsider your data structure choice
- Use slices when order matters, maps when fast lookups matter

**Note:** Unlike some languages with ordered dictionaries (Python 3.7+), Go maps are intentionally unordered by design.

---

### Receiver Functions (Methods)

Go allows you to define **methods** on types using **receiver functions**. A receiver is a special parameter that associates a function with a type.

**Basic syntax:**
```go
// Type definition
type Rectangle struct {
    Width  float64
    Height float64
}

// Method with receiver
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Usage
rect := Rectangle{Width: 10, Height: 5}
area := rect.Area()  // 50
```

**Value receivers vs Pointer receivers:**

**Value receiver** - receives a copy of the type:
```go
func (r Rectangle) ScaleValue(factor float64) {
    r.Width *= factor   // Modifies the COPY, not the original
    r.Height *= factor
}

rect := Rectangle{Width: 10, Height: 5}
rect.ScaleValue(2)
fmt.Println(rect.Width)  // Still 10 - original unchanged
```

**Pointer receiver** - receives a pointer to the type:
```go
func (r *Rectangle) ScalePointer(factor float64) {
    r.Width *= factor   // Modifies the original
    r.Height *= factor
}

rect := Rectangle{Width: 10, Height: 5}
rect.ScalePointer(2)
fmt.Println(rect.Width)  // 20 - original modified
```

**When to use pointer receivers:**
- When you need to modify the receiver
- When the type is large (avoids copying)
- When the type has pointer receiver methods already (consistency)
- For most struct methods (common convention)

**When to use value receivers:**
- When the type is small and cheap to copy (int, bool, small structs)
- When the receiver should not be modified
- For immutable operations
- For basic types

**Methods on any type:**
```go
// Can define methods on custom types, not just structs
type Celsius float64

func (c Celsius) ToFahrenheit() float64 {
    return float64(c)*9/5 + 32
}

temp := Celsius(25.0)
fmt.Println(temp.ToFahrenheit())  // 77
```

**Methods on built-in types (via type alias):**
```go
type IntSlice []int

func (is IntSlice) Sum() int {
    total := 0
    for _, v := range is {
        total += v
    }
    return total
}

numbers := IntSlice{1, 2, 3, 4, 5}
fmt.Println(numbers.Sum())  // 15
```

**Method chaining:**
```go
type Builder struct {
    value string
}

func (b *Builder) Append(s string) *Builder {
    b.value += s
    return b
}

func (b *Builder) String() string {
    return b.value
}

result := new(Builder).
    Append("Hello").
    Append(" ").
    Append("World").
    String()
    
fmt.Println(result)  // Hello World
```

**Multiple methods on same type:**
```go
type Account struct {
    balance float64
}

func (a *Account) Deposit(amount float64) {
    a.balance += amount
}

func (a *Account) Withdraw(amount float64) error {
    if amount > a.balance {
        return errors.New("insufficient funds")
    }
    a.balance -= amount
    return nil
}

func (a Account) Balance() float64 {
    return a.balance
}

acc := &Account{}
acc.Deposit(100)
acc.Withdraw(30)
fmt.Println(acc.Balance())  // 70
```

**Receiver naming conventions:**
- Use short names (usually 1-2 letters)
- Be consistent across methods of same type
- Common: first letter(s) of type name: `r` for Rectangle, `u` for User
- Use `this` or `self` is NOT idiomatic in Go

**Important notes:**
- You cannot define methods on types from other packages
- Pointer and value receivers can be called on both pointers and values (Go handles conversion)
- Methods are just functions with a receiver parameter
- Methods enable interface satisfaction for polymorphism

**Example showing automatic conversion:**
```go
type Counter struct {
    count int
}

func (c *Counter) Increment() {
    c.count++
}

// Both work - Go automatically converts
c1 := Counter{}
c1.Increment()  // Go converts to (&c1).Increment()

c2 := &Counter{}
c2.Increment()  // Direct call
```

**Methods vs Functions:**
```go
// Function - standalone
func CalculateArea(r Rectangle) float64 {
    return r.Width * r.Height
}

// Method - associated with type
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Usage difference
area1 := CalculateArea(rect)  // Function call
area2 := rect.Area()           // Method call (more OOP-like)
```

---

## Summary

Go combines the best of both worlds:

- **Strong + Static typing**: Catches errors early, runs fast, great tooling support
- **Compiled**: Fast execution, easy deployment, no runtime dependencies
- **Cross-platform**: Write once, deploy everywhere

This makes Go particularly well-suited for:
- Microservices and backend systems (like your Solar-Log work!)
- CLI tools and utilities
- Cloud-native applications
- High-performance network services
- DevOps tooling

---

*Last updated: 2026-01-03*
