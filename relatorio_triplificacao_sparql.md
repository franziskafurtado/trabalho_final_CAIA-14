# Triplificação de CSV e consultas SPARQL

## Domínio

Violência doméstica e interpessoal contra mulheres, com base em amostra de 10 registros do CSV da SES/MG.

## Estrutura da triplificação

Cada linha do CSV foi transformada em um indivíduo da classe `vd:NotificacaoViolencia`. Para cada notificação também foram criados indivíduos relacionados: uma vítima (`vd:VitimaMulher`), um autor envolvido (`vd:AutorEnvolvido`), um município (`vd:Municipio`), um local de ocorrência (`vd:LocalOcorrencia`) e um ou mais tipos de violência (`vd:TipoViolencia`).

## Mapeamento CSV -> Knowledge Graph

| Coluna CSV | Propriedade RDF | Regra de mapeamento |
|---|---|---|
| `dt_notific` | `vd:dataNotificacao` | Literal xsd:date da notificação |
| `dt_nasc` | `vd:dataNascimento` | Literal xsd:date da vítima |
| `nu_idade_n` | `vd:idade` | Literal xsd:integer da vítima |
| `cs_sexo` | `vd:possuiSexo` | Objeto da classe vd:Sexo ligado à vítima |
| `cs_raca` | `vd:possuiRacaCor` | Objeto da classe vd:RacaCor |
| `id_mn_resi` | `vd:ocorreuEmMunicipio` | Objeto da classe vd:Municipio |
| `local_ocor` | `vd:ocorreuEmLocal` | Objeto da classe vd:LocalOcorrencia |
| `out_vezes` | `vd:possuiRecorrencia` | Objeto da classe vd:RecorrenciaViolencia |
| `les_autop` | `vd:possuiTipoViolencia` | Quando Sim, adiciona ex:violencia_autoprovocada |
| `viol_fisic` | `vd:possuiTipoViolencia` | Quando Sim, adiciona ex:violencia_fisica |
| `viol_psico` | `vd:possuiTipoViolencia` | Quando Sim, adiciona ex:violencia_psicologica |
| `viol_sexu` | `vd:possuiTipoViolencia` | Quando Sim, adiciona ex:violencia_sexual |
| `num_envolv` | `vd:possuiNumeroEnvolvidos` | Objeto da classe vd:NumeroEnvolvidos |
| `autor_sexo` | `vd:possuiSexo` | Objeto da classe vd:Sexo ligado ao autor envolvido |
| `orient_sex` | `vd:possuiOrientacaoSexual` | Objeto da classe vd:OrientacaoSexual |
| `ident_gen` | `vd:possuiIdentidadeGenero` | Objeto da classe vd:IdentidadeGenero |

## Exemplo de triples geradas

```turtle
ex:notificacao_001 a vd:NotificacaoViolencia ;
    rdfs:label "Notificação 001"@pt ;
    vd:dataNotificacao "2025-12-25"^^xsd:date ;
    vd:possuiVitima ex:vitima_001 ;
    vd:ocorreuEmMunicipio ex:municipio_virgolandia ;
    vd:ocorreuEmLocal ex:localocorrencia_residencia ;
    vd:possuiTipoViolencia ex:violencia_fisica .
```

## Consultas SPARQL e respostas esperadas

### Pergunta 1: Quantas notificações existem por município na amostra?

```sparql
PREFIX vd: <http://example.org/vd#>
PREFIX ex: <http://example.org/recurso/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

# Pergunta 1: Quantas notificações existem por município na amostra?
SELECT ?municipioLabel (COUNT(?notificacao) AS ?totalNotificacoes)
WHERE {
  ?notificacao a vd:NotificacaoViolencia ;
               vd:ocorreuEmMunicipio ?municipio .
  ?municipio rdfs:label ?municipioLabel .
}
GROUP BY ?municipioLabel
ORDER BY DESC(?totalNotificacoes) ?municipioLabel
```

**Resposta esperada na amostra:**

| municipioLabel | totalNotificacoes |
| --- | --- |
| Ibiá | 6 |
| Governador Valadares | 1 |
| Nazareno | 1 |
| Uberaba | 1 |
| Virgolândia | 1 |

### Pergunta 2: Quais notificações ocorreram em residência e tiveram violência física?

```sparql
PREFIX vd: <http://example.org/vd#>
PREFIX ex: <http://example.org/recurso/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

# Pergunta 2: Quais notificações ocorreram em residência e tiveram violência física?
SELECT ?notificacaoLabel ?data ?municipioLabel ?idade
WHERE {
  ?notificacao a vd:NotificacaoViolencia ;
               rdfs:label ?notificacaoLabel ;
               vd:dataNotificacao ?data ;
               vd:ocorreuEmLocal ex:localocorrencia_residencia ;
               vd:possuiTipoViolencia ex:violencia_fisica ;
               vd:ocorreuEmMunicipio ?municipio ;
               vd:possuiVitima ?vitima .
  ?municipio rdfs:label ?municipioLabel .
  ?vitima vd:idade ?idade .
}
ORDER BY ?data
```

**Resposta esperada na amostra:**

| notificacaoLabel | data | municipioLabel | idade |
| --- | --- | --- | --- |
| Notificação 006 | 2025-06-29 | Ibiá | 20 |
| Notificação 004 | 2025-07-29 | Ibiá | 39 |
| Notificação 010 | 2025-10-16 | Uberaba | 13 |
| Notificação 008 | 2025-10-22 | Ibiá | 43 |
| Notificação 002 | 2025-12-19 | Governador Valadares | 22 |
| Notificação 001 | 2025-12-25 | Virgolândia | 33 |

### Pergunta 3: Quais notificações pertencem a municípios com mais de 1 caso na amostra?

```sparql
PREFIX vd: <http://example.org/vd#>
PREFIX ex: <http://example.org/recurso/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

# Pergunta 3: Quais notificações pertencem a municípios com mais de 1 caso na amostra?
# Esta consulta usa subconsulta para calcular o total por município e depois lista os casos.
SELECT ?municipioLabel ?totalMunicipio ?notificacaoLabel ?data
WHERE {
  {
    SELECT ?municipio (COUNT(?notificacaoInterna) AS ?totalMunicipio)
    WHERE {
      ?notificacaoInterna a vd:NotificacaoViolencia ;
                          vd:ocorreuEmMunicipio ?municipio .
    }
    GROUP BY ?municipio
    HAVING (COUNT(?notificacaoInterna) > 1)
  }

  ?notificacao a vd:NotificacaoViolencia ;
               rdfs:label ?notificacaoLabel ;
               vd:dataNotificacao ?data ;
               vd:ocorreuEmMunicipio ?municipio .
  ?municipio rdfs:label ?municipioLabel .
}
ORDER BY DESC(?totalMunicipio) ?municipioLabel ?data
```

**Resposta esperada na amostra:**

| municipioLabel | totalMunicipio | notificacaoLabel | data |
| --- | --- | --- | --- |
| Ibiá | 6 | Notificação 005 | 2025-06-29 |
| Ibiá | 6 | Notificação 006 | 2025-06-29 |
| Ibiá | 6 | Notificação 004 | 2025-07-29 |
| Ibiá | 6 | Notificação 003 | 2025-08-30 |
| Ibiá | 6 | Notificação 008 | 2025-10-22 |
| Ibiá | 6 | Notificação 009 | 2025-10-27 |

### Pergunta 4: Qual é a média de idade das vítimas por tipo de violência?

```sparql
PREFIX vd: <http://example.org/vd#>
PREFIX ex: <http://example.org/recurso/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

# Pergunta 4: Qual é a média de idade das vítimas por tipo de violência?
SELECT ?tipoViolenciaLabel (AVG(?idade) AS ?mediaIdade) (COUNT(?notificacao) AS ?totalCasos)
WHERE {
  ?notificacao a vd:NotificacaoViolencia ;
               vd:possuiTipoViolencia ?tipoViolencia ;
               vd:possuiVitima ?vitima .
  ?tipoViolencia rdfs:label ?tipoViolenciaLabel .
  ?vitima vd:idade ?idade .
}
GROUP BY ?tipoViolenciaLabel
ORDER BY DESC(?totalCasos)
```

**Resposta esperada na amostra:**

| tipoViolenciaLabel | mediaIdade | totalCasos |
| --- | --- | --- |
| Violência física | 25.5 | 8 |
| Violência psicológica | 25.33333333333333333333333333 | 3 |
| Violência sexual | 6 | 1 |
| Violência autoprovocada | 13 | 1 |

### Pergunta 5: Quais casos recorrentes tiveram mais de um tipo de violência associado?

```sparql
PREFIX vd: <http://example.org/vd#>
PREFIX ex: <http://example.org/recurso/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

# Pergunta 5: Quais casos recorrentes tiveram mais de um tipo de violência associado?
# Esta consulta usa subconsulta para contar os tipos de violência por notificação.
SELECT ?notificacaoLabel ?data ?municipioLabel ?totalTiposViolencia
WHERE {
  {
    SELECT ?notificacao (COUNT(DISTINCT ?tipoViolencia) AS ?totalTiposViolencia)
    WHERE {
      ?notificacao a vd:NotificacaoViolencia ;
                   vd:possuiTipoViolencia ?tipoViolencia .
    }
    GROUP BY ?notificacao
    HAVING (COUNT(DISTINCT ?tipoViolencia) > 1)
  }

  ?notificacao rdfs:label ?notificacaoLabel ;
               vd:dataNotificacao ?data ;
               vd:ocorreuEmMunicipio ?municipio ;
               vd:possuiRecorrencia ?recorrencia .
  ?municipio rdfs:label ?municipioLabel .
  ?recorrencia rdfs:label "Sim"@pt .
}
ORDER BY DESC(?totalTiposViolencia) ?data
```

**Resposta esperada na amostra:**

| notificacaoLabel | data | municipioLabel | totalTiposViolencia |
| --- | --- | --- | --- |
| Notificação 010 | 2025-10-16 | Uberaba | 2 |
