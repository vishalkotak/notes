# CPython Internals

![rw-book-cover](https://m.media-amazon.com/images/I/61lFNpIBuwL._SY160.jpg)

## Metadata
- Author: [[Anthony Shaw]]
- Full Title: CPython Internals
- Category: #books

## Highlights
- The unique thing about CPython is that it contains both a runtime and the shared language specification that all other Python implementations use. ([Location 223](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=223))
    -  
- The Python language specification is the document that describes the Python language. For example, it says that assert is a reserved keyword and that [] is used for indexing, slicing, and creating empty lists. ([Location 224](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=224))

- For C, C++, and other compiled languages, the list of commands you need to execute to load, link, and compile your code in the right order can be very long. When compiling applications from source, you need to link any external libraries in the system. It would be unrealistic to expect the developer to know the locations of all of these libraries and to copy and paste them into the command line, so make and configure are commonly used in C/C++ projects to automate the creation of a build script. ([Location 555](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=555))
    
- The generated makefile is similar to a shell script and is broken into sections called targets. ([Location 561](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=561))

- The macOS, Linux, and Windows build processes have flags for profile-guided optimization (PGO). ([Location 744](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=744))
    
- PGO works by doing an initial compilation, then profiling the application by running a series of tests. ([Location 745](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=745))
    
- The profile is then analyzed, and the compiler makes changes to the binary that improve performance. ([Location 746](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=746))
    -  
- For CPython, the profiling stage runs python -m test --pgo, which executes the regression tests specified in Lib/test/libregrtest/pgo.py. ([Location 747](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=747))
    -  
- Function inlining: If a function is regularly called from another function, then it will be inlined, or copied into the calling function, to reduce the stack size. ([Location 758](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=758))
    -  
- Virtual call speculation and inlining: If a virtual function call frequently targets a certain function, then PGO can insert a conditionally ([Location 759](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=759))
    -  
    - Note: What does virtual call mean?
- executed direct call to that function. The direct call can then be inlined. ([Location 760](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=760))
    -  
- Register allocation optimization: Based on profile data results, the PGO will optimize register allocation. ([Location 761](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=761))
    -  
- Basic block optimization: Basic block optimization allows commonly executed basic blocks that temporally execute within a given frame to be placed in the same locality, or set of pages. It minimizes the number of pages used, which minimizes memory overhead. Hot spot optimization: Functions that the program spends the most execution time on can be optimized for speed. Function layout optimization: After PGO analyzes the call graph, functions that tend to be along the same execution path are moved to the same section of the compiled application. Conditional branch optimization: PGO can look at a decision branch, like an if … else if or switch statement, and spot the most commonly used path. For example, if there are ten cases in a switch statement, and one is used 95 percent of the time, then that case will be moved to the top so that it will be executed immediately in the code path. Dead spot separation: Code that isn’t called during PGO is moved to a separate section of the application. ([Location 762](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=762))
    -  
- One consideration when choosing a compiler is the system portability requirements. Java and .NET CLR will compile into an intermediary language so that the compiled code is portable across multiple system architectures. C, Go, C++, and Pascal will compile into an executable binary. This binary is built for the platform on which it was compiled. ([Location 779](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=779))
    -  
- Python applications are typically distributed as source code. The role of the Python interpreter is to convert the Python source code and execute it in one step. The CPython runtime compiles your code when it runs for the first time. This step is invisible to the regular user. ([Location 783](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=783))
    -  
- Python code isn’t compiled into machine code. It’s compiled into a low-level intermediary language called bytecode. This bytecode is stored in .pyc files and cached for execution. If you run the same Python application twice without changing the source code, then it will be faster on the second execution. This is because it loads the compiled bytecode instead of recompiling each time. ([Location 784](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=784))
    -  
- Self-hosted compilers are compilers written in the language they compile, such as the Go compiler. This is done by a process known as bootstrapping. Source-to-source compilers are compilers written in another language that already has a compiler. ([Location 792](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=792))
    -  
- The compiler needs strict rules for the grammatical structure for the language before it tries to execute it. ([Location 816](https://readwise.io/to_kindle?action=open&asin=B0BCNSDSYP&location=816))
    -  
