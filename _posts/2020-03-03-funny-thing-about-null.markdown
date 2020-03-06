---
layout: post
title: "One Funny thing about JavaScript NULL"
subtitle: "It's not what you think it is"
categories: Javascript
background: "/img/posts/06.jpg/"
---

While I was reading the draft of Dan Abramov's Just Javascript, I stumbled upon the type of javascript `null`.

Try this out on your javascript console
`typeof null`

Turns out that null gets considered as an object rather than a primitive data type, which is -- a bug. :(

This is a bug in JavaScript. A bug that has a history attached to it. It is so intense that it cannot be fixed now because almost the entire internet will crash because of it. Funny!

Now, According to the History

> The “typeof null” bug is a remnant from the first version of JavaScript. In this version, values were stored in 32 bit units, which consisted of a small type tag (1–3 bits) and the actual data of the value. The type tags were stored in the lower bits of the units

Strange. There's more.
This happens because of the following code

```c++
JS_PUBLIC_API(JSType)
JS_TypeOfValue(JSContext *cx, jsval v)
{
JSType type = JSTYPE_VOID;
JSObject *obj;
JSObjectOps *ops;
JSClass *clasp;

        CHECK_REQUEST(cx);
        if (JSVAL_IS_VOID(v)) {  // (1)
            type = JSTYPE_VOID;
        } else if (JSVAL_IS_OBJECT(v)) {  // (2)
            obj = JSVAL_TO_OBJECT(v);
            if (obj &&
                (ops = obj->map->ops,
                 ops == &js_ObjectOps
                 ? (clasp = OBJ_GET_CLASS(cx, obj),
                    clasp->call || clasp == &js_FunctionClass) // (3,4)
                 : ops->call != 0)) {  // (3)
                type = JSTYPE_FUNCTION;
            } else {
                type = JSTYPE_OBJECT;
            }
        } else if (JSVAL_IS_NUMBER(v)) {
            type = JSTYPE_NUMBER;
        } else if (JSVAL_IS_STRING(v)) {
            type = JSTYPE_STRING;
        } else if (JSVAL_IS_BOOLEAN(v)) {
            type = JSTYPE_BOOLEAN;
        }
        return type;
    }

```

> Psst, While I was previewing the markdown of this code snippet, the browser crashed and it went into Infinite loop. 🤔

The steps performed by the above code are:

- At (1), the engine first checks whether the value v is undefined (VOID). This check is performed by comparing the value via equals:
  `#define JSVAL_IS_VOID(v) ((v) == JSVAL_VOID)`
- The next check (2) is whether the value has an object tag. If it additionally is either callable (3) or its internal property [[Class]] marks it as a function (4) then v is a function. Otherwise, it is an object. This is the result that is produced by typeof null.
- The subsequent checks are for number, string and boolean. There is not even an explicit check for null, which could be performed by the following C macro.
  `#define JSVAL_IS_NULL(v) ((v) == JSVAL_NULL)`

###### There might arise one question, is `typeof([])` also a bug?

###### `typeof([])` returns an Object which is correct because arrays are treated as objects and not primitive data types.

Subscribe to Dan's course if you'd like, I love it because it is reengineering my brain to perceieve the most out of Javascript.

[Just JavaScript](https://justjavascript.com/)
