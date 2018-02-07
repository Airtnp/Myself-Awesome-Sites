# Compilers

## Intro
* Compiler
* + Lexical Analysis
* + Parsing
* + Semantic Analysis
* + Optimization
* + Code Generation
* Intermediate language
* Types
* + parameterization (* -> *)
* + subtyping (inheritance)

## Lexical analysis
* Tokens
* + Identifier
* + Literal
* + Keyword
* + Whitespace
* + Operator
* Step1: Define finte set of tokens
* Step2: Describe what kind of strings belongs to each token
* Impl1: Recognize substring of tokens
* Impl2: Return value or lexeme(substring) of the token
* Input String => Lexemes => Identify Lexemes
* Left-to-right scan: need lookahead
* Regular Expression
* + epsilon (empty)
* + character
* + union (A+B / A|B / [AB])
* + - range [a-z]
* + - excluded range [^a-z]
* + concatenation (AB)
* + Iteration (A*)
* + - A+ = AA*
* + - A? = A|ε
* Finite automata
* + DFA
* + NFA
* Lexical Spec => RegExp => NFA => DFA => Tables
* Lexical Spec => RegExp
* + regexp for each lexemes
* + L(R) = L(R1) + L(R2) + ... + L(Rn)
* + Test input on L(R)
* + - pick the longest possible string in L(R)
* + - pick the first token rule applies (like if as keyword instead of identifier)
* + - error handling => a last rule matching all bad strings
* Finite Automata (implement regexp)
* + epsilon-move
* + - DFA (no epsilon-move), one transition per input per state, fast
* + - NFA (has epsilon-move), can have multiple choice
* RegExp => NFA
* + eps: eps-move
* + input alpha: alpha-move
* + AB: A-eps-B
* + A|B: E-eps-A-eps-F + E-eps-B-eps-F
* + A*: E-eps-A-eps-F + A-eps-E + E-eps-F
* NFA => DFA
* + [Thompson](https://zhuanlan.zhihu.com/p/31158595)
* + simulate NFA
* + each state of DFA = a non-empty subset of states of the NFA
* + start state = set of NFA states reachable through eps-move from NFA start state
* + S-alpha-S' iff S' is the set of NFA states reachable from any state in S aftering seeing the input alpha, considering eps-move
* + eps-closure: DFS/BFS
* + DFA can be huge => 2^N - 1 subsets
* DFA minimization
* + [Hopcroft](https://zhuanlan.zhihu.com/p/31166841)
* + Given Q. First divide to A = {End State}, N = Q \ A
* + Consider N, if character c can divide into P, Q which P -c- J1 and Q -c-J2, then repeat this procedure on P, Q individually. Otherwise, choice next character.
* + Repeat this for A
* DFA => Table
* + States:Input Symbol
* + T[i, a] = k: Si-alpha-Sk
* + Execution: Read T[i, a]

## Parsing
* Not re: a^nb^n
* Input: sequence of tokens from lexer
* Output: parse tree
* CFG (Context-free grammar)
* + natural notation for recursive structure
* + A set of terminals T
* + A set of non-terminals N
* + A start symbol S (non-terminal)
* + A set of productions
* + - X -> Y1Y2...Yn where X in N and Y in T | N | {eps}
* + Begin with S (string)
* + Replace non-terminal X to Y1...Yn
* + Repeat until no non-terminal
* flex: re
* bison: cfg
* Derivation
* + sequence of productions (S -> ... -> ...)
* + can be drawn as a tree
* + start as root
* + production as children
* + in-order traversal of leaves => original input
* + Left-most / Right-most
* Ambiguity
* + `E = E + E | E * E | (E) | id`
* + => `E = E' + E | E'`
* + => `E' = id * E' | id | (E) * E' | (E)`
* + more than one parse tree
* + rewrite
* + operator precedence
* + - precedence climbing (Shunting yard)
* + `E = if E then E | if E then E else E | Other`
* + - `if E1 if E2 then E3 else E4`
* + - `else` match the closest unmatched `then`
```
E -> MIF | UIF
MIF -> if E then MIF else MIF | ...
UIF -> if E then E | if E then MIF else UIF
```
* + precedence and associativity
```
Error kind    Example    Detected by
Lexical       $          lexer
Syntax        x*%        parser
Semantic      int x; y=x type checker
Correctness   user       user
```
* Error handling
* + report error
* + recorver from error
* + no slow-down for compilation
* + panic mode
* + - discard token until one with a clear role (synchronizing tokens) is found
* + - statement/expression terminators
* + error production
* + - specify in grammar of common mistakes
* + automatic local/global correction
* + - try token insertion/deletion
* + - exhausive search
* Top-down parsing (Recursive Descent)
* + from top
* + from left to right
* + recursive descent
* + Token stream => Try rules in order => Mismatch backtrack
* + cannot backtrack since a production is successful
* + needs grammars where at most 1 production can succeed for a non-terminal
```c++
bool term(TOKEN tkn) { return *next++ == tkn; }
bool Sn() { ... } // nth production
bool S() { 
    TOKEN* current = next;
    return (next = current, S1()) || (next = current, S2()) || ... ;
} // all production
```
* + left-recursive => infinite loop
* + - `S -> Sa`
* + - elimination of left-recursion
* + - - `S -> S a | b` => `S -> b S'` | `S' -> a S' | esp`
* + - - `S -> S a1 | ... | S an | b1 | ... | bn`
* + - - => `S  -> b1 S' | ... | bn S'`
* + - - => `S' -> a1 S' | ... | an S' | esp`
* + - - What if `S -> A a | b` | `A -> S c`, then `S -> A a -> S c A`
* + - - [左递归消除](http://www.cnblogs.com/nano94/p/4020775.html)
```
Arrange all non-terminal symbol as order A1, ..., An
For i = 1 to n {
    For j = 1 to i - 1 {
        Denote Aj -> s1 | ... | sk
        Replace Ai -> Aj t to
            Ai -> s1 t | ... | sk t
        Eliminate left-recursion in Ai -> Ai a | ...
    }
}
```
* Predictive parser
* + look next few tokens
* + no backtracking
* + LL(k) (leeft-to-right-scan leftmost-derivation predict-based-on-k-tokens-of-lookahead)
* + LL(1) used often
* LL(1)
* + nonterminal left-most A
* + lookahead next input t
* + unique-production A -> a to use (or no production => error state)
* + left-factor grammar: factor out common prefixes of productions
* + [左公因式提取]()
* + parser table E[A, t]
* + - `A -> a -> t b` (t in First(A)) | `A -> a -> eps` & `S -> b A t c` (t in Follow(A))
```
First(X) = { t | X ->* t a } | { eps | X ->* eps }
First(t) = { t }
If X -> eps | (X -> A1...An & eps in First(Ai))
    eps in First(X)
If X -> A1...An a
    First(a) in First(X)

Follow(X) = { t | S ->*  b X t c }
$ in Follow(S)
For production A -> a X b
    First(b) \ { eps } in Follow(X)
For production A -> a X b while eps in First(b)
    Follow(A) in Follow(X)

For each production A -> a
    For each terminal t in First(a)
        T[A, t] = a
    If eps in First(a)
        For each t in Follow(A)
            T[A, t] = a
    If eps in First(a)
        If $ in Follow(A)
            T[A, $] = a
```
* + stack recording froniter of parse tree (top = leftmost pending terminal/non-terminal)
* Bottom-Up Parsing
* + reduce a string to start symbol by inverting productions
* + right-most derivation
* + split string into two substrings (stack)
```
shift: ABC|xyz => ABCx|yz (push to stack)
reduce: given A->xy, Cbxy|ijk = CbA|ijk (pop symbols and push non-terminals)
```
* + conflict
* + - shift-reduce
* + - reduce-reduce

## Semantics
* AST
* + grammar symbol => attributes
* + - terminal symbol (lexcial token) can be calculated by the lexer
* + production => action
* + - X->Y1...Yn
* Action => Syntax-directed translation
* + Declarative
* + - order of resolution is not specified
* + - parser figures it out
* + Imperative
* + - order of evaluation is fixed
* + - important if the actions change global state
* Dependency Graph
* + AST node => val (this value can be a ast)
* + compute all children
* + synthesized attributes (from descendents)
* + inherited attributes (from parent/siblings)
