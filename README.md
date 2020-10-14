# Integrity Checking

## Intoduction

This readme covers the general philosophy behind the Integrity libraries, which share a common interface across multiple languages. The individual language implementations are:
Language|Link
--------|----
Python | [GitHub integrity for python](https://github.com/RisingTyde/integrity-py)
JavaScript | [GitHub integrity for JavScript (node & browser)](https://github.com/RisingTyde/integrity-js)
c# | [GitHub integrity for c#](https://github.com/RisingTyde/integrity-cs)
c++ | [GitHub integrity for c++](https://github.com/RisingTyde/integrity-cpp)

The intention is to provide a common interface so that as you switch between languages you can add integrity checks without having to remember different functions, or how they work. There are a set of core functions and some languages have some extra functions added to help with type checking.

## What is an integrity check?

Integrity checks should indicate that a coding error has occured. If an integrity check fires in the field, or in testing, it means a code change is required, either 
* some other code is in violation of a(correct) assumption, and needs to be changed
* your code (the one with the check in it) made a false assumption, and needs to change
* the check itself is wrong e.g. variable name was misspelled

Integrity checks *should not* be used for:
* Higher level validation of user input (but can be used in lower levels which assume the validation was done correctly)
* Environmental checks. If the fix to the problem is to fix the environment, then Integrity checks were the wrong way of detecting the problem. Examples of environmental errors could be: missing file, internet not available, database down. Generaly environmental errors are raised by the libraries you use, but if necessary explicit environmental problems should be considered and the code written to deal with them.

If in doubt ask: 'If this check fires, does it automatically indicate a coding error?' If the answer is yes, then use an integrity check. If no, then do some other form of error checking.

## Aren't Integrity checks the same as asserts?
The philisophical intention is somewhat similar, but the asserts themselves are mad, bad, and dangerous to know.
### Asserts are mad
The implementaiton of assert across varuious languages is perplexing, to say the least. Here are some of the issues:
* In c# there are two forms of assert - one is active in production and one is active only in debug mode.
* In Java assert will throw an exception derived from 'Error' instead of derived from Exception which makes it dangerous to use, as much general error handling code only deals with exceptions derived from Exception. This makes it dangerous and unpredicatble to use assert.
### Asserts are bad
Asserts can themselves introduce bugs, because of the way they have been implemented in the language. For example:
* In PHP the assert function will evaluate the passed in variable and if it is a string execute it as code! The has been deprecated, but still works.
* In Python if you forget the assert syntax and call it like a function i.e.
```python
assert(1 == 2) # incorrect assert sytax
```
instead of
```python
assert 1 == 2 # correct assert syntax
```
then the assert will *always* pass, defeating the whole point of having it.
* In JavaScript and Python there is the potential to accidentally do an assignment instead of a comparasion. If you use assert (or console.assert in JavaScript) the assignment will go by unnoticed (apart from potentially wrecking your code).
* There is potential for the assert message to cause a problem. 
* Finally, the way most languages implement assert encourages the developer to build the message strings even if the assert would pass, which is an overhead your program does not need
### Asserts are dangerous to know
The general problem with asserts is that they will probably not be active in production code. This means that the code you test is not the same as the code you release. Any accidental side effects from the asserts will not be present in production code. Integrity checking has a multiplier effect on your unit tests, your test team, the support team, and other developers (see below). You lose all this if the asserts are striped out of the code.

## Aren't Integrity checks just the same as: if(check) { throw } ?

Yes, they are. If you prefer writing the checks explicitly then knock yourself out. The Integrity libraries are just helpers to make life a little bit easier. Just to be clear (using Python for the example),
```Python
Integrity.check(a == b, "Oopsies", a, b)
```
is the same as:
```Python
if(a != b):
    raise ValueError("Oopsies, " + str(a) + ", " + str(b))
```
The advantages of using the Integrity library are:
* It is very clear to other programmers that you are testing an assumption
* The message building is less likely to go wrong
* Some checks like checkIsValidNumber do multiple checks, and doing it by hand would be tedious

Having said all that, the critical thing is that the check is made, and it doesn't really matter whether you use the Integrity library or not, it is just a helper which is consistent across languages, which is nice if you jump between languages.

## The philosophy behind the Integrity libraries

### Don't introduce new exceptions
The libraries all use exception classes which already exist in the native languages. This means that there is no need to indroduce a dependency on the Integrity library in the catching code.

### Be as performant as possible
Perfromance is prioritised, which means:
* The string building of the message only occurs after a check fails
* There isn't any fancy stuff like callback hooks, which would add extra overhead

### Be as safe as possible
* If you accidentally do an assignment instead of a comparasion, an exception will be raised. Note that this is not possible in c# (for example) because the compiler will not allow you to do it, but could occur in JavaScript or Python. This means that the error will be noticed the first time the code is executed, which is better than having a strange side effect.
* The message building is both deferred and as safe as possible. By passing variables as a list or arguments, the string building can be defered until the check fails, and also done in a safe way.
* For type safe languages like c#, there are some Integrity functions which don't make sense for certain types. The prime example is CheckIsValidNumber, which only makes sense for floats and doubles, but does not make sense for ints, longs etc. This is because it is impossible for an int or long to *not* be a valid number. However, a developer could accidentally pass an int or long in, which will always pass. The problem is that in order to do this the compiler will autobox the int or long to a float or double, which incurs a performance penalty, and will then do some pointless checks on it (checking for NaN etc.). To avoid this, overloaded variants are provided which do nothing. The overload for CheckIsValidNumber for an int simply returns. Although it does not make sense for the developer to have called this on an int, this is the safest and most performant way of dealing with this situation.

### Be as consistent across languages as possible
The message building part is the same across all languages, but there are some minor variations in the function names to accommodate language idiosyncracies:
* In c# the first letter of each function is capitalised, as per c# convention
* In Python the word 'Null' is replaces with 'None'
* In JavaScript the word 'Null' means both null and undefined.
* In type safe languages some functions are ommitted, as the compiler makes the check meaningless. For example, checkIsString does not make any sense in c# (but does in javaScript and Python) 
