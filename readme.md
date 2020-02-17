# @prettybad/forester
## a forest for the trees

If a tree falls on a forester, does anyone hear them scream?

## Forest
### @prettybad/forest
<dl>
  <dt>Forest (n):</dt>
  <dd>data structure; a collection of trees.</dd>
</dl>

@prettybad/forest is _normalized_. This means, rather than nesting the
nodes in trees in the forest under one another as children:

1. Each node receives a number.
2. Relationships between nodes are written by reference to the number.

Thanks to normalization, copying a node is cheap. Having many of the same
node across many trees is cheap. It is easy to add copy-on-write semantics
to a normalized data structure. Normalization and copy-on-write satisfy a
use-case common with, for example, multiple layouts or views of the same
data; such as in a visual text editor, or layout editor, or theme viewer,
et cetera.

Other properties:

- Forests are immutable, but mutative alternates exist for each method.
- Forests place no mutability or shape constraints on nodes.
- Forests can, however, enforce copy-on-write semantics for node mutation.
- Forest node relationship changes do not modify nodes.
- Forest node references can be weakly held, to permit garbage collection.
- Forest nodes can be roots, to represent independent sub-trees.
- Forest nodes can be children, to represent tree branches.
- Forest nodes can be detached, to represent tombstone deletion or as an
  intermediate state for an in-transplant sub-tree.
- A Forest is (almost) trivially a rope, for editing long strings.
- A Forest can be a fixed-size data structure (e.g. TypedArrays in JS).

**Relationship changes in a Forest are cheap**: although a new Forest is
returned by default (unless the mutative method-call is used), the only
part of the Forest that must be reallocated is the specific sub-array
where the relationship change has occurred.

**Forest traversals are array iterations**: the nodes and the
relationships between them in a Forest are all represented linearly by
arrays. No pointer chasing, no 'left'/'right', no 'if not null' in your
breadth-first searches. `forest.children(node)` is literally an array of
children. If it's empty, then it's empty; if there are twenty, then there
are twenty; if anything in there is 'null', then you did that yourself
(and I judge you for that \<(-\_\_-\<) \<(T o T)\> (\>-\_\_-)\>).

Clearly, since it's really just a couple of arrays, it really is a pretty
bad forest. (Also, it lacks any sort of peat or pine scent.)

## Parse
### @prettybad/parse (AKA parsimony)
<dl>
  <dt>Parse (v):</dt>
  <dd>extrapolate and represent meaning from text.</dd>
  <dt>Parsimony (n):</dt>
  <dd>being to-the-point. As in: brief, terse, laconic.</dd>
</dl>

@prettybad/parse is a parser-generator library. Given a sequence of
tokens, a Forest is constructed that matches sub-sequences of tokens and
constructs meaningful data structures out of them.

Since the output is a Forest, multiple top-level terms can be parsed at
once. Multiple files can be parsed at once. Files can be represented as
tree-roots, where each child is its own top-level term. Modules can be
represented as tree-roots, where each file is a child of the module. Et
cetera. Thanks to cheap copies, imports/exports/cross-references between
parse units can be resolved and materialized instantaneously. Dangling
references will be caught during parse, and can either be marked in the
resulting Forest for later reconciliation or can throw a fatal error.

Obviously, since the parser will fatally error if references between nodes
cannot be resolved, it really is a pretty bad parser. It can't parse C,
that's for sure!

## Lex
### @prettybad/lex (AKA sexylexy)
<dl>
  <dt>Lex (v; Computer Science):</dt>
  <dd>perform lexical analysis.</dd>
  <dt>Lexer (n; Computer Science):</dt>
  <dd>a program that performs lexical analysis.</dd>
  <dt>Lexical analysis (n):</dt>
  <dd>grouping signs into tokens; e.g. letters → words.</dd>
  <dt>Sexylexy (n; proper):</dt>
  <dd>a damn fine s-expression lexer.</dd>
</dl>

@prettybad/lex is a lexer-generator library, targeted at delimited terms.
Generally speaking, this makes it unsuitable for arbitrary grammars; its
focus is on s-expressions, and similarly delimited, structured formats.
This focus on s-expressions is where the "sexy" comes from in "sexylexy."

A @prettybad lexer outputs a tree of tokens appropriate for use by a
@prettybad parser. The definition of the lexer is, effectively, a
definition of the Tokens and how they are arranged.

A Token is defined by its delimiters and a matching predicate for its
content (either a regular expression, or another Token). A Token can have
0, 1, or 2 delimiters. A 0-delimiter token can match anything. A
1-delimiter token is left-delimited, like using `'` for a symbol in Lisp.
A 2-delimiter token is delimited by a pair, like `"..."` or `(...)`.

Note that delimiters cannot occur within a token, except where the token
definition allows recursive nesting.

Tokens are lexed _independently_. That is: once a token is entered, only
its definition is considered by the lexer. For example, with the following
token definitions (where `...` stands for "any text"):

- Token: `(...)`
- Token: `[...]`

The inputs:

- `[[]]` begins to lex as token 2, but errors on the 2nd `[`.
- `(()`  begins to lex as token 1, but errors on the 2nd `(`.
- `[)))((()]` will lex as `[...]`.
- `(]]][[]])` will lex as `(...)`.

Once one of the tokens has been entered, only its definition is considered
by the lexer. However, if the token definitions were instead:

- Token: `parens`: `()` of `<parens>` or `<square>` or `...`
- Token: `square`: `[]` of `<square>` or `<parens>` or `...`

Then the first matching alternate for the token's contents will win every
time. So, the inputs:

- `[[]]` will lex as `square(square(...))`
- `(()`  begins to lex as `parens(parens(...))`, but errors on missing `)`
- `(([]))`    will lex as `parens(parens(square()))`
- `[)))(()(]` will lex as `square(... parens() ...)`
- `(]][[]]])` will lex as `parens(... square(square()) ...)`

An emphasis on delimiters means that @prettybad/lex is effectively
incapable of lexical analysis on free text. A focus on highly-structured
textual inputs keeps the code simpler and cleaner. @prettybad/lex is not,
by itself, a good compiler front-end; however, it is useful as a 2nd stage
in any good compiler front-end: if you cannot transform your input text
into an unambiguously-delimited set of expressions that can be modelled by
definitions in @prettybad/lex, then you probably have a very ambiguous
grammar, and your troubles are only just beginning.

Clearly, with all of these restrictions, it really is a pretty bad lexer.
It can't lex C++, that's for sure! It may not even be able to lex C!

## Putting it all together
An end-to-end textual analysis using these libraries looks like:

  text → lex → tokens → parse → nodes → forest

Where the lexer is defined and generated using @prettybad/lex; the parser
is defined and generated using @prettybad/parser; and the forest is an
instance of @prettybad/forest (AKA Forest).

The input text is a string or a rope (where the rope is a Forest).

Tokens are objects defined by the lexer definition, and expected as input
by the parser definition.

Nodes of the resulting Forest are transformations from Tokens, as defined
by the lexer and consumed by the parser, into arbitrary objects, as
defined by the parser and expected as the node-type(s) of the Forest.

There you have it, then: a pretty bad textual analysis pipeline. Just
don't miss the forest for the...

Seeya.
