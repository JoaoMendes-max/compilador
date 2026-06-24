# Compilador

Compilador de um subconjunto de C para uma ISA RISC de 16 bits própria.
Traduz `.c` → assembly de ponta a ponta: análise léxica, sintática, semântica,
geração de IR, alocação de registos e geração de código.

## Pipeline

```
.c → Lexer (Flex) → Parser (Bison) → AST
   → Análise Semântica (2 passes) → AST anotada
   → IR Lowering (3 endereços, registos virtuais)
   → Liveness → Interferência → Pre-color → Chaitin-Briggs → Spill (iterativo)
   → Code Generation → output.asm
```

`main.c` corre cada fase em sequência. O IR só corre se a semântica não tiver
erros; o codegen só corre se a alocação de registos convergir. Para o assembly
final correr, é ligado contra `runtime/runtime.asm` (helpers de mul/div/mod).

## Build e execução

Requer `gcc`, `flex` e `bison`.

```sh
make
./compiler programa.c      # gera output.asm
```

## Estrutura

```
Lexer/        Scanner Flex (lexer.l)
Parser/       Gramática Bison + AST (parser.y, ASTree, ASTPrint)
Semantic/     Análise semântica em 2 passes, símbolos, tipos, arena
IR/           IR e lowering (decl / expr / stmt)
CodeGen/      Geração de código
  RegAlloc/   Liveness, interferência, pre-color, Chaitin-Briggs, spill
Util/         Logger, NodeTypes
runtime/      runtime.asm (mul / div / mod por software)
assembler/    Assembler autónomo (asm → hex)
scripts/      Helpers de build e teste
test_files/   Programas de teste (IR, regalloc, semântica)
docs/         Especificações + relatório
main.c        Driver da pipeline
```

## Testes

```sh
make
scripts/test_compile_all.sh   # compila todos os testes e compara com snapshots
```

Os programas estão em `test_files/` (`IR_checks/`, `RegisterAllocation/`,
`semantic_checks/`).

## Limitações conhecidas

- Ponteiros para funções / chamadas indiretas — rejeitados.
- Struct/union por valor (cópia, parâmetros, retorno) — só acesso a campos.
- Floating point — bloqueado na semântica.
- `long long` / aritmética de 64 bits — não suportado.
- Inicializadores agregados (`{1, 2, 3}`) — arrays/structs ficam a zero.
- Funções variádicas — não suportadas.

## Documentação

- [Relatório do compilador (PDF)](docs/Compiler_Report.pdf)
- [docs/Compiler Overview.md](docs/Compiler%20Overview.md) — visão geral da pipeline.
- [docs/IR Specification.md](docs/IR%20Specification.md) — referência do IR.
- [docs/Code Generation Specification.md](docs/Code%20Generation%20Specification.md) — geração de código.
- [abi_spec.md](abi_spec.md) — convenção de registos e chamadas.
