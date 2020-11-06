# Typing, Compiling, Interpreting

## Strongly typed languages

- Java: strong and static:
Strong: A variable must have its type defined upfront and cannot have other value than that from there on.

```java
int x = 10;  // strong typed: x's type must be explicitly declared => accomodate space in memory statically
x = 'a';  // statically typed: error, x's type (integer) cannot change dynamically to accommodate a string;
```

- Python/PHP: strong and dynamic:

```python
"3" + 5  # TypeError for implicitly coercion is not allowed, thus making Python a Strongly Typed Language.
```

- C: static and weak:

```c
```

- JavaScript: weak and dynamic:

```javascript
1 + 2 + "3"  // => "33", implicitly converts variable to string, thus, Weakly Typed Language.
```


## Definitions

In 1974, Liskov and Zilles defined a strongly-typed language as one in which "whenever an object is passed from a calling function to a called function, its type must be compatible with the type declared in the called function."

In 1977, Jackson wrote, "In a strongly typed language each data area will have a distinct type and each process will state its communication requirements in terms of these types."[3]

## Keywords

- Compile time

## References

- <https://en.wikipedia.org/wiki/Strong_and_weak_typing>
- <https://hackernoon.com/i-finally-understand-static-vs-dynamic-typing-and-you-will-too-ad0c2bd0acc7>

## Compiled vs Interpreted

## Summary

Strong: no implicitly coercion. E.g., Python, Java. `1 + "3"  # TypeError`
Weak: allow implicit coercion. E.g.: Javascript. `1 + 2 + "3"  // "33"`.

Static: mostly because type checking is performed during compiling. E.g.: Java. Implies that variable types must be defined before used.

```java
int x = 10;

static void sum_one(int operand) {
    return operand + 1;
}

sum_one("1");  // Error while compiling, sum_one must receive an `int` instance.
```

Dynamic: type errors happen at runtime. So it allows variables to not have defined type before being used.

```python
x = 10
not_number = "10"

def sum_one(operand) {
    return operand + 1
}

sum_one(not_number);  # TypeError during runtime, sum_one must receive an `int` instance.
```