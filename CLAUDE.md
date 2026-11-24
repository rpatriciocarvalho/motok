# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projeto

Localizador de vagas públicas de moto em São Paulo. Pacote base: `com.rodrigopatricio.motok`.

**Estado atual:** esqueleto gerado pelo Spring Initializr (`MotokApplication` + teste de contexto). Nenhuma lógica de domínio ainda.

## Stack

- Java 25, Spring Boot 4.2 (Maven, via `./mvnw`)
- PostgreSQL + PostGIS (única fonte de verdade)
- Kafka / Kafka Streams (opcional, atrás de feature flag)
- AWS: ECS Fargate, RDS, MSK, Bedrock, Amazon Location Service
- Dependências atuais no `pom.xml`: actuator, webmvc, restclient, driver postgresql. Ainda **não** há JPA/JDBC, migração (Flyway/Liquibase), Kafka nem Testcontainers; adicionar conforme necessário.

## Comandos

```bash
docker compose up -d db                 # PostGIS local (postgis/postgis:18, db "motok", postgres/postgres, porta 5432)
./mvnw spring-boot:run                  # rodar a aplicação
./mvnw verify                           # build + todos os testes
./mvnw test -Dtest=NomeDaClasse         # uma classe de teste
./mvnw test -Dtest=NomeDaClasse#metodo  # um único teste
```

Não há lint/formatter configurado.

## Regras fixas de arquitetura

- **v1 sempre online:** mapa, busca por endereço e rota no Google Maps funcionam sem depender de Kafka ou chatbot.
- **Feature flags:** Kafka (somente eventos de ocupação) e chatbot ficam desligados por padrão; a v1 deve funcionar com ambos desligados.
- **Kafka nunca na importação de dados.** A importação grava direto no PostgreSQL, que é a única fonte de verdade (Kafka não é fonte de estado).
- **O chatbot (Bedrock) não acessa o banco**: só chama as tools expostas pela API.
- **Dev local** com Docker Compose; **AWS via IaC**, destruída após a demo.
- **Testes de integração com Testcontainers** (PostGIS real, e Kafka quando a flag estiver ligada); não usar H2.

## Estrutura de módulos (proposta, ainda não criada)

Monólito modular por pacote sob `com.rodrigopatricio.motok`:

- `vaga`: entidade, repositório e consultas geoespaciais (PostGIS)
- `importacao`: carga de dados públicos direto no PostgreSQL (sem Kafka)
- `busca`: geocodificação de endereço (Amazon Location Service) e busca por proximidade
- `rota`: geração de link de rota para o Google Maps
- `ocupacao`: eventos de ocupação, produtores/Streams (flag)
- `chatbot`: integração Bedrock e tools da API (flag)
- `config`: feature flags e configuração de infraestrutura

Infra como código (ex.: Terraform/CDK) em diretório separado, ex. `infra/`, a decidir.

## Repositório público

O código é público no GitHub; tudo que for commitado fica exposto no histórico.

- **Nada de segredos ou identificadores de conta no repositório:** chaves (Google Maps, AWS), senhas, IDs de conta, ARNs, endpoints de RDS/MSK, nomes de bucket. Configuração vem de variáveis de ambiente (Spring `${VAR}`); `.env` e `application-local.*` com valores reais entram no `.gitignore`. Se algo sensível for commitado por engano, é preciso rotacionar a credencial, não apenas removê-la.
- As credenciais `postgres/postgres` do `compose.yaml` são só para dev local; nunca reutilizar em AWS.
- **IaC sem valores fixos de conta/região:** usar variáveis e fornecer um arquivo de exemplo (`*.example`) com placeholders.
- Manter um `.env.example` com os nomes das variáveis exigidas, sem valores reais.
- **Dados públicos importados:** registrar fonte e licença no README e respeitar os termos de uso; não commitar dumps grandes, só o script de importação e instruções de download.
- Usar a API do Google Maps apenas para gerar o link de rota (sem chave no código cliente); respeitar os termos de uso.
- Não incluir dados pessoais (nomes, e-mails, contas) em código, testes, fixtures, mensagens de commit ou documentação.
- Demo na AWS é efêmera: documentar como subir e derrubar (`destroy`) sem expor o ambiente.

## Convenções

Código limpo; comentários só quando o porquê não for óbvio.
