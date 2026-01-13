# 02 - Criação das Migrations

## 📚 O que são Migrations?

Migrations são arquivos que descrevem a estrutura do banco de dados de forma versionada. Elas permitem:

- ✅ Criar e alterar tabelas de forma programática
- ✅ Manter histórico de mudanças no schema
- ✅ Facilitar deploy e sincronização entre ambientes
- ✅ Reverter alterações (rollback)

## 🗂️ Ordem de Criação das Migrations

**⚠️ IMPORTANTE:** Migrations devem ser criadas na ordem de dependência:

1. **Tabelas independentes** (sem foreign keys)
2. **Tabelas dependentes** (com foreign keys)

### Ordem para este projeto:

1. `users` (já existe)
2. `livros`
3. `autores`
4. `autor_livro` (depende de livros e autores)
5. `exemplares` (depende de livros)
6. `access_tokens` (já existe)
7. `reservas` (depende de users e livros)
8. `emprestimos` (depende de users)
9. `item_emprestimo` (depende de emprestimos e exemplares)

## 📝 Migration 1: Atualizar Tabela Users

A migration `users` já existe, mas precisamos adicionar campos específicos.

**Arquivo:** `database/migrations/1768302505585_create_users_table.ts`

```typescript
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'users'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id').notNullable()
      table.string('matricula', 50).notNullable().unique().comment('Matrícula do usuário')
      table.string('nome', 255).notNullable().comment('Nome completo')
      table.string('email', 191).notNullable().unique().comment('Email do usuário')
      table.string('telefone', 20).nullable().comment('Telefone de contato')
      table.date('data_cadastro').notNullable().comment('Data de cadastro')
      table
        .string('tipo', 20)
        .notNullable()
        .defaultTo('aluno')
        .comment('Tipo de usuário (aluno, professor, funcionario)')
      table.string('password').notNullable()

      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').nullable()

      // Índices
      table.index('matricula')
      table.index('tipo')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

**Observações importantes:**
- Email limitado a 191 caracteres (limite de índice UNIQUE no MySQL com utf8mb4)
- Campo `matricula` é UNIQUE para identificação única
- Campo `tipo` com valor padrão 'aluno'

## 📝 Migration 2: Criar Tabela Livros

**Arquivo:** `database/migrations/1768302505586_create_livros_table.ts`

```typescript
/**
 * Migration para criar tabela livros
 * 
 * @description Armazena informações dos livros do acervo da biblioteca
 * @author Sistema
 * @since 2025-01-13
 */
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'livros'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.string('isbn', 20).primary().comment('ISBN do livro (chave primária)')
      table.string('titulo', 255).notNullable().comment('Título do livro')
      table.integer('ano_publicacao').unsigned().notNullable().comment('Ano de publicação')
      table.integer('edicao').unsigned().nullable().comment('Edição do livro')
      table.integer('id_editora').unsigned().nullable().comment('ID da editora')
      table.integer('id_categoria').unsigned().nullable().comment('ID da categoria')
      table.string('editora', 100).nullable().comment('Nome da editora')
      table.string('categoria', 50).nullable().comment('Categoria/gênero do livro')
      
      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').notNullable()

      // Índices para otimização
      table.index('titulo')
      table.index('id_editora')
      table.index('id_categoria')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

**Observações importantes:**
- ISBN como chave primária (string)
- Índices em campos frequentemente buscados
- Campos nullable para editora e categoria (futuras expansões)

## 📝 Migration 3: Criar Tabela Autores

**Arquivo:** `database/migrations/1768302505587_create_autores_table.ts`

```typescript
/**
 * Migration para criar tabela autores
 * 
 * @description Armazena informações dos autores de livros
 * @author Sistema
 * @since 2025-01-13
 */
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'autores'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id').primary().comment('ID do autor (chave primária)')
      table.string('nome', 255).notNullable().comment('Nome completo do autor')
      table.date('data_nascimento').nullable().comment('Data de nascimento do autor')
      
      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').notNullable()

      // Índice para busca por nome
      table.index('nome')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

## 📝 Migration 4: Criar Tabela Autor_Livro (Pivot)

**Arquivo:** `database/migrations/1768302505588_create_autor_livro_table.ts`

```typescript
/**
 * Migration para criar tabela autor_livro
 * 
 * @description Tabela pivot para relacionamento N:N entre autores e livros
 * @author Sistema
 * @since 2025-01-13
 */
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'autor_livro'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id').primary().comment('ID do relacionamento (chave primária)')
      table
        .integer('id_autor')
        .unsigned()
        .notNullable()
        .references('id')
        .inTable('autores')
        .onDelete('CASCADE')
        .onUpdate('CASCADE')
        .comment('ID do autor (FK)')
      table
        .string('isbn_livro', 20)
        .notNullable()
        .references('isbn')
        .inTable('livros')
        .onDelete('CASCADE')
        .onUpdate('CASCADE')
        .comment('ISBN do livro (FK)')
      table.integer('ordem').unsigned().nullable().comment('Ordem do autor (1º, 2º, etc)')
      table.string('papel', 50).nullable().comment('Papel do autor (autor, coautor, organizador, etc)')
      
      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').notNullable()

      // Índices compostos para otimização
      table.unique(['id_autor', 'isbn_livro'])
      table.index('id_autor')
      table.index('isbn_livro')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

**Observações importantes:**
- `CASCADE` em `onDelete`: ao deletar autor/livro, remove relacionamento
- `CASCADE` em `onUpdate`: ao atualizar PK, atualiza FK
- Índice UNIQUE composto evita duplicação de relacionamento

## 📝 Migration 5: Criar Tabela Exemplares

**Arquivo:** `database/migrations/1768302505589_create_exemplares_table.ts`

```typescript
/**
 * Migration para criar tabela exemplares
 * 
 * @description Armazena os exemplares físicos de cada livro
 * @author Sistema
 * @since 2025-01-13
 */
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'exemplares'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.string('codigo', 50).primary().comment('Código do exemplar (chave primária)')
      table
        .string('isbn_livro', 20)
        .notNullable()
        .references('isbn')
        .inTable('livros')
        .onDelete('RESTRICT')
        .onUpdate('CASCADE')
        .comment('ISBN do livro (FK)')
      table.string('localizacao', 100).nullable().comment('Localização física na biblioteca')
      table
        .string('status', 20)
        .notNullable()
        .defaultTo('disponivel')
        .comment('Status do exemplar (disponivel, emprestado, reservado, manutencao)')
      
      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').notNullable()

      // Índices para otimização
      table.index('isbn_livro')
      table.index('status')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

**Observações importantes:**
- `RESTRICT` em `onDelete`: impede deletar livro com exemplares
- Status padrão 'disponivel'

## 📝 Migration 6: Criar Tabela Reservas

**Arquivo:** `database/migrations/1768302505591_create_reservas_table.ts`

```typescript
/**
 * Migration para criar tabela reservas
 * 
 * @description Armazena as reservas de livros feitas pelos usuários
 * @author Sistema
 * @since 2025-01-13
 */
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'reservas'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id').primary().comment('ID da reserva (chave primária)')
      table
        .string('matricula_usuario', 50)
        .notNullable()
        .references('matricula')
        .inTable('users')
        .onDelete('RESTRICT')
        .onUpdate('CASCADE')
        .comment('Matrícula do usuário (FK)')
      table
        .string('isbn_livro', 20)
        .notNullable()
        .references('isbn')
        .inTable('livros')
        .onDelete('RESTRICT')
        .onUpdate('CASCADE')
        .comment('ISBN do livro (FK)')
      table.date('data_expiracao').notNullable().comment('Data de expiração da reserva')
      table
        .string('status', 20)
        .notNullable()
        .defaultTo('ativa')
        .comment('Status da reserva (ativa, atendida, cancelada, expirada)')
      table
        .string('codigo_exemplar_atribuido', 50)
        .nullable()
        .comment('Código do exemplar atribuído à reserva')
      
      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').notNullable()

      // Índices para otimização
      table.index('matricula_usuario')
      table.index('isbn_livro')
      table.index('status')
      table.index('data_expiracao')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

## 📝 Migration 7: Criar Tabela Emprestimos

**Arquivo:** `database/migrations/1768302505592_create_emprestimos_table.ts`

```typescript
/**
 * Migration para criar tabela emprestimos
 * 
 * @description Armazena os empréstimos realizados pelos usuários
 * @author Sistema
 * @since 2025-01-13
 */
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'emprestimos'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id').primary().comment('ID do empréstimo (chave primária)')
      table
        .string('matricula_usuario', 50)
        .notNullable()
        .references('matricula')
        .inTable('users')
        .onDelete('RESTRICT')
        .onUpdate('CASCADE')
        .comment('Matrícula do usuário (FK)')
      table.date('data_retirada').notNullable().comment('Data de retirada dos livros')
      table
        .string('status', 20)
        .notNullable()
        .defaultTo('ativo')
        .comment('Status do empréstimo (ativo, finalizado, atrasado)')
      
      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').notNullable()

      // Índices para otimização
      table.index('matricula_usuario')
      table.index('status')
      table.index('data_retirada')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

## 📝 Migration 8: Criar Tabela Item_Emprestimo

**Arquivo:** `database/migrations/1768302505593_create_item_emprestimo_table.ts`

```typescript
/**
 * Migration para criar tabela item_emprestimo
 * 
 * @description Armazena os itens (exemplares) de cada empréstimo
 * @autor Sistema
 * @since 2025-01-13
 */
import { BaseSchema } from '@adonisjs/lucid/schema'

export default class extends BaseSchema {
  protected tableName = 'item_emprestimo'

  async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.increments('id').primary().comment('ID do item (chave primária)')
      table
        .integer('id_emprestimo')
        .unsigned()
        .notNullable()
        .references('id')
        .inTable('emprestimos')
        .onDelete('CASCADE')
        .onUpdate('CASCADE')
        .comment('ID do empréstimo (FK)')
      table
        .string('codigo_exemplar', 50)
        .notNullable()
        .references('codigo')
        .inTable('exemplares')
        .onDelete('RESTRICT')
        .onUpdate('CASCADE')
        .comment('Código do exemplar (FK)')
      table.date('data_prevista_devolucao').notNullable().comment('Data prevista de devolução')
      table.date('data_devolucao').nullable().comment('Data real de devolução')
      table
        .string('status_item', 20)
        .notNullable()
        .defaultTo('emprestado')
        .comment('Status do item (emprestado, devolvido, atrasado)')
      
      table.timestamp('created_at').notNullable()
      table.timestamp('updated_at').notNullable()

      // Índices para otimização
      table.index('id_emprestimo')
      table.index('codigo_exemplar')
      table.index('status_item')
      table.index('data_prevista_devolucao')
    })
  }

  async down() {
    this.schema.dropTable(this.tableName)
  }
}
```

## 🚀 Executando as Migrations

### Passo 1: Executar Migrations

```powershell
node ace migration:fresh
```

**Saída esperada:**
```
[ success ] Dropped tables successfully
[ info ] Upgrading migrations version from "1" to "2"
❯ migrated database/migrations/1768302505585_create_users_table
❯ migrated database/migrations/1768302505586_create_livros_table
❯ migrated database/migrations/1768302505587_create_autores_table
❯ migrated database/migrations/1768302505588_create_autor_livro_table
❯ migrated database/migrations/1768302505589_create_exemplares_table
❯ migrated database/migrations/1768302505590_create_access_tokens_table
❯ migrated database/migrations/1768302505591_create_reservas_table
❯ migrated database/migrations/1768302505592_create_emprestimos_table
❯ migrated database/migrations/1768302505593_create_item_emprestimo_table

Migrated in 693 ms
```

### Passo 2: Verificar Tabelas Criadas

```powershell
mysql -u root -p biblioteca_db -e "SHOW TABLES;"
```

**Saída esperada:**
```
+---------------------------+
| Tables_in_biblioteca_db   |
+---------------------------+
| access_tokens             |
| adonis_schema             |
| adonis_schema_versions    |
| autor_livro               |
| autores                   |
| emprestimos               |
| exemplares                |
| item_emprestimo           |
| livros                    |
| reservas                  |
| users                     |
+---------------------------+
```

## 📊 Comandos Úteis de Migration

```powershell
# Executar migrations pendentes
node ace migration:run

# Reverter última migração
node ace migration:rollback

# Reverter todas as migrations
node ace migration:reset

# Dropar tudo e recriar
node ace migration:fresh

# Ver status das migrations
node ace migration:status
```

## ✅ Checklist

Antes de prosseguir, confirme:

- [ ] Todas as 9 migrations criadas
- [ ] Comando `migration:fresh` executado com sucesso
- [ ] 11 tabelas criadas no banco de dados
- [ ] Sem erros de foreign key ou sintaxe

## 🎯 Próximo Passo

Banco de dados estruturado! Agora vamos criar os models.

➡️ **[03-criacao-models.md](03-criacao-models.md)**

---

**Tempo estimado:** 30-40 minutos  
**Dificuldade:** ⭐⭐ Intermediário
