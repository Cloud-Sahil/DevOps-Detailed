# Python Interview Questions

## Basic Questions

### 1. What is Python?

Python is a high-level, interpreted, object-oriented programming language.

---

### 2. Difference between List and Tuple?

| List | Tuple |
|--------|--------|
| Mutable | Immutable |
| [] | () |

---

### 3. Difference between Deep Copy and Shallow Copy?

Shallow copy copies references.
Deep copy creates independent objects.

---

### 4. What is PEP8?

Python coding standard guidelines.

---

### 5. What is __init__()?

Constructor method executed during object creation.

---

### 6. Difference between == and is?

== compares values.

is compares memory locations.

---

### 7. What is Lambda Function?

Anonymous single-line function.

---

### 8. What is a Dictionary?

Key-value data structure.

---

### 9. What is List Comprehension?

Short syntax to create lists.

```python
nums = [x*x for x in range(5)]
```

---

### 10. What is Exception Handling?

Handling runtime errors using try-except.

---

# Intermediate Questions

### 11. What are Decorators?

Functions that modify another function behavior.

---

### 12. What are Generators?

Functions using yield keyword.

---

### 13. Difference between Threading and Multiprocessing?

Threading → Shared Memory

Multiprocessing → Separate Memory

---

### 14. What is GIL?

Global Interpreter Lock.

Allows one thread to execute Python bytecode at a time.

---

### 15. What is Virtual Environment?

Isolated Python environment.

---

### 16. What is Pip?

Python package manager.

---

### 17. What are Modules and Packages?

Module → Single .py file

Package → Collection of modules

---

### 18. What is Serialization?

Converting object into storable format.

---

### 19. What is JSON?

JavaScript Object Notation.

---

### 20. What is REST API?

HTTP based communication between applications.

---

# Advanced Questions

### 21. Explain Python Memory Management.

Uses reference counting and garbage collection.

---

### 22. What is Monkey Patching?

Changing class behavior at runtime.

---

### 23. What is Method Resolution Order (MRO)?

Order used to search inherited methods.

---

### 24. What is Metaclass?

Class that creates classes.

---

### 25. Explain Context Manager.

Uses with statement for resource handling.

---

### 26. What is Asyncio?

Framework for asynchronous programming.

---

### 27. Difference between Process and Thread?

Process → Independent

Thread → Lightweight execution unit

---

### 28. What is Pickling?

Object serialization mechanism.

---

### 29. What is Unpickling?

Deserialization process.

---

### 30. Explain Python Garbage Collection.

Automatically removes unused objects.

---

# Scenario-Based Questions

## 31

Application consuming high CPU.

How will you troubleshoot?

### Answer

- Check top
- Check logs
- Profile code
- Identify infinite loops
- Optimize algorithms

---

## 32

Python application running slow.

### Answer

- Check memory
- Check DB queries
- Check API calls
- Use profiling tools

---

## 33

Script works locally but fails in production.

### Answer

- Compare Python versions
- Compare packages
- Check environment variables
- Check permissions

---

## 34

Application consuming high memory.

### Answer

- Use memory profiler
- Check memory leaks
- Optimize object creation

---

## 35

API response taking too long.

### Answer

- Check backend logs
- Check DB latency
- Check external API dependency

---

## 36

Python service crashing repeatedly.

### Answer

- Review logs
- Check exceptions
- Verify resources

---

## 37

Requirements.txt not installing.

### Answer

- Verify package versions
- Upgrade pip
- Check internet access

---

## 38

Multithreaded application not improving performance.

### Answer

- Check GIL impact
- Use multiprocessing

---

## 39

Database connections exhausted.

### Answer

- Use connection pooling
- Close idle connections

---

## 40

File upload failing in production.

### Answer

- Check file size limits
- Check permissions
- Check disk space

---

## 41

Logs not generated.

### Answer

- Verify logging configuration
- Check write permissions

---

## 42

Dockerized Python app exits immediately.

### Answer

- Check CMD
- Check entrypoint
- Review container logs

---

## 43

Application inaccessible behind Nginx.

### Answer

- Check Nginx config
- Verify backend service

---

## 44

Python API returns 500 error.

### Answer

- Review stack trace
- Validate inputs

---

## 45

Kubernetes Pod running Python app restarts.

### Answer

- Check kubectl logs
- Check probes
- Verify resources

---

## 46

AWS Lambda timing out.

### Answer

- Increase timeout
- Optimize code

---

## 47

Jenkins pipeline failing for Python project.

### Answer

- Check dependencies
- Verify virtual environment

---

## 48

Terraform automation script failing.

### Answer

- Check credentials
- Check state access

---

## 49

Python script cannot access S3.

### Answer

- Verify IAM permissions
- Check AWS credentials

---

## 50

How would you debug a Python production incident end-to-end?

### Answer

1. Verify issue
2. Check logs
3. Check metrics
4. Check infrastructure
5. Reproduce issue
6. Apply fix
7. Validate fix
8. Document RCA
