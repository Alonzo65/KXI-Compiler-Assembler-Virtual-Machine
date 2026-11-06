# KXI Compiler, Assembler & Virtual Machine

A compiler toolchain built for my senior project. This repository brings together a **Python compiler for KXI**, a **C++ assembler**, and a **C++ virtual machine**, showing how a source program becomes assembly, then binary instructions, and finally a running program.

KXI is an educational language with C-like syntax, classes, methods, and typed variables. The project explores both compiler design and the runtime needed to execute generated code.

## How it works

```text
KXI source (.kxi or a text file)
    |
    v
Python compiler
  Tokenization -> Parsing / AST -> Semantic checks -> Transformations
    |
    v
output.asm
    |
    v
C++ assembler -> output.bin -> C++ virtual machine -> Console input / output
```

The assembler and virtual machine share one executable. Passing an `.asm` file assembles it; passing a `.bin` file executes it. These are two separate commands.

## Project highlights

- **Compiler front end:** tokenization and parsing with PLY, producing an abstract syntax tree (AST).
- **Semantic analysis:** scoped symbol tables and visitor passes for declarations, types, assignments, function calls, parameters, return values, and control flow.
- **Code generation:** tree transformations and visitor passes that produce assembly for the custom virtual machine.
- **Assembler:** two passes to resolve labels, validate instructions and operands, and encode instructions and `.INT` / `.BYT` data directives.
- **Virtual machine:** registers, arithmetic, branching, memory access, stack and heap management, and console input/output through trap instructions.
- **AST visualization:** an option to export the parsed program as a PNG using pydot and Graphviz.

The language grammar includes classes, constructors, public/private members, arrays, arithmetic and Boolean expressions, `if` / `else`, `while`, `switch`, `break`, `return`, `cin`, and `cout`.

## Finding your way around

The repository retains the original nested project folders:

| Component | Location |
| --- | --- |
| Python compiler | [compilercs4491-compiler4490-c4f508764044/compilercs4491-compiler4490-c4f508764044/](compilercs4491-compiler4490-c4f508764044/compilercs4491-compiler4490-c4f508764044/) |
| C++ assembler and VM | [FinalAssembler-main/FinalAssembler-main/AsssemblerVM/AssemblerVM/](FinalAssembler-main/FinalAssembler-main/AsssemblerVM/AssemblerVM/) |

Start with these files when exploring the implementation:

| File | Role |
| --- | --- |
| `compilerLex.py` | Keywords, token definitions, and lexer rules. |
| `Parser.py` | Grammar rules and the compiler's command-line entry point. |
| `nodes.py` | AST nodes and assembly instruction nodes. |
| `symboltable.py` | Symbol tables, activation records, and literal labels. |
| `semStarter.py` | Runs the semantic checking passes. |
| `visitor.py` | Semantic checks, tree transformations, visualization, and assembly generation. |
| `Main.cpp` | Selects assembly or execution based on the input file extension. |
| `Assembler.h` / `syntaxChecker.h` | Assembly parsing, label resolution, operand validation, and binary generation. |
| `VM.h` | Loads and executes the generated binary. |

## Build and run

The instructions below use **Windows and PowerShell** for the complete toolchain.

### Requirements

- **Python 3** for the compiler.
- **Visual Studio 2022** with the **Desktop development with C++** workload, the v143 toolset, and a Windows SDK for the assembler/VM. Use the project's **Debug / x64** configuration, which enables C++20.
- **Graphviz**, with `dot` available on `PATH`, if you want AST images or want to run the existing parser tests.

The compiler folder includes copies of **PLY** and **pydot**. Install **pyparsing** for pydot's DOT parsing dependency; you do not need to reinstall the bundled libraries.

### 1. Get the repository

```powershell
git clone https://github.com/Alonzo65/KXI-CompilerxAssembler.git
cd KXI-CompilerxAssembler
```

### 2. Build the assembler and virtual machine

Open this solution in Visual Studio 2022:

```text
FinalAssembler-main/FinalAssembler-main/AsssemblerVM/AssemblerVM/AsssemblerVM.sln
```

Select **Debug** and **x64**, then choose **Build > Build Solution**. With the default output settings, the executable is created at:

```text
FinalAssembler-main/FinalAssembler-main/AsssemblerVM/AssemblerVM/x64/Debug/AsssemblerVM.exe
```

The spelling `AsssemblerVM` (three consecutive `s` characters) matches the existing project and executable names.

### 3. Prepare the compiler

From the repository root, enter the compiler directory and install the additional Python dependency:

```powershell
cd .\compilercs4491-compiler4490-c4f508764044\compilercs4491-compiler4490-c4f508764044
python -m pip install pyparsing
```

Keep this directory as your working directory for the following commands.

### 4. Compile a small program

Save this as `example.kxi` in the compiler directory:

```text
void kxi2023 main() {
    int answer = 6 * 7;
    cout << answer;
    cout << '\n';
}
```

Run semantic checking first, resolve any reported errors, then generate assembly:

```powershell
python Parser.py example.kxi -s
python Parser.py example.kxi -c
```

Compilation writes **`output.asm`** in the current directory, replacing any existing file with that name.

### 5. Assemble and execute

Still in the compiler directory, point to the executable you built:

```powershell
$vm = "..\..\FinalAssembler-main\FinalAssembler-main\AsssemblerVM\AssemblerVM\x64\Debug\AsssemblerVM.exe"

& $vm .\output.asm
& $vm .\output.bin
```

The first command creates `output.bin`; the second runs it. The example prints:

```text
42
```

## Compiler modes

Use `python Parser.py <source-file> <flag>` from the compiler directory:

| Flag | Action |
| --- | --- |
| `-l` | Print each token's type, line number, and value. |
| `-p` | Parse the source and render its AST to `output.png`. Requires Graphviz. |
| `-s` | Build symbol tables and run semantic checks. |
| `-c` | Run the compilation passes and write `output.asm`. |

For example, inspect the tokens in the included demo:

```powershell
python Parser.py C.kxi -l
```

Run `-s` before `-c` when working on a program: the compilation driver can continue after semantic diagnostics, so an output file alone does not establish that the source is valid.

## Included examples and checks

- **`C.kxi`** is a larger interactive demo with recursive Fibonacci calculations, nested conditionals, arithmetic precedence, method calls, and switch statements. Its comments describe the expected inputs.
- **`lexer-test.kxi`, `syntax-test.kxi`, and `semantic-test.kxi`** contain material for inspecting individual compiler stages, including intentional error cases.
- **`test_pytest.py`** contains parser checks using the included text fixtures. Its helper also renders AST images, so Graphviz is required. Install `pytest` and run `python -m pytest test_pytest.py` from the compiler directory.
- **Assembly examples** in the assembler/VM directory include `Arrays1.asm`, `Arrays2.asm`, `HeapP1.asm`, and `Proj5Rec.asm` for exploring arrays, heap allocation, and recursion at the assembly level.

## Project notes

This repository preserves the senior project's source, examples, and development artifacts. The compiler and assembler are separate tools connected through the generated `.asm` file. Generated `.bin` files use this project's custom format and are executed by its virtual machine.

For a first look at the design, follow `compilerLex.py` -> `Parser.py` -> `visitor.py`, then explore `Assembler.h` and `VM.h` to see how the generated program is assembled and executed.
