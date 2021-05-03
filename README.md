Information Hiding in Scheme
============================

Say you wanted to implement an [abstract data type][] in [R5RS Scheme][].
How would you do it?

_Could_ you do it?  On first blush, it might appear that you can't.
There is no module system, and the data structuring facilities that
Scheme provides are really primitive.  If you represent your data type
as a list, any client code could just take it apart with `car` and `cdr`,
and, worse, could `cons` together any other thing and pass that off
as your data type.

But there is a way.  More than one, in fact.  Here is one way that I
I think is one of the easier ones to grasp.  It relies on two properties
of Scheme:

*   Scheme _does_ have an opaque data structure: the function value.
*   `(eq? (list 1) (list 1))` is `#f`, that is, each cons-cell can
    be distinguished from every other cons-cell.

(The former would not be true if one could examine the definition of a
function value.  The latter would not be true if Scheme implemented
hash-consing.)

### The Basic Idea

The basic idea is straightforward.  We define a function which we call
a _module_.  When called, the module generates a few values internally.
One of these values is a unique cons-cell which we'll call the
_secret token_ of the module.  Other values are functions, which we'll
call _operations_.

Each operation requires a token to work, and only performs its intended
function when this token matches the secret token defined by the module.

The value that the module returns to its caller may contain the
operations — it may simply be a list of the operations — but it may
not contain the secret token.  Likewise, the values returned by the
operations must not contain the secret token.

(These values may be functions which have _closed over_ the secret
token.  But they should not _be_ the secret token nor _return_ it.)

In this way, clients of the module may use the operations of the module
without having to know, nor being able to alter or counterfeit, the
concrete data format that the operations work on.

### `seal` and `open`

Because every operation follows the same pattern — pass the secret
token to a given opaque object to obtain the internal representation,
examine or modify the internal representation, then possibly create
a new opaque object — it is useful to write a pair of helper functions to
"open" and "seal" these objects using the module's secret token, and
then write all operations using these helpers.

`seal` takes some data, and returns a function which does the following:
it takes a token, and if that token matches the secret token, it returns
the data, otherwise it returns an error value.

`open` takes an opaque object, and calls it using the secret token
returning the data.  (The `open` helper "knows" the secret token
because they are both defined inside the module; the definition
of the function called `open` closes over the secret token value.)

### Example Code

See the file [`information-hiding.scm`](information-hiding.scm) for
an example of using this technique to implement a stack ADT.

[abstract data type]: https://en.wikipedia.org/wiki/Abstract_data_type
[R5RS Scheme]: https://schemers.org/Documents/Standards/R5RS/
