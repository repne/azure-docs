---
title: Data types in Bicep
description: Describes the data types that are available in Bicep
ms.topic: conceptual
ms.date: 09/22/2021
---

# Data types in Bicep

This article describes the data types supported in [Bicep](./overview.md).

## Supported types

Within a Bicep, you can use these data types:

* array
* bool
* int
* object
* secureObject - indicated by modifier in Bicep
* secureString - indicated by modifier in Bicep
* string

## Arrays

Arrays start with a left bracket (`[`) and end with a right bracket (`]`). In Bicep, an array must be declared in multiple lines. Don't use commas between values.

In an array, each item is represented by the [any type](bicep-functions-any.md). You can have an array where each item is the same data type, or an array that holds different data types.

The following example shows an array of integers and an array different types.

```bicep
var integerArray = [
  1
  2
  3
]

var mixedArray = [
  resourceGroup().name
  1
  true
  'example string'
]
```

Arrays in Bicep are zero-based. In the following example, the expression `exampleArray[0]` evaluates to 1 and `exampleArray[2]` evaluates to 3. The index of the indexer may itself be another expression. The expression `exampleArray[index]` evaluates to 2. Integer indexers are only allowed on expression of array types.

```bicep
var index = 1

var exampleArray = [
  1
  2
  3
]
```

## Booleans

When specifying boolean values, use `true` or `false`. Don't surround the value with quotation marks.

```bicep
param exampleBool bool = true
```

## Integers

When specifying integer values, don't use quotation marks.

```bicep
param exampleInt int = 1
```

In Bicep, integers are 64-bit integers. When passed as inline parameters, the range of values may be limited by the SDK or command-line tool you use for deployment. For example, when using PowerShell to deploy a Bicep, integer types can range from -2147483648 to 2147483647. To avoid this limitation, specify large integer values in a [parameter file](parameter-files.md). Resource types apply their own limits for integer properties.

Floating point, decimal or binary formats aren't currently supported.

## Objects

Objects start with a left brace (`{`) and end with a right brace (`}`). In Bicep, an object must be declared in multiple lines. Each property in an object consists of key and value. The key and value are separated by a colon (`:`). An object allows any property of any type.

In Bicep, the key isn't enclosed by quotes. Don't use commas to between properties.

```bicep
param exampleObject object = {
  name: 'test name'
  id: '123-abc'
  isCurrent: true
  tier: 1
}
```

Property accessors are used to access properties of an object. They're constructed using the `.` operator.

```bicep
var a = {
  b: 'Dev'
  c: 42
  d: {
    e: true
  }
}

output result1 string = a.b // returns 'Dev'
output result2 int = a.c // returns 42
output result3 bool = a.d.e // returns true
```

Property accessors can be used with any object, including parameters and variables of object types and object literals. Using a property accessor on an expression of non-object type is an error.

You can also use the `[]` syntax to access a property. The following example returns `Development`.

```bicep
var environmentSettings = {
  dev: {
    name: 'Development'
  }
  prod: {
    name: 'Production'
  }
}

output accessorResult string = environmentSettings['dev'].name
```

## Strings

In Bicep, strings are marked with singled quotes, and must be declared on a single line. All Unicode characters with code points between *0* and *10FFFF* are allowed.

```bicep
param exampleString string = 'test value'
```

The following table lists the set of reserved characters that must be escaped by a backslash (`\`) character:

| Escape Sequence | Represented value | Notes |
|:-|:-|:-|
| `\\` | `\` ||
| `\'` | `'` ||
| `\n` | line feed (LF) ||
| `\r` | carriage return (CR) ||
| `\t` | tab character ||
| `\u{x}` | Unicode code point `x` | **x** represents a hexadecimal code point value between *0* and *10FFFF* (both inclusive). Leading zeros are allowed. Code points above *FFFF* are emitted as a surrogate pair.
| `\$` | `$` | Only escape when followed by `{`. |

```bicep
// evaluates to "what's up?"
var myVar = 'what\'s up?'
```

All strings in Bicep support interpolation. To inject an expression, surround it by `${` and `}`. Expressions that are referenced can't span multiple lines.

```bicep
var storageName = 'storage${uniqueString(resourceGroup().id)}
```

## Multi-line strings

In Bicep, multi-line strings are defined between three single quote characters (`'''`) followed optionally by a newline (the opening sequence), and three single quote characters (`'''` - the closing sequence). Characters that are entered between the opening and closing sequence are read verbatim, and no escaping is necessary or possible.

> [!NOTE]
> Because the Bicep parser reads all characters as is, depending on the line endings of your Bicep file, newlines can be interpreted as either `\r\n` or `\n`.
> Interpolation is not currently supported in multi-line strings.
> Multi-line strings containing `'''` are not supported.

```bicep
// evaluates to "hello!"
var myVar = '''hello!'''

// evaluates to "hello!" because the first newline is skipped
var myVar2 = '''
hello!'''

// evaluates to "hello!\n" because the final newline is included
var myVar3 = '''
hello!
'''

// evaluates to "  this\n    is\n      indented\n"
var myVar4 = '''
  this
    is
      indented
'''

// evaluates to "comments // are included\n/* because everything is read as-is */\n"
var myVar5 = '''
comments // are included
/* because everything is read as-is */
'''

// evaluates to "interpolation\nis ${blocked}"
// note ${blocked} is part of the string, and is not evaluated as an expression
myVar6 = '''interpolation
is ${blocked}'''
```

## Secure strings and objects

Secure string uses the same format as string, and secure object uses the same format as object. With Bicep, you add the `@secure()` modifier to a string or object.

When you set a parameter to a secure string or secure object, the value of the parameter isn't saved to the deployment history and isn't logged. However, if you set that secure value to a property that isn't expecting a secure value, the value isn't protected. For example, if you set a secure string to a tag, that value is stored as plain text. Use secure strings for passwords and secrets.

The following example shows two secure parameters:

```bicep
@secure()
param password string

@secure()
param configValues object
```

## Data type assignability

In Bicep, a value of one type (source type) can be assigned to another type (target type). The following table shows which source type (listed horizontally) can or can't be assigned to which target type (listed vertically). In the table, `X` means assignable, empty space means not assignable, and `?` means only if they types are compatible.

| Types | `any` | `error` | `string` | `number` | `int` | `bool` | `null` | `object` | `array` | named resource | named module | `scope` |
|-|-|-|-|-|-|-|-|-|-|-|-|-|
| `any`          |X| |X|X|X|X|X|X|X|X|X|X|
| `error`        | | | | | | | | | | | | |
| `string`       |X| |X| | | | | | | | | |
| `number`       |X| | |X|X| | | | | | | |
| `int`          |X| | | |X| | | | | | | |
| `bool`         |X| | | | |X| | | | | | |
| `null`         |X| | | | | |X| | | | | |
| `object`       |X| | | | | | |X| | | | |
| `array`        |X| | | | | | | |X| | | |
| `resource`     |X| | | | | | | | |X| | |
| `module`       |X| | | | | | | | | |X| |
| `scope`        | | | | | | | | | | | |?|
| **named resource** |X| | | | | | |?| |?| | |
| **named module**   |X| | | | | | |?| | |?| |

## Next steps

To learn about the structure and syntax of Bicep, see [Bicep file structure](./file.md).
