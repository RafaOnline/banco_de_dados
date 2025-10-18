# 🗄️ Comandos SQL — Guia Completo

Este documento reúne os principais comandos SQL utilizados para **manipulação e gerenciamento de bancos de dados**, com exemplos e descrições práticas.  
Compatível com PostgreSQL e outros SGBDs relacionais.

---

## 🧩 Criação e Estrutura de Tabelas
```sql
CREATE TABLE aluno (
    id SERIAL PRIMARY KEY,            -- auto incremento
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    idade INT CHECK (idade >= 0),     -- validação de valor
    sexo CHAR(1) DEFAULT 'M'          -- valor padrão
);
```
🔹 CHECK é ótimo para impor regras dentro da própria tabela.
Exemplo: impedir idades negativas.

## 🔒 Constraints (Restrições)

### **UNIQUE**
Impede que uma coluna tenha valores repetidos.
```sql
ALTER TABLE <tabela> 
ADD CONSTRAINT <nome_constraint> UNIQUE (<coluna>);
```

### **NOT NULL**
Impede que uma coluna receba valores nulos (vazios).
```slq
ALTER TABLE <tabela> 
ALTER COLUMN <coluna> SET NOT NULL;
```

## ✏️ Alterações em Tabelas

### Renomear Coluna
```sql
ALTER TABLE <tabela>
RENAME COLUMN <nome_antigo> TO <nome_novo>;
```

### Remover Coluna
```sql
ALTER TABLE <tabela>
DROP COLUMN <coluna>;
```

## 🔗 Chaves Estrangeiras (Foreign Keys)

### Adicionar FK
```sql
ALTER TABLE <tabela>
ADD COLUMN <coluna> <tipo> REFERENCES <tabela_referencia>(<coluna_referencia>);
```

### Referenciar FK
```sql
FOREIGN KEY (<coluna_atual>)
REFERENCES <tabela_referencia>(<coluna_referencia>);
```

### Adicionar FK a uma tabela existente
```sql
ALTER TABLE curso
ADD FOREIGN KEY (codigonivel)
REFERENCES nivel;
```
⚠️ Para remover uma coluna com FK, primeiro elimine as referências.

## 🧹 Exclusões

### Remover Coluna
```sql
ALTER TABLE <tabela> DROP <coluna>;
```

### Remover Tabela
```sql
DROP TABLE <tabela>;
```

### Remover em CASCADE
Elimina a tabela e todas as dependências.
```sql
DROP TABLE nivel CASCADE;
```
⚠️ Evite usar CASCADE sem necessidade, pois pode causar perda de dados em cascata.

## 🧩 Inserção de Dados

### Inserir Valores
```sql
INSERT INTO <tabela> (coluna1, coluna2, colunaN)
VALUES (valor1, valor2, valorN);
```

### Inserir com Atualização Condicional
Atualiza apenas quando houver conflito.
```sql
INSERT INTO clientes (nome, email, idade)
VALUES ('João Silva', 'teste@gmail.com', 31)
ON CONFLICT (email) DO UPDATE 
SET idade = EXCLUDED.idade;
```

## 📋 Cópia de Estruturas e Dados

### Copiar Estrutura da Tabela (sem dados)
```sql
CREATE TABLE clientes_backup AS TABLE clientes WITH NO DATA;
```

### Copiar Dados Específicos
```sql
INSERT INTO clientes_backup (nome, email, idade)
SELECT nome, email, idade 
FROM clientes 
WHERE idade > 30;
```

## 🔄 Atualizações

### Atualizar Dados
```sql
UPDATE disciplina 
SET cargahoraria = 70 
WHERE codigodisciplina = 2 
RETURNING *;
```

### Atualizar Múltiplos IDs
```sql
UPDATE clientes 
SET idreg = 1 
WHERE id IN (1, 2);
```

## Alterar Chave Primária / FK
```sql
ALTER TABLE cursodisciplina
DROP CONSTRAINT cursodisciplina_curso,
ADD CONSTRAINT cursodisciplina_curso
FOREIGN KEY (codigocurso) REFERENCES curso (codigocurso)
ON UPDATE CASCADE;
```

## Pontos de Transação
```sql
SAVEPOINT nome_ponto;
ROLLBACK TO nome_ponto;
```


## 🔍 Consultas (SELECT)

### Consulta Específica
```sql
SELECT coluna1 AS "Apelido 1",
       coluna2 AS "Apelido 2"
FROM tabela;


SELECT codigoaluno AS "Matrícula", 
       nome AS "Nome do discente",
       dtnascimento AS "Data de nascimento"
FROM aluno;
```

## 🕒 Funções de Data e Hora

### Função	Retorno
```sql
current_date	Data de hoje
current_time	Hora atual
current_timestamp	Data e hora
extract(campo from fonte)	Extrai partes da data/hora
Exemplo:
SELECT CURRENT_DATE AS "Data Atual",
       CURRENT_TIME AS "Hora Atual",
       CURRENT_TIMESTAMP AS "Data e Hora Atuais",
       EXTRACT(DOY FROM CURRENT_DATE) AS "Dia do Ano",
       EXTRACT(DOW FROM CURRENT_DATE) AS "Dia da Semana",
       EXTRACT(DAY FROM CURRENT_DATE) AS "Dia Atual",
       EXTRACT(MONTH FROM CURRENT_DATE) AS "Mês Atual",
       EXTRACT(YEAR FROM CURRENT_DATE) AS "Ano Atual",
       EXTRACT(CENTURY FROM CURRENT_DATE) AS "Século Atual";
```

### 📆 Nome do Dia da Semana
```sql
SELECT CASE 
    WHEN EXTRACT(DOW FROM CURRENT_DATE) = 0 THEN 'domingo'
    WHEN EXTRACT(DOW FROM CURRENT_DATE) = 1 THEN 'segunda-feira'
    WHEN EXTRACT(DOW FROM CURRENT_DATE) = 2 THEN 'terça-feira'
    WHEN EXTRACT(DOW FROM CURRENT_DATE) = 3 THEN 'quarta-feira'
    WHEN EXTRACT(DOW FROM CURRENT_DATE) = 4 THEN 'quinta-feira'
    WHEN EXTRACT(DOW FROM CURRENT_DATE) = 5 THEN 'sexta-feira'
    WHEN EXTRACT(DOW FROM CURRENT_DATE) = 6 THEN 'sábado'
END AS "Nome do dia da semana";
```

### 👶 Calculando Idade e Faixa Etária
```sql
SELECT nome,
       dtnascimento,
       AGE(dtnascimento) AS "Idade [ano/mês/dia]",
       EXTRACT(YEAR FROM AGE(dtnascimento)) AS "Idade do Aluno"
FROM aluno;
```

### 📊 Funções de Agregação
| Função |	Descrição |
|:------:|:----------:|
| COUNT(*) | Conta número de linhas|
| MIN() |	Retorna menor valor|
| MAX() |	Retorna maior valor|
| AVG() |	Calcula média|
| SUM() |	Soma valores|
| STDDEV() |	Calcula desvio padrão|
| VARIANCE() |	Calcula variância|

### 👁️ Criar Visões (Views)
```sql
CREATE VIEW vteste AS
SELECT nome,
       EXTRACT(YEAR FROM AGE(dtnascimento)) AS "Idade do Aluno",
       CASE 
           WHEN EXTRACT(YEAR FROM AGE(dtnascimento)) <= 20 THEN '1. até 20 anos'
           WHEN EXTRACT(YEAR FROM AGE(dtnascimento)) BETWEEN 21 AND 30 THEN '2. 21 a 30 anos'
           WHEN EXTRACT(YEAR FROM AGE(dtnascimento)) BETWEEN 31 AND 40 THEN '3. 31 a 40 anos'
           WHEN EXTRACT(YEAR FROM AGE(dtnascimento)) BETWEEN 41 AND 50 THEN '4. 41 a 50 anos'
           WHEN EXTRACT(YEAR FROM AGE(dtnascimento)) BETWEEN 51 AND 60 THEN '5. 51 a 60 anos'
           WHEN EXTRACT(YEAR FROM AGE(dtnascimento)) > 60 THEN '6. mais de 60 anos'
       END AS "Faixa Etária"
FROM aluno;
```

## 👥 Manipulação de Strings

### Concatenar Nomes
```sql
SELECT id, prim_nome || ' ' || ult_nome AS "Nome completo" 
FROM empregado;
```

## ⚙️ Operadores Lógicos e Comparativos
| Operador | Significado |
|:--------:|:-----------:|
| <	| Menor |
| <= | Menor ou igual |
| > | Maior |
| >= | Maior ou igual |
| = | Igual |
| <> | ou !=	Diferente 
| AND | Conjunção |
| OR | Disjunção |
| NOT | Negação |

## 🧠 Filtros com WHERE
```sql
SELECT nome, dtnascimento, sexo
FROM aluno
WHERE sexo = 'F' AND EXTRACT(MONTH FROM dtnascimento) = 11;
```

### IN
```sql
SELECT nome, dtnascimento
FROM aluno
WHERE EXTRACT(MONTH FROM dtnascimento) IN (7, 8, 9, 10, 11, 12);
```

### BETWEEN
```sql
SELECT nome
FROM aluno
WHERE EXTRACT(YEAR FROM dtnascimento) BETWEEN 1985 AND 2005;
```

### LIKE / NOT LIKE
```sql
SELECT nome FROM aluno WHERE nome LIKE '%COSTA%';
SELECT nome FROM aluno WHERE nome NOT LIKE '%MARIA%';
```

### lower()
```sql
SELECT lower(nome) FROM aluno;
```

### Pesquisar e Contar Emails
```sql
SELECT COUNT(*) AS quantidade
FROM aluno
WHERE email LIKE '%@gmail.%';
```

### NULL
```sql
SELECT * FROM aluno WHERE email IS NULL;
SELECT * FROM aluno WHERE email IS NOT NULL;
```

## 🧾 Ordenação e Agrupamento

### ORDER BY
```sql
SELECT nome, dtnascimento
FROM aluno
ORDER BY nome ASC;  -- ou DESC
```

### DISTINCT
```sql
SELECT DISTINCT sexo FROM funcionario;
```

### GROUP BY
```sql
SELECT sexo, COUNT(*) AS quantidade
FROM funcionario
GROUP BY sexo;
```
## 🔄 Junções (JOINS)

### INNER JOIN (junção mais comum)
```sql
SELECT c.nome AS curso, n.descricao AS nivel
FROM curso c
INNER JOIN nivel n ON n.codigonivel = c.codigonivel;
```
🔹 Retorna apenas os registros que têm correspondência nas duas tabelas.

### LEFT JOIN
```sql
SELECT c.codigocurso, c.nome, n.codigonivel, n.descricao
FROM curso c
LEFT JOIN nivel n ON n.codigonivel = c.codigonivel;
```

### RIGHT JOIN
```sql
SELECT c.codigocurso, c.nome, n.codigonivel, n.descricao
FROM curso c
RIGHT JOIN nivel n ON n.codigonivel = c.codigonivel;
```

### FULL JOIN
```sql
SELECT c.codigocurso, c.nome, n.codigonivel, n.descricao
FROM curso c
FULL JOIN nivel n ON n.codigonivel = c.codigonivel;
```

## 🔁 INTERSECT
Retorna registros que estão em ambas as tabelas.
```sql
SELECT nome, cpf FROM aluno
INTERSECT
SELECT nome, cpf FROM cliente;
```

## 📚 Resumo CRUD
| Operação | Descrição |
|:--------:|:---------:|
| CREATE |	Criar registros ou tabelas |
| READ | (SELECT)	Consultar dados |
| UPDATE |	Atualizar registros |
| DELETE |	Excluir registros |
