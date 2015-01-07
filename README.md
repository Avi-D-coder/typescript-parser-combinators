Some basic parser combinators. Not as pretty as in a language that supports operator overloading but still pretty good. Looking at the example grammar and parsing some s-expressions with `Grammar.parse` is a pretty good way to get a handle on it. The example grammar demonstrates pretty much all the combinators and how to handle recursion with `Parser.delay`. Everything is documented below.

## Base Parser
The inductive definition of the parsers bottoms out at `Parser.m` which takes a predicate function as a parameter. A predicate function can be anything as long as it returns a boolean value. Some examples of basic parsers

```
var match_a : Parser = Parser.m(x => x === 'a');
var match_b : Parser = Parser.m(x => x === 'b');
var match_a_or_b : Parser = Parser.m(x => x.test(/[ab]/));
var whitespace : Parser = Parser.m(x => x.test(/\s/));
```

Notice in the above examples that any function that takes a single element from the input stream and then tests it for some property is a valid way to construct basic parsers. In fact the elements of the input stream do not have to be characters so the full generality comes into effect when you start parsing things like `['abc', 'def', 'hij']`. In that instance the "elements" of the input stream are not single characters but the actual strings `'abc'`, `'def'`, `'hij'` and the basic parsers that would match those elements would be

```
var match_abc : Parser = Parser.m(x => x === 'abc');
var match_def : Parser = Parser.m(x => x === 'def');
var match_hij : Parser = Parser.m(x => x === 'hij');
```

## Non-base parsers
Once you have the base parsers for the basic lexical elements it is time to combine them into compound parsers. There are two very basic constructors to master here and almost everything else is either reducible to those two or is close enough in semantics that you won't lose anything even if you pretend they are reducible.

### Alternation
Just like with regular expressions you sometimes want to express the structure of something as a set of alternatives. For example, to test the set of integers and pick out all the two digit numbers that start with '2' or '3' you could use the regular expression `/(2|3)[0-9]/`. The pipe, `|`, is known as the alternation operator/combinator and the equivalent combinator exists for parsers. TypeScript does not have operator overloading so the alternation combinator is a method instead of a symbol. To combine two parsers using alternation you use the `.or` method. The regular expression example expressed as a parser would look as follows

```
var match_2 : Parser = Parser.m(x => x === '2');
var match_2 : Parser = Parser.m(x => x === '3');
var match_digit : Parser = Parser.m(x => /[0-9]/.test(x));
var regex_parser : Parser = (match_2.or(match_3)).then(match_digit);
```

That last parser also brings us to our next combinator.

### Sequencing
Sequencing is implicit with regular expressions and between alternation and sequencing it is the more primitive operation. To match the string '23' as a regular expression you just stick that string inside a regular expression `/23/` and whatever is inside is implicitly sequenced. With parser combinators we have to be a bit more explicit and the method we use to sequence parsers is `.then`. So that example spelled out with parsers looks like

```
var match_2 : Parser = Parser.m(x => x === '2');
var match_3 : Parser = Parser.m(x => x === '3');
var match_23 : Parser = match_2.then(match_3);
```
