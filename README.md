# Resumão de Banco de Dados (SQL)

Este material é para **releitura rápida**, foco em **entendimento prático** e não decoreba.

## 📚 Índice

1. [Conceitos Fundamentais](#1-conceitos-fundamentais)
2. [SELECT (a base de tudo)](#2-select-a-base-de-tudo)
3. [WHERE (filtragem)](#3-where-filtragem)
4. [ORDER BY (ordenação)](#4-order-by-ordenação)
5. [Funções de Agregação](#5-funções-de-agregação)
6. [GROUP BY](#6-group-by-o-ponto-que-mais-confunde)
7. [HAVING](#7-having-where-do-group-by)
8. [JOINs](#8-joins-liga-tabelas)
9. [Índices](#9-índices-performance)
10. [Planner e Performance](#10-planner-e-performance)
11. [Erros Comuns](#11-erros-comuns-pra-lembrar-sempre)
12. [Mentalidade Correta](#12-mentalidade-correta)
13. [Transações](#13-transações)
14. [Isolamento de Transações](#14-isolamento-de-transações)
15. [Fenômenos Clássicos](#15-fenômenos-clássicos)
16. [Serialização](#16-serialização-serializable)
17. [MVCC](#17-mvcc-multi-version-concurrency-control)
18. [Locks](#18-locks-visão-prática)
19. [Hotspot](#19-hotspot)
20. [Window Functions](#20-window-functions-funções-de-janela)
21. [PARTITION BY](#21-partition-by)
22. [ORDER BY em Window Functions](#22-order-by-em-window-functions)
23. [Funções de Janela Comuns](#23-funções-de-janela-comuns)
24. [Quando usar Window Functions](#24-quando-usar-window-functions)
25. [Conexão entre os Conceitos](#25-conexão-entre-os-conceitos)
26. [Frase pra Guardar](#26-frase-pra-guardar)
27. [Algoritmos de JOIN](#27-algoritmos-de-join-muito-importante)
28. [Subqueries](#28-subqueries)
29. [Estruturas de Dados no Banco](#29-estruturas-de-dados-no-banco)
30. [Complexidade Big-O](#30-complexidade-big-o-versão-banco-de-dados)
31. [Planner / Optimizer](#31-planner--optimizer-revisão-rápida)
32. [O que NÃO vale obsessão](#32-o-que-não-vale-obsessão-agora)
33. [Checklist Mental de Entrevista](#33-checklist-mental-de-entrevista)
34. [Nível Realista](#34-nível-realista-opinião-honesta)
35. [Frase Final](#35-frase-final-pra-guardar)

---

## 1. Conceitos Fundamentais

### Tabela
- Conjunto de **linhas (registros)** e **colunas (atributos)**.
- Cada linha representa uma entidade (ex: um usuário).

### Chaves
- **Primary Key (PK)**: identifica unicamente uma linha.
- **Foreign Key (FK)**: referencia a PK de outra tabela.

### Normalização (resumo honesto)
- Evita dados duplicados.
- Deixa o banco mais consistente.
- Às vezes **desnormalizar é ok** por performance (com consciência).

---

## 2. SELECT (a base de tudo)

```sql
SELECT coluna1, coluna2
FROM tabela
WHERE condição;
```

Ordem lógica de execução:
1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY

(Isso explica muita confusão 😅)

---

## 3. WHERE (filtragem)

- Filtra **linhas antes de agrupar**.

```sql
WHERE idade > 18
AND ativo = true
```

Operadores comuns:
- =, !=, >, <, >=, <=
- AND / OR / NOT
- IN, BETWEEN, LIKE, IS NULL

---

## 4. ORDER BY (ordenação)

- Ordena o **resultado final**.

```sql
ORDER BY nome ASC
ORDER BY data DESC
```

---

## 5. Funções de Agregação

Trabalham **em conjunto com GROUP BY**.

- `COUNT(*)` → conta linhas
- `COUNT(coluna)` → conta valores **não nulos**
- `SUM(coluna)` → soma
- `AVG(coluna)` → média
- `MIN / MAX`

Exemplo:
```sql
SELECT COUNT(*)
FROM usuarios;
```

---

## 6. GROUP BY (o ponto que mais confunde)

Regra de ouro:
> **Tudo que está no SELECT e NÃO é agregação, TEM que estar no GROUP BY**.

Exemplo correto:
```sql
SELECT status, COUNT(*)
FROM pedidos
GROUP BY status;
```

Errado:
```sql
SELECT status, data, COUNT(*)
FROM pedidos
GROUP BY status; -- data quebra a regra
```

### Por que não posso usar `GROUP BY *`?
Porque o banco precisa saber **como agrupar**, e `*` não define grupos lógicos.

---

## 7. HAVING (WHERE do GROUP BY)

- Filtra **depois da agregação**.

```sql
SELECT status, COUNT(*)
FROM pedidos
GROUP BY status
HAVING COUNT(*) > 10;
```

Diferença clara:
- WHERE → antes de agrupar
- HAVING → depois de agrupar

---

## 8. JOINs (liga tabelas)

### INNER JOIN
- Só traz registros que existem nas duas tabelas.

### LEFT JOIN
- Traz tudo da esquerda, mesmo sem correspondência.

Exemplo:
```sql
SELECT u.nome, p.valor
FROM usuarios u
LEFT JOIN pedidos p ON p.usuario_id = u.id;
```

---

## 9. Índices (performance)

- Aceleram buscas.
- Não são mágicos.

⚠️ Evite índices em:
- Colunas com **baixa cardinalidade** (boolean, status simples).
- Tabelas muito pequenas.

Índices ajudam mais em:
- WHERE
- JOIN
- ORDER BY

---

## 10. Planner e Performance

- O banco escolhe o plano mais barato.
- Às vezes **Seq Scan é melhor que índice**.
- Índice errado pode piorar performance.

Nunca force otimização sem medir.

---

## 11. Erros comuns (pra lembrar sempre)

- Usar GROUP BY sem entender agregação.
- Criar índice pra tudo.
- Recriar ORM sem perceber 😂
- Otimizar cedo demais.

---

## 12. Mentalidade correta

- SQL não é decoreba → é **lógica**.
- Pense em:
  1. Que linhas eu quero?
  2. Vou agrupar?
  3. O que quero contar/somar?

Se você consegue responder isso, a query sai.

---

📌 **Use esse resumo pra revisão rápida.**
Se quiser, dá pra criar:
- versão ultra-curta (1 página)
- checklist de prova
- exercícios resolvidos
- mapa mental


## Concorrência, Transações, Performance e Análise

---

## 13. Transações

Transação = conjunto de operações que deve ser executado como **uma unidade lógica**.

### ACID
- **Atomicidade**: tudo ou nada
- **Consistência**: regras do banco respeitadas
- **Isolamento**: transações não se atrapalham
- **Durabilidade**: commit não se perde

---

## 14. Isolamento de Transações

Define **o quanto uma transação enxerga da outra**.

### Read Uncommitted
- Pode ler dados sujos (dirty read)
- Quase não usado

### Read Committed (padrão no Postgres)
- Só lê dados já commitados
- Pode ocorrer **non-repeatable read**

### Repeatable Read
- Leituras repetidas retornam o mesmo valor
- Ainda pode ocorrer **phantom read**

### Serializable
- Simula execução totalmente sequencial
- Mais seguro
- Mais custo

📌 Quanto maior o isolamento → mais segurança → menos concorrência.

---

## 15. Fenômenos Clássicos

- **Dirty Read**: ler dado não commitado
- **Non-repeatable Read**: mesmo SELECT retorna valores diferentes
- **Phantom Read**: novas linhas aparecem entre leituras

---

## 16. Serialização (Serializable)

- O banco garante que o resultado seja equivalente a uma execução **uma de cada vez**.
- Não significa execução realmente sequencial.
- Pode abortar transações automaticamente.

Use quando:
- Regra de negócio é crítica
- Erro não é aceitável

---

## 17. MVCC (Multi-Version Concurrency Control)

Ideia central:
> Leituras não bloqueiam escritas e vice-versa.

Como funciona:
- Cada transação vê uma **versão dos dados**
- UPDATE cria nova versão da linha
- Leituras usam snapshot

Vantagens:
- Alta concorrência
- Menos lock

Custo:
- Mais uso de disco/memória
- Precisa de vacuum (Postgres)

---

## 18. Locks (visão prática)

- Lock protege recursos
- Lock demais = gargalo

Tipos comuns:
- Row-level lock
- Table-level lock

MVCC reduz lock, mas **não elimina**.

---

## 19. Hotspot

Hotspot = ponto do sistema com **acesso excessivo concorrente**.

Exemplos:
- UPDATE constante na mesma linha
- Sequência global mal usada
- Tabela de contador

Consequências:
- Lock
- Contenção
- Queda de performance

Soluções comuns:
- Sharding lógico
- Counters distribuídos
- Reduzir escrita

---

## 20. Window Functions (funções de janela)

Permitem calcular valores **sem colapsar linhas**.

Diferença-chave:
- GROUP BY → reduz linhas
- WINDOW → mantém linhas

Exemplo:
```sql
SELECT nome,
       salario,
       AVG(salario) OVER () AS media_geral
FROM funcionarios;
```

## Diagrama de execução:
![Diagrama Window Functions](images/windowFunctions.png)

---

## 21. PARTITION BY

Divide a janela em grupos lógicos.

```sql
SELECT nome,
       departamento,
       salario,
       AVG(salario) OVER (PARTITION BY departamento)
FROM funcionarios;
```

---

## 22. ORDER BY em Window Functions

Define ordem dentro da janela.

```sql
SUM(valor) OVER (
  PARTITION BY cliente_id
  ORDER BY data
) AS acumulado
```

---

## 23. Funções de Janela Comuns

- ROW_NUMBER()
- RANK()
- DENSE_RANK()
- LAG()
- LEAD()
- SUM() OVER
- AVG() OVER

Usos típicos:
- Ranking
- Acumulados
- Comparar linha atual com anterior

---

## 24. Quando usar Window Functions

Use quando:
- Precisa de agregação + detalhe
- Ranking
- Relatórios

Evite quando:
- GROUP BY resolve
- Dataset enorme sem índice

---

## 25. Conexão entre os conceitos

- MVCC viabiliza concorrência
- Isolamento controla visibilidade
- Serializable força consistência
- Hotspot quebra performance
- Window Functions resolvem análise sem gambiarra

---

## 26. Frase pra guardar

> Performance ruim quase sempre é concorrência mal pensada.

---

📌 Use esse resumo como referência técnica.
Se quiser, posso criar:
- mapa mental desses conceitos
- exemplos de bugs reais por isolamento errado
- perguntas de entrevista com resposta curta


Este documento é o **fechamento do ciclo**: o que vale a pena revisar, o que é bônus e o que NÃO precisa pirar agora.

## 27. Algoritmos de JOIN (MUITO IMPORTANTE)

O planner escolhe **como** fazer o JOIN, não só **qual** JOIN.

### Nested Loop Join
- Um loop dentro do outro
- Bom para tabelas pequenas ou quando há índice
- Ruim para grandes volumes sem índice

Modelo mental:
> Para cada linha da tabela A, varre a B

---

### Hash Join
- Cria uma hash table em memória
- Muito rápido para igualdade (=)
- Não funciona com range

Requisitos:
- Memória suficiente
- Condição de igualdade

---

### Merge Join
- Exige dados ordenados
- Muito eficiente em grandes volumes
- Funciona bem com índices ordenados

Requisito-chave:
- ORDER BY compatível

---

📌 Entrevista gosta da frase:
> “O banco escolhe o tipo de JOIN baseado em custo.”

---

## 28. Subqueries

### Subquery Escalar
Retorna um único valor.

```sql
SELECT nome
FROM usuarios
WHERE salario > (SELECT AVG(salario) FROM usuarios);
```

---

### Subquery com IN / EXISTS

- `IN` → pode materializar resultado
- `EXISTS` → para na primeira correspondência

Dica prática:
> EXISTS costuma ser melhor que IN em grandes volumes.

---

### Subquery vs JOIN

- JOIN → quando quer dados relacionados
- Subquery → quando quer condição lógica

Planner muitas vezes transforma um no outro.

---

## 29. Estruturas de Dados no Banco

### Árvores (principalmente B-Tree)

- Base dos índices mais comuns
- Busca O(log n)
- Suporta range, ordenação

Por isso B-Tree ≠ árvore binária simples.

---

### Hash Index
- Busca O(1)
- Apenas igualdade
- Pouco usado em Postgres

---

## 30. Complexidade Big-O (VERSÃO BANCO DE DADOS)

Não é algoritmo puro, é **intuição de custo**.

- Seq Scan → O(n)
- Index Scan → O(log n)
- Nested Loop → O(n × m)
- Hash Join → O(n + m)
- Merge Join → O(n + m)

⚠️ Mas:
> Banco escolhe baseado em custo real, não só Big-O.

---

## 31. Planner / Optimizer (REVISÃO RÁPIDA)

O planner decide:
- Tipo de JOIN
- Uso de índice
- Ordem das tabelas

Baseado em:
- Estatísticas
- Cardinalidade
- Custo estimado

`EXPLAIN ANALYZE` é seu melhor amigo.

---

## 32. O que NÃO vale obsessão agora

- Implementar árvore na mão
- Provar Big-O formalmente
- Tunagem extrema
- Lock interno de baixo nível

Isso é pleno/sênior.

---

## 33. Checklist Mental de Entrevista

Você PRECISA conseguir explicar:

- Diferença entre WHERE e HAVING
- GROUP BY corretamente
- JOINs e quando usar cada um
- Índice: quando ajuda e quando atrapalha
- MVCC em alto nível
- Isolamento em linguagem simples
- Hash vs Merge vs Nested Join

Se você explica isso com calma → você passa.

---

## 34. Nível realista (opinião honesta)

Com tudo que você estudou até aqui:

✅ Passa em entrevista júnior
✅ Se destaca em SQL
✅ Demonstra maturidade técnica

O diferencial não é saber o nome — é saber **explicar com exemplo simples**.

---

## 35. Frase final pra guardar

> Banco de dados não é sobre decorar SQL, é sobre custo, concorrência e intenção.


