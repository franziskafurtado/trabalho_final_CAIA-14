## Consultas SPARQL elaboradas com `CONSTRUCT`

Além das consultas `SELECT`, foram elaboradas duas consultas mais avançadas utilizando `CONSTRUCT`. Esse tipo de consulta não retorna uma tabela, mas sim um novo conjunto de triplas RDF, permitindo construir subgrafos ou gerar novos conhecimentos derivados a partir dos dados existentes no Knowledge Graph.

As consultas abaixo percorrem mais de um nó do grafo, relacionando notificações, vítimas, autores envolvidos, municípios, locais de ocorrência, recorrência e tipos de violência.

---

## Consulta elaborada 1

### Pergunta

**Quais notificações recorrentes ocorreram em residência e qual é o subgrafo completo desses casos, incluindo vítima, autor envolvido, município, local, recorrência, raça/cor, sexo e tipos de violência?**

### Caminho percorrido no grafo

A consulta percorre os seguintes caminhos:

```text
NotificacaoViolencia
→ possuiVitima
→ VitimaMulher
→ possuiRacaCor
→ RacaCor

NotificacaoViolencia
→ possuiVitima
→ VitimaMulher
→ possuiSexo
→ Sexo

NotificacaoViolencia
→ possuiAutorEnvolvido
→ AutorEnvolvido
→ possuiSexo
→ Sexo

NotificacaoViolencia
→ ocorreuEmMunicipio
→ Municipio

NotificacaoViolencia
→ ocorreuEmLocal
→ LocalOcorrencia

NotificacaoViolencia
→ possuiRecorrencia
→ RecorrenciaViolencia

NotificacaoViolencia
→ possuiTipoViolencia
→ TipoViolencia
```

### Query SPARQL

```sparql
PREFIX vd: <http://example.org/vd#>
PREFIX ex: <http://example.org/recurso/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

CONSTRUCT {
  ?notificacao a vd:CasoRecorrenteEmResidencia ;
               rdfs:label ?notificacaoLabel ;
               vd:dataNotificacao ?data ;
               vd:possuiVitima ?vitima ;
               vd:possuiAutorEnvolvido ?autor ;
               vd:ocorreuEmMunicipio ?municipio ;
               vd:ocorreuEmLocal ?local ;
               vd:possuiRecorrencia ?recorrencia ;
               vd:possuiTipoViolencia ?tipoViolencia .

  ?vitima a vd:VitimaMulher ;
          rdfs:label ?vitimaLabel ;
          vd:idade ?idade ;
          vd:possuiSexo ?sexoVitima ;
          vd:possuiRacaCor ?racaCor ;
          vd:possuiOrientacaoSexual ?orientacaoSexual ;
          vd:possuiIdentidadeGenero ?identidadeGenero .

  ?autor a vd:AutorEnvolvido ;
         rdfs:label ?autorLabel ;
         vd:possuiSexo ?sexoAutor .

  ?municipio rdfs:label ?municipioLabel .
  ?local rdfs:label ?localLabel .
  ?recorrencia rdfs:label ?recorrenciaLabel .
  ?tipoViolencia rdfs:label ?tipoViolenciaLabel .
  ?sexoVitima rdfs:label ?sexoVitimaLabel .
  ?sexoAutor rdfs:label ?sexoAutorLabel .
  ?racaCor rdfs:label ?racaCorLabel .
  ?orientacaoSexual rdfs:label ?orientacaoSexualLabel .
  ?identidadeGenero rdfs:label ?identidadeGeneroLabel .
}
WHERE {
  ?notificacao a vd:NotificacaoViolencia ;
               rdfs:label ?notificacaoLabel ;
               vd:dataNotificacao ?data ;
               vd:possuiVitima ?vitima ;
               vd:possuiAutorEnvolvido ?autor ;
               vd:ocorreuEmMunicipio ?municipio ;
               vd:ocorreuEmLocal ?local ;
               vd:possuiRecorrencia ?recorrencia ;
               vd:possuiTipoViolencia ?tipoViolencia .

  ?local rdfs:label "Residencia"@pt .
  ?recorrencia rdfs:label "Sim"@pt .

  ?vitima rdfs:label ?vitimaLabel ;
          vd:idade ?idade ;
          vd:possuiSexo ?sexoVitima ;
          vd:possuiRacaCor ?racaCor ;
          vd:possuiOrientacaoSexual ?orientacaoSexual ;
          vd:possuiIdentidadeGenero ?identidadeGenero .

  ?autor rdfs:label ?autorLabel ;
         vd:possuiSexo ?sexoAutor .

  ?municipio rdfs:label ?municipioLabel .
  ?local rdfs:label ?localLabel .
  ?recorrencia rdfs:label ?recorrenciaLabel .
  ?tipoViolencia rdfs:label ?tipoViolenciaLabel .
  ?sexoVitima rdfs:label ?sexoVitimaLabel .
  ?sexoAutor rdfs:label ?sexoAutorLabel .
  ?racaCor rdfs:label ?racaCorLabel .
  ?orientacaoSexual rdfs:label ?orientacaoSexualLabel .
  ?identidadeGenero rdfs:label ?identidadeGeneroLabel .
}
```

### Explicação

Essa consulta cria um novo subgrafo contendo apenas os casos que satisfazem duas condições:

```text
local da ocorrência = Residencia
recorrência da violência = Sim
```

O resultado da consulta classifica essas notificações como:

```text
vd:CasoRecorrenteEmResidencia
```

Com isso, a consulta não apenas recupera dados, mas também cria uma nova representação semântica para os casos recorrentes ocorridos em residência.

---

## Consulta elaborada 2

### Pergunta

**Como construir um grafo-resumo com o total de notificações e a média de idade das vítimas por município e tipo de violência?**

### Caminho percorrido no grafo

A consulta percorre o seguinte caminho:

```text
NotificacaoViolencia
→ ocorreuEmMunicipio
→ Municipio

NotificacaoViolencia
→ possuiTipoViolencia
→ TipoViolencia

NotificacaoViolencia
→ possuiVitima
→ VitimaMulher
→ idade
```

### Recursos SPARQL utilizados

Essa consulta utiliza:

```text
CONSTRUCT
Subconsulta
COUNT
AVG
BIND
IRI
CONCAT
ENCODE_FOR_URI
```

### Query SPARQL

```sparql
PREFIX vd: <http://example.org/vd#>
PREFIX ex: <http://example.org/recurso/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

CONSTRUCT {
  ?resumo a vd:ResumoMunicipioTipoViolencia ;
          rdfs:label ?resumoLabel ;
          vd:resumoMunicipio ?municipio ;
          vd:resumoTipoViolencia ?tipoViolencia ;
          vd:totalNotificacoes ?totalNotificacoes ;
          vd:mediaIdadeVitimas ?mediaIdade .

  ?municipio rdfs:label ?municipioLabel .
  ?tipoViolencia rdfs:label ?tipoViolenciaLabel .
}
WHERE {
  {
    SELECT ?municipio ?municipioLabel ?tipoViolencia ?tipoViolenciaLabel
           (COUNT(DISTINCT ?notificacao) AS ?totalNotificacoes)
           (AVG(?idade) AS ?mediaIdade)
    WHERE {
      ?notificacao a vd:NotificacaoViolencia ;
                   vd:ocorreuEmMunicipio ?municipio ;
                   vd:possuiTipoViolencia ?tipoViolencia ;
                   vd:possuiVitima ?vitima .

      ?vitima vd:idade ?idade .
      ?municipio rdfs:label ?municipioLabel .
      ?tipoViolencia rdfs:label ?tipoViolenciaLabel .
    }
    GROUP BY ?municipio ?municipioLabel ?tipoViolencia ?tipoViolenciaLabel
  }

  BIND(
    IRI(
      CONCAT(
        "http://example.org/recurso/resumo_",
        ENCODE_FOR_URI(STR(?municipioLabel)),
        "_",
        ENCODE_FOR_URI(STR(?tipoViolenciaLabel))
      )
    ) AS ?resumo
  )

  BIND(
    CONCAT(
      "Resumo: ",
      STR(?municipioLabel),
      " - ",
      STR(?tipoViolenciaLabel)
    ) AS ?resumoLabel
  )
}
```

### Explicação

Essa consulta cria um novo grafo de resumo a partir das notificações existentes. Para cada combinação entre município e tipo de violência, é criado um novo recurso da classe:

```text
vd:ResumoMunicipioTipoViolencia
```

Cada recurso de resumo contém:

```text
Município relacionado
Tipo de violência relacionado
Total de notificações
Média de idade das vítimas
```

Exemplos de recursos que podem ser criados:

```text
ex:resumo_Ibiá_Violência_física
ex:resumo_Ibiá_Violência_psicológica
ex:resumo_Uberaba_Violência_física
```

Essa consulta demonstra a geração de conhecimento derivado, pois cria novos nós de resumo a partir dos dados já existentes no Knowledge Graph.

---

## Versão auxiliar em `SELECT` para conferência da segunda consulta

Como a consulta `CONSTRUCT` retorna triplas RDF, também pode ser utilizada uma versão em `SELECT` para visualizar os resultados em formato de tabela no GraphDB.

```sparql
PREFIX vd: <http://example.org/vd#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?municipioLabel ?tipoViolenciaLabel
       (COUNT(DISTINCT ?notificacao) AS ?totalNotificacoes)
       (AVG(?idade) AS ?mediaIdade)
WHERE {
  ?notificacao a vd:NotificacaoViolencia ;
               vd:ocorreuEmMunicipio ?municipio ;
               vd:possuiTipoViolencia ?tipoViolencia ;
               vd:possuiVitima ?vitima .

  ?vitima vd:idade ?idade .
  ?municipio rdfs:label ?municipioLabel .
  ?tipoViolencia rdfs:label ?tipoViolenciaLabel .
}
GROUP BY ?municipioLabel ?tipoViolenciaLabel
ORDER BY DESC(?totalNotificacoes) ?municipioLabel ?tipoViolenciaLabel
```

### Finalidade da versão `SELECT`

Essa versão não cria novas triplas, mas facilita a visualização dos resultados em tabela. Ela pode ser usada para tirar print no GraphDB e demonstrar que os dados agregados utilizados na consulta `CONSTRUCT` estão corretos.

---

## Considerações sobre as consultas elaboradas

As duas consultas apresentadas são mais complexas do que consultas simples de seleção, pois percorrem múltiplos nós do Knowledge Graph e utilizam recursos avançados de SPARQL.

A primeira consulta constrói um subgrafo específico para casos recorrentes ocorridos em residência, permitindo representar semanticamente uma nova categoria de caso. A segunda consulta gera um grafo-resumo com informações agregadas por município e tipo de violência, criando novos recursos derivados dos dados originais.

Essas consultas demonstram o uso do SPARQL não apenas para recuperação de dados, mas também para construção de novas informações dentro de um grafo de conhecimento.
