---
link: https://gagor.pro/2026/01/a-practical-guide-to-building-a-parser-in-go/
byline: Tom
site: Tom's Blog
date: 2026-01-04T01:00
excerpt: A developer&rsquo;s journey of building a recursive descent parser in Go, from basic concepts to tackling left recursion with Packrat memoization.
tags:
  - golang
  - parser
slurped: 2026-02-24T10:32
title: A Practical guide to building a parser in Go
---

## Intro

It was during the Christmas period when I was looking for a book to read. I picked up [“Pragmatic Programmer”](https://gagor.pro/book/2017/the-pragmatic-programmer/) from the shelf.

![Cover of book The Pragmatic Programmer by Andrew Hunt, David Thomas](app://obsidian.md/book/2017/the-pragmatic-programmer/images/book-cover.webp)

The Pragmatic Programmer

Your journey to mastery

Authors: Andrew Hunt, David Thomas

I had already read parts of this book, but I wanted something lightweight and entertaining. It was this book that taught me to learn a new programming language each year, to ask “why am I in this meeting?” and other valuable lessons. It’s old, from a different era, and that’s partly what makes it fun. The authors loved using Perl to simplify daily tasks, and I did the same long time ago. I had a Perl script to automate web hosting configuration at one company, and another to move files between invoice systems. In my defense, this was long before Ansible and it was cutting-edge at the time 😁

Anyway, there’s also a chapter on custom DSLs ([Domain Specific Languages](https://en.wikipedia.org/wiki/Domain-specific_language) ), with an example grammar in BNF notation. BNF stands for [“Backus-Naur Form”](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form) - a formal notation for defining the syntax of programming languages and grammars. BNF was the original iteration, quite mathematical in nature. Later came EBNF (“Extended BNF”)[1](#fn:1) and ABNF (“Augmented BNF”)[2](#fn:2).

In the book there’s a task at the end of the chapter, to:

> Write a BNF grammar to validate time specifications. This grammar should accept each of these examples: 4pm, 7:38pm, 23:42, 3:16, 3:16am

The chapter included two more tasks: implement a simple BNF syntax analyzer using `bison` or `yacc`, or try implementing it yourself in Perl.

I had some free time, but I wasn’t interested in learning those ancient tools just to use them once. However, I’d been learning Go for the past year and hadn’t had many opportunities to write substantial software in it, despite building a few personal tools. I’d written many micro-languages and DSLs in Python, Groovy, and Ruby, but never in Go. This seemed like the perfect opportunity.

In 2-3 hours, I had a very basic script and quickly discovered how challenging parser writing is and how low-level Go feels compared to other languages. I started searching for books on compilers and parsers to better understand the concepts. Then I discovered the “Dragon Book” and realized I had found my passion project.

Yes, it’s one of the books from “Hackers” 😁

At that point, I knew I had to finish this 😁

## The Journey Begins: Three-Act Architecture

My goal was to build a parser that could validate strings against a BNF-like grammar[3](#fn:3).

BNF syntax is deceptively simple:

```
<symbol> ::= __expression__
```

Where:

- `<symbol>` is a “non-terminal” - it consists of or resolves to other symbols,
- `::=` separates the symbol name from its definition,
- `__expression__` is one or more sequences of terminals or non-terminals.

Basic BNF expressions use operators like:

- `|` - the OR operator, combining alternatives,
- `"string"` - literal text.

To make grammars more concise, EBNF and ABNF added operators like:

- `+` - one or more occurrences,
- `*` - zero or more occurrences,
- `?` - optional (zero or one),
- `(` and `)` - grouping.

My initial approach was simple, but as I added more features, I realized a solid architecture was essential. The solution was to split the process into three acts: the Lexer, the Parser, and the AST Builder.

### Act I: The Lexer - Tokenizing the grammar

Info

**The Lexer** converts raw grammar text (a string) into a sequence of tokens. For example, `<expr> ::= <term> "+" <term>` becomes tokens representing identifiers, operators, and literals. This process, called tokenization or lexical analysis, transforms characters into meaningful symbols that the parser can understand.

The first step in parsing is reading the BNF grammar itself. I used Go’s `bufio.Reader` to read character by character (“magic runes” 😉).

The basic lexer structure:

```
type Lexer struct {
	r *bufio.Reader
}

func NewLexer(r io.Reader) *Lexer {
	return &Lexer{r: bufio.NewReader(r)}
}

func (l *Lexer) Next() (Token, error) {
	// 1. skip whitespace and comments
	var ch rune
	var err error
	for {
		ch, _, err = l.r.ReadRune()
		if err == io.EOF {
			return Token{Type: EOF}, nil
		}
		if err != nil {
			return Token{}, err
		}
...
```

The lexer checks each character and returns tokens when patterns are detected. With `ReadRune()`, you get individual characters that can be compared and collected with `strings.Builder`. If you read too far, `UnreadRune()` lets you step back:

```
// 2. identifier
if isIdentStart(ch) {
	var sb strings.Builder
	sb.WriteRune(ch)

	for {
		ch, _, err := l.r.ReadRune()
		if err != nil {
			if err == io.EOF {
				break
			}
			return Token{}, err
		}
		if !isIdentPart(ch) {
			l.r.UnreadRune()
			break
		}
		sb.WriteRune(ch)
	}

	return Token{
		Type: IDENT,
		Text: sb.String(),
	}, nil
}
```

This straightforward approach with `strings.Builder` was new to me. For the complete implementation, see [lexer.go](https://github.com/tgagor/go-bnf/blob/main/bnf/lexer.go) .

### Act II: The Parser - Building a grammar AST

Info

**The Parser** consumes the stream of tokens from the lexer and builds an Abstract Syntax Tree (AST) representing the grammar’s structure. An AST is a tree where nodes represent rules, expressions, sequences, and choices. For example, `<number> ::= <digit> | <digit><number>` would have a Choice node with two Sequence children.

Once tokenization is complete, the parser takes over. It consumes tokens and builds an AST of the grammar structure. Consider this simple grammar:

```
<number> ::= <digit> | <digit><number>
<digit>  ::= "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
```

A key challenge is handling forward references and recursion. A rule might reference another rule that hasn’t been parsed yet (like `<number>` referring to `<digit>`), or refer to itself. The solution was a two-pass approach:

1. **First pass:** Parse all rules into an AST without resolving references. `NonTerminal` nodes store the name of the rule they refer to.
2. **Second pass:** Traverse the AST and “link” `NonTerminal` nodes to their actual rule nodes, creating a fully connected graph.

The lexer produces tokens:

```
// TokenType represents the category of a lexed token.
type TokenType int

const (
	EOF      TokenType = iota
	IDENT              // generic identifier or BNF rule name
	NT_IDENT           // BNF rule name explicitly in angle brackets (<rule>)
	STRING             // quoted string literal
	REGEX              // regex pattern enclosed in /.../
	ASSIGN             // the ::= operator
	PIPE               // the | operator
	STAR               // the * operator
	PLUS               // the + operator
	QMARK              // the ? operator
	LPAREN             // the ( operator
	RPAREN             // the ) operator
)

// Token represents a single atom (lexeme) in the input BNF grammar.
type Token struct {
	Type TokenType
	Text string
}
```

The parser iterates over these tokens and builds the AST tree. See [parser.go](https://github.com/tgagor/go-bnf/blob/main/bnf/parser.go) for details.

### Act III: The Matcher - Walking the tree

Info

**The Matcher** takes the fully parsed AST and matches an input string against it. Each node in the tree implements a `Match` method that checks if the input string at a given position matches that node’s rule.

With the grammar fully parsed and in memory as a walkable AST, the final act is matching an input string. Each node implements a `Match` method:

```
type Node interface {
    Match(input string, pos int) []int
}
```

- A `Terminal` node checks if the input string at the current position matches its literal value.
- A `Sequence` node tries to match its children one after another.
- A `Choice` node tries each child and collects all successful matches.

graph TD
    A{{"Node Interface  
Match input string, pos → []int"}} --> B["Terminal  
matches literal string"]
    A --> C["Sequence  
matches children in order"]
    A --> D["Choice  
tries each child"]

A grammar rule becomes a tree like this:

graph TD
    E{{"Rule: number ::= digit | digit number"}} --> F(("Choice node"))
    F --> G["Sequence: digit"]
    F --> H["Sequence: digit number"]
    G --> I[["Terminal: digit"]]
    H --> J[["Terminal: digit"]]
    H --> K[["NonTerminal: number"]]

This recursive design allows clean matching logic. When everything was connected, I was happy with the architecture and started writing grammars to test - until I hit “left recursion” 😁

## Left recursion and the Packrat solution

Things became complicated when I tried to implement a grammar for arithmetic expressions:

```
Expr ::= Expr "+" Term | Term
Term ::= "a"
```

This is a **left-recursive** grammar because `Expr` refers to itself as its first symbol. A naive recursive descent parser would enter an infinite loop trying to match `Expr` by calling itself without consuming input. My initial implementation did exactly that 🤷

Warning

**Left recursion** occurs when a rule’s first symbol is itself. For example, `Expr ::= Expr "+" Term` tries to match `Expr` by first matching `Expr`, leading to infinite recursion. This is a fundamental challenge in recursive descent parsing.

The solution was to implement **Packrat parsing**[4](#fn:4) using **memoization** - caching parsing results. If the parser is asked to match the same rule at the same position twice, it returns the cached result instead of re-evaluating.

To implement this, I introduced a `Context` object that holds the memoization table (the “packrat cache”).

```
type memoKey struct {
    node Node
    pos  int
}

type Context struct {
    input string
    memo  map[memoKey]*memoEntry
}
```

However, simple memoization isn’t enough for left recursion. The key insight is handling a cache hit for a rule currently _in progress_. The solution involved an iterative approach:

1. When a recursive rule is first encountered at a given position, it’s marked as “in progress” and a seed result (failure) is stored.
2. The parser then continues, and if it finds a productive path (one that consumes input), it updates the memoized result for the recursive rule.
3. The process is repeated, “growing” the match until no more input can be consumed. This iterative process effectively “unrolls” the left recursion.
4. To manage this, an `activeCounts` tracker was added to the context to keep track of active recursive calls and ensure that results are only memoized when the entire recursive evaluation is complete.

This robust implementation of Packrat parsing not only solved the left recursion problem but also gave the parser linear time complexity for many grammars, a huge performance win.

## From Validator to true Parser: Building the input AST

Initially, the parser was just a validator - it could tell you if a string matched, but not _how_ it matched. From a practical perspective, this was useless - you’d never know where you made a mistake or how to fix it.

The next major step in the journey was to evolve it into a true parser that could generate an AST of the _input string_.

This required changing the `Match` function to return not just the end positions, but also the nodes of the parse tree. The new API included a `Parse` method that returns an `ASTNode`.

```
type ASTNode struct {
    Name     string
    Value    string
    Children []*ASTNode
}
```

This was a significant change that rippled through the codebase, but the clean architecture paid off. With the AST, the parser could now provide a structured representation of the input, which is essential for any further processing, like interpretation or code generation. It also allowed for much better error reporting, as the parser could show the partial tree it had built before it failed. It could present where the failure happen and what symbols were expected there. The tool could be useful now!

## The Ultimate Test: Parsing Bash

With a now-robust parser in hand, I decided to put it to the ultimate test: parsing a real-world, complex grammar for the Bash shell. This adventure turned out to be a great learning experience. The Bash grammar was full of ambiguities and left recursion, pushing the parser to its limits.

It quickly became apparent that the performance was not good enough. The parser would hang for minutes on a simple 34-line bash script. Profiling revealed that the big number of backtracking paths was causing an exponential explosion in execution time.

This led to two key improvements:

1. **Recursion/Growth Limits**: To prevent hanging on pathological grammars, I added limits on recursive calls and growth iterations. If limits are exceeded, the parser fails with a clear message suggesting that the grammar is too complex or ambiguous.
    
2. **Regex Support**: Many Bash grammar parts could be expressed much more concisely and efficiently with regex. For example, a word could be `/[a-zA-Z_][a-zA-Z0-9_]*/` instead of recursive rules. Adding regex literals to the grammar syntax made the Bash grammar simpler and faster.
    

## Lessons learned

This project was a fantastic journey. It started as a simple textbook exercise and evolved into a powerful, robust Packrat parser.

My initial approach - just matching strings - wasn’t sufficient and didn’t scale. Rethinking it to separate the lexer, parser, and matcher was crucial. It made the codebase easier to understand, maintain, and extend. A more complex architecture, but far more extensible. When I added comments support, the lexer was the only place to change; the parser didn’t need to care. Adding new syntax required only extending the lexer and parser, leaving the matching logic untouched.

It was also a nice reminded, that theory matters. Learning concepts like left recursion and Packrat parsing was essential for building a correct and efficient parser. When I understood them, it was just relaxed matching of building blocks.

Go is a great language for this kind of projects. Its focus on simplicity, clear interfaces, and strong tooling made it a pleasure to work with. It reminds me a bit times when I was still coding in C, but easier. It’s also crazy fast. Except the `bash` grammar example, that was really complex, most other examples where parsed and analyzed in less that 0.3 ms.

The final result is a tool that is not only capable of parsing complex grammars but also serves as a testament to the learning process itself. The full source code is available on GitHub at [https://github.com/tgagor/go-bnf](https://github.com/tgagor/go-bnf) .
