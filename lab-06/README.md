# Modelo para Apresentação do Lab06 - Patologias, Medicamentos e Efeitos Colaterais em Cypher


# Equipe `Sexteto Sinistro`

# Subgrupo `LGPD`
* `Gustavo Ferreira Gitzel` - `223559`
* `Maria Eduarda Elias Rocha` - `248408`
* `Pedro Sanchez Bitencourt` - `231133`

## Tarefa de Cypher sobre Patologias, Medicamentos e Efeitos Colaterais

## Exercício

Faça a projeção em relação a Patologia, ou seja, conecte patologias que são tratadas pela mesma droga.

### Resolução
~~~cypher
MATCH (p1:Pathology)<-[a]-(d:Drug)-[b]->(p2:Pathology)
MERGE (p1)<-[c:Connect]->(p2)
ON CREATE SET c.weight=1
ON MATCH SET c.weight=c.weight+1
~~~
~~~cypher
MATCH (p1:Pathology)<-[:Connect]->(p2:Pathology)
RETURN p1, p2
LIMIT 20
~~~
# Trabalhando com Efeitos Colaterais

## Exercício

Construa um grafo ligando os medicamentos aos efeitos colaterais (com pesos associados) a partir dos registros das pessoas, ou seja, se uma pessoa usa um medicamento e ela teve um efeito colateral, o medicamento deve ser ligado ao efeito colateral.

### Resolução
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/pathology.csv' AS line
CREATE (:Pathology { code: line.code, name: line.name})
~~~
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug.csv' AS line
CREATE (:Drug {code: line.code, name: line.name})
~~~
~~~cypher
create index for (d:Drug) on d.code
create index for (p:Pathology) on p.code
~~~
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
CREATE (:Person {code: line.idperson})

create index for(p:Person) on p.code
~~~
~~~cypher
load csv WITH HEADERS FROM  'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MATCH (d:Drug {code: line.codedrug})
MATCH (p:Person {code: line.idperson})
MERGE (p)-[u:Use]->(d)
ON CREATE SET u.weight=1
ON MATCH SET u.weight=u.weight+1
~~~
~~~cypher
load csv WITH HEADERS FROM  'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS line
MATCH (p1:Pathology {code: line.codePathology})
MATCH (p2:Person {code: line.idPerson})
MERGE (p2)-[h:Had]->(p1)
ON CREATE SET h.weight=1
ON MATCH SET h.weight=h.weight+1
~~~
~~~cypher
MATCH (pa:Pathology)<-[a]-(pe:Person)-[b]->(d:Drug)
MERGE (d)<-[c:Consequence]->(pa)
ON CREATE SET c.weight=1
ON MATCH SET c.weight=c.weight+1
~~~
~~~cypher
MATCH (d:Drug)<-[:Consequence]->(pa:Pathology)
RETURN d, pa
LIMIT 20
~~~

## Exercício

Que tipo de análise interessante pode ser feita com esse grafo?

Proponha um tipo de análise e escreva uma sentença em Cypher que realize a análise.

### Resolução
Uma possível analise seria encontrar um medicamento que tenha ao menos 40 ocorrências de pelo menos 1 medicamento, apontando assim os medicamentos que teriam mais efeitos colaterais registrados.
~~~cypher
MATCH (d:Drug)<-[c:Consequence]->(pa:Pathology)
where c.weight > 40
RETURN d, pa 
LIMIT 20
~~~