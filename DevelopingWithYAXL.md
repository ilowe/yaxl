## Reading XML ##

Use the `parse` function to obtain a hierarchy of `Element` instances.

```
>>> import yaxl
>>> x = yaxl.parse("""
... 	<person id="1234">
... 		<name>Bob Smith</name>
... 		<age>200</age>
... 	</person>
... """)
>>> x
<person id="1234"><name>Bob Smith</name><age>200</age></person>
>>> x['id']
'1234'
>>> str(x[0])
'Bob Smith'
>>> str(x.children[0])
'Bob Smith'
>>> x[0]
<name>Bob Smith</name>
>>> x[1]
<age>200</age>
```

#### Valid sources ####

The `parse` method is very versatile: it accepts a file path, a file-like object, a URL or a string as the source from which to load `Element`s.

This source can contain either a full document or a well-formed XML fragment.

## Creating new elements ##

You can create new elements by using the `Element` constructor or the `append` method of an existing `Element`. Both accept a name for the element (which may contain a properly bound [prefix](namespace.md)(#xml\_namespaces)), a `dict` of attributes (all non-string values are converted to strings) and a `text` parameter that sets the text node contents of the element being created. Both return the newly created element so you can chain calls to them.

```
>>> from yaxl import *
>>> person = Element('person', {'id': 1234})
>>> person.append('name', text='Bob Smith')
<name>Bob Smith</name>
>>> person
<person id="1234"><name>Bob Smith</name></person>
>>> person.append('age').appendTextNode('200')
>>> person
<person id="1234"><name>Bob Smith</name><age>200</age></person>
>>> person.append('address').append('street', text='Coconut Grove')
<street>Coconut Grove</street>
>>> person[0]
<name>Bob Smith</name>
>>> person[1]
<age>200</age>
>>> person[2]
<address><street>Coconut Grove</street></address>
>>> person[2][0]
<street>Coconut Grove</street>
>>> str(person[2][0])
'Coconut Grove'
```

You can also `append` an element to another:

```
>>> from yaxl import *
>>> person = Element('person', {'id': '1234'})
>>> age = Element('age', text=200)
>>> person.append(age)
<age>200</age>
>>> person
<person id="1234"><age>200</age></person>
```

Note that trying to create a new element whose name uses an unbound [prefix](namespace.md)(#xml\_namespaces) will raise an `Exception`.

## Manipulating `Element`s ##

  * [contents](String.md)(#string\_contents)
  * [elements](Locating.md)(#locating\_elements)
  * [Attributes](XML.md)(#xml\_attributes)
  * [names](Element.md)(#element\_qname)
  * [Namespaces](XML.md)(#xml\_namespaces)

### String contents ###

The string value of an element is the same as in XPath.

```
>>> from yaxl import *
>>> x = Element('x', text='something')
>>> str(x)
'something'
>>> '%s' % x
'something'
>>> x
<x>something</x>
```

You can also append strings to an element's contents:

```
>>> x = Element('x', text='Hello')
>>> x += ', World!'
>>> str(x)
'Hello, World!'
```

### Locating elements ###

You can access the elements in a document (or any sub-tree) in three ways.

  * Evaluate an XPath expression using the element as the context
  * Iterate over the element's children
  * Index the element itself
  * Access the `children` property of the element

#### Using XPath ####

You can also use XPath to retrieve elements' children and attributes. The XPath support on an `Element` allows you to execute arbitrary XPath expressions using that `Element` as the execution context. "Call" the element and supply an XPath expression to evaluate.

The result of evaluating an XPath expression is [defined](http://www.w3.org/TR/xpath#section-Introduction) as being one of: a node-set, a boolean, a number or a string. To simplify usage, yaxl returns a single value when the result is single and a tuple when the result is a node-set. If the returned node-set has only one element, however, that element is unwrapped and returned alone. To modify this, set the optional `return_nodelist` parameter to `True` to force yaxl to always return a zero-or-more element sequence.

```
>>> from yaxl import *
>>> x = Element('x')
>>> y = x.append('y')
>>> z = y.append('z')
>>> x('z')
>>> x('y')
<y><z /></y>
>>> x('/')
<x><y><z /></y></x>
>>> x('//z')
<z />
>>> x('//z', return_nodelist=True)
(<z />,)
```

#### Iterate over the element's children ####

Iterating over an element's children will yield non-text nodes in document order:

```
>>> from yaxl import *
>>> x = Element('x')
>>> x += 'Hello '
>>> y = x.append('y', text='world')
>>> x
<x>Hello <y>world</y></x>
>>> x += ', this is a test!'
>>> x
<x>Hello <y>world</y>, this is a test!</x>
>>> for child in x:
... 	print repr(child)
<y>world</y>
```

#### Indexing the element ####

Indexing the element like a list is a shortcut for indexing the list of an element's children.

```
>>> from yaxl import *
>>> x = Element('x')
>>> y = x.append('y')
>>> z = x.append('z')
>>> x[0]
<y />
>>> x[1]
<z />
```

#### Access the `children` property ####

Each element has a `children` property that is a list of all its children.

```
>>> from yaxl import *
>>> x = Element('x')
>>> y = x.append('y')
>>> z = x.append('z')
>>> x.children
[<y />, <z />]
```

### XML Attributes ###

You can access all of an element's attributes via the `attributes` property of the element.

```
>>> from yaxl import *
>>> x = Element('t', {'id': 1234, 'name': 'John Doe'})
>>> x['id']
'1234'
>>> x['id'] = 5
>>> x
<t id="5" name="John Doe" />
>>> x['name'] = 17
>>> x
<t id="5" name="17" />
```

Note that trying to modify an attribute whose qname uses an unbound namespace prefix will raise an `Exception`.

```
>>> from yaxl import *
>>> x = Element('t', {'id': 1234, 'name': 'John Doe'})
>>> x['test:x'] = 5
Traceback (most recent call last):
...
UndeclaredNamespaceException: test was not declared prior to use with localname x was not bound to a URI before use
```

### Element names ###

XML elements are named by a QName property. YAXL `Element`s have an attribute called `qname` that simulates the QName property. It does so by `join`ing its `ns` property (the namespace prefix the element is defined in) and its `localname` property (i.e. the local name of the element in that namespace).

```
>>> from yaxl import *
>>> x = Element('x')
>>> x
<x />
>>> x.qname
'x'
>>> x.ns
>>> x.localname
'x'
>>> x['xmlns:ex'] = 'http://example.org'
>>> x.ns = 'ex'
>>> x.localname = 'mytag'
>>> x
<ex:mytag xmlns:ex="http://example.org" />
>>> x['xmlns:ex2'] = 'http://example2.org'
>>> x.qname = 'ex2:anothertag'
>>> x
<ex2:anothertag xmlns:ex2="http://example2.org" />
```

Although setting an element's `qname` property to a value with an un-mapped namespace prefix normally raises an `UndeclaredNamespaceException`, you can use namespaces mapped in the `namespaces` keyword parameter of calls to the `Element` constructor and the `append` method.

### XML Namespaces ###

In order to use a namespace in an element you need to first map the namespace's prefix to its URL. Elements "inherit" the namespaces declared in their parents. To map a new namespace in an element, set an `xmlns:*` attribute to the URI to which the prefix should be bound.

```
>>> from yaxl import *
>>> x = Element('t', {'id': 1234, 'name': 'John Doe'})
>>> x['xmlns:test'] = 'http://example.org'
>>> x['test:x'] = 5
>>> x
<t xmlns:test="http://example.org" test:x="5" id="1234" name="John Doe" />
```

The default namespace of each element is specified by its `xmlns` attribute which is normally unset. Setting this attribute will change the default namespace of the element.

```
>>> from yaxl import *
>>> x = Element('x')
>>> x['xmlns']
>>> x
<x />
>>> x['xmlns'] = 'http://example.org'
>>> x
<x xmlns="http://example.org" />
```

The `xml` prefix is mapped by default to http://www.w3.org/XML/1998/namespace(http://www.w3.org/XML/1998/namespace).

```
>>> from yaxl import *
>>> x = Element('x')
>>> x['xmlns:xml']
'http://www.w3.org/XML/1998/namespace'
```

## Writing XML ##

Use the `repr` function, and the `asdoc` and `write` methods to output XML from a tree of `Element`s.

### Output methods ###

#### The `repr` function ####

Using `repr` on an `Element` returns an XML representation of the element. It is the method used by default in the Python interpreter so evaluating an `Element` at the prompt will print out it's XML representation.

```
>>> from yaxl import *
>>> x = Element('x')
>>> x += 'This is a text'
>>> x
<x>This is a text</x>
```

The only downside to this output method is that it returns an XML fragment and not a well-formed document.

#### The `asdoc` method ####

You can retrieve a well-formed XML document using an `Element` as the root with the `asdoc` method.

```
>>> from yaxl import *
>>> x = Element('x')
>>> x.asdoc()
'<?xml version="1.0" encoding="UTF8"?><x />'
```

As you can see, the default encoding for XML documents produced by the `asdoc` method is `UTF8`.

#### The `write` method ####

The `write` method can be used for output to files or other file-like objects. It will use the `write` method on any object supplied to output it's `Element`.

```
>>> from StringIO import StringIO
>>> from yaxl import *
>>> io = StringIO()
>>> x = Element('x')
>>> x.write(io)
>>> io.getvalue()
'<x />'
```

The `write` method, like the `repr` function, outputs an XML fragment, not a valid document. It can be used, however, when building XML files programmatically (e.g. when customizing processing instructions).

### Encoding ###

To set a document's encoding, pass an `encoding` keyword parameter to any of the `Element` output methods (`__repr__`, `asdoc`, `write`).

```
>>> from yaxl import *
>>> x = Element('x')
>>> x.asdoc(encoding='iso-8859-2')
'<?xml version="1.0" encoding="iso-8859-2"?><x />'
```

**Remember**, by default, all documents are encoded in UTF8.