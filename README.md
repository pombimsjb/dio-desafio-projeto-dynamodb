Repositório do desafio de projeto "Boas práticas com DynamoDB" do Bootcamp Banco "PAN Java Developer"


## Foi instalado e utlizado as seguintes ferramentas:

- AWS DynamoDB
- AWS CLI (Client) para execução em linha de comando
  - Configurando:
  - Foi feito o download no link [https://aws.amazon.com/pt/cli/](https://aws.amazon.com/pt/cli/).
  - Logo após no CMD do windows foi executado o comando aws configure.
  - Foi preenchido os dados de ACCESS KEY ID, SECRET ACCESS KEY, REGION NAME, OUTPUT FORMAT.

------

### Etapas de desenvolvimento do projeto
- Criação da Tabela/coleção 'musica' 

: Código ficou diferente do exemplo pois a máquina windows não aceitava as quebras de linha "\"

~~~ 
    aws dynamodb create-table --table-name Musica --attribute-definitions AttributeName=Artista,AttributeType=S AttributeName=TituloMusica,AttributeType=S --key-schema AttributeName=Artista,KeyType=HASH AttributeName=TituloMusica,KeyType=RANGE --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=5
~~~

------

- Para inserção do primeiro item com dados foi utilizado o arquivo itemMusica.json:
: Código ficou diferente do exemplo pois a máquina windows não aceitava as quebras de linha "\"

~~~
aws dynamodb put-item --table-name Musica --item file://itemMusica.json
~~~

------

- Para inserção de múltiplos itens foi utilizado o arquivo itemMusica.json e itemMusicaSystem.json em seguida:
: Código ficou diferente do exemplo pois a máquina windows não aceitava as quebras de linha "\"

~~~
aws dynamodb batch-write-item --request-items file://batchMusica.json
~~~
~~~
aws dynamodb batch-write-item --request-items file://batchMusicaSystem.json
~~~

------

### Ao chegar na parte de pesquisa por índice primário me deparei com meu primeiro problema.

#### O código abaixo retornava o erro "Expecting property name enclosed in double quotes

~~~
aws dynamodb query --table-name Musica --key-condition-expression "Artista = :artista" --expression-attribute-values '{":artista":{"S":"Iron Maiden"}}'
~~~

#### Foi formatado o código conforme abaixo para execução no windows porém retornou outro erro de parametros.

~~~
aws dynamodb query --table-name Musica --key-condition-expression "Artista = :artista" --expression-attribute-values '{\":artista\":{\"S\":\"Iron Maiden\"}}'
~~~

#### Foi então que chegamos a seguinte solução:

~~~
aws dynamodb query --table-name Musica --key-condition-expression "Artista = :artista" --expression-attribute-values  '{\":artista\":{\"S\":""Iron Maiden""}}'
~~~

#### Foi aplicado uma formação encontrada no [StackOverflow](https://stackoverflow.com/questions/51861707/error-parsing-parameter-expression-attribute-values-invalid-json-expecting) juntamente com uma explicação retirada do chat GPT onde ele indica que em casos em que se faz necessária a formatação de um JSON para CLI no windows, frases, títulos entre outros que contenham mais de uma palavra a ( \" ) não é aplicável.

------

### Utilizando o contexto acima, ao chegar na expressão abaixo tornou-se "complicado" utilizar o windows e passei para instalação do Ubuntu para windows Subsystem e comecei a trabalhar com os códigos conforme os exemplo.
### Neste código cria-se um index global secundário baseado no título do álbum.
~~~
aws dynamodb update-table \
    --table-name Musica \
    --attribute-definitions AttributeName=TituloAlbum,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"TituloAlbum-index\",\"KeySchema\":[{\"AttributeName\":\"TituloAlbum\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
~~~

------

### Utilizei do código abaixo para fazer uma pesquisa pelo index secundário baseado no título do album:
~~~
aws dynamodb query \
    --table-name Musica \
    --index-name TituloAlbum-index \
    --key-condition-expression "TituloAlbum = :nome" \
    --expression-attribute-values  '{":nome":{"S":"Fear of the Dark"}}'
~~~

------

### Criar index global secundário baseado no "Nome do artista" e no "Título do albúm". (Múltiplos parâmetros)

~~~
aws dynamodb update-table \
    --table-name Musica \
    --attribute-definitions\
        AttributeName=Artista,AttributeType=S \
        AttributeName=TituloAlbum,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ArtistaTituloAlbum-index\",\"KeySchema\":[{\"AttributeName\":\"Artista\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"TituloAlbum\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5},\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
~~~

------

### Código para pesquisa de index secundário com base no "Nome do artista" e no "Título do álbum"
~~~
aws dynamodb query \
    --table-name Musica \
    --index-name ArtistaTituloAlbum-index \
    --key-condition-expression "Artista = :v_artista and TituloAlbum = :v_tituloAlbum" \
    --expression-attribute-values  '{":v_artista":{"S":"Iron Maiden"},":v_tituloAlbum":{"S":"Fear of the Dark"} }'
~~~

------

### Código para criar index global secundário baseado no "Título da música" e no "Ano"
~~~
aws dynamodb update-table \
    --table-name Musica \
    --attribute-definitions\
        AttributeName=TituloMusica,AttributeType=S \
        AttributeName=AnoMusica,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"TituloAnoMusica-index\",\"KeySchema\":[{\"AttributeName\":\"TituloMusica\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AnoMusica\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5},\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
~~~

------

### Código para pesquisar pelo index secundário baseado no "Título da música" e no "ano".
~~~
aws dynamodb query \
    --table-name Musica \
    --index-name TituloAnoMusica-index \
    --key-condition-expression "TituloMusica = :v_titulo_musica and AnoMusica = :v_ano_musica" \
    --expression-attribute-values  '{":v_titulo_musica":{"S":"Wasting Love"},":v_ano_musica":{"S":"1992"} }'
~~~

------

### Código para pesquisar item por "Artista" e "Título da música"(Index primário) com uso do arquivo com os dados.
~~~
aws dynamodb query \
    --table-name Musica \
    --key-condition-expression "Artista = :artista and TituloMusica = :tituloMusica" \
    --expression-attribute-values file://chavesCondicoes.json
~~~