---
title: Lazily load jquery selectors
date: 2016-09-01 09:00:01 Z
categories:
- js,
- good
- practices
layout: post
---

It's a known bad practice to repeat in your code different calls to jquery with the same selector, so often the recommended way is to call those in a variable.But what happens when you want to refer to many different selectors??

Another thing I have been trying is to avoid adding CSS classes in javascript code, and rather pass them as options of the functions/classes in the code.

For that I created this little method, that given an object where you can specify the different selectors makes all the jquery calls automatically.

```
  function loadSelectors(sel) {
    var result = {}
    
    for (var i in sel) {
      result[i] = (function() {
        var element;
        return () => element || (element = $(i));
      })();
    }
    
    return result;
  }
```

The usage is pretty simple:

```
let selectors = {
	container: '.js-container',
	sidebar: '.js-sidebar'
};
let $selectors = loadSelectors(selectors);

$selectors.container().hide()
```

The resulting object `$selectors` contains functions that will return the corresponding objects, but wrapped into jquery calls. The good thing is that they are lazily loaded and thus they are only called once.





