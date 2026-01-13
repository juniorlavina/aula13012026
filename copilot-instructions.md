# AdonisJS API - Copilot Instructions

## Arquitetura e Framework

Este projeto utiliza **AdonisJS v6** com TypeScript e **Lucid ORM**, focado em APIs REST/JSON.

### Estrutura de Diretórios

- `app/`: Código da aplicação (models, middleware, exceptions)
- `config/`: Arquivos de configuração (database, auth, cors, etc)
- `start/`: Inicialização (routes, kernel, env)
- `database/migrations/`: Migrações do banco de dados
- `bin/`: Entry points (server, console, test)
- `tests/`: Testes com Japa

---

## 🎯 WORKFLOW: Criação de API Completa a partir de Diagrama

Quando receber um diagrama de banco de dados ou requisitos de API, siga este processo **rigorosamente** e na ordem apresentada:

### Fase 1: Análise do Diagrama
1. Identificar todas as **entidades** (tabelas)
2. Mapear **relacionamentos** (1:1, 1:N, N:N)
3. Listar **atributos** de cada entidade
4. Identificar **chaves primárias** e **foreign keys**
5. Determinar **endpoints necessários** (CRUD + operações especiais)

### Fase 2: Implementação (Ordem Obrigatória)

**⚠️ REGRA CRÍTICA DE VALIDAÇÃO:**
Após completar CADA etapa de implementação (migrations, models, controllers, services, validators, rotas, etc), você DEVE:
1. Executar `get_errors` para verificar erros de TypeScript/ESLint
2. Corrigir TODOS os erros encontrados antes de prosseguir
3. Nunca avançar para a próxima etapa com erros pendentes

#### 2.1 Migrations (Criar TODAS antes de prosseguir)
- Criar migrations na ordem correta: **entidades independentes primeiro, depois as dependentes**
- Incluir foreign keys com `onDelete` e `onUpdate` apropriados
- Incluir índices para otimização de queries
- SEMPRE incluir `up()` e `down()` completos

**Template de Migration:**
```typescript
/**
 * Migration para criar tabela [NOME_TABELA]
 * 
 * @description [Descrição do propósito da tabela]
 * @author Sistema
 * @since [Data]
 */
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'nome_tabela'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id').primary()
      // Colunas com comentários
      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').notNullable()
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

#### 2.2 Models (Criar TODOS após migrations)
- Decoradores `@column` para todas as colunas
- Decoradores de relacionamento: `@hasOne`, `@hasMany`, `@belongsTo`, `@manyToMany`, `@hasManyThrough`
- Hooks quando necessário: `@beforeSave`, `@afterCreate`, etc.
- Usar `serializeAs: null` para campos sensíveis (senha, tokens)
- **⚠️ OBRIGATÓRIO**: Após criar TODOS os models, executar `get_errors` para verificar e corrigir erros de TypeScript/ESLint

**Template de Model:**
```typescript
/**
 * Model [NOME]
 * 
 * @description [Descrição da entidade]
 * @author Sistema
 */
import { DateTime } from 'luxon'
import { BaseModel, column, hasMany, belongsTo } from '@adonisjs/lucid/orm'
import type { HasMany, BelongsTo } from '@adonisjs/lucid/types/relations'

export default class NomeModel extends BaseModel {
  /**
   * Chave primária
   */
  @column({ isPrimary: true })
  declare id: number

  /**
   * [Descrição do campo]
   */
  @column()
  declare campo: string

  /**
   * Data de criação do registro
   */
  @column.dateTime({ autoCreate: true })
  declare createdAt: DateTime

  /**
   * Data da última atualização
   */
  @column.dateTime({ autoCreate: true, autoUpdate: true })
  declare updatedAt: DateTime

  /**
   * Relacionamento: [Descrição]
   */
  @hasMany(() => RelatedModel)
  declare related: HasMany<typeof RelatedModel>
}
```

#### 2.3 Controllers (Criar TODOS após models)
- Implementar CRUD completo: `index`, `store`, `show`, `update`, `destroy`
- Validação de dados em TODAS as operações
- Tratamento de erros apropriado
- Retornar status HTTP corretos (200, 201, 400, 404, 500)
- **⚠️ OBRIGATÓRIO**: Após criar TODOS os controllers, executar `get_errors` para verificar e corrigir erros de TypeScript/ESLint

**Template de Controller:**
```typescript
/**
 * Controller para [RECURSO]
 * 
 * @description Gerencia operações CRUD de [RECURSO]
 * @author Sistema
 */
import type { HttpContext } from '@adonisjs/core/http'
import Model from '#models/model'

export default class ModelsController {
  /**
   * Lista todos os registros
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Array>} Lista de registros
   */
  async index({ response }: HttpContext) {
    const records = await Model.all()
    return response.ok(records)
  }

  /**
   * Cria um novo registro
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Registro criado
   */
  async store({ request, response }: HttpContext) {
    // Validação aqui
    const data = request.only(['campo1', 'campo2'])
    const record = await Model.create(data)
    return response.created(record)
  }

  /**
   * Exibe um registro específico
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Registro encontrado
   */
  async show({ params, response }: HttpContext) {
    const record = await Model.findOrFail(params.id)
    return response.ok(record)
  }

  /**
   * Atualiza um registro
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Registro atualizado
   */
  async update({ params, request, response }: HttpContext) {
    const record = await Model.findOrFail(params.id)
    const data = request.only(['campo1', 'campo2'])
    await record.merge(data).save()
    return response.ok(record)
  }

  /**
   * Remove um registro
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<void>}
   */
  async destroy({ params, response }: HttpContext) {
    const record = await Model.findOrFail(params.id)
    await record.delete()
    return response.noContent()
  }
}
```

#### 2.4 Services (Quando necessário)
Criar services quando:
- Lógica de negócio complexa
- Operações envolvendo múltiplos models
- Integrações externas
- Processamento assíncrono

**Template de Service:**
```typescript
/**
 * Service para [FUNCIONALIDADE]
 * 
 * @description [Descrição do propósito]
 * @author Sistema
 */
export default class NomeService {
  /**
   * [Descrição do método]
   * 
   * @param {Type} param - Descrição do parâmetro
   * @returns {Promise<Type>} Descrição do retorno
   */
  async metodo(param: Type): Promise<Type> {
    // Lógica de negócio
    return result
  }
}
```

#### 2.5 Rotas (Configurar TODAS após controllers)
- Agrupar rotas relacionadas
- Aplicar middleware `auth` em rotas protegidas
- Usar prefixos semânticos (`/
- **⚠️ OBRIGATÓRIO**: Após configurar rotas, executar `get_errors` para verificar e corrigir erros de TypeScript/ESLintapi/v1`)
- Nomear rotas para referência

**Template de Rotas:**
```typescript
/**
 * Rotas para [RECURSO]
 */
import router from '@adonisjs/core/services/router'
import { middleware } from '#start/kernel'

// Rotas públicas
router.group(() => {
  router.post('/login', 'AuthController.login')
  router.post('/register', 'AuthController.register')
}).prefix('/api/auth')

// Rotas protegidas
router.group(() => {
  router.resource('resources', 'ResourcesController').apiOnly()
}).prefix('/api').use(middleware.auth())
```

#### 2.6 Autenticação (OBRIGATÓRIO para APIs protegidas)

Sempre implementar autenticação quando houver rotas protegidas:

**AuthController com:**
- `register()` - Criar novo usuário e retornar token
- `login()` - Autenticar e retornar token
- `logout()` - Revogar token atual
- `me()` - Retornar dados do usuário autenticado

**Rotas de autenticação:**
```typescript
// Rotas públicas
router.group(() => {
  router.post('/register', '#controllers/auth_controller.register')
  router.post('/login', '#controllers/auth_controller.login')
}).prefix('/api/auth')

// Rotas protegidas
router.group(() => {
  router.get('/me', '#controllers/auth_controller.me')
  router.post('/logout', '#controllers/auth_controller.logout')
  // ... outras rotas protegidas
}).prefix('/api').use(middleware.auth())
```

**Hook de senha no User model:**
```typescript
@beforeSave()
static async hashPassword(user: User) {
  if (user.$dirty.password) {
    user.password = await hash.make(user.password)
  }
}
```

### Fase 3: Testes (OBRIGATÓRIO)
Criar testes para:
- ✅ Cada endpoint CRUD
- ✅ Autenticação (login, register, logout)
- ✅ Validações de dados
- ✅ Relacionamentos entre models

**Template de Teste:**
```typescript
/**
 * Testes para [RECURSO]
 * 
 * @description Testa operações CRUD de [RECURSO]
 */
import { test } from '@japa/runner'

test.group('Resources', () => {
  test('deve listar todos os recursos', async ({ client }) => {
    const response = await client.get('/api/resources')
    response.assertStatus(200)
    response.assertBodyContains([])
  })

  test('deve criar um novo recurso', async ({ client }) => {
    const response = await client.post('/api/resources').json({
      campo: 'valor'
    })
    response.assertStatus(201)
  })
})
```

### Fase 4: Execução e Validação

#### 4.1 Controle de Versão Git (OBRIGATÓRIO)

**SEMPRE criar repositório Git antes de iniciar o desenvolvimento:**

1. **Inicializar repositório** (se ainda não existir):
   ```bash
   git init
   git add .
   git commit -m "feat: projeto inicial AdonisJS v6"
   ```

2. **Solicitar commits a cada funcionalidade implementada:**
   - Após criar migrations: `git add . && git commit -m "feat: adicionar migrations [RECURSO]"`
   - Após criar models: `git add . && git commit -m "feat: adicionar models [RECURSO]"`
   - Após criar controllers: `git add . && git commit -m "feat: adicionar controllers [RECURSO]"`
   - Após implementar autenticação: `git add . && git commit -m "feat: implementar autenticação"`
   - Após criar testes: `git add . && git commit -m "test: adicionar testes [RECURSO]"`
   - Após correções: `git add . && git commit -m "fix: corrigir [PROBLEMA]"`

3. **Padrão de mensagens de commit (Conventional Commits):**
   - `feat:` - Nova funcionalidade
   - `fix:` - Correção de bug
   - `test:` - Adição ou modificação de testes
   - `docs:` - Documentação
   - `refactor:` - Refatoração de código
   - `chore:` - Tarefas de manutenção

**⚠️ REGRA CRÍTICA:**
- NUNCA executar commits automaticamente
- SEMPRE solicitar ao usuário para executar o commit manualmente
- Sugerir a mensagem de commit apropriada
- Confirmar que o commit foi realizado antes de prosseguir para próxima funcionalidade

#### 4.2 Configuração do Banco de Dados (OBRIGATÓRIO ANTES DE MIGRATIONS)

**SEMPRE solicitar ao usuário:**
1. Criar banco de dados MySQL com nome apropriado
2. Criar arquivo `.env` baseado em `.env.example` (se existir)
3. Configurar variáveis de ambiente:
   ```env
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_USER=root
   DB_PASSWORD=sua_senha
   DB_DATABASE=nome_do_banco
   ```

**Se o usuário tiver dificuldades**, oferecer script para criar banco automaticamente:

**PowerShell (Windows):**
```powershell
# Criar banco via MySQL CLI
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS nome_do_banco CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

**Bash (Linux/Mac):**
```bash
# Criar banco via MySQL CLI
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS nome_do_banco CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

**Validar conexão antes de prosseguir:**
```bash
node ace list
```

#### 4.3 Executar Migrations e Testes
1. **Executar migrations**: `node ace migration:run`
2. **Compilar código**: Verificar erros TypeScript
3. **Executar testes**: `node ace test`
4. **Iniciar servidor**: `node ace serve --watch`
5. **Validar endpoints** com ferramenta REST (Insomnia/Postman)

**Após cada etapa de validação bem-sucedida, solicitar commit:**
```bashRepositório Git inicializado**
- [ ] **Banco de dados criado e configurado no .env**
- [ ] **Conexão com banco validada**
- [ ] Todas as migrations criadas e executadas
- [ ] Todos os models com relacionamentos configurados
- [ ] Todos os controllers com CRUD completo
- [ ] Services criados quando necessário
- [ ] Rotas configuradas e agrupadas
- [ ] Autenticação implementada (se necessário)
- [ ] Todos os testes criados e passando
- [ ] Código compila sem erros
- [ ] Documentação Doxygen completa
- [ ] Servidor funcionando e endpoints testados
- [ ] **Commits realizados para cada funcionalidade implementada**
- [ ] Rotas configuradas e agrupadas
- [ ] Autenticação implementada (se necessário)
- [ ] Todos os testes criados e passando
- [ ] Código compila sem erros
- [ ] Documentação Doxygen completa
- [ ] Servidor funcionando e endpoints testados

---

---

## Convenções Críticas

### 1. Imports com Path Aliases

Use `#` para importar módulos da aplicação:

```typescript
import User from '#models/user'
import AuthMiddleware from '#middleware/auth_middleware'
import '#start/env'
```

### 2. Autenticação via Access Tokens

- Guard padrão: `api` (configurado em [config/auth.ts](config/auth.ts))
- Provider: `DbAccessTokensProvider` com modelo `User`
- Uso: `ctx.auth.authenticateUsing()` no middleware
- Modelo User tem `User.accessTokens` para gerenciar tokens

### 3. Middleware Stack

Em [start/kernel.ts](start/kernel.ts):

**Server middleware** (todas as requisições):
1. `container_bindings_middleware` - Injeta HttpContext e Logger no container
2. `force_json_response_middleware` - Força respostas JSON (altera header Accept)
3. `@adonisjs/cors/cors_middleware`

**Router middleware**:
1. `@adonisjs/core/bodyparser_middleware`
2. `@adonisjs/auth/initialize_auth_middleware`

**Named middleware**:
- `auth` - Aplique em rotas protegidas: `router.get('/profile', handler).use(middleware.auth())`

### 4. Models com Lucid ORM

**Estrutura Básica:**
- Herdam de `BaseModel` com mixins via `compose()`
- User usa `withAuthFinder` para autenticação
- Timestamps: `@column.dateTime({ autoCreate: true, autoUpdate: true })`
- Senha: use `{ serializeAs: null }` para ocultar em JSON
- Acesso ao hash: `hash.use('scrypt')`

**Relacionamentos:**
- `@hasOne(() => Model)` - 1:1 (ex: User tem Profile)
- `@hasMany(() => Model)` - 1:N (ex: User tem Posts)
- `@belongsTo(() => Model)` - Inverso de hasOne/hasMany
- `@manyToMany(() => Model)` - N:N (usa tabela pivot)
- `@hasManyThrough([() => Model1, () => Model2])` - Através de modelo intermediário

**Exemplo completo com relacionamentos:**
```typescript
import { BaseModel, column, hasMany, belongsTo, manyToMany } from '@adonisjs/lucid/orm'
import type { HasMany, BelongsTo, ManyToMany } from '@adonisjs/lucid/types/relations'

export default class Post extends BaseModel {
  @column({ isPrimary: true })
  declare id: number

  @column()
  declare userId: number

  // Relacionamento N:1
  @belongsTo(() => User)
  declare author: BelongsTo<typeof User>

  // Relacionamento 1:N
  @hasMany(() => Comment)
  declare comments: HasMany<typeof Comment>

  // Relacionamento N:N
  @manyToMany(() => Tag, {
    pivotTable: 'post_tag',
    pivotColumns: ['created_at'] // Colunas extras da pivot
  })
  declare tags: ManyToMany<typeof Tag>
}
```

**Hooks de Model:**
```typescript
import { BaseModel, beforeSave, afterCreate } from '@adonisjs/lucid/orm'

export default class User extends BaseModel {
  @beforeSave()
  static async hashPassword(user: User) {
    if (user.$dirty.password) {
      user.password = await hash.make(user.password)
    }
  }

  @afterCreate()
  static async sendWelcomeEmail(user: User) {
    // Enviar email de boas-vindas
  }
}
```

**⚠️ REGRAS CRÍTICAS SOBRE HOOKS:**

1. **NUNCA chamar hooks manualmente**
   - Hooks são executados **automaticamente** pelo Lucid ORM
   - `@beforeSave()`, `@afterCreate()`, etc. NÃO são funções utilitárias
   - Chamadas manuais causam comportamento indefinido

2. **🚫 PROIBIDO:**
   ```typescript
   // NUNCA faça isso!
   User.hashPassword(user)
   await User.sendWelcomeEmail(user)
   ```

3. **✅ CORRETO:**
   ```typescript
   // Hooks executam automaticamente durante operações do model
   await User.create({ email: 'user@example.com', password: 'secret' })
   // beforeSave() executa automaticamente e faz hash da senha
   
   await user.save()
   // beforeSave() executa automaticamente se houver alterações
   ```

4. **Disponíveis:**
   - `@beforeSave()` - Antes de INSERT/UPDATE
   - `@afterSave()` - Depois de INSERT/UPDATE
   - `@beforeCreate()` - Antes de INSERT
   - `@afterCreate()` - Depois de INSERT
   - `@beforeUpdate()` - Antes de UPDATE
   - `@afterUpdate()` - Depois de UPDATE
   - `@beforeDelete()` - Antes de DELETE
   - `@afterDelete()` - Depois de DELETE
   - `@beforeFind()` - Antes de SELECT (um registro)
   - `@afterFind()` - Depois de SELECT (um registro)
   - `@beforeFetch()` - Antes de SELECT (múltiplos)
   - `@afterFetch()` - Depois de SELECT (múltiplos)

### 5. Operações CRUD com Lucid

**Create:**
```typescript
// Método 1: create
const user = await User.create({ email: 'user@example.com' })

// Método 2: new + save
const user = new User()
user.email = 'user@example.com'
await user.save()

// Criar múltiplos
await User.createMany([{ email: 'a@b.com' }, { email: 'c@d.com' }])
```

**Read:**
```typescript
// Por ID
const user = await User.find(1) // retorna null se não encontrar
const user = await User.findOrFail(1) // lança exceção se não encontrar

// Por campo
const user = await User.findBy('email', 'user@example.com')
const user = await User.findByOrFail('email', 'user@example.com')

// Múltiplos
const users = await User.findMany([1, 2, 3])
const users = await User.all()

// Com query builder
const users = await User.query().where('isActive', true).orderBy('id', 'desc')

// Preload de relacionamentos
const posts = await Post.query()
  .preload('author')
  .preload('comments', (query) => {
    query.where('isApproved', true)
  })

// Agregações de relacionamento
const posts = await Post.query().withCount('comments')
```

**Update:**
```typescript
const user = await User.findOrFail(1)
user.email = 'new@example.com'
await user.save()

// Ou usando merge
await user.merge({ email: 'new@example.com' }).save()
```

**Delete:**
```typescript
const user = await User.findOrFail(1)
await user.delete()

// Bulk delete
await User.query().where('isVerified', false).delete()
```

**Métodos Idempotentes:**
```typescript
// firstOrCreate - busca ou cria
const user = await User.firstOrCreate(
  { email: 'user@example.com' },
  { password: 'secret', name: 'User' }
)

// updateOrCreate - atualiza ou cria
await User.updateOrCreate(
  { email: 'user@example.com' },
  { name: 'Updated Name' }
)

// fetchOrCreateMany - busca ou cria múltiplos
await User.fetchOrCreateMany('email', [
  { email: 'a@b.com', name: 'A' },
  { email: 'c@d.com', name: 'C' }
])
```

### 6. Banco de Dados MySQL

- Cliente: `mysql2`
- Configuração em [config/database.ts](config/database.ts)
- Variáveis de ambiente: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_DATABASE`
- Migrations em `database/migrations/` com natural sort

**Migrations - Tipos de Colunas Comuns:**
```typescript
table.increments('id') // AUTO_INCREMENT PRIMARY KEY
table.string('email', 255) // VARCHAR(255)
table.text('description') // TEXT
table.integer('count') // INTEGER
table.boolean('is_active') // BOOLEAN
table.decimal('price', 10, 2) // DECIMAL(10,2)
table.timestamp('created_at') // TIMESTAMP
table.json('metadata') // JSON

// Foreign Keys
table.integer('user_id').unsigned().references('id').inTable('users').onDelete('CASCADE')

// Índices
table.unique('email')
table.index('status')
```

### 7. Validação de Env

Todas as variáveis de ambiente são validadas em [start/env.ts](start/env.ts):
- `Env.schema.string()`, `Env.schema.number()`, `Env.schema.enum()`
- Formato host: `{ format: 'host' }`

---

## Padrões de Documentação

### Comentários Doxygen

**TODOS os arquivos devem ter:**
- Header com descrição, autor, data
- Comentários Doxygen em classes, métodos, parâmetros
- Comentários inline para lógica complexa

**Exemplo:**
```typescript
/**
 * Controller para gerenciar usuários
 * 
 * @description Implementa operações CRUD para entidade User
 * @author Sistema
 * @since 2025-01-13
 */
export default class UsersController {
  /**
   * Lista todos os usuários ativos
   * 
   * @param {HttpContext} ctx - Contexto HTTP do AdonisJS
   * @returns {Promise<Array<User>>} Lista de usuários
   * @throws {Exception} Se houver erro na consulta
   */
  async index({ response }: HttpContext): Promise<void> {
    // Busca usuários ativos ordenados por data de criação
    const users = await User.query()
      .where('isActive', true)
      .orderBy('createdAt', 'desc')
    
    return response.ok(users)
  }
}
```

---

### 1. Imports com Path Aliases

Use `#` para importar módulos da aplicação:

```typescript
import User from '#models/user'
import AuthMiddleware from '#middleware/auth_middleware'
import '#start/env'
```

### 2. Autenticação via Access Tokens

- Guard padrão: `api` (configurado em [config/auth.ts](config/auth.ts))
- Provider: `DbAccessTokensProvider` com modelo `User`
- Uso: `ctx.auth.authenticateUsing()` no middleware
- Modelo User tem `User.accessTokens` para gerenciar tokens

### 3. Middleware Stack

Em [start/kernel.ts](start/kernel.ts):

**Server middleware** (todas as requisições):
1. `container_bindings_middleware` - Injeta HttpContext e Logger no container
2. `force_json_response_middleware` - Força respostas JSON (altera header Accept)
3. `@adonisjs/cors/cors_middleware`

**Router middleware**:
1. `@adonisjs/core/bodyparser_middleware`
2. `@adonisjs/auth/initialize_auth_middleware`

**Named middleware**:
- `auth` - Aplique em rotas protegidas: `router.get('/profile', handler).use(middleware.auth())`

### 4. Models com Lucid ORM

- Herdam de `BaseModel` com mixins via `compose()`
- User usa `withAuthFinder` para autenticação
- Timestamps: `@column.dateTime({ autoCreate: true, autoUpdate: true })`
- Senha: use `{ serializeAs: null }` para ocultar em JSON
- Acesso ao hash: `hash.use('scrypt')`

### 5. Banco de Dados MySQL

- Cliente: `mysql2`
- Configuração em [config/database.ts](config/database.ts)
- Variáveis de ambiente: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_DATABASE`
- Migrations em `database/migrations/` com natural sort

### 6. Validação de Env

Todas as variáveis de ambiente são validadas em [start/env.ts](start/env.ts):
- `Env.schema.string()`, `Env.schema.number()`, `Env.schema.enum()`
- Formato host: `{ format: 'host' }`

## Comandos de Desenvolvimento

```bash
# Servidor de desenvolvimento
node ace serve --watch

# Executar migrations
node ace migration:run

# Reverter migrations
node ace migration:rollback

# Testes
node ace test

# Criar migration
node ace make:migration <nome>

# Criar modelo
node ace make:model <nome>

# Criar middleware
node ace make:middleware <nome>
```

## Padrões de Rotas

Em [start/routes.ts](start/routes.ts):

```typescript
import router from '@adonisjs/core/services/router'
import { middleware } from '#start/kernel'

// Rota pública
router.get('/', async () => ({ hello: 'world' }))

// Rota protegida
router.get('/profile', handler).use(middleware.auth())

// Grupo de rotas
router.group(() => {
  router.post('/login', handler)
  router.post('/register', handler)
}).prefix('/api')
```

## Testes com Japa

- Configuração em [tests/bootstrap.ts](tests/bootstrap.ts)
- Plugins: `assert`, `apiClient`, `pluginAdonisJS`
- Testes funcionais/e2e iniciam HTTP server automaticamente
- Entry point: [bin/test.ts](bin/test.ts) com `NODE_ENV=test`

## Exception Handling

- Handler customizado em [app/exceptions/handler.ts](app/exceptions/handler.ts)
- Herda de `ExceptionHandler`
- `debug = !app.inProduction` para stack traces detalhados
- Métodos: `handle()` para resposta, `report()` para logging

## Observações Importantes

- Sempre retorne JSON: o middleware `force_json_response_middleware` garante isso
- Use dependency injection via container resolver quando necessário
- Migrations devem ter métodos `up()` e `down()`
- Services disponíveis via imports: `@adonisjs/core/services/{router,server,app,hash,testUtils}`
