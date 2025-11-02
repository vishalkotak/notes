# Writing a Compiler in Go

![rw-book-cover](https://m.media-amazon.com/images/I/51TID4Qup3L._SY160.jpg)

## Metadata
- Author: [[Thorsten Ball]]
- Full Title: Writing a Compiler in Go
- Category: #books

## Highlights
- The difference between a stack and a register machine is – put in the most simple terms – whether the machine uses a stack to do its computations (like we did in our example above) or registers (virtual ones!). The debate’s still open on what’s the better (read: faster) choice, since it’s mostly about trade-offs and which ones you’re prepared to make. ([Location 465](https://readwise.io/to_kindle?action=open&asin=B07FZWWVQT&location=465))
    - Tags: [[orange]] 
- In our example above we used a switch statement do the dispatching in the run loop of our machine. ([Location 480](https://readwise.io/to_kindle?action=open&asin=B07FZWWVQT&location=480))
    - Tags: [[orange]] 
- But we would also have to write a new compiler for every computer architecture we want to run our programs on. That’s a lot of work. Instead, we can translate our programs into instructions for a virtual machine. And the virtual machine itself runs on as many architectures as its implementation language. In the case of the Go programming language that’s pretty portable. ([Location 506](https://readwise.io/to_kindle?action=open&asin=B07FZWWVQT&location=506))
    - Tags: [[orange]] 
- There are two possible orders, called little endian and big endian. Little endian means that the least significant byte of the original data comes first and is stored in the lowest memory address. Big endian is the opposite: the most significant byte comes first. ([Location 564](https://readwise.io/to_kindle?action=open&asin=B07FZWWVQT&location=564))
    - Tags: [[orange]] 
