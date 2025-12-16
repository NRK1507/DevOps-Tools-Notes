# 🐍 Python Basics: Syntax & Semantics

Welcome to Python!  
This guide explains **Syntax and Semantics** in a simple, beginner-friendly way with clean formatting that renders correctly on GitHub.

----------------------------------------------

## 📖 Table of Contents
- What is Syntax?
- What is Semantics?
- Comments in Python
- Basic Syntax Rules
- Indentation
- Line Continuation
- Common Errors
- Summary

----------------------------------------------

## 🔹 What is Syntax?

**Syntax** refers to the rules of writing Python code.

Think of syntax like **grammar in English**.  
If grammar is wrong, the sentence makes no sense.

### ✅ Correct Syntax

```python
print("Hello World")
```

### ❌ Incorrect Syntax

```python
print "Hello World"
```

🚫 Python will stop and show an error if syntax is wrong.

----------------------------------------------

## 🔹 What is Semantics?

**Semantics** refers to the **meaning of the code**.

Even if code is written correctly, it must also make sense logically.

### Example (Correct Syntax, Wrong Meaning)

```python
a = 10
b = 0
print(a / b)
```

✔ Syntax is correct  
❌ Logic is wrong → runtime error

----------------------------------------------

## 💬 Comments in Python

Comments help humans understand the code.  
Python ignores comments during execution.

### Single-line Comment

```python
# This prints a message
print("Hello World")
```

### Multi-line Comment (Common Practice)

```python
# This is a multi-line comment
# used to explain
# multiple lines
```
----------------------------------------------

## 🧩 Basic Syntax Rules

### 🔠 Case Sensitivity

Python is case-sensitive.

```python
name = "Raj"
Name = "raj"

print(name)
print(Name)
```

Output:
```
Raj
raj
```

----------------------------------------------

## 📐 Indentation (Very Important)

Python uses indentation to define blocks of code.

- Indentation is mandatory
- Common standard: **4 spaces**

```python
age = 32

if age > 30:
    print(age)

print(age)
```

Output:
```
32
32
```

----------------------------------------------

### Correct Indentation Example

```python
if True:
    print("Correct Indentation")
    if False:
        print("This will not print")
    print("This will print")

print("Outside the if block")
```

----------------------------------------------

## ➡️ Line Continuation

Use a backslash (`\`) to continue a long statement.

```python
total = 1 + 2 + 3 + 4 + 5 + 6 + 7 + \
        4 + 5 + 6

print(total)
```

Output:
```
43
```

----------------------------------------------

## 🧱 Multiple Statements on One Line

You *can* write multiple statements using `;`  
(Not recommended for beginners)

```python
x = 5; y = 10; z = x + y
print(z)
```
----------------------------------------------

## ⚠️ Common Errors

### ❌ NameError

Occurs when a variable is not defined.

```python
a = b
```

Error:
```
NameError: name 'b' is not defined
```

----------------------------------------------

### ❌ Indentation Error

Occurs when indentation is incorrect.

```python
if age > 30:
print(age)
```

---

## ✅ Summary

| Concept | Meaning |
|------|-------|
| Syntax | How code is written |
| Semantics | What code means |
| Case Sensitivity | `name` ≠ `Name` |
| Indentation | Mandatory |
| Typing | Dynamic |

---

🎯 **Final Tip:**  
Correct syntax helps Python run your code.  
Correct semantics ensures your code does the *right thing*.

🚀 Happy Python Learning!
