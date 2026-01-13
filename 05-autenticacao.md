# 05 - Implementação de Autenticação

## 📚 O que é Autenticação?

A autenticação é o processo de verificar a identidade de um usuário. No AdonisJS v6, utilizamos **Access Tokens** (similares a JWT) para:

- ✅ Registrar novos usuários
- ✅ Autenticar usuários existentes
- ✅ Proteger rotas privadas
- ✅ Gerenciar sessões via tokens

## 🔐 Fluxo de Autenticação

```
1. Registro (POST /api/auth/register)
   ↓
   Criar usuário + Gerar token
   ↓
   Retornar usuário e token

2. Login (POST /api/auth/login)
   ↓
   Verificar email + senha
   ↓
   Gerar token
   ↓
   Retornar usuário e token

3. Requisição Autenticada
   ↓
   Enviar token no header: Authorization: Bearer <token>
   ↓
   Middleware valida token
   ↓
   Permite acesso ao endpoint

4. Logout (POST /api/auth/logout)
   ↓
   Revogar token atual
   ↓
   Retornar mensagem de sucesso
```

## 📝 Controller de Autenticação

**Arquivo:** `app/controllers/auth_controller.ts`

```typescript
/**
 * Controller para Autenticação
 * 
 * @description Gerencia registro, login, logout e perfil de usuários
 * @author Sistema
 * @since 2025-01-13
 */
import type { HttpContext } from '@adonisjs/core/http'
import User from '#models/user'
import { DateTime } from 'luxon'

export default class AuthController {
  /**
   * Registra um novo usuário
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Usuário criado e token de acesso
   */
  async register({ request, response }: HttpContext) {
    const data = request.only(['matricula', 'nome', 'email', 'telefone', 'tipo', 'password'])

    // Define data de cadastro como hoje
    const user = await User.create({
      ...data,
      dataCadastro: DateTime.now(),
    })

    // Gera token de acesso
    const token = await User.accessTokens.create(user)

    return response.created({
      user,
      token: token.value!.release(),
    })
  }

  /**
   * Autentica um usuário
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Usuário autenticado e token de acesso
   */
  async login({ request, response }: HttpContext) {
    const { email, password } = request.only(['email', 'password'])

    // Verifica credenciais
    const user = await User.verifyCredentials(email, password)

    // Gera token de acesso
    const token = await User.accessTokens.create(user)

    return response.ok({
      user,
      token: token.value!.release(),
    })
  }

  /**
   * Retorna dados do usuário autenticado
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Dados do usuário
   */
  async me({ auth, response }: HttpContext) {
    const user = auth.user!
    await user.load('reservas', (query) => {
      query.preload('livro').where('status', 'ativa')
    })
    await user.load('emprestimos', (query) => {
      query
        .preload('itens', (subQuery) => {
          subQuery.preload('exemplar', (itemQuery) => {
            itemQuery.preload('livro')
          })
        })
        .where('status', 'ativo')
    })

    return response.ok(user)
  }

  /**
   * Desautentica o usuário (revoga token)
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Mensagem de sucesso
   */
  async logout({ auth, response }: HttpContext) {
    const user = auth.user!
    await User.accessTokens.delete(user, user.currentAccessToken.identifier)

    return response.ok({ message: 'Logout realizado com sucesso' })
  }
}
```

## 🔑 Entendendo os Métodos

### 1. register() - Registro de Usuário

```typescript
async register({ request, response }: HttpContext) {
  // 1. Extrair dados do body
  const data = request.only(['matricula', 'nome', 'email', 'telefone', 'tipo', 'password'])

  // 2. Criar usuário (senha é hasheada automaticamente pelo AuthFinder)
  const user = await User.create({
    ...data,
    dataCadastro: DateTime.now(),
  })

  // 3. Gerar token de acesso
  const token = await User.accessTokens.create(user)

  // 4. Retornar usuário e token
  return response.created({
    user,
    token: token.value!.release(),
  })
}
```

**Observações:**
- `User.accessTokens.create()` gera um token único
- `token.value!.release()` extrai o valor do token para enviar ao client
- Senha é hasheada automaticamente pelo mixin `withAuthFinder`

### 2. login() - Autenticação de Usuário

```typescript
async login({ request, response }: HttpContext) {
  const { email, password } = request.only(['email', 'password'])

  // Verifica se email e senha estão corretos
  const user = await User.verifyCredentials(email, password)

  // Gera novo token
  const token = await User.accessTokens.create(user)

  return response.ok({
    user,
    token: token.value!.release(),
  })
}
```

**Observações:**
- `User.verifyCredentials()` é fornecido pelo mixin `withAuthFinder`
- Lança exceção se credenciais inválidas (status 400)

### 3. me() - Perfil do Usuário

```typescript
async me({ auth, response }: HttpContext) {
  const user = auth.user! // Usuário autenticado

  // Carregar relacionamentos
  await user.load('reservas', (query) => {
    query.preload('livro').where('status', 'ativa')
  })
  await user.load('emprestimos', (query) => {
    query.preload('itens').where('status', 'ativo')
  })

  return response.ok(user)
}
```

**Observações:**
- `auth.user!` acessa o usuário autenticado (disponível após middleware auth)
- Carrega apenas reservas/empréstimos ativos

### 4. logout() - Desautenticação

```typescript
async logout({ auth, response }: HttpContext) {
  const user = auth.user!

  // Revoga o token atual
  await User.accessTokens.delete(user, user.currentAccessToken.identifier)

  return response.ok({ message: 'Logout realizado com sucesso' })
}
```

**Observações:**
- `user.currentAccessToken.identifier` identifica o token atual
- Token é removido da tabela `access_tokens`

## 🔐 Como o Hash de Senha Funciona

O mixin `withAuthFinder` no model User:

```typescript
const AuthFinder = withAuthFinder(() => hash.use('scrypt'), {
  uids: ['email'],
  passwordColumnName: 'password',
})
```

Fornece automaticamente:

1. **Hash na criação/atualização:**
   ```typescript
   const user = await User.create({ password: '123456' })
   // Senha é hasheada automaticamente antes de salvar
   ```

2. **Verificação de credenciais:**
   ```typescript
   const user = await User.verifyCredentials('email@example.com', '123456')
   // Compara hash armazenado com senha fornecida
   ```

## 🛡️ Middleware de Autenticação

O middleware `auth` já está configurado em `start/kernel.ts`:

```typescript
export const middleware = router.named({
  auth: () => import('#middleware/auth_middleware'),
})
```

**Como funciona:**

1. Client envia requisição com header:
   ```
   Authorization: Bearer <token>
   ```

2. Middleware extrai token e valida:
   ```typescript
   await auth.authenticateUsing(['api'])
   ```

3. Se válido, usuário fica disponível em `auth.user`
4. Se inválido, retorna 401 Unauthorized

## ✅ Validando a Autenticação

Execute o comando para verificar erros:

```powershell
node ace list
```

## 📊 Endpoints de Autenticação

| Método | Rota | Controller | Autenticação | Descrição |
|--------|------|------------|--------------|-----------|
| POST | /api/auth/register | register() | ❌ Pública | Criar usuário |
| POST | /api/auth/login | login() | ❌ Pública | Autenticar |
| GET | /api/me | me() | ✅ Privada | Perfil do usuário |
| POST | /api/logout | logout() | ✅ Privada | Desautenticar |

## 🧪 Testando Autenticação (Manual)

### 1. Registrar Usuário

```bash
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

**Resposta esperada:**
```json
{
  "user": {
    "id": 1,
    "matricula": "2024001",
    "nome": "João Silva",
    "email": "joao@example.com",
    ...
  },
  "token": "oat_xxx.yyy.zzz"
}
```

### 2. Login

```bash
POST http://localhost:3333/api/auth/login
Content-Type: application/json

{
  "email": "joao@example.com",
  "password": "senha123"
}
```

### 3. Acessar Perfil (Com Token)

```bash
GET http://localhost:3333/api/me
Authorization: Bearer oat_xxx.yyy.zzz
```

### 4. Logout

```bash
POST http://localhost:3333/api/logout
Authorization: Bearer oat_xxx.yyy.zzz
```

## ⚠️ Erros Comuns

### Erro 401: "E_UNAUTHORIZED_ACCESS"

**Causa:** Token inválido ou ausente

**Solução:** Enviar token correto no header Authorization

### Erro 400: "Invalid credentials"

**Causa:** Email ou senha incorretos

**Solução:** Verificar credenciais

### Erro 422: "Validation failed"

**Causa:** Dados obrigatórios ausentes

**Solução:** Verificar campos required (matricula, email, password, etc)

## ✅ Checklist

Antes de prosseguir, confirme:

- [ ] AuthController criado
- [ ] 4 métodos implementados (register, login, me, logout)
- [ ] Comando `node ace list` executa sem erros
- [ ] Entendimento do fluxo de tokens

## 🎯 Próximo Passo

Autenticação pronta! Agora vamos configurar as rotas.

➡️ **[06-rotas.md](06-rotas.md)**

---

**Tempo estimado:** 30-40 minutos  
**Dificuldade:** ⭐⭐ Intermediário
