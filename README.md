# dio-live-dynamodb
Repositório sobre o Amazon DynamoDB

### Serviço utilizado
  - Amazon DynamoDB
  - Amazon CLI para execução em linha de comando

### Login no sistema da Amazon
- Efeturar a clonagem do repositório
- Acessar o caminho da pasta src pelo WSL, para execução dos arquivos .json contidos nela
- Digitar ```aws configure``` para inserir as chaves de acesso
  - AWS Access Key ID []:
  - AWS Secret Access Key []:
  - Default region name []: us-east-1
  - Default output format []: json

### Comandos para execução do experimento:


- Criar uma tabela

```
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```
- Saída no terminal

![Criação da Tabela](./img/tabela_criada.PNG)

- Serviço na Amazon

![Criação da Tabela](./img/tabela_criada_nuvem.PNG)

- Inserir um item

```
aws dynamodb put-item \
    --table-name Music \
    --item file://itemmusic.json \
```

- Serviço na Amazon

![Adição de um intem](./img/item_inserido_na_tabela_nuvem.PNG)

- Inserir múltiplos itens

```
aws dynamodb batch-write-item \
    --request-items file://batchmusic.json
```

- Serviço na Amazon

![Adição de vários intens](./img/itens_inseridos_na_tabela_nuvem.PNG)

- Criar um index global secundário baseado no título do álbum

```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Serviço na Amazon

![Adição de vários intens](./img/index_global_album_nuvem.PNG)

- Criar um index global secundário baseado no nome do artista e no título do álbum

```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=Artist,AttributeType=S \
        AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ArtistAlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"Artist\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Serviço na Amazon

![Criação de Index Artista Album](./img/index_artista_album.PNG)

- Criar um index global secundário baseado no título da música e no ano

```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=SongTitle,AttributeType=S \
        AttributeName=SongYear,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"SongTitleYear-index\",\"KeySchema\":[{\"AttributeName\":\"SongTitle\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"SongYear\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Serviço na Amazon

![Criação de Index Artista Album](./img/criacao_index_musica_ano.PNG)

- Pesquisar item por artista

```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist" \
    --expression-attribute-values  '{":artist":{"S":"Iron Maiden"}}'
```

- Saída no terminal

![Pesquisa por Artista](./img/pesquisa_item_por_artista_realizada.PNG)

- Pesquisar item por artista e título da música

```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist and SongTitle = :title" \
    --expression-attribute-values file://keyconditions.json
```
- Saída no terminal

![Pesquisa por Artista e Musica](./img/pequisar_item_por_artista_musica_sem_index_realizada.PNG)

- Pesquisar item por música

```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "SongTitle = :title" \
    --expression-attribute-values  '{":title":{"S":"The Apparition"}}'
```

- Saída no terminal

![Pesquisa por Música](./img/pesquisar_item_por_musica_sem_index_falha.PNG)

- Pesquisa pelo index secundário baseado no título do álbum

```
aws dynamodb query \
    --table-name Music \
    --index-name AlbumTitle-index \
    --key-condition-expression "AlbumTitle = :name" \
    --expression-attribute-values  '{":name":{"S":"Fear of the Dark"}}'
```

- Saída no terminal

![Pesquisa por Música](./img/pesquisa_index_album.PNG)

- Pesquisa pelo index secundário baseado no nome do artista e no título do álbum

```
aws dynamodb query \
    --table-name Music \
    --index-name ArtistAlbumTitle-index \
    --key-condition-expression "Artist = :v_artist and AlbumTitle = :v_title" \
    --expression-attribute-values  '{":v_artist":{"S":"Iron Maiden"},":v_title":{"S":"Fear of the Dark"} }'
```

- Saída no terminal

![Pesquisa Index por Artista Álbum](./img/pesquisa_index_artista_album.PNG)

- Pesquisa pelo index secundário baseado no título da música e no ano

```
aws dynamodb query \
    --table-name Music \
    --index-name SongTitleYear-index \
    --key-condition-expression "SongTitle = :v_song and SongYear = :v_year" \
    --expression-attribute-values  '{":v_song":{"S":"Wasting Love"},":v_year":{"S":"1992"} }'
```

- Saída no terminal

![Pesquisa Index por Música Ano](./img/pesquisa_index_musica_ano.PNG)
