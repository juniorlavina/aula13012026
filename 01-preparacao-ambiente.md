# 01 - Preparação do Ambiente

## 📦 Verificação de Pré-requisitos

Antes de iniciar, verifique se possui as ferramentas necessárias instaladas.

### 1. Verificar Node.js

```powershell
node --version
# Deve retornar v20.x.x ou superior
```

### 2. Verificar MySQL

```powershell
mysql --version
# Deve retornar mysql Ver 8.x.x ou superior
```

### 3. Verificar Git

```powershell
git --version
# Deve retornar git version 2.x.x ou superior
```

## 🗄️ Configuração do Banco de Dados

### Passo 1: Criar o Banco de Dados

Abra o MySQL e crie o banco de dados para o projeto:

```powershell
mysql -u root -p
```

No console do MySQL, execute:

```sql
CREATE DATABASE biblioteca_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
EXIT;
```

**Alternativa via linha de comando:**

```powershell
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS biblioteca_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

### Passo 2: Verificar Criação

```powershell
mysql -u root -p -e "SHOW DATABASES;"
```

Você deve ver `biblioteca_db` na lista.

## 📁 Estrutura Inicial do Projeto

O projeto AdonisJS já deve estar criado. Verifique a estrutura:

```
aula-13012026/
├── app/
│   ├── controllers/
│   ├── exceptions/
│   ├── middleware/
│   └── models/
├── bin/
├── config/
├── database/
│   └── migrations/
├── start/
├── tests/
├── .env.example
├── adonisrc.ts
├── package.json
└── tsconfig.json
```

## 🔐 Configuração do Arquivo .env

### Passo 1: Criar o Arquivo .env

Crie o arquivo `.env` na raiz do projeto com o seguinte conteúdo:

```env
TZ=UTC
PORT=3333
HOST=localhost
LOG_LEVEL=info
APP_KEY=gFQNzjp5vMZP7Hx_8oqMY_WRfqJKcOVu
NODE_ENV=development

DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASSWORD=
DB_DATABASE=biblioteca_db
```

### Passo 2: Configurar a Senha do MySQL

Edite a linha `DB_PASSWORD=` e adicione sua senha do MySQL:

```env
DB_PASSWORD=sua_senha_aqui
```

### Passo 3: Validar Configuração

Execute o comando para listar os comandos disponíveis:

```powershell
node ace list
```

Se não houver erros de conexão, a configuração está correta!

## 🔧 Inicialização do Git

### Passo 1: Verificar Repositório Git

Se ainda não houver um repositório Git, inicialize:

```powershell
git init
```

### Passo 2: Criar Commit Inicial

```powershell
git add .
git commit -m "feat: projeto inicial AdonisJS v6"
```

## ✅ Checklist de Preparação

Antes de prosseguir, confirme:

- [ ] Node.js instalado e funcionando
- [ ] MySQL instalado e rodando
- [ ] Banco de dados `biblioteca_db` criado
- [ ] Arquivo `.env` configurado com credenciais corretas
- [ ] Comando `node ace list` executa sem erros
- [ ] Repositório Git inicializado
- [ ] Commit inicial realizado

## 🔍 Resolução de Problemas Comuns

### Erro: "Access denied for user"

**Problema:** Credenciais do MySQL incorretas.

**Solução:** Verifique `DB_USER` e `DB_PASSWORD` no arquivo `.env`.

### Erro: "Unknown database"

**Problema:** Banco de dados não foi criado.

**Solução:** Execute novamente o comando de criação do banco.

### Erro: "ECONNREFUSED"

**Problema:** MySQL não está rodando.

**Solução:** Inicie o serviço MySQL:

```powershell
# Windows (com MySQL como serviço)
net start MySQL80

# Ou inicie pelo MySQL Workbench/XAMPP/WAMP
```

## 🎯 Próximo Passo

Ambiente configurado com sucesso! Agora vamos criar a estrutura do banco de dados.

➡️ **[02-criacao-migrations.md](02-criacao-migrations.md)**

---

**Tempo estimado:** 10-15 minutos  
**Dificuldade:** ⭐ Básico
