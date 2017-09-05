---
layout: post
title: "Exercise in Python: remove blanks from strings"
date: 2017-01-24 16:51
updated: 2017-01-25 08:51
comments: true
published: true
categories: [Python, exercise, profile, basic algorithms]
sharing: true
description: "A basic exercise in Python for removing blanks from a given string"
cover:
---
This morning, after reading [this very nice post](http://lemire.me/blog/2017/01/20/how-quickly-can-you-remove-spaces-from-a-string/), I decided to challenge myself in Python and to have a look at the impact of [mispredicted branches](https://en.wikipedia.org/wiki/Branch_misprediction) in a language different from C/C++. The basic idea was to use only [Python builtins](https://docs.python.org/3.6/library/functions.html): external libraries are not allowed!

As a benchmark, I grabbed a large text file from [P. Norvig's website](http://norvig.com/big.txt), which is 6'488'666 byte long.

**The final answer?** Yes, **mispredicted branches** have a huge impact in Python too.

**The hidden answer?** Python dictionaries ever stop to surprise me: they are REALLY efficient.

**NOTE**: The followig code snippets were executed in a Python 3.5 notebook, on a windows machine, running Windows 10 and [Anaconda Python 3.5 64 bits](https://www.continuum.io/downloads).
You can find my notebook on my [Blog GitHub repo](https://github.com/stegua/MyBlogEntries). Don't ask me why, but this blog entry is better visualized directly on [GitHub](https://github.com/stegua/MyBlogEntries/blob/master/Remove%2Bblanks%2Bfrom%2Ba%2Bstring.ipynb).

**UPDATE**: Well, most of the time I would use my first implementation based on the ```filter``` builtin function, and I would try for alternative implementations only after a profiler has shown
that removing blanks is a true bottleneck of my whole program. As written in the title, this post is meant as a basic exercise in Python.

### First attempt: Functional style
In Python, I prefer to write as much code in functional style as possible, relying on the 3 basic functions:

1. [```map```](https://docs.python.org/3/library/functions.html#map)
2. [```filter```](https://docs.python.org/3/library/functions.html#filter)
3. [```reduce```](https://docs.python.org/3/library/functools.html#functools.reduce) (this is in the functools module and it is not a true builtin)

Therefore, after few preliminaries, here is my first code snippet:

```python
import cProfile

# Download file from: 'http://norvig.com/big.txt'
big = open('big.txt', 'r')

# Read the while file
test = big.read()

def RemoveBlanksFilter(in_str):
    result = filter(lambda c: c != '\r' and c != '\t' and c != ' ', in_str)
    return "".join(result)

test_result = RemoveBlanksFilter(test)

cProfile.run('RemoveBlanksFilter(test)')
```

             6488671 function calls in 1.956 seconds
    
       Ordered by: standard name
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.000    0.000    1.955    1.955 <ipython-input-3-eeb7d3495697>:1(RemoveBlanksFilter)
      6488666    0.870    0.000    0.870    0.000 <ipython-input-3-eeb7d3495697>:2(<lambda>)
            1    0.000    0.000    1.956    1.956 <string>:1(<module>)
            1    0.000    0.000    1.956    1.956 {built-in method builtins.exec}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
            1    1.085    1.085    1.955    1.955 {method 'join' of 'str' objects}
    
    
    

Wow, I didn't realize that I would have call the lambda function for every single byte of my input file. This is clearly too much overhead.

### 2nd attempt: remove function calls overhead
Let me drop my functional style, and write a plain old for-loop:


```python
def RemoveBlanks(in_str):
    result = []
    for c in in_str:
        if c != '\r' and c != '\t' and c != ' ':
            result.append(c)
    return "".join(result)

print('Is test passed:', test_result == RemoveBlanks(test))
```

    Is test passed: True
    


```python
cProfile.run('RemoveBlanks(test)')
```

             5452148 function calls in 1.566 seconds
    
       Ordered by: standard name
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    1.210    1.210    1.553    1.553 <ipython-input-6-5e45e3056bc2>:1(RemoveBlanks)
            1    0.012    0.012    1.566    1.566 <string>:1(<module>)
            1    0.000    0.000    1.566    1.566 {built-in method builtins.exec}
      5452143    0.310    0.000    0.310    0.000 {method 'append' of 'list' objects}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
            1    0.033    0.033    0.033    0.033 {method 'join' of 'str' objects}
    
    
    

Mmm... we just shift the problem to the list append function calls. Maybe we can do better by working in place.

### 3rd attempt: work in place
Well, almost in place: Python string are immutable; therefore, we first copy the string into a list, and then we work in place over the copied list.


```python
def RemoveBlanksInPlace(in_str):
    buffer = list(in_str)
    pos = 0
    for c in in_str:
        if c != '\r' and c != '\t' and c != ' ':
            buffer[pos] = c
            pos += 1
    return "".join(buffer[:pos])

print('Is test passed:', test_result == RemoveBlanksInPlace(test))
```

    Is test passed: True
    

```python
cProfile.run('RemoveBlanksInPlace(test)')
```

             5 function calls in 1.158 seconds
    
       Ordered by: standard name
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    1.113    1.113    1.145    1.145 <ipython-input-9-99d36ae6359e>:1(RemoveBlanksInPlace)
            1    0.013    0.013    1.158    1.158 <string>:1(<module>)
            1    0.000    0.000    1.158    1.158 {built-in method builtins.exec}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
            1    0.032    0.032    0.032    0.032 {method 'join' of 'str' objects}
    
    
    

Ok, working in place does have an impact. Let me go on the true point: avoiding *mispredicted branches*.
    
### 4th attempt: to avoid mispredicted branches
As in the [original blog post](http://lemire.me/blog/2017/01/20/how-quickly-can-you-remove-spaces-from-a-string/):


```python
def RemoveBlanksNoBranch(in_str):
    # Build table
    table = []
    for ic in range(256):
        c = chr(ic)
        if c == '\r' or c == '\t' or c == ' ':
            table.append(0)
        else:
            table.append(1)
                
    # Removal
    buffer = list(in_str)
    pos = 0
    for c in in_str:
        buffer[pos] = c
        pos += table[ord(c)]  # ord() is a function --> bottleneck
    return "".join(buffer[:pos])


print('Is test passed:', test_result == RemoveBlanksNoBranch(test))
```

    Is test passed: True
    

```python
cProfile.run('RemoveBlanksNoBranch(test)')
```

             6489183 function calls in 1.474 seconds
    
       Ordered by: standard name
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    1.235    1.235    1.460    1.460 <ipython-input-12-1bd75a3de21d>:1(RemoveBlanksNoBranch)
            1    0.014    0.014    1.474    1.474 <string>:1(<module>)
          256    0.000    0.000    0.000    0.000 {built-in method builtins.chr}
            1    0.000    0.000    1.474    1.474 {built-in method builtins.exec}
      6488666    0.192    0.000    0.192    0.000 {built-in method builtins.ord}
          256    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
            1    0.033    0.033    0.033    0.033 {method 'join' of 'str' objects}
    
    
    
Ouch!!! These are getting even worse! Why? Well, 'ord' is a function, so we are getting back the overhead of function calls. Can we do better by using a dictionary instead of an array?

### 5th attempt: use a dictionary
Let me use a dictionary in order to avoid the 'ord' function calls.


```python
def RemoveBlanksNoBranchDict(in_str):
    if type(in_str) != str:
        raise TypeError('This function works only for strings')
    
    # Build table
    table = {}
    for ic in range(256):
        c = chr(ic)
        if c == '\r' or c == '\t' or c == ' ':
            table[c] = 0
        else:
            table[c] = 1
                
    # Removal
    buffer = list(in_str)
    pos = 0
    for c in in_str:
        buffer[pos] = c
        pos += table[c]
    return "".join(buffer[:pos])


print('Is test passed:', test_result == RemoveBlanksNoBranchDict(test))
```

    Is test passed: True
    

```python
cProfile.run('RemoveBlanksNoBranchDict(test)')
```

             261 function calls in 0.771 seconds
    
       Ordered by: standard name
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.724    0.724    0.758    0.758 <ipython-input-15-46ad4c3f0b26>:1(RemoveBlanksNoBranchDict)
            1    0.013    0.013    0.771    0.771 <string>:1(<module>)
          256    0.000    0.000    0.000    0.000 {built-in method builtins.chr}
            1    0.000    0.000    0.771    0.771 {built-in method builtins.exec}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
            1    0.034    0.034    0.034    0.034 {method 'join' of 'str' objects}
    
    
    
Oooh, yes! Now we can see that without **mispredicted branches** we can really speed up our algorithm.

Is this the best pythonic solution? No, surely not, but still it is an interesting remark to keep in mind when coding.

### Final remark: a simple pythonic solution
Likely, the simplest pythonic solution is just to use the 'replace' string function as follows:


```python
def RemoveBlanksBuiltin(in_str):
    s1 = in_str.replace('\r','')
    s2 = s1.replace('\t','')
    return s2.replace(' ','')


print('Is test passed:', test_result == RemoveBlanksBuiltin(test))
```

    Is test passed: True
    


```python
cProfile.run('RemoveBlanksBuiltin(test)')
```

             7 function calls in 0.065 seconds
    
       Ordered by: standard name
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.001    0.001    0.064    0.064 <ipython-input-18-58fd6655cfba>:1(RemoveBlanksBuiltin)
            1    0.001    0.001    0.065    0.065 <string>:1(<module>)
            1    0.000    0.000    0.065    0.065 {built-in method builtins.exec}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
            3    0.063    0.021    0.063    0.021 {method 'replace' of 'str' objects}

Here we are, the best solution is indeed to use a builtin function, whenever it is possible, even if this was not the real aim of this exercise.

Please, let me know if you have some comments or a different solution in Python.
