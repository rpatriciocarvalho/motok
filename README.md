# motok

Encontre vagas públicas de moto perto do seu destino. Informe um endereço ou use sua localização atual, veja as vagas mais próximas no mapa ou em lista e abra a rota no Google Maps.

## Funcionalidades

- Mapa com a sua localização e as vagas ao redor, com alternância entre mapa e lista
- Busca por endereço
- Abre o Google Maps com a vaga como parada e o endereço informado como destino final
- Importação em lote dos dados abertos da prefeitura (shapefile e PDF com geocoding)
- Eventos de ocupação via Kafka, com score por vaga e mapa ao vivo
- Chatbot em linguagem natural com tool calling sobre a API de busca

## Arquitetura

```
Web (mapa e chat) ──► API Spring Boot ──► PostgreSQL + PostGIS
                         │  ▲                    ▲
                         │  └── Bedrock (tools)  │ carga em lote
                         ▼                       Importador
                       Kafka ──► Kafka Streams (score de ocupação) ──► PostgreSQL
```

## Stack

| Camada | Tecnologia |
| --- | --- |
| Backend | Java 25, Spring Boot |
| Banco | PostgreSQL com PostGIS |
| Streaming | Apache Kafka, Kafka Streams |
| IA | Spring AI, Amazon Bedrock |
| Geocoding | Amazon Location Service |
| Infra | AWS (ECS Fargate, RDS, MSK, S3, CloudFront), Terraform |
| Testes | JUnit, Testcontainers, `TopologyTestDriver` |
| CI/CD | GitHub Actions |

## Como rodar localmente

Pré-requisitos: JDK 25 e Docker.

```bash
# sobe PostgreSQL/PostGIS e Kafka
docker compose up -d

# importa o dataset
./mvnw spring-boot:run -Dspring-boot.run.profiles=import

# sobe a API
./mvnw spring-boot:run
```

Para ligar os recursos opcionais:

```bash
export FEATURE_OCUPACAO=true
export FEATURE_CHATBOT=true
```

> Ajuste os comandos e nomes de variáveis conforme o build escolhido (Maven ou Gradle).

## API

| Endpoint | Função |
| --- | --- |
| `GET /vagas?lat&lng&raio&horario` | Vagas próximas, com distância, score e link do Google Maps |
| `GET /vagas/{id}` | Detalhe de uma vaga |
| `POST /vagas/{id}/eventos` | Registra evento de ocupação |
| `POST /chat` | Pergunta em linguagem natural, resposta com lista de vagas |

## Testes

```bash
./mvnw verify
```

Os testes de integração usam Testcontainers (PostGIS e Kafka). O classificador de trechos do geocoding é testado com exemplos reais, sem chamar a API externa.

## Deploy

Provisionado com Terraform.

```bash
terraform apply
terraform destroy
```

## Dados

Os dados vêm dos dados abertos das prefeituras, inicialmente.