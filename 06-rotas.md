# 06 - Configuração de Rotas

## 📚 O que são Rotas?

Rotas mapeiam URLs para controllers/métodos. No AdonisJS:

- ✅ Definem endpoints da API
- ✅ Aplicam middlewares (autenticação, validação)
- ✅ Agrupam rotas relacionadas
- ✅ Definem métodos HTTP (GET, POST, PUT, DELETE)

## 🏗️ Estrutura de Rotas

**Arquivo:** `start/routes.ts`

```typescript
/**
 * Rotas da API de Biblioteca
 * 
 * @description Define rotas públicas e protegidas da aplicação
 * @author Sistema
 * @since 2025-01-13
 */
import router from '@adonisjs/core/services/router'
import { middleware } from '#start/kernel'

/**
 * Rota raiz
 */
router.get('/', async () => {
  return {
    message: 'API de Biblioteca',
    version: '1.0.0',
    endpoints: {
      auth: '/api/auth',
      recursos: '/api',
    },
  }
})

/**
 * Rotas públicas de autenticação
 */
router
  .group(() => {
    router.post('/register', '#controllers/auth_controller.register')
    router.post('/login', '#controllers/auth_controller.login')
  })
  .prefix('/api/auth')

/**
 * Rotas protegidas (requerem autenticação)
 */
router
  .group(() => {
    // Perfil do usuário
    router.get('/me', '#controllers/auth_controller.me')
    router.post('/logout', '#controllers/auth_controller.logout')

    // Livros
    router.get('/livros', '#controllers/livros_controller.index')
    router.post('/livros', '#controllers/livros_controller.store')
    router.get('/livros/:isbn', '#controllers/livros_controller.show')
    router.put('/livros/:isbn', '#controllers/livros_controller.update')
    router.delete('/livros/:isbn', '#controllers/livros_controller.destroy')

    // Autores
    router.get('/autores', '#controllers/autores_controller.index')
    router.post('/autores', '#controllers/autores_controller.store')
    router.get('/autores/:id', '#controllers/autores_controller.show')
    router.put('/autores/:id', '#controllers/autores_controller.update')
    router.delete('/autores/:id', '#controllers/autores_controller.destroy')

    // Exemplares
    router.get('/exemplares', '#controllers/exemplares_controller.index')
    router.post('/exemplares', '#controllers/exemplares_controller.store')
    router.get('/exemplares/:codigo', '#controllers/exemplares_controller.show')
    router.put('/exemplares/:codigo', '#controllers/exemplares_controller.update')
    router.delete('/exemplares/:codigo', '#controllers/exemplares_controller.destroy')

    // Reservas
    router.get('/reservas', '#controllers/reservas_controller.index')
    router.post('/reservas', '#controllers/reservas_controller.store')
    router.get('/reservas/minhas', '#controllers/reservas_controller.minhasReservas')
    router.get('/reservas/:id', '#controllers/reservas_controller.show')
    router.put('/reservas/:id', '#controllers/reservas_controller.update')
    router.delete('/reservas/:id', '#controllers/reservas_controller.destroy')

    // Empréstimos
    router.get('/emprestimos', '#controllers/emprestimos_controller.index')
    router.post('/emprestimos', '#controllers/emprestimos_controller.store')
    router.get('/emprestimos/meus', '#controllers/emprestimos_controller.meusEmprestimos')
    router.get('/emprestimos/:id', '#controllers/emprestimos_controller.show')
    router.put('/emprestimos/:id', '#controllers/emprestimos_controller.update')
    router.delete('/emprestimos/:id', '#controllers/emprestimos_controller.destroy')
    router.post(
      '/emprestimos/itens/:itemId/devolver',
      '#controllers/emprestimos_controller.devolverItem'
    )
  })
  .prefix('/api')
  .use(middleware.auth())
```

## 🔍 Entendendo os Componentes

### 1. Importações

```typescript
import router from '@adonisjs/core/services/router'
import { middleware } from '#start/kernel'
```

- `router`: Objeto para definir rotas
- `middleware`: Named middlewares (auth, cors, etc)

### 2. Rota Raiz (/)

```typescript
router.get('/', async () => {
  return {
    message: 'API de Biblioteca',
    version: '1.0.0',
    endpoints: {
      auth: '/api/auth',
      recursos: '/api',
    },
  }
})
```

**Propósito:** Documentar endpoints disponíveis

**Teste:**
```bash
GET http://localhost:3333/
```

### 3. Grupo de Rotas Públicas

```typescript
router
  .group(() => {
    router.post('/register', '#controllers/auth_controller.register')
    router.post('/login', '#controllers/auth_controller.login')
  })
  .prefix('/api/auth')
```

**Características:**
- `.group()`: Agrupa rotas relacionadas
- `.prefix()`: Adiciona prefixo a todas as rotas do grupo
- Sem `.use(middleware.auth())`: Rotas públicas

**Rotas resultantes:**
- `POST /api/auth/register`
- `POST /api/auth/login`

### 4. Grupo de Rotas Protegidas

```typescript
router
  .group(() => {
    router.get('/me', '#controllers/auth_controller.me')
    router.get('/livros', '#controllers/livros_controller.index')
    // ... outras rotas
  })
  .prefix('/api')
  .use(middleware.auth()) // ← Middleware de autenticação
```

**Características:**
- `.use(middleware.auth())`: Aplica middleware a todas as rotas
- Requer header `Authorization: Bearer <token>`

### 5. Sintaxe de Controller

```typescript
router.get('/livros', '#controllers/livros_controller.index')
```

**Formato:** `#controllers/pasta/arquivo.metodo`

**Alternativa (import explícito):**
```typescript
import LivrosController from '#controllers/livros_controller'
router.get('/livros', [LivrosController, 'index'])
```

### 6. Parâmetros de Rota

```typescript
router.get('/livros/:isbn', '#controllers/livros_controller.show')
```

**`:isbn`** - Parâmetro dinâmico

**No controller:**
```typescript
async show({ params }: HttpContext) {
  const isbn = params.isbn // Acessa parâmetro
}
```

### 7. Rotas Customizadas

```typescript
router.get('/reservas/minhas', '#controllers/reservas_controller.minhasReservas')
router.post('/emprestimos/itens/:itemId/devolver', '#controllers/emprestimos_controller.devolverItem')
```

**Observações:**
- Rotas customizadas devem vir **antes** das rotas com parâmetros
- Evita conflito: `/reservas/minhas` vs `/reservas/:id`

## 📊 Mapa Completo de Rotas

### Autenticação (Públicas)

| Método | Rota | Controller | Descrição |
|--------|------|------------|-----------|
| POST | /api/auth/register | register() | Criar usuário |
| POST | /api/auth/login | login() | Autenticar |

### Perfil (Protegidas)

| Método | Rota | Controller | Descrição |
|--------|------|------------|-----------|
| GET | /api/me | me() | Dados do usuário |
| POST | /api/logout | logout() | Desautenticar |

### Livros (Protegidas)

| Método | Rota | Controller | Descrição |
|--------|------|------------|-----------|
| GET | /api/livros | index() | Listar livros |
| POST | /api/livros | store() | Criar livro |
| GET | /api/livros/:isbn | show() | Exibir livro |
| PUT | /api/livros/:isbn | update() | Atualizar livro |
| DELETE | /api/livros/:isbn | destroy() | Deletar livro |

### Autores (Protegidas)

| Método | Rota | Controller | Descrição |
|--------|------|------------|-----------|
| GET | /api/autores | index() | Listar autores |
| POST | /api/autores | store() | Criar autor |
| GET | /api/autores/:id | show() | Exibir autor |
| PUT | /api/autores/:id | update() | Atualizar autor |
| DELETE | /api/autores/:id | destroy() | Deletar autor |

### Exemplares (Protegidas)

| Método | Rota | Controller | Descrição |
|--------|------|------------|-----------|
| GET | /api/exemplares | index() | Listar exemplares |
| POST | /api/exemplares | store() | Criar exemplar |
| GET | /api/exemplares/:codigo | show() | Exibir exemplar |
| PUT | /api/exemplares/:codigo | update() | Atualizar exemplar |
| DELETE | /api/exemplares/:codigo | destroy() | Deletar exemplar |

### Reservas (Protegidas)

| Método | Rota | Controller | Descrição |
|--------|------|------------|-----------|
| GET | /api/reservas | index() | Listar reservas |
| POST | /api/reservas | store() | Criar reserva |
| GET | /api/reservas/minhas | minhasReservas() | Minhas reservas |
| GET | /api/reservas/:id | show() | Exibir reserva |
| PUT | /api/reservas/:id | update() | Atualizar reserva |
| DELETE | /api/reservas/:id | destroy() | Deletar reserva |

### Empréstimos (Protegidas)

| Método | Rota | Controller | Descrição |
|--------|------|------------|-----------|
| GET | /api/emprestimos | index() | Listar empréstimos |
| POST | /api/emprestimos | store() | Criar empréstimo |
| GET | /api/emprestimos/meus | meusEmprestimos() | Meus empréstimos |
| GET | /api/emprestimos/:id | show() | Exibir empréstimo |
| PUT | /api/emprestimos/:id | update() | Atualizar empréstimo |
| DELETE | /api/emprestimos/:id | destroy() | Deletar empréstimo |
| POST | /api/emprestimos/itens/:itemId/devolver | devolverItem() | Devolver item |

## 🔧 Comandos Úteis

### Listar Todas as Rotas

```powershell
node ace list:routes
```

**Saída esperada:**
```
GET    /                                        closure
POST   /api/auth/register                       auth_controller.register
POST   /api/auth/login                          auth_controller.login
GET    /api/me                                  auth_controller.me        [auth]
POST   /api/logout                              auth_controller.logout    [auth]
GET    /api/livros                              livros_controller.index   [auth]
...
```

**Legenda:**
- `[auth]` indica rotas protegidas

### Testar Rota Específica

```powershell
# Rota raiz
curl http://localhost:3333/

# Login
curl -X POST http://localhost:3333/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"joao@example.com","password":"senha123"}'
```

## ✅ Validando as Rotas

Execute o comando:

```powershell
node ace list:routes
```

Verifique se:
- Rotas públicas (`/api/auth/*`) não têm `[auth]`
- Rotas protegidas (`/api/*`) têm `[auth]`
- Todas as rotas estão mapeadas

## ⚠️ Ordem das Rotas

**❌ Errado:**
```typescript
router.get('/reservas/:id', ...) // Mais genérica primeiro
router.get('/reservas/minhas', ...) // Nunca será alcançada!
```

**✅ Correto:**
```typescript
router.get('/reservas/minhas', ...) // Específica primeiro
router.get('/reservas/:id', ...) // Genérica depois
```

## ✅ Checklist

Antes de prosseguir, confirme:

- [ ] Arquivo `start/routes.ts` atualizado
- [ ] Rotas públicas sem middleware auth
- [ ] Rotas protegidas com `.use(middleware.auth())`
- [ ] Comando `node ace list:routes` lista todas as rotas
- [ ] Rotas customizadas antes das rotas com parâmetros

## 🎯 Próximo Passo

Rotas configuradas! Agora vamos testar e validar a API.

➡️ **[07-validacao-testes.md](07-validacao-testes.md)**

---

**Tempo estimado:** 20-30 minutos  
**Dificuldade:** ⭐⭐ Intermediário
