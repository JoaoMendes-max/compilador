# Compiler

Compiler for a subset of C targeting a custom 16-bit RISC ISA (custom-designed processor). Translates `.c` → assembly end to end: lexical analysis, syntax analysis, semantic analysis, IR generation, register allocation, and code generation.

## Pipeline

```
.c → Lexer (Flex) → Parser (Bison) → AST
   → Semantic Analysis (2 passes) → annotated AST
   → IR Lowering (three-address, virtual registers)
   → Liveness → Interference → Pre-color → Chaitin-Briggs → Spill (iterative)
   → Code Generation → output.asm

```

`main.c` runs each phase in sequence. The IR only runs if semantic analysis has no errors; codegen only runs if register allocation converges. For the final assembly to run, it is linked against `runtime/runtime.asm` (mul/div/mod helpers).

## Build and run

Requires `gcc`, `flex`, and `bison`.

```
make
./compiler program.c      # generates output.asm
```

## Structure

```
Lexer/        Flex scanner (lexer.l)
Parser/       Bison grammar + AST (parser.y, ASTree, ASTPrint)
Semantic/     2-pass semantic analysis, symbols, types, arena
IR/           IR and lowering (decl / expr / stmt)
CodeGen/      Code generation
  RegAlloc/   Liveness, interference, pre-color, Chaitin-Briggs, spill
Util/         Logger, NodeTypes
runtime/      runtime.asm (software mul / div / mod)
assembler/    Standalone assembler (asm → hex)
scripts/      Build and test helpers
test_files/   Test programs (IR, regalloc, semantics)
docs/         Specifications + report
main.c        Pipeline driver

```

## Tests

```
make
scripts/test_compile_all.sh   # compiles every test and compares against snapshots
```

The programs are located in `test_files/` (`IR_checks/`, `RegisterAllocation/`, `semantic_checks/`).

## Known limitations

* Function pointers / indirect calls — rejected.
* Struct/union by value (copy, parameters, return) — field access only.
* Floating point — blocked at the semantic level.
* `long long` / 64-bit arithmetic — not supported.
* Aggregate initializers (`{1, 2, 3}`) — arrays/structs are left zeroed.
* Variadic functions — not supported.

## Documentation

* [Compiler Report (PDF)](https://github.com/JoaoMendes-max/compilador/blob/main/docs/Compiler_Report.pdf)
* [docs/Compiler Overview.md](https://github.com/JoaoMendes-max/compilador/blob/main/docs/Compiler%20Overview.md) — pipeline overview.
* [docs/IR Specification.md](https://github.com/JoaoMendes-max/compilador/blob/main/docs/IR%20Specification.md) — IR reference.
* [docs/Code Generation Specification.md](https://github.com/JoaoMendes-max/compilador/blob/main/docs/Code%20Generation%20Specification.md) — code generation.
* [abi_spec.md](https://github.com/JoaoMendes-max/compilador/blob/main/abi_spec.md) — register and calling convention.
