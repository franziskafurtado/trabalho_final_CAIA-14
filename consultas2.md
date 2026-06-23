### Pergunta

**Quais notificações recorrentes ocorreram em residência e qual é o subgrafo completo desses casos, incluindo vítima, autor envolvido, município, local, recorrência, raça/cor, sexo e tipos de violência?**


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

### Pergunta

**Como construir um grafo-resumo com o total de notificações e a média de idade das vítimas por município e tipo de violência?**

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