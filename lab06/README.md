## Tarefa de Cypher e o FDA Adverse Event Reporting System (FAERS)

## Exercício 1

Escreva uma sentença em Cypher que crie o medicamento de nome `Metamizole`, código no DrugBank `DB04817`.

### Resolução
~~~cypher
CREATE(:Drug {drugbank:"DB04817", name:"Metamizole"})
~~~

## Exercício 2

Considerando que a `Dipyrone` e `Metamizole` são o mesmo medicamento com nomes diferentes, crie uma aresta com o rótulo `:SameAs` que ligue os dois.

### Resolução
~~~cypher
MATCH(a:Drug {name:"Dipyrone"})
MATCH(b:Drug {name:"Metamizole"})
CREATE(a)-[:SameAs]->(b)

MATCH(b:Drug {name:"Dipyrone"})
MATCH(a:Drug {name:"Metamizole"})
CREATE(a)-[:SameAs]->(b)
~~~

## Exercício 3

Use o `DELETE` para excluir o relacionamento que você criou (apenas ele).

### Resolução
~~~cypher
MATCH (:Drug {name:"Dipyrone"})-[e]->(:Drug {name:"Metamizole"})
DELETE e

MATCH (:Drug {name:"Metamizole"})-[e]->(:Drug {name:"Dipyrone"})
DELETE e
~~~

## Exercício 4

Faça a projeção em relação a Patologia, ou seja, conecte patologias que são tratadas pela mesma droga.

### Resolução
~~~cypher
MATCH (p1:Pathology)<-[a]-(d:Drug)-[b]->(p2:Pathology)
WHERE a.weight > 20 AND b.weight > 20
MERGE (p1)<-[m:MesmaDrogaTrata]->(p2)
ON CREATE SET m.weight=1
ON MATCH SET m.weight=m.weight+1
~~~

## Exercício 5

Construa um grafo ligando os medicamentos aos efeitos colaterais (com pesos associados) a partir dos registros das pessoas, ou seja, se uma pessoa usa um medicamento e ela teve um efeito colateral, o medicamento deve ser ligado ao efeito colateral.

### Resolução
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
CREATE (:drugUse {drugCode:line.codedrug, codePathology:line.codepathology,idPerson:line.idperson})

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS line
CREATE (:sideEffect {idPerson:line.idPerson, codePathology:line.` codePathology`})

MATCH (u:drugUse)
MATCH (s:sideEffect)
MATCH (d:Drug)
MATCH (p:Pathology)
WHERE d.code=u.drugCode AND u.idPerson=s.idPerson AND s.codePathology=p.code
MERGE (d)-[h:hasSideEffect]->(p)
ON CREATE SET h.weight=1
ON MATCH SET h.weight=h.weight+1

MATCH (d:Drug)-[h:hasSideEffect]->(p:Pathology)
WHERE h.weight>30
RETURN d,p
~~~

## Exercício 6

Que tipo de análise interessante pode ser feita com esse grafo?

Proponha um tipo de análise e escreva uma sentença em Cypher que realize a análise.

### Resolução

Pode ser feita uma análise que leva em conta os efeitos colaterais mais reportados de cada medicamento e juntar medicamentos com o mesmo tipo de efeito colateral, a partir disso, podemos tentar criar uma classificação de cada medicamento de acordo com sua periculosidade (como as cores das tarjas dos medicamentos).

~~~cypher
MATCH (d1:Drug)-[h1:hasSideEffect]->(:Pathology)<-[h2:hasSideEffect]-(d2:Drug)
WHERE h1.weight > 20 AND h2.weight > 20
MERGE (d1)<-[a:Alike]->(d2)
ON CREATE SET a.weight=1
ON MATCH SET a.weight=a.weight+1

MATCH (d1:Drug)<-[:Alike]->(d2:Drug)
RETURN d1,d2
~~~