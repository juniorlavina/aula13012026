# 07 - Validação e Testes da API

## 📚 Validação da Implementação

Antes de testar endpoints, precisamos validar que tudo foi implementado corretamente.

## ✅ Passo 1: Verificar Erros TypeScript/ESLint

Execute o comando para listar erros:

```powershell
node ace list
```

**Se houver erros:** Corrija antes de prosseguir.

**Saída esperada (sem erros):**
```
Available commands:
  add                 Install and configure a package
  build               Build application for production
  ...
```

## ✅ Passo 2: Iniciar Servidor

```powershell
node ace serve --hmr
```

**Saída esperada:**
```
[ info ] starting HTTP server...
[ info ] watching file system for changes...
╭─────────────────────────────────────────────────╮
│                                                 │
│    Server address: http://localhost:3333        │
│    Watch Mode: HMR                           │
│    Ready in: 1.2 s                              │
│                                                 │
╰─────────────────────────────────────────────────╯
```

## ✅ Passo 3: Verificar Migrations

```powershell
node ace migration:status
```

**Saída esperada:**
```
❯ completed database/migrations/1768302505585_create_users_table
❯ completed database/migrations/1768302505586_create_livros_table
❯ completed database/migrations/1768302505587_create_autores_table
...
```

Todas devem estar `completed`.

## 🧪 Testes Manuais com Postman/Insomnia

### Configurar Ambiente

**Base URL:** `http://localhost:3333`

### Teste 1: Rota Raiz (GET /)

**Request:**
```
GET http://localhost:3333/
```

**Response esperada (200 OK):**
```json
{
  "message": "API de Biblioteca",
  "version": "1.0.0",
  "endpoints": {
    "auth": "/api/auth",
    "recursos": "/api"
  }
}
```

### Teste 2: Registrar Usuário (POST /api/auth/register)

**Request:**
```
POST http://localhost:3333/api/auth/register
Content-Type: application/json

{
  "matricula": "2024001",
  "nome": "João Silva",
  "email": "joao@example.com",
  "telefone": "(11) 98765-4321",
  "tipo": "aluno",
  "password": "senha123"
}
```

**Response esperada (201 Created):**
```json
{
  "user": {
    "id": 1,
    "matricula": "2024001",
    "nome": "João Silva",
    "email": "joao@example.com",
    "telefone": "(11) 98765-4321",
    "tipo": "aluno",
    "dataCadastro": "2025-01-13",
    "createdAt": "2025-01-13T10:30:00.000-03:00",
    "updatedAt": "2025-01-13T10:30:00.000-03:00"
  },
  "token": "oat_MQ.xxx...yyy"
}
```

**⚠️ Anotar o token retornado!**

### Teste 3: Login (POST /api/auth/login)

**Request:**
```
POST http://localhost:3333/api/auth/login
Content-Type: application/json

{
  "email": "joao@example.com",
  "password": "senha123"
}
```

**Response esperada (200 OK):**
```json
{
  "user": {
    "id": 1,
    "matricula": "2024001",
    ...
  },
  "token": "oat_MQ.xxx...yyy"
}
```

### Teste 4: Acessar Perfil (GET /api/me) - Autenticado

**Request:**
```
GET http://localhost:3333/api/me
Authorization: Bearer oat_MQ.xxx...yyy
```

**Response esperada (200 OK):**
```json
{
  "id": 1,
  "matricula": "2024001",
  "nome": "João Silva",
  "email": "joao@example.com",
  ...
  "reservas": [],
  "emprestimos": []
}
```

### Teste 5: Criar Autor (POST /api/autores) - Autenticado

**Request:**
```
POST http://localhost:3333/api/autores
Authorization: Bearer oat_MQ.xxx...yyy
Content-Type: application/json

{
  "nome": "Machado de Assis",
  "dataNascimento": "1839-06-21"
}
```

**Response esperada (201 Created):**
```json
{
  "id": 1,
  "nome": "Machado de Assis",
  "dataNascimento": "1839-06-21",
  "createdAt": "2025-01-13T10:35:00.000-03:00",
  "updatedAt": "2025-01-13T10:35:00.000-03:00"
}
```

### Teste 6: Criar Livro (POST /api/livros) - Autenticado

**Request:**
```
POST http://localhost:3333/api/livros
Authorization: Bearer oat_MQ.xxx...yyy
Content-Type: application/json

{
  "isbn": "978-8535908770",
  "titulo": "Dom Casmurro",
  "anoPublicacao": 1899,
  "edicao": 1,
  "editora": "Companhia das Letras",
  "categoria": "Romance",
  "autores": [1]
}
```

**Response esperada (201 Created):**
```json
{
  "isbn": "978-8535908770",
  "titulo": "Dom Casmurro",
  "anoPublicacao": 1899,
  ...
  "autores": [
    {
      "id": 1,
      "nome": "Machado de Assis",
      ...
    }
  ]
}
```

### Teste 7: Listar Livros (GET /api/livros) - Autenticado

**Request:**
```
GET http://localhost:3333/api/livros
Authorization: Bearer oat_MQ.xxx...yyy
```

**Response esperada (200 OK):**
```json
[
  {
    "isbn": "978-8535908770",
    "titulo": "Dom Casmurro",
    ...
    "autores": [
      {
        "id": 1,
        "nome": "Machado de Assis"
      }
    ],
    "exemplares": []
  }
]
```

### Teste 8: Criar Exemplar (POST /api/exemplares) - Autenticado

**Request:**
```
POST http://localhost:3333/api/exemplares
Authorization: Bearer oat_MQ.xxx...yyy
Content-Type: application/json

{
  "codigo": "EX-001",
  "isbnLivro": "978-8535908770",
  "localizacao": "Estante A, Prateleira 3",
  "status": "disponivel"
}
```

**Response esperada (201 Created):**
```json
{
  "codigo": "EX-001",
  "isbnLivro": "978-8535908770",
  "localizacao": "Estante A, Prateleira 3",
  "status": "disponivel",
  ...
  "livro": {
    "isbn": "978-8535908770",
    "titulo": "Dom Casmurro",
    ...
  }
}
```

### Teste 9: Criar Reserva (POST /api/reservas) - Autenticado

**Request:**
```
POST http://localhost:3333/api/reservas
Authorization: Bearer oat_MQ.xxx...yyy
Content-Type: application/json

{
  "isbnLivro": "978-8535908770",
  "dataExpiracao": "2025-01-20"
}
```

**Response esperada (201 Created):**
```json
{
  "id": 1,
  "matriculaUsuario": "2024001",
  "isbnLivro": "978-8535908770",
  "dataExpiracao": "2025-01-20",
  "status": "ativa",
  ...
  "livro": {
    "isbn": "978-8535908770",
    "titulo": "Dom Casmurro",
    ...
  }
}
```

### Teste 10: Criar Empréstimo (POST /api/emprestimos) - Autenticado

**Request:**
```
POST http://localhost:3333/api/emprestimos
Authorization: Bearer oat_MQ.xxx...yyy
Content-Type: application/json

{
  "dataRetirada": "2025-01-13",
  "itens": [
    {
      "codigoExemplar": "EX-001",
      "dataPrevistaDevolucao": "2025-01-27"
    }
  ]
}
```

**Response esperada (201 Created):**
```json
{
  "id": 1,
  "matriculaUsuario": "2024001",
  "dataRetirada": "2025-01-13",
  "status": "ativo",
  ...
  "itens": [
    {
      "id": 1,
      "idEmprestimo": 1,
      "codigoExemplar": "EX-001",
      "dataPrevistaDevolucao": "2025-01-27",
      "statusItem": "emprestado",
      ...
      "exemplar": {
        "codigo": "EX-001",
        "status": "emprestado",
        "livro": {
          "isbn": "978-8535908770",
          "titulo": "Dom Casmurro"
        }
      }
    }
  ]
}
```

### Teste 11: Devolver Item (POST /api/emprestimos/itens/:itemId/devolver)

**Request:**
```
POST http://localhost:3333/api/emprestimos/itens/1/devolver
Authorization: Bearer oat_MQ.xxx...yyy
```

**Response esperada (200 OK):**
```json
{
  "id": 1,
  "idEmprestimo": 1,
  "codigoExemplar": "EX-001",
  "dataPrevistaDevolucao": "2025-01-27",
  "dataDevolucao": "2025-01-13",
  "statusItem": "devolvido",
  ...
  "exemplar": {
    "codigo": "EX-001",
    "status": "disponivel",
    ...
  }
}
```

### Teste 12: Minhas Reservas (GET /api/reservas/minhas)

**Request:**
```
GET http://localhost:3333/api/reservas/minhas
Authorization: Bearer oat_MQ.xxx...yyy
```

**Response esperada (200 OK):**
```json
[
  {
    "id": 1,
    "matriculaUsuario": "2024001",
    "isbnLivro": "978-8535908770",
    "status": "ativa",
    ...
    "livro": {
      "isbn": "978-8535908770",
      "titulo": "Dom Casmurro"
    }
  }
]
```

### Teste 13: Meus Empréstimos (GET /api/emprestimos/meus)

**Request:**
```
GET http://localhost:3333/api/emprestimos/meus
Authorization: Bearer oat_MQ.xxx...yyy
```

**Response esperada (200 OK):**
```json
[
  {
    "id": 1,
    "matriculaUsuario": "2024001",
    "status": "finalizado",
    ...
    "itens": [...]
  }
]
```

### Teste 14: Logout (POST /api/logout)

**Request:**
```
POST http://localhost:3333/api/logout
Authorization: Bearer oat_MQ.xxx...yyy
```

**Response esperada (200 OK):**
```json
{
  "message": "Logout realizado com sucesso"
}
```

## 🚨 Testando Erros

### Erro 401: Rota protegida sem token

**Request:**
```
GET http://localhost:3333/api/livros
# Sem header Authorization
```

**Response esperada (401 Unauthorized):**
```json
{
  "errors": [
    {
      "message": "Unauthorized access"
    }
  ]
}
```

### Erro 400: Credenciais inválidas

**Request:**
```
POST http://localhost:3333/api/auth/login
Content-Type: application/json

{
  "email": "joao@example.com",
  "password": "senhaerrada"
}
```

**Response esperada (400 Bad Request):**
```json
{
  "errors": [
    {
      "message": "Invalid user credentials"
    }
  ]
}
```

### Erro 404: Recurso não encontrado

**Request:**
```
GET http://localhost:3333/api/livros/ISBN-INVALIDO
Authorization: Bearer oat_MQ.xxx...yyy
```

**Response esperada (404 Not Found):**
```json
{
  "errors": [
    {
      "message": "E_ROW_NOT_FOUND: Row not found"
    }
  ]
}
```

## 📊 Collection Postman/Insomnia

Crie uma collection com todas as requisições acima e configure:

### Variáveis de Ambiente

```json
{
  "baseUrl": "http://localhost:3333",
  "token": "{{token}}"
}
```

### Scripts de Pós-Requisição (Login/Register)

```javascript
// Salvar token automaticamente
const response = pm.response.json();
pm.environment.set("token", response.token);
```

## ✅ Checklist de Testes

Antes de finalizar, confirme:

- [ ] Servidor rodando sem erros
- [ ] Registro de usuário funciona
- [ ] Login retorna token
- [ ] Token é aceito em rotas protegidas
- [ ] CRUD de livros funciona
- [ ] CRUD de autores funciona
- [ ] CRUD de exemplares funciona
- [ ] Criação de reservas funciona
- [ ] Criação de empréstimos funciona
- [ ] Devolução de itens funciona
- [ ] Relacionamentos carregados corretamente (autores, exemplares, etc)
- [ ] Erro 401 em rotas protegidas sem token
- [ ] Erro 400 com credenciais inválidas
- [ ] Erro 404 com IDs inexistentes

## 🎯 Finalização

Parabéns! Você completou a criação de uma API REST completa com AdonisJS v6!

### 📝 Commit Final

```powershell
git add .
git commit -m "feat: API de biblioteca completa

- Migrations para 9 tabelas
- Models com relacionamentos 1:1, 1:N, N:N
- Controllers CRUD completos
- Autenticação via tokens
- Rotas públicas e protegidas
- Testes manuais validados"
```

### 📚 Próximos Passos (Opcional)

1. **Validação de Dados:** Implementar VineJS validators
2. **Testes Automatizados:** Criar testes com Japa
3. **Documentação:** Gerar Swagger/OpenAPI
4. **Deploy:** Publicar em produção (Heroku, Vercel, etc)
5. **Performance:** Adicionar cache com Redis
6. **Notificações:** Enviar emails de confirmação
7. **Paginação:** Implementar em endpoints `index()`
8. **Filtros:** Adicionar busca e filtros avançados

---

**Tempo total da aula:** ~4-5 horas  
**Dificuldade geral:** ⭐⭐⭐ Intermediário/Avançado  
**Data:** 13 de janeiro de 2026
