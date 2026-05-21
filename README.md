# Atividade Avaliativa 2 - Banco de Dados II

## Objetivo

Executar a aplicação Laravel, identificar as consultas SQL geradas pelo ORM (Eloquent), executar as consultas diretamente no MySQL e analisar possíveis problemas de desempenho e otimizações.

---

## Tecnologias utilizadas

- Laravel
- MySQL 8.4
- Docker
- Docker Compose
- PHP 8.4
- Composer

---

# Configuração do ambiente

## docker-compose.yml

```yaml
services:

  mysql:
    image: mysql:8.4
    container_name: db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app_biblioteca
      MYSQL_USER: dev
      MYSQL_PASSWORD: minhasenha
    ports:
      - "3307:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init-db:/docker-entrypoint-initdb.d

  php:
    build: .
    container_name: php
    working_dir: /app
    volumes:
      - ./:/app
    ports:
      - "8000:8000"

  composer:
    image: composer
    container_name: composer
    working_dir: /app
    volumes:
      - ./:/app

volumes:
  mysql_data:
```

---

## Dockerfile

```dockerfile
FROM php:8.4-cli

RUN docker-php-ext-install pdo_mysql

WORKDIR /app
```

---

## Configuração do .env

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=app_biblioteca
DB_USERNAME=dev
DB_PASSWORD=minhasenha
```

---

# Comandos utilizados

## Subir containers

```bash
docker compose up -d --build
```

---

## Instalar dependências

```bash
docker compose run --rm composer install
```

---

## Gerar chave da aplicação

```bash
docker compose run --rm php php artisan key:generate
```

---

## Executar migrations

```bash
docker compose run --rm php php artisan migrate
```

---

## Popular banco de dados

```bash
docker compose run --rm php php artisan db:seed
```

---

## Executar aplicação

```bash
docker compose run --rm -p 8000:8000 php php artisan serve --host=0.0.0.0
```

Aplicação disponível em:

```text
http://localhost:8000
```

---

# Captura das consultas SQL

Foi utilizado o Laravel Debugbar para visualizar as consultas SQL executadas pelo ORM.

## Instalação da Debugbar

```bash
docker compose run --rm composer require barryvdh/laravel-debugbar --dev
```

---

# Consulta analisada

```sql
select * from pessoas;
```

---

## EXPLAIN

```sql
EXPLAIN select * from pessoas;
```

Resultado:

```text
type = ALL
key = NULL
rows = 11
```

---

# Problemas identificados

## Uso de SELECT *

A consulta utiliza:

```sql
SELECT *
```

Problemas:

- retorno de colunas desnecessárias
- maior uso de memória
- menor escalabilidade

---

## Full Table Scan

O EXPLAIN apresentou:

```text
type = ALL
```

Indicando varredura completa da tabela.

Neste caso o comportamento é esperado, pois não há cláusula WHERE.

---

# Possíveis otimizações

## Selecionar apenas colunas necessárias

Em vez de:

```sql
select * from pessoas;
```

Utilizar:

```sql
select id, nome, email from pessoas;
```

---

## Otimização no Laravel

Em vez de:

```php
Pessoa::all();
```

Utilizar:

```php
Pessoa::select('id', 'nome', 'email')->get();
```

---

# Execução das consultas diretamente no MySQL

## Acesso ao MySQL

```bash
docker exec -it db mysql -u dev -p
```

Senha:

```text
minhasenha
```

---

## Selecionar banco

```sql
USE app_biblioteca;
```

---

# Considerações finais

A aplicação funcionou corretamente após configuração do ambiente Docker e ajustes no container PHP.

A análise inicial mostrou:

- consultas simples
- ausência aparente de N+1 queries
- possibilidade de otimização removendo SELECT *
- uso do EXPLAIN para análise de desempenho