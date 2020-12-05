## Tarefas com Publicações

## Questão 1
Construa uma comando SELECT que retorne dados equivalentes a este XPath
~~~xquery
//individuo[idade>20]/@nome
~~~

### Resolução
~~~sql
SELECT i.nome
FROM fichario f, individuos i
WHERE f.tipo = individuo AND f.chave = i.nome AND i.idade>20
~~~

## Questão 2
Qual a outra maneira de escrever esta query sem o where?

~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~
### Resolução
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo[idade>17])
return {data($i/@nome)}
~~~

## Questão 3
Escreva uma consulta SQL equivalente ao XQuery:
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')

for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~

### Resolução
~~~sql
SELECT i.nome
FROM fichario f, individuos i
WHERE f.tipo = individuo AND f.chave = i.nome AND i.idade>17
~~~

## Questão 4
Retorne quantas publicações são posteriores ao ano de 2011.

### Resolução
~~~xquery
let $info := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml")

return count($info/publications/publication[year>2011])
~~~

## Questão 5
Retorne a categoria cujo `<label>` em inglês seja 'e-Science Domain'.

### Resolução
~~~xquery
let $info := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml")

return $info/publications/categories/category[label/@lang="en-US" and label="e-Science Domain"]
~~~

## Questão 6
Retorne as publicações associadas à categoria cujo `<label>` em inglês seja 'e-Science Domain'. A associação entre o label e a key da categoria deve ser feita na consulta.

### Resolução
~~~xquery
let $info := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml")

for $pub in $info/publications/publication,
    $cat in $info/publications/categories/category[label/@lang="en-US" and label="e-Science Domain"]
where $pub/key = $cat/@key
return $pub
~~~

## Tarefas com DRON e PubChem

## Questão 1

Liste o nome de todas as classificações que estão apenas dois níveis imediatamente abaixo da raiz.

### Resolução
~~~xquery
let $info := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml")

for $i in $info/drug/drug/drug
return {data($i/@name)}
~~~

## Questão 2

Apresente todas as classificações de um componente a sua escolha (diferente de `Acetylsalicylic Acid`) que estejam hierarquicamente dois níveis acima. Note que no exemplo dado em sala foi considerado um nível hierárquico acima.

### Resolução
~~~xquery
let $info := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml")

for $i in $info//drug[drug/drug/@name="OLMESARTAN"]
let $gr := $i/@name
group by $gr
order by $gr
return {data($gr),'&#xa;'}
~~~

## Questão 3

### Questão 3.1

Liste todos os códigos ChEBI dos componentes disponíveis.

#### Resolução
~~~xquery
let $info := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml")

for $i in $info//drug[substring(@id,1,36) = "http://purl.obolibrary.org/obo/CHEBI"]
let $gr := substring($i/@id,38)
group by $gr
order by $gr
return {$gr ,'&#xa;'}
~~~

### Questão 3.2

Liste todos os códigos ChEBI e primeiro nome (sinônimo) de cada um dos componentes disponíveis.

#### Resolução
~~~xquery
let $info := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml")

for $i in $info//drug[substring(@id,1,36) = "http://purl.obolibrary.org/obo/CHEBI"]
let $gr := substring($i/@id,38)
let $gr_name := $i/@name
group by $gr,$gr_name
order by $gr,$gr_name
return {$gr, $gr_name ,'&#xa;'}
~~~

### Questão 3.3

Para cada código ChEBI, liste os sinônimos e o nome que aparece para o mesmo componente no DRON (se existir). Para isso faça um JOIN com o DRON.

http://purl.obolibrary.org/obo/DRON_00020015"
#### Resolução
~~~xquery
let $info := doc("https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml")

for $i in $info//drug[substring(@id,1,36) = "http://purl.obolibrary.org/obo/CHEBI"],
    $j in $info//drug[substring(@id,1,35) = "http://purl.obolibrary.org/obo/DRON"]
where $i/@name = $j/@name
let $gr_chebi := substring($i/@id,38)
let $gr_dron  := substring($j/@id,37)
let $gr_name := $i/@name
group by $gr_chebi,$gr_name, $gr_dron
order by $gr_chebi, $gr_dron ,$gr_name
return {$gr_chebi, $gr_dron, $gr_name ,'&#xa;'}
~~~