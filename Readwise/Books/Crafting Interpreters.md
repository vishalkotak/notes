# Crafting Interpreters

![rw-book-cover](https://m.media-amazon.com/images/I/81dXJigFHiL._SY160.jpg)

## Metadata
- Author: [[Robert Nystrom]]
- Full Title: Crafting Interpreters
- Category: #books

## Highlights
- Or you can write a virtual machine (VM), a program that emulates a hypothetical chip supporting your virtual architecture at runtime. Running bytecode in a VM is slower than translating it to native code ahead of time because every instruction must be simulated at runtime each time it executes. ([Location 370](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=370))
    - Tags: [[pink]] 
- In a statically typed language like C++, method lookup typically happens at compile time based on the static type of the instance, giving you static dispatch. In contrast, dynamic dispatch looks up the class of the actual instance object at runtime. This is how virtual methods in statically typed languages and all methods in a dynamically typed language like Lox work. ([Location 780](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=780))
    - Tags: [[orange]] 
- Each of these blobs of characters is called a lexeme. ([Location 1086](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1086))
    - Tags: [[orange]] 
- That means the parser wants to know not just that it has a lexeme for some identifier, but that it has a reserved word, and which keyword it is. ([Location 1092](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1092))
    - Tags: [[orange]] 
- Since the scanner has to walk each character in the literal to correctly identify it, it can also convert that textual representation of a value to the living runtime object that will be used by the interpreter later. ([Location 1117](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1117))
    - Tags: [[orange]] 
- It’s sort of like advance(), but doesn’t consume the character. This is called lookahead. ([Location 1342](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1342))
    - Tags: [[orange]] 
- Comments are lexemes, but they aren’t meaningful, and the parser doesn’t want to deal with them. So when we reach the end of the comment, we don’t call addToken(). When we loop back around to start the next lexeme, start gets reset and the comment’s lexeme disappears in a puff of smoke. ([Location 1348](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1348))
    - Tags: [[orange]] 
- (This is why we used peek() to find the newline ending a comment instead of match(). We want that newline to get us here so we can update line.) ([Location 1362](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1362))
    - Tags: [[pink]] 
- Consider what would happen if a user named a variable orchid. The scanner would see the first two letters, or, and immediately emit an or keyword token. This gets us to an important principle called maximal munch. When two lexical grammar rules can both match a chunk of code that the scanner is looking at, whichever one matches the most characters wins. ([Location 1471](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1471))
    - Tags: [[orange]] 
- That rule states that if we can match orchid as an identifier and or as a keyword, then the former wins. This is also why we tacitly assumed, previously, that <= should be scanned as a single <= token and not < followed by =. ([Location 1475](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1475))
    - Tags: [[orange]] 
- Maximal munch means we can’t easily detect a reserved word until we’ve reached the end of what might instead be an identifier. ([Location 1485](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1485))
    - Tags: [[orange]] 
- regular language. That was fine for our scanner, which emits a flat sequence of tokens. But regular languages aren’t powerful enough to handle expressions which can nest arbitrarily deeply. ([Location 1651](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1651))
    - Tags: [[orange]] 
- We need a bigger hammer, and that hammer is a context-free grammar (CFG). ([Location 1652](https://readwise.io/to_kindle?action=open&asin=B09BCCVLCL&location=1652))
    - Tags: [[orange]] 
