# 🚀 Setup Local - Desafio Itaú

Este guia explica como configurar o ambiente local para rodar os microserviços do **Desafio Itaú - Compra Programada de Ações**.

---

## ⚠️ ATENÇÃO

Este README cobre **apenas a configuração da infraestrutura** (PostgreSQL, Kafka, Zookeeper e Schema do banco).

**Para instruções específicas sobre como rodar cada microserviço**, consulte os READMEs individuais nos respectivos repositórios:

- 📦 [Common Library](https://github.com/desafio-itau/itau-common-library) - Biblioteca compartilhada
- 👥 [Cliente Service](https://github.com/desafio-itau/itau-srv-gerenciamento-clientes) - Microserviço de gestão de clientes
- 📈 [Trading Service](https://github.com/desafio-itau/itau-srv-trading) - Microserviço de operações de trading

---

## 📋 Pré-requisitos

- **Docker** e **Docker Compose** instalados
- **Java 17+** instalado
- **Maven 3.8+** instalado
- **Git** instalado

## 📂 Estrutura de Pastas

Você precisará criar uma pasta raiz para organizar todos os projetos:

```
desafio-itau/
├── infraestrutura/          # Docker Compose + Schema
│   ├── docker-compose.yml
│   ├── env/
    |   └── .env
│   └── schema/
│       └── schema.sql
├── common-library/          # Biblioteca compartilhada
├── cliente-service/         # Microserviço de clientes
└── trading-service/         # Microserviço de trading
```

---

## 🛠️ Passo 1: Criar a Estrutura de Pastas

```bash
# Criar pasta raiz do projeto
mkdir desafio-itau
cd desafio-itau

# Criar pasta para o schema
mkdir -p infraestrutura/schema

# Criar pasta para as variáveis de ambiente
mkdir -p infraestrutura/schema
```

---

## 🐳 Passo 2: Configurar Docker Compose

### 2.1 Baixar o docker-compose.yml

Acesse o repositório: [desafio-itau/itau-infra](https://github.com/desafio-itau/itau-infra)

Copie o conteúdo do arquivo `docker-compose.yml` e cole em:

```
desafio-itau/infraestrutura/docker-compose.yml
```

### 2.2 Criar arquivo .env

Crie o arquivo `.env` dentro da pasta `infraestrutura/env`:

```bash
cd infraestrutura/env
touch .env
```

Adicione o seguinte conteúdo ao `.env`:

```env
# Configurações do PostgreSQL
DESAFIO_ITAU_DB_USER=usuario_do_seu_banco
DESAFIO_ITAU_DB_PASSWORD=senha_do_seu_banco
DESAFIO_ITAU_DB_NAME=nome_do_seu_banco
```

### 2.3 Baixar o schema.sql

Acesse o repositório: [desafio-itau/itau-sql-schema](https://github.com/desafio-itau/itau-sql-schema)

Copie o conteúdo do arquivo `schema.sql` e cole em:
```
desafio-itau/infraestrutura/schema/schema.sql
```

---

## 🚀 Passo 3: Subir a Infraestrutura

Agora que tudo está configurado, suba os containers:

```bash
cd infraestrutura
docker-compose up -d
```

**Aguarde** cerca de 30 segundos para todos os serviços ficarem saudáveis.

### Verificar status

```bash
docker-compose ps
```

Você deve ver todos os containers com status `Up (healthy)`:
```
NAME                COMMAND                  SERVICE      STATUS
itau-postgres       "docker-entrypoint.s…"   postgres     Up (healthy)
itau-kafka          "/etc/confluent/dock…"   kafka        Up (healthy)
itau-zookeeper      "/etc/confluent/dock…"   zookeeper    Up (healthy)
itau-kafka-ui       "/docker-entrypoint.…"   kafka-ui     Up
```

---

## 📊 Passo 4: Verificar se o Banco Foi Criado

Conecte ao PostgreSQL e verifique as tabelas:

```bash
docker exec -it itau-postgres psql -U postgres -d clientes_db
```

Dentro do PostgreSQL, execute:
```sql
\dt
```

Você deve ver todas as tabelas criadas:
```
                    List of relations
 Schema |         Name          | Type  |  Owner   
--------+-----------------------+-------+----------
 public | cestas_recomendacao   | table | postgres
 public | clientes              | table | postgres
 public | contas_graficas       | table | postgres
 public | cotacoes              | table | postgres
 public | custodias             | table | postgres
 public | distribuicoes         | table | postgres
 public | eventos_ir            | table | postgres
 public | itens_cesta           | table | postgres
 public | ordens_compra         | table | postgres
 public | rebalanceamentos      | table | postgres
```

Para sair do PostgreSQL:
```sql
\q
```

---

## 📦 Passo 5: Clonar e Instalar a Common Library

A `common-library` contém classes compartilhadas entre os microserviços (exceptions, configurações, etc).

### 5.1 Clonar o repositório

```bash
cd desafio-itau
git clone https://github.com/desafio-itau/itau-common-library.git
cd common-library
```

### 5.2 Instalar localmente

Como você **não tem acesso ao GitHub Packages**, instale a biblioteca no repositório Maven local:

```bash
mvn clean install -DskipTests
```

Isso instalará a biblioteca em `~/.m2/repository` e ficará disponível para os outros projetos.

---

## 🎯 Passo 6: Clonar e Rodar os Microserviços

### 6.1 Cliente Service

```bash
cd desafio-itau
git clone https://github.com/desafio-itau/itau-srv-gerenciamento-clientes.git
cd cliente-service

# Rodar o microserviço
mvn spring-boot:run
```

O serviço estará disponível em: `http://localhost:8080`

### 6.2 Trading Service

```bash
cd desafio-itau
git clone https://github.com/desafio-itau/itau-srv-trading.git
cd trading-service

# Rodar o microserviço
mvn spring-boot:run
```

O serviço estará disponível em: `http://localhost:8081`

---

## 🌐 Acessos

### PostgreSQL
- **Host:** `localhost`
- **Porta:** `5433`
- **Database:** `clientes_db`
- **Usuário:** `postgres`
- **Senha:** `postgres`

**JDBC URL:**
```
jdbc:postgresql://localhost:5433/nome_seu_banco
```

### Kafka
- **Bootstrap Server:** `localhost:9092`

### Kafka UI (Interface Web)
- **URL:** http://localhost:8090
- Visualize tópicos, mensagens e consumidores em tempo real

### Microserviços
- **Cliente Service:** http://localhost:8080
- **Trading Service:** http://localhost:8081
- **Swagger (Cliente):** http://localhost:8080/swagger-ui.html
- **Swagger (Trading):** http://localhost:8081/swagger-ui.html

---

## 🧪 Testando a Configuração

### Teste 1: PostgreSQL

```bash
docker exec -it itau-postgres psql -U postgres -d clientes_db -c "SELECT * FROM contas_graficas WHERE tipo = 'MASTER';"
```

Deve retornar a conta Master criada automaticamente:
```
 id | cliente_id | numero_conta | tipo   | data_criacao 
----+------------+--------------+--------+--------------
  1 |            | MASTER-001   | MASTER | 2026-02-27...
```

### Teste 2: Kafka

```bash
# Listar tópicos
docker exec -it itau-kafka kafka-topics --list --bootstrap-server localhost:9092

# Criar tópico de teste
docker exec -it itau-kafka kafka-topics --create \
  --topic teste \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1

# Enviar mensagem
echo "Hello Kafka" | docker exec -i itau-kafka kafka-console-producer \
  --topic teste \
  --bootstrap-server localhost:9092

# Consumir mensagem
docker exec -it itau-kafka kafka-console-consumer \
  --topic teste \
  --bootstrap-server localhost:9092 \
  --from-beginning \
  --max-messages 1
```

### Teste 3: Common Library

Verifique se a biblioteca foi instalada:

```bash
ls ~/.m2/repository/com/itau/common-library/
```

Deve mostrar a versão instalada (ex: `0.0.1`).

---

## 🛑 Comandos Úteis

### Parar os containers

```bash
cd infraestrutura
docker-compose down
```

### Parar e REMOVER volumes (apaga dados do banco)

```bash
docker-compose down -v
```

### Ver logs em tempo real

```bash
# Todos os serviços
docker-compose logs -f

# Apenas PostgreSQL
docker-compose logs -f postgres

# Apenas Kafka
docker-compose logs -f kafka
```

### Reiniciar tudo do zero

```bash
cd infraestrutura
docker-compose down -v
docker-compose up -d
```

Aguarde 30 segundos e verifique novamente.

---

## ⚠️ Troubleshooting

### Erro: "Porta 5433 já em uso"

Edite o `.env` e altere a porta:
```env
DESAFIO_ITAU_DB_PORT=5434
```

Depois edite o `docker-compose.yml`:
```yaml
ports:
  - "${DESAFIO_ITAU_DB_PORT}:5432"
```

### Erro: "Common library não encontrada"

Certifique-se de ter executado:
```bash
cd common-library
mvn clean install -DskipTests
```

### Kafka não está iniciando

Verifique se o Zookeeper está saudável:
```bash
docker-compose logs zookeeper
```

Aguarde alguns segundos e tente novamente.

### Schema não foi criado no banco

Verifique se o arquivo `schema/schema.sql` existe e execute:
```bash
docker-compose down -v
docker-compose up -d
```

---
