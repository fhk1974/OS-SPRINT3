# Sprint 3 - Operating Systems

Implementação da Sprint 3 da disciplina de Operating Systems – Challenge XP 2025 (FIAP).

## Estrutura do Projeto
- **NGINX**: servidor web rodando no Ubuntu/WSL2.  
- **Docker + SQLite**: banco de dados em contêiner com volume externo.  
- **API Node.js**: endpoints para inserir e listar usuários no SQLite.  
- **Shell Script + Cronjob**: anonimização automática de dados sensíveis (nome, CPF).  

## Pré-requisitos
- Ubuntu 22.04 (WSL2 ou VM)  
- Docker instalado  
- NGINX configurado  

## Passos de Configuração

### 1. Banco SQLite em contêiner
```bash
docker run -d --name xp-sqlite \
  -v /opt/xp-sprint3/sqlite_data:/data \
  --restart unless-stopped \
  nouchka/sqlite3

### 2. Criar tabela usuarios
docker exec -i xp-sqlite sqlite3 /data/app.db <<'SQL'
CREATE TABLE IF NOT EXISTS usuarios (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  nome TEXT,
  cpf  TEXT
);
SQL

### 3. API Node.js
cd api
docker build -t xp-node-api .
docker run -d --name xp-node-api \
  -p 3000:3000 \
  -v /opt/xp-sprint3/sqlite_data:/data \
  xp-node-api

### 4. Testes da API
# Inserir usuário
curl -X POST http://localhost:3000/usuarios \
  -H "Content-Type: application/json" \
  -d '{"nome":"Fabio","cpf":"12345678900"}'

# Listar usuários
curl http://localhost:3000/usuarios

### 5. Cronjob + Script
chmod +x /opt/xp-sprint3/scripts/anonymize.sh
crontab -e
# adicionar a linha abaixo para rodar todo dia às 23h
0 23 * * * /opt/xp-sprint3/scripts/anonymize.sh

