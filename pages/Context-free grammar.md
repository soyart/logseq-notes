- > Let $G$ be a CFG, then $L(G) =$ language of all strings generated by $G$
- A [[Push-down automata]] can recognize a string built from a CFG
- # Definition $G = (V, \Sigma, R, S)$
	- $V$ finite set of variables
	- $\Sigma$ finite set of terminal symbols
	- $R$ finite set of rules
	- $S$ a start variable
	- ## Example
		- $G_1$
		  $S \rightarrow  0S1$
		  $S \rightarrow  R$
		  $R \rightarrow  \epsilon$
		- **Variables** -> symbols that only appear on the LHS $= \{S, R\}$
		- **Terminals** -> symbols that only appear on RHS $= \{0, 1\}$
		- **Start variable** $S$ -> the root of all strings generated by this grammar $=S$
	- ### For $u, v \in (V \cup \Sigma)^\ast$
		- $u \Rightarrow v$ if can go from $u$ to $v$ with one step in $G$
		- $u \xRightarrow{\ast} v$ if can go with some number of substitutions
			- $u \Rightarrow u_1 \Rightarrow u_2 \Rightarrow \dots \Rightarrow v$ is called a derivation of $v$ from $u$
	- Then the definition of the language is $L(G) = \{w \mid w \in \Sigma^\ast$ and $S \xRightarrow{\ast} w\}$
- # Substitutions
	- ### Expand $S$ into RHS until only terminals remain
	- $G_1$
	  id:: 65a18e03-1693-4786-8f40-949ecf0c8097
	  $S \rightarrow  0S1$
	  $S \rightarrow  R$
	  $R \rightarrow  \epsilon$
		- In this case, $S$ and $R$ are variables, with $S$ being start variable, and terminals $= \{0, 1, \epsilon\}$
		- $G_1$ is *context-free*, because LHS only contains 1 variable symbol per rule
			- If a grammar rule has >1 LHS, then the grammar has *context*
		- We can further *simplify* $G_1$  by combining the 2 rules for $S$ :
		  $S \mapsto 0S1 \mid R$
		  $R \mapsto \epsilon$
		- This CFG can generate: $\{\epsilon, 01, 0011, 000111\}$, which is what [$\{0^k1^k \mid k \geq 0\}$](((65a173fb-af9f-49f8-9269-c92c2df58776))) describe
		- The generated strings are strings of the CFL
		- Expansion steps for $0011$ (*parse tree*)
		- {{renderer code_diagram,tikz}}
		  id:: 65a18fbb-2ed9-4307-931e-107d437bc62a
			- ```tikz
			  \usepackage{tikz}
			  \usetikzlibrary{positioning}
			  \begin{document}
			  \begin{tikzpicture}[
			  roundnode/.style={circle, draw=green!60, fill=green!5, very thick, minimum size=7mm},
			  squarednode/.style={rectangle, draw=blue!60, fill=red!5, very thick, minimum size=5mm},
			  ]
			  %Nodes
			  \node[roundnode]    (startvar)     {S};
			  
			  \node[roundnode]    (s1)           [below=of startvar] {S};
			  \node[squarednode]  (left0_0)      [left=of s1] {0};
			  \node[squarednode]  (right0_1)     [right=of s1 ] {1};
			  
			  \node[roundnode]    (s2)           [below=of s1] {S};
			  \node[squarednode]  (left1_0)      [left=of s2] {0};
			  \node[squarednode]  (right1_1)     [right=of s2] {1};
			  
			  \node[roundnode]    (r0)           [below=of s2] {R};
			  \node[squarednode]  (e)            [below=of r0] {$\epsilon$};
			  
			  %Lines
			  \draw[->] (startvar.south) -- (left0_0.north);
			  \draw[->] (startvar.south) -- (s1.north);
			  \draw[->] (startvar.south) -- (right0_1.north);
			  
			  \draw[->] (s1.south) -- (left1_0.north);
			  \draw[->] (s1.south) -- (s2.north);
			  \draw[->] (s1.south) -- (right1_1.north);
			  
			  \draw[->] (s2.south) -- (r0.north);
			  \draw[->] (r0.south) -- (e.north);
			  
			  \end{tikzpicture}
			  \end{document}
			  
			  ```
	- $G_2$
	  $S \rightarrow  0S$
	  $R \rightarrow  RR$
		- $G_2$ does not expand into terminals, only variables, so no strings will be produced.
		- $G_2$ **is also a CFG**, *but* its language just happens to be the *empty language* $\varnothing$
	- $G_3$
	  $E \rightarrow  E+T \mid T$
	  $T \rightarrow T\times F \mid F$
	  $F \rightarrow (F) \mid a$
		- $V_{G_3} = \{E, T, F\}$
		- $\Sigma_{G_3} = \{a, +, \times, (, )\}$
		- $S_{G_3} = T$
		- $G_3$ can generate these strings:
			- $a$
			- $a+a$
			- $a+a+a$
			- $a+a \times a$
			- $(a+a) \times a$
		- $G_3$ or similar grammars may be used in programming languages to parse arithmetic expression
		- #### $G_3$ also has precedence of $\times$ over $+$, as seen in the [parse tree](((65a18fbb-2ed9-4307-931e-107d437bc62a)))
			- This is because rule for $\times$ sit lower in the parse tree
			- So when parsing any strings, $G_3$ will evaluate $\times$ before $+$, unless the addition is also in paren, because paren sits lower in the parse tree than $\times$
			- We can build a different [ambiguous grammar](((65a2eedb-9300-450a-aa4e-81d6552b15e4))), $G_a$, that can generate the same strings as $G_3$, but with ambiguity (i.e. $G_a$ does not contain precedence of $\times$:
				- $E \rightarrow E + E \mid E \times E \mid (E) \mid a$
- # Ambiguity
  id:: 65a2eedb-9300-450a-aa4e-81d6552b15e4
	- If a string has >1 possible parse trees when parsed by a grammar $G$, then $G$ is ambiguous
	- i.e. ambiguity is when we may have different meanings from parsing the same string
	- For example, consider this English sentence string: `the boy saw the girl with the mirror`
	- We have ambiguity here - who had the mirror?
		- Did the boy hold the mirror when seeing the girl, *or* did the boy saw that the girl had mirror