---
layout: post
mathjax: true
title: On Javascript Engines, Code Auditing
date: 2019-04-07
permalink: /:title/
description: "Adventures approaching real targets"
tags: [exploitation]
share: true
---

# Introduction

After watching @nedwill's CCC talk[^1] I figured it was time to really try
and approach real targets. As I began this ~~scaringly difficult~~ learning
project I had to come to terms with the feeling Frodo must have felt when
he was tasked with one of the most important tasks in the lore.
I'm writing this as an aspiring security researcher (not necessarily in the
academic <accezzione> of the word), with no previous exposure to JavaScript
engines or compiler theory.



Besides that, I approached it as a learning project, on in which I wanted
to try and find existing CVEs. Even though I'm not particularly well versed
in exploitation, I felt as if I had to get my auditing skills down first.

Downloading the source code turned out to be
the most painful part of it to be honest.

---
Table of Contents:
* TOC
{:toc}

# Setup

Since I did not feel like getting stuck installing tools I just went on with Sublime Text 3.
Due to my unfamiliarity with svn I went ahead with git's mirrors. When it came down to
choosing which targets to audit, I sticked with WebKit as it has many many resources and v8
While trying to find the vulnerabilities myself, I tried to comment the code while trying
to understand most of it.

# Warming up

Coming from a CTF-esque background I started looking for integer overflows.
The exact commit I checked my codebase against is `956c6759d161d678125b0b9b8128e3fe6bd37c69`
<!-- change this phrase as it is very bad -->
(for the curious, I picked that commit because I ran `git rev-list -1 --before="2017-01-20 12:00" master`).
The first vulnerability we are going to discuss is CVE-2017-2464, a v8 integer overflow in the Zone allocator
that can be leveraged to achieve an heap overflow. Assuming one is acquainted with integer overflows, this
vulnerability is quite straighforward. Readers are free to look for the vulnerability themselves as an exercise.

---

### CVE-2017-2464

> from bug tracker [...]

The code is quite lengthy, and as such, I following along with the code in a text editor.
The vulnerable function is in `Source/JavaScriptCore/builtins/ArrayPrototype.js`.

```javascript
function concatSlowPath()
{
    "use strict";

    if (this == null)
        @throwTypeError("Array.prototype.concat requires that |this| not be null or undefined");

    var currentElement = @Object(this);

    // [1]
    var constructor;
    if (@isArray(currentElement)) {
        constructor = currentElement.constructor;
        [2]
        if (@isArrayConstructor(constructor) && @Array !== constructor)
            constructor = @undefined;
        else if (@isObject(constructor)) {
            constructor = constructor.@speciesSymbol;
            if (constructor === null)
                constructor = @Array;
        }
    }

    var argCount = arguments.length;
    var result;

    if (constructor === @Array || constructor === @undefined)
        result = @newArrayWithSize(0);
    else
        result = new constructor(0);
    // [3]
    var resultIsArray = @isJSArray(result);

    var resultIndex = 0;
    var argIndex = 0;

    do {
        // [4] This is the codepath of interest as the else clause explicitly checks for overflows
        let spreadable = @isObject(currentElement) && currentElement.@isConcatSpreadableSymbol;
        if ((spreadable === @undefined && @isArray(currentElement)) || spreadable) {
            let length = @toLength(currentElement.length);
            // [5]
            if (resultIsArray && @isJSArray(currentElement)) {
                // [6]
                @appendMemcpy(result, currentElement, resultIndex);
                resultIndex += length;
            } else {
                if (length + resultIndex > @MAX_SAFE_INTEGER)
                    @throwTypeError("length exceeded the maximum safe integer");
                // [7]
                for (var i = 0; i < length; i++) {
                    if (i in currentElement)
                        @putByValDirect(result, resultIndex, currentElement[i]);
                    resultIndex++;
                }
            }
        } else {
            if (resultIndex >= @MAX_SAFE_INTEGER)
                @throwTypeError("length exceeded the maximum safe integer");
            @putByValDirect(result, resultIndex++, currentElement);
        }
        currentElement = arguments[argIndex];
    } while (argIndex++ < argCount);

    // [8] what if the vunlerability consists in increasing resultIndex
    result.length = resultIndex;
    return result;
}

```

Notes taken during my analysis:

[1] This block isn't too interesting, it is used to get the constructor
of the object that called ".concat"

[2] concat can receive multiple arguments
ie: `"a".concat("b", "c")`

[3] non-array objects can too call concat, so before
    proceeding any further, make sure 'result' is the
    of the same type as *this

[4] We have this check so that if some array from a different global object
    calls this map they don't get an array with the `Array.prototype` of the
    other global object.

[5] This will be later used to decide which path to take

[4] This is the codepath of interest as the else clause explicitly checks for overflows

[5] Again, this kind of overflow can only be triggered with arrays

[?1] appendMemcpy is probably safe, though it is called in the
     wrong way

[6]  For non-array objects that can bee iterated upon, follow a slightly different road
     `@putByValDirect(values, index, argument)`;

[7]  This will be later used to decide which path to take

[X]  We have this check so that if some array from a different global object
     calls this map they don't get an array with the Array.prototype of the
     other global object.

[4] This is the codepath of interest as the else clause explicitly checks for overflows

[?] what if the vunlerability consists in increasing resultIndex


```cpp
bool JSArray::appendMemcpy(ExecState* exec, VM& vm, unsigned startIndex, JSC::JSArray* otherArray)
{
    auto scope = DECLARE_THROW_SCOPE(vm);

    // I can only copy in the fast way
    if (!canFastCopy(vm, otherArray))
        return false;

    // A memcpy is being called. In order to do the copy,
    // everything from type to size must be the same. In
    // order to figure out how much mem to allocate, and
    // how to use it, it is necessary to know how the object
    // is layed out first.
    IndexingType type = indexingType();
    IndexingType copyType = mergeIndexingTypeForCopying(otherArray->indexingType());
    // Depending on the type of the array to be copied, take a different
    // path
    if (type == ArrayWithUndecided && copyType != NonArray) {
        if (copyType == ArrayWithInt32)
            convertUndecidedToInt32(vm);
        else if (copyType == ArrayWithDouble)
            convertUndecidedToDouble(vm);
        else if (copyType == ArrayWithContiguous)
            convertUndecidedToContiguous(vm);
        else {
            ASSERT(copyType == ArrayWithUndecided);
            return true;
        }
    } else if (type != copyType)
        return false;

    unsigned otherLength = otherArray->length();
    unsigned newLength = startIndex + otherLength;
    if (newLength >= MIN_SPARSE_ARRAY_INDEX)
        return false;

    // make sure the array is contiguous
    if (!ensureLength(vm, newLength)) {
        throwOutOfMemoryError(exec, scope);
        return false;
    }
    ASSERT(copyType == indexingType());

    if (type == ArrayWithDouble)
        memcpy(butterfly()->contiguousDouble().data() + startIndex, otherArray->butterfly()->contiguousDouble().data(), sizeof(JSValue) * otherLength);
    else
        memcpy(butterfly()->contiguous().data() + startIndex, otherArray->butterfly()->contiguous().data(), sizeof(JSValue) * otherLength);

    return true;
}
```

As the name suggests, `concatSlowpath` is a function used by v8 to `concat` arrays without optimizinations

# v8's internals

# WebKit's Internals

#

[^1]: [Ned's talk]() is great as it addresses topics that, in his words, are
    less publicly discussed yet are more prohibitive to aspiring security researchers
(not in the academic sense of the words)
