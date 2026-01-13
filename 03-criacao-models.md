# 03 - Criação dos Models

## 📚 O que são Models?

Models (ou Modelos) são classes que representam as entidades do banco de dados no código. No AdonisJS com Lucid ORM, eles:

- ✅ Mapeiam tabelas do banco para classes TypeScript
- ✅ Definem relacionamentos entre entidades
- ✅ Fornecem métodos para CRUD (Create, Read, Update, Delete)
- ✅ Aplicam validações e transformações de dados

## 🏗️ Estrutura de um Model

Todo model no AdonisJS herda de `BaseModel` e usa decoradores TypeScript:

```typescript
import { BaseModel, column } from '@adonisjs/lucid/orm'

export default class NomeModel extends BaseModel {
  @column({ isPrimary: true })
  declare id: number

  @column()
  declare nome: string
}
```

## 📝 Model 1: Atualizar User

O model User já existe, mas precisamos adicionar os novos campos e relacionamentos.

**Arquivo:** `app/models/user.ts`

```typescript
/**
 * Model User
 * 
 * @description Representa um usuário do sistema de biblioteca
 * @author Sistema
 * @since 2025-01-13
 */
import { DateTime } from 'luxon'
import hash from '@adonisjs/core/services/hash'
import { compose } from '@adonisjs/core/helpers'
import { BaseModel, column, hasMany } from '@adonisjs/lucid/orm'
import { withAuthFinder } from '@adonisjs/auth/mixins/lucid'
import { DbAccessTokensProvider } from '@adonisjs/auth/access_tokens'
import type { HasMany } from '@adonisjs/lucid/types/relations'
import Reserva from '#models/reserva'
import Emprestimo from '#models/emprestimo'

const AuthFinder = withAuthFinder(() => hash.use('scrypt'), {
  uids: ['email'],
  passwordColumnName: 'password',
})

export default class User extends compose(BaseModel, AuthFinder) {
  /**
   * Chave primária
   */
  @column({ isPrimary: true })
  declare id: number

  /**
   * Matrícula única do usuário
   */
  @column()
  declare matricula: string

  /**
   * Nome completo do usuário
   */
  @column()
  declare nome: string

  /**
   * Email do usuário
   */
  @column()
  declare email: string

  /**
   * Telefone de contato
   */
  @column()
  declare telefone: string | null

  /**
   * Data de cadastro
   */
  @column.date()
  declare dataCadastro: DateTime

  /**
   * Tipo de usuário (aluno, professor, funcionario)
   */
  @column()
  declare tipo: string

  /**
   * Senha do usuário (não serializada)
   */
  @column({ serializeAs: null })
  declare password: string

  /**
   * Data de criação do registro
   */
  @column.dateTime({ autoCreate: true })
  declare createdAt: DateTime

  /**
   * Data da última atualização
   */
  @column.dateTime({ autoCreate: true, autoUpdate: true })
  declare updatedAt: DateTime | null

  /**
   * Relacionamento: Usuário tem várias reservas
   */
  @hasMany(() => Reserva, {
    foreignKey: 'matriculaUsuario',
    localKey: 'matricula',
  })
  declare reservas: HasMany<typeof Reserva>

  /**
   * Relacionamento: Usuário tem vários empréstimos
   */
  @hasMany(() => Emprestimo, {
    foreignKey: 'matriculaUsuario',
    localKey: 'matricula',
  })
  declare emprestimos: HasMany<typeof Emprestimo>

  static accessTokens = DbAccessTokensProvider.forModel(User)
}
```

**Observações importantes:**
- `withAuthFinder` fornece métodos de autenticação (verifyCredentials, etc)
- `serializeAs: null` impede que a senha seja retornada em JSON
- Relacionamentos `hasMany` usam `matricula` como chave (não `id`)

## 📝 Model 2: Criar Livro

**Arquivo:** `app/models/livro.ts`

```typescript
/**
 * Model Livro
 * 
 * @description Representa um livro do acervo da biblioteca
 * @author Sistema
 * @since 2025-01-13
 */
import { DateTime } from 'luxon'
import { BaseModel, column, hasMany, manyToMany } from '@adonisjs/lucid/orm'
import type { HasMany, ManyToMany } from '@adonisjs/lucid/types/relations'
import Autor from '#models/autor'
import Exemplar from '#models/exemplar'
import Reserva from '#models/reserva'

export default class Livro extends BaseModel {
  /**
   * ISBN do livro (chave primária)
   */
  @column({ isPrimary: true })
  declare isbn: string

  /**
   * Título do livro
   */
  @column()
  declare titulo: string

  /**
   * Ano de publicação
   */
  @column()
  declare anoPublicacao: number

  /**
   * Edição do livro
   */
  @column()
  declare edicao: number | null

  /**
   * ID da editora
   */
  @column()
  declare idEditora: number | null

  /**
   * ID da categoria
   */
  @column()
  declare idCategoria: number | null

  /**
   * Nome da editora
   */
  @column()
  declare editora: string | null

  /**
   * Categoria/gênero do livro
   */
  @column()
  declare categoria: string | null

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
   * Relacionamento: Livro tem vários autores (N:N)
   */
  @manyToMany(() => Autor, {
    pivotTable: 'autor_livro',
    localKey: 'isbn',
    pivotForeignKey: 'isbn_livro',
    relatedKey: 'id',
    pivotRelatedForeignKey: 'id_autor',
    pivotColumns: ['ordem', 'papel'],
  })
  declare autores: ManyToMany<typeof Autor>

  /**
   * Relacionamento: Livro tem vários exemplares
   */
  @hasMany(() => Exemplar, {
    foreignKey: 'isbnLivro',
  })
  declare exemplares: HasMany<typeof Exemplar>

  /**
   * Relacionamento: Livro tem várias reservas
   */
  @hasMany(() => Reserva, {
    foreignKey: 'isbnLivro',
  })
  declare reservas: HasMany<typeof Reserva>
}
```

**Observações importantes:**
- `manyToMany` define relacionamento N:N com tabela pivot
- `pivotColumns` permite acessar campos extras da pivot (ordem, papel)
- ISBN como chave primária (string, não incremental)

## 📝 Model 3: Criar Autor

**Arquivo:** `app/models/autor.ts`

```typescript
/**
 * Model Autor
 * 
 * @description Representa um autor de livros
 * @author Sistema
 * @since 2025-01-13
 */
import { DateTime } from 'luxon'
import { BaseModel, column, manyToMany } from '@adonisjs/lucid/orm'
import type { ManyToMany } from '@adonisjs/lucid/types/relations'
import Livro from '#models/livro'

export default class Autor extends BaseModel {
  /**
   * Chave primária
   */
  @column({ isPrimary: true })
  declare id: number

  /**
   * Nome completo do autor
   */
  @column()
  declare nome: string

  /**
   * Data de nascimento do autor
   */
  @column.date()
  declare dataNascimento: DateTime | null

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
   * Relacionamento: Autor tem vários livros (N:N)
   */
  @manyToMany(() => Livro, {
    pivotTable: 'autor_livro',
    localKey: 'id',
    pivotForeignKey: 'id_autor',
    relatedKey: 'isbn',
    pivotRelatedForeignKey: 'isbn_livro',
    pivotColumns: ['ordem', 'papel'],
  })
  declare livros: ManyToMany<typeof Livro>
}
```

## 📝 Model 4: Criar Exemplar

**Arquivo:** `app/models/exemplar.ts`

```typescript
/**
 * Model Exemplar
 * 
 * @description Representa um exemplar físico de um livro
 * @author Sistema
 * @since 2025-01-13
 */
import { DateTime } from 'luxon'
import { BaseModel, column, belongsTo, hasMany } from '@adonisjs/lucid/orm'
import type { BelongsTo, HasMany } from '@adonisjs/lucid/types/relations'
import Livro from '#models/livro'
import ItemEmprestimo from '#models/item_emprestimo'

export default class Exemplar extends BaseModel {
  /**
   * Código do exemplar (chave primária)
   */
  @column({ isPrimary: true })
  declare codigo: string

  /**
   * ISBN do livro
   */
  @column()
  declare isbnLivro: string

  /**
   * Localização física na biblioteca
   */
  @column()
  declare localizacao: string | null

  /**
   * Status do exemplar (disponivel, emprestado, reservado, manutencao)
   */
  @column()
  declare status: string

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
   * Relacionamento: Exemplar pertence a um livro
   */
  @belongsTo(() => Livro, {
    foreignKey: 'isbnLivro',
  })
  declare livro: BelongsTo<typeof Livro>

  /**
   * Relacionamento: Exemplar tem vários itens de empréstimo
   */
  @hasMany(() => ItemEmprestimo, {
    foreignKey: 'codigoExemplar',
  })
  declare itensEmprestimo: HasMany<typeof ItemEmprestimo>
}
```

**Observações importantes:**
- `belongsTo` define relacionamento inverso (N:1)
- Código como chave primária (string)

## 📝 Model 5: Criar Reserva

**Arquivo:** `app/models/reserva.ts`

```typescript
/**
 * Model Reserva
 * 
 * @description Representa uma reserva de livro feita por um usuário
 * @author Sistema
 * @since 2025-01-13
 */
import { DateTime } from 'luxon'
import { BaseModel, column, belongsTo } from '@adonisjs/lucid/orm'
import type { BelongsTo } from '@adonisjs/lucid/types/relations'
import User from '#models/user'
import Livro from '#models/livro'

export default class Reserva extends BaseModel {
  /**
   * Chave primária
   */
  @column({ isPrimary: true })
  declare id: number

  /**
   * Matrícula do usuário
   */
  @column()
  declare matriculaUsuario: string

  /**
   * ISBN do livro
   */
  @column()
  declare isbnLivro: string

  /**
   * Data de expiração da reserva
   */
  @column.date()
  declare dataExpiracao: DateTime

  /**
   * Status da reserva (ativa, atendida, cancelada, expirada)
   */
  @column()
  declare status: string

  /**
   * Código do exemplar atribuído à reserva
   */
  @column()
  declare codigoExemplarAtribuido: string | null

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
   * Relacionamento: Reserva pertence a um usuário
   */
  @belongsTo(() => User, {
    foreignKey: 'matriculaUsuario',
    localKey: 'matricula',
  })
  declare usuario: BelongsTo<typeof User>

  /**
   * Relacionamento: Reserva pertence a um livro
   */
  @belongsTo(() => Livro, {
    foreignKey: 'isbnLivro',
  })
  declare livro: BelongsTo<typeof Livro>
}
```

## 📝 Model 6: Criar Emprestimo

**Arquivo:** `app/models/emprestimo.ts`

```typescript
/**
 * Model Emprestimo
 * 
 * @description Representa um empréstimo realizado por um usuário
 * @author Sistema
 * @since 2025-01-13
 */
import { DateTime } from 'luxon'
import { BaseModel, column, belongsTo, hasMany } from '@adonisjs/lucid/orm'
import type { BelongsTo, HasMany } from '@adonisjs/lucid/types/relations'
import User from '#models/user'
import ItemEmprestimo from '#models/item_emprestimo'

export default class Emprestimo extends BaseModel {
  /**
   * Chave primária
   */
  @column({ isPrimary: true })
  declare id: number

  /**
   * Matrícula do usuário
   */
  @column()
  declare matriculaUsuario: string

  /**
   * Data de retirada dos livros
   */
  @column.date()
  declare dataRetirada: DateTime

  /**
   * Status do empréstimo (ativo, finalizado, atrasado)
   */
  @column()
  declare status: string

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
   * Relacionamento: Emprestimo pertence a um usuário
   */
  @belongsTo(() => User, {
    foreignKey: 'matriculaUsuario',
    localKey: 'matricula',
  })
  declare usuario: BelongsTo<typeof User>

  /**
   * Relacionamento: Emprestimo tem vários itens
   */
  @hasMany(() => ItemEmprestimo, {
    foreignKey: 'idEmprestimo',
  })
  declare itens: HasMany<typeof ItemEmprestimo>
}
```

## 📝 Model 7: Criar ItemEmprestimo

**Arquivo:** `app/models/item_emprestimo.ts`

```typescript
/**
 * Model ItemEmprestimo
 * 
 * @description Representa um item (exemplar) de um empréstimo
 * @author Sistema
 * @since 2025-01-13
 */
import { DateTime } from 'luxon'
import { BaseModel, column, belongsTo } from '@adonisjs/lucid/orm'
import type { BelongsTo } from '@adonisjs/lucid/types/relations'
import Emprestimo from '#models/emprestimo'
import Exemplar from '#models/exemplar'

export default class ItemEmprestimo extends BaseModel {
  static table = 'item_emprestimo'

  /**
   * Chave primária
   */
  @column({ isPrimary: true })
  declare id: number

  /**
   * ID do empréstimo
   */
  @column()
  declare idEmprestimo: number

  /**
   * Código do exemplar
   */
  @column()
  declare codigoExemplar: string

  /**
   * Data prevista de devolução
   */
  @column.date()
  declare dataPrevistaDevolucao: DateTime

  /**
   * Data real de devolução
   */
  @column.date()
  declare dataDevolucao: DateTime | null

  /**
   * Status do item (emprestado, devolvido, atrasado)
   */
  @column()
  declare statusItem: string

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
   * Relacionamento: ItemEmprestimo pertence a um empréstimo
   */
  @belongsTo(() => Emprestimo, {
    foreignKey: 'idEmprestimo',
  })
  declare emprestimo: BelongsTo<typeof Emprestimo>

  /**
   * Relacionamento: ItemEmprestimo pertence a um exemplar
   */
  @belongsTo(() => Exemplar, {
    foreignKey: 'codigoExemplar',
  })
  declare exemplar: BelongsTo<typeof Exemplar>
}
```

**Observações importantes:**
- `static table = 'item_emprestimo'` sobrescreve nome da tabela
- Por padrão, Lucid pluraliza o nome da classe

## 🔍 Tipos de Relacionamentos Implementados

### 1. hasMany (1:N)
User tem muitas Reservas e Empréstimos:
```typescript
@hasMany(() => Reserva)
declare reservas: HasMany<typeof Reserva>
```

### 2. belongsTo (N:1)
Reserva pertence a um User:
```typescript
@belongsTo(() => User)
declare usuario: BelongsTo<typeof User>
```

### 3. manyToMany (N:N)
Livro tem muitos Autores (via pivot):
```typescript
@manyToMany(() => Autor, {
  pivotTable: 'autor_livro'
})
declare autores: ManyToMany<typeof Autor>
```

## ✅ Validando os Models

Execute o comando para verificar erros:

```powershell
node ace list
```

Se não houver erros TypeScript, os models estão corretos!

## 📊 Testando Relacionamentos no REPL

```powershell
node ace repl
```

No REPL:
```javascript
// Carregar um livro com autores
const livro = await (await import('#models/livro')).default
  .query()
  .preload('autores')
  .first()

console.log(livro?.autores)
```

## ✅ Checklist

Antes de prosseguir, confirme:

- [ ] 7 models criados (User, Livro, Autor, Exemplar, Reserva, Emprestimo, ItemEmprestimo)
- [ ] Todos os relacionamentos configurados
- [ ] Comando `node ace list` executa sem erros
- [ ] Imports com `#models/` funcionando

## 🎯 Próximo Passo

Models prontos! Agora vamos criar os controllers.

➡️ **[04-criacao-controllers.md](04-criacao-controllers.md)**

---

**Tempo estimado:** 40-50 minutos  
**Dificuldade:** ⭐⭐⭐ Intermediário/Avançado
