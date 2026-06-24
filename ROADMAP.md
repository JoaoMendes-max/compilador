# Roadmap — melhorias (V2)

Lista de melhorias a explorar nesta V2. Para cada uma: introdução breve,
termos para pesquisar, e os trade-offs. A ideia é implementá-las por mim,
uma de cada vez.

---

## 1. SSA e o que vem com ele

A coluna vertebral. São sub-passos encadeados.

### 1a. Promoção de variáveis para registos
Hoje cada variável vive num slot de memória e é lida/escrita a cada uso.
A técnica tira da memória os escalares cujo endereço nunca é tirado (`&x`),
passando-os a valores em registos. (O termo "mem2reg" é só o nome do *pass*
da LLVM; a técnica chama-se **register promotion** e faz-se ao construir SSA.)
- **Pesquisar:** `register promotion`, `scalar replacement`, `SSA construction`.

### 1b. Dominância
"A domina B" se todo o caminho do início até B passa por A. A *dominance
frontier* é onde é preciso colocar as φ. Infraestrutura para SSA, loops, LICM.
- **Pesquisar:** `dominator tree`, `dominance frontier`,
  `Cooper Harvey Kennedy simple fast dominance`.

### 1c. Forma SSA + φ
Cada valor é definido exatamente uma vez. Quando o mesmo valor pode vir de
blocos diferentes, insere-se uma **φ-function** no merge que escolhe a versão
consoante o caminho. Resultado: cadeias def→uso exatas.
- **Pesquisar:** `static single assignment form`, `phi function placement`,
  `Braun simple efficient SSA` (mais fácil), `Cytron SSA` (clássico).

### 1d. Saída de SSA
As φ não existem em hardware: convertem-se em cópias nos predecessores. Cuidado
com os bugs clássicos (*lost copy* e *swap*), que obrigam a cópias em paralelo.
- **Pesquisar:** `SSA destruction`, `out of SSA`, `lost copy problem`,
  `swap problem`, `parallel copy`.

### 1e. Alocação de registos para SSA
Em SSA o grafo de interferência é *chordal* → coloração **ótima em tempo
polinomial**, sem a heurística iterativa do Chaitin-Briggs. Alternativa simples:
*linear scan*. (O coalescing morto do regalloc atual revive aqui.)
- **Pesquisar:** `SSA based register allocation`, `chordal graph coloring`,
  `Hack Goos register allocation SSA`, `linear scan register allocation`,
  `iterated register coalescing`.

**Livros base:** Cooper & Torczon, *Engineering a Compiler*; *SSA Book*
(Rastello & Bouchez, PDF grátis).

---

## 2. Escalonamento de instruções (stalls)

Num CPU com pipeline, alguns resultados (ex.: loads) só ficam prontos uns ciclos
depois; se a instrução seguinte os precisa já, o CPU *stalla*. O escalonador
reordena as instruções do bloco, pondo trabalho independente no meio.
- **Trade-off:** reordenar alonga os tempos de vida → mais pressão de registos →
  mais spills. Decidir se corre antes ou depois da alocação.
- **Atenção:** só compensa se o processador tiver pipeline com latências reais.
  Se for single-cycle, não faz nada — confirmar a microarquitetura primeiro.
- **Pesquisar:** `list scheduling`, `instruction scheduling`,
  `dependence DAG`, `pipeline hazards`, `software pipelining` (avançado).

---

## 3. Peephole

Janela deslizante de 1–3 instruções sobre o assembly final, substituindo padrões
ineficientes (remover `MOV r,r`, colapsar load-após-store, identidades
algébricas). Simples mas local — não raciocina globalmente.
- **Bom primeiro projeto:** baixo risco, feedback imediato.
- **Pesquisar:** `peephole optimization`, `Davidson Fraser peephole`,
  `redundant load store elimination`, `superoptimization` (avançado).

---

## 4. Garbage collection

Decidir o tipo só depois de entender os eixos de design.

**Como descobre o lixo:** *tracing* (segue ponteiros a partir das raízes) vs
*reference counting* (conta referências; falha ciclos).
- **Pesquisar:** `tracing garbage collection`, `reference counting`,
  `tri-color marking`.

**O que faz com o lixo (tracing):** *mark-sweep* (não move, fragmenta) vs
*copying* (compacta, mas usa metade do heap e precisa de ponteiros precisos) vs
*mark-compact*.
- **Pesquisar:** `mark sweep`, `Cheney copying collector`, `mark compact`.

**Quanto sabe sobre ponteiros:** *conservador* (não precisa do compilador) vs
*preciso* (compilador emite *stack maps*; obrigatório para mover objetos).
- **Pesquisar:** `conservative garbage collection`, `precise GC`, `stack maps`,
  `Boehm conservative collector`.

**Produção (contexto):** *geracional*, *incremental/concorrente* (latência).
- **Pesquisar:** `generational hypothesis`, `write barrier`,
  `Garbage Collection Handbook`.

**Sugestão (a decidir):** o *conservador mark-sweep não-movedor* é o que se
integra com menos esforço (não precisa de stack maps), mas é uma de várias
opções.

---

## Ordem sugerida (phase ordering)

```
AST → IR → SSA → otimizações em SSA (constprop, DCE, GVN)
   → saída de SSA → alocação de registos → [scheduling, se houver pipeline]
   → peephole → assembly
```

Por ROI: **peephole** (praticar) → **SSA** (a fundação) → **GC** (isolado) →
**scheduling** (só se o CPU stalla).
