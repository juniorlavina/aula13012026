# 04 - Criação dos Controllers

## 📚 O que são Controllers?

Controllers são responsáveis por:

- ✅ Receber requisições HTTP
- ✅ Processar dados (validação, lógica de negócio)
- ✅ Interagir com models
- ✅ Retornar respostas HTTP

## 🏗️ Estrutura CRUD Padrão

Todo controller deve implementar os 5 métodos CRUD:

1. **index()** - Listar todos os registros (GET /recurso)
2. **store()** - Criar novo registro (POST /recurso)
3. **show()** - Exibir um registro (GET /recurso/:id)
4. **update()** - Atualizar registro (PUT /recurso/:id)
5. **destroy()** - Deletar registro (DELETE /recurso/:id)

## 📝 Controller 1: LivrosController

**Arquivo:** `app/controllers/livros_controller.ts`

```typescript
/**
 * Controller para Livros
 * 
 * @description Gerencia operações CRUD de livros
 * @author Sistema
 * @since 2025-01-13
 */
import type { HttpContext } from '@adonisjs/core/http'
import Livro from '#models/livro'

export default class LivrosController {
  /**
   * Lista todos os livros
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Array>} Lista de livros
   */
  async index({ response }: HttpContext) {
    const livros = await Livro.query().preload('autores').preload('exemplares')
    return response.ok(livros)
  }

  /**
   * Cria um novo livro
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Livro criado
   */
  async store({ request, response }: HttpContext) {
    const data = request.only([
      'isbn',
      'titulo',
      'anoPublicacao',
      'edicao',
      'idEditora',
      'idCategoria',
      'editora',
      'categoria',
    ])

    const livro = await Livro.create(data)
    
    // Associar autores se fornecidos
    const autoresIds = request.input('autores', [])
    if (autoresIds.length > 0) {
      await livro.related('autores').attach(autoresIds)
    }

    await livro.load('autores')
    return response.created(livro)
  }

  /**
   * Exibe um livro específico
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Livro encontrado
   */
  async show({ params, response }: HttpContext) {
    const livro = await Livro.query()
      .where('isbn', params.isbn)
      .preload('autores')
      .preload('exemplares')
      .preload('reservas')
      .firstOrFail()

    return response.ok(livro)
  }

  /**
   * Atualiza um livro
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<Object>} Livro atualizado
   */
  async update({ params, request, response }: HttpContext) {
    const livro = await Livro.findOrFail(params.isbn)
    const data = request.only([
      'titulo',
      'anoPublicacao',
      'edicao',
      'idEditora',
      'idCategoria',
      'editora',
      'categoria',
    ])

    await livro.merge(data).save()

    // Atualizar autores se fornecidos
    const autoresIds = request.input('autores')
    if (autoresIds !== undefined) {
      await livro.related('autores').sync(autoresIds)
    }

    await livro.load('autores')
    return response.ok(livro)
  }

  /**
   * Remove um livro
   * 
   * @param {HttpContext} ctx - Contexto HTTP
   * @returns {Promise<void>}
   */
  async destroy({ params, response }: HttpContext) {
    const livro = await Livro.findOrFail(params.isbn)
    await livro.delete()
    return response.noContent()
  }
}
```

**Observações importantes:**
- `preload()` carrega relacionamentos (evita N+1 queries)
- `attach()` associa registros em relacionamento N:N
- `sync()` sincroniza relacionamentos (remove antigos, adiciona novos)
- `response.ok()` retorna status 200
- `response.created()` retorna status 201
- `response.noContent()` retorna status 204

## 📝 Controller 2: AutoresController

**Arquivo:** `app/controllers/autores_controller.ts`

```typescript
/**
 * Controller para Autores
 * 
 * @description Gerencia operações CRUD de autores
 * @author Sistema
 * @since 2025-01-13
 */
import type { HttpContext } from '@adonisjs/core/http'
import Autor from '#models/autor'

export default class AutoresController {
  /**
   * Lista todos os autores
   */
  async index({ response }: HttpContext) {
    const autores = await Autor.query().preload('livros')
    return response.ok(autores)
  }

  /**
   * Cria um novo autor
   */
  async store({ request, response }: HttpContext) {
    const data = request.only(['nome', 'dataNascimento'])
    const autor = await Autor.create(data)
    return response.created(autor)
  }

  /**
   * Exibe um autor específico
   */
  async show({ params, response }: HttpContext) {
    const autor = await Autor.query().where('id', params.id).preload('livros').firstOrFail()

    return response.ok(autor)
  }

  /**
   * Atualiza um autor
   */
  async update({ params, request, response }: HttpContext) {
    const autor = await Autor.findOrFail(params.id)
    const data = request.only(['nome', 'dataNascimento'])
    await autor.merge(data).save()
    return response.ok(autor)
  }

  /**
   * Remove um autor
   */
  async destroy({ params, response }: HttpContext) {
    const autor = await Autor.findOrFail(params.id)
    await autor.delete()
    return response.noContent()
  }
}
```

## 📝 Controller 3: ExemplaresController

**Arquivo:** `app/controllers/exemplares_controller.ts`

```typescript
/**
 * Controller para Exemplares
 * 
 * @description Gerencia operações CRUD de exemplares
 * @author Sistema
 * @since 2025-01-13
 */
import type { HttpContext } from '@adonisjs/core/http'
import Exemplar from '#models/exemplar'

export default class ExemplaresController {
  /**
   * Lista todos os exemplares
   */
  async index({ response }: HttpContext) {
    const exemplares = await Exemplar.query().preload('livro')
    return response.ok(exemplares)
  }

  /**
   * Cria um novo exemplar
   */
  async store({ request, response }: HttpContext) {
    const data = request.only(['codigo', 'isbnLivro', 'localizacao', 'status'])
    const exemplar = await Exemplar.create(data)
    await exemplar.load('livro')
    return response.created(exemplar)
  }

  /**
   * Exibe um exemplar específico
   */
  async show({ params, response }: HttpContext) {
    const exemplar = await Exemplar.query()
      .where('codigo', params.codigo)
      .preload('livro')
      .preload('itensEmprestimo')
      .firstOrFail()

    return response.ok(exemplar)
  }

  /**
   * Atualiza um exemplar
   */
  async update({ params, request, response }: HttpContext) {
    const exemplar = await Exemplar.findOrFail(params.codigo)
    const data = request.only(['localizacao', 'status'])
    await exemplar.merge(data).save()
    await exemplar.load('livro')
    return response.ok(exemplar)
  }

  /**
   * Remove um exemplar
   */
  async destroy({ params, response }: HttpContext) {
    const exemplar = await Exemplar.findOrFail(params.codigo)
    await exemplar.delete()
    return response.noContent()
  }
}
```

## 📝 Controller 4: ReservasController

**Arquivo:** `app/controllers/reservas_controller.ts`

```typescript
/**
 * Controller para Reservas
 * 
 * @description Gerencia operações CRUD de reservas
 * @author Sistema
 * @since 2025-01-13
 */
import type { HttpContext } from '@adonisjs/core/http'
import Reserva from '#models/reserva'

export default class ReservasController {
  /**
   * Lista todas as reservas
   */
  async index({ response }: HttpContext) {
    const reservas = await Reserva.query().preload('usuario').preload('livro')
    return response.ok(reservas)
  }

  /**
   * Cria uma nova reserva
   */
  async store({ request, response, auth }: HttpContext) {
    const user = auth.user!
    const data = request.only(['isbnLivro', 'dataExpiracao'])
    
    // Define matrícula do usuário autenticado
    const reserva = await Reserva.create({
      ...data,
      matriculaUsuario: user.matricula,
      status: 'ativa',
    })

    await reserva.load('livro')
    return response.created(reserva)
  }

  /**
   * Exibe uma reserva específica
   */
  async show({ params, response }: HttpContext) {
    const reserva = await Reserva.query()
      .where('id', params.id)
      .preload('usuario')
      .preload('livro')
      .firstOrFail()

    return response.ok(reserva)
  }

  /**
   * Atualiza uma reserva
   */
  async update({ params, request, response }: HttpContext) {
    const reserva = await Reserva.findOrFail(params.id)
    const data = request.only(['status', 'codigoExemplarAtribuido', 'dataExpiracao'])
    await reserva.merge(data).save()
    await reserva.load('usuario')
    await reserva.load('livro')
    return response.ok(reserva)
  }

  /**
   * Remove uma reserva
   */
  async destroy({ params, response }: HttpContext) {
    const reserva = await Reserva.findOrFail(params.id)
    await reserva.delete()
    return response.noContent()
  }

  /**
   * Lista reservas do usuário autenticado
   */
  async minhasReservas({ auth, response }: HttpContext) {
    const user = auth.user!
    const reservas = await Reserva.query()
      .where('matriculaUsuario', user.matricula)
      .preload('livro')
      .orderBy('createdAt', 'desc')

    return response.ok(reservas)
  }
}
```

**Observações importantes:**
- `auth.user!` acessa usuário autenticado (! = non-null assertion)
- Método `minhasReservas()` é uma rota customizada (não CRUD padrão)

## 📝 Controller 5: EmprestimosController

**Arquivo:** `app/controllers/emprestimos_controller.ts`

```typescript
/**
 * Controller para Empréstimos
 * 
 * @description Gerencia operações CRUD de empréstimos
 * @author Sistema
 * @since 2025-01-13
 */
import type { HttpContext } from '@adonisjs/core/http'
import Emprestimo from '#models/emprestimo'
import ItemEmprestimo from '#models/item_emprestimo'
import Exemplar from '#models/exemplar'
import { DateTime } from 'luxon'

export default class EmprestimosController {
  /**
   * Lista todos os empréstimos
   */
  async index({ response }: HttpContext) {
    const emprestimos = await Emprestimo.query()
      .preload('usuario')
      .preload('itens', (query) => {
        query.preload('exemplar', (subQuery) => {
          subQuery.preload('livro')
        })
      })

    return response.ok(emprestimos)
  }

  /**
   * Cria um novo empréstimo
   */
  async store({ request, response, auth }: HttpContext) {
    const user = auth.user!
    const { dataRetirada, itens } = request.only(['dataRetirada', 'itens'])

    // Criar empréstimo
    const emprestimo = await Emprestimo.create({
      matriculaUsuario: user.matricula,
      dataRetirada: dataRetirada || DateTime.now().toSQLDate(),
      status: 'ativo',
    })

    // Criar itens do empréstimo
    if (itens && Array.isArray(itens)) {
      for (const item of itens) {
        await ItemEmprestimo.create({
          idEmprestimo: emprestimo.id,
          codigoExemplar: item.codigoExemplar,
          dataPrevistaDevolucao: item.dataPrevistaDevolucao,
          statusItem: 'emprestado',
        })

        // Atualizar status do exemplar
        const exemplar = await Exemplar.findOrFail(item.codigoExemplar)
        exemplar.status = 'emprestado'
        await exemplar.save()
      }
    }

    await emprestimo.load('itens', (query) => {
      query.preload('exemplar', (subQuery) => {
        subQuery.preload('livro')
      })
    })

    return response.created(emprestimo)
  }

  /**
   * Exibe um empréstimo específico
   */
  async show({ params, response }: HttpContext) {
    const emprestimo = await Emprestimo.query()
      .where('id', params.id)
      .preload('usuario')
      .preload('itens', (query) => {
        query.preload('exemplar', (subQuery) => {
          subQuery.preload('livro')
        })
      })
      .firstOrFail()

    return response.ok(emprestimo)
  }

  /**
   * Atualiza um empréstimo
   */
  async update({ params, request, response }: HttpContext) {
    const emprestimo = await Emprestimo.findOrFail(params.id)
    const data = request.only(['status'])
    await emprestimo.merge(data).save()
    await emprestimo.load('usuario')
    await emprestimo.load('itens')
    return response.ok(emprestimo)
  }

  /**
   * Remove um empréstimo
   */
  async destroy({ params, response }: HttpContext) {
    const emprestimo = await Emprestimo.findOrFail(params.id)
    await emprestimo.delete()
    return response.noContent()
  }

  /**
   * Lista empréstimos do usuário autenticado
   */
  async meusEmprestimos({ auth, response }: HttpContext) {
    const user = auth.user!
    const emprestimos = await Emprestimo.query()
      .where('matriculaUsuario', user.matricula)
      .preload('itens', (query) => {
        query.preload('exemplar', (subQuery) => {
          subQuery.preload('livro')
        })
      })
      .orderBy('createdAt', 'desc')

    return response.ok(emprestimos)
  }

  /**
   * Devolve um item de empréstimo
   */
  async devolverItem({ params, response }: HttpContext) {
    const item = await ItemEmprestimo.findOrFail(params.itemId)
    
    item.dataDevolucao = DateTime.now()
    item.statusItem = 'devolvido'
    await item.save()

    // Atualizar status do exemplar
    const exemplar = await Exemplar.findOrFail(item.codigoExemplar)
    exemplar.status = 'disponivel'
    await exemplar.save()

    // Verificar se todos os itens foram devolvidos
    const emprestimo = await Emprestimo.findOrFail(item.idEmprestimo)
    await emprestimo.load('itens')
    
    const todosDevolvidos = emprestimo.itens.every((i) => i.statusItem === 'devolvido')
    if (todosDevolvidos) {
      emprestimo.status = 'finalizado'
      await emprestimo.save()
    }

    await item.load('exemplar')
    return response.ok(item)
  }
}
```

**Observações importantes:**
- Preload aninhado: `query.preload('exemplar', subQuery => ...)`
- Lógica de negócio: atualiza status do exemplar ao emprestar/devolver
- Método `devolverItem()` é uma operação customizada

## ✅ Validando os Controllers

Execute o comando para verificar erros:

```powershell
# Verificar erros TypeScript/ESLint
node ace list
```

Se houver erros, execute:

```powershell
# Compilar e verificar
npm run build
```

## 📊 Métodos HTTP e Status Codes

| Método | Rota | Controller | Status |
|--------|------|------------|--------|
| GET | /livros | index() | 200 OK |
| POST | /livros | store() | 201 Created |
| GET | /livros/:isbn | show() | 200 OK |
| PUT | /livros/:isbn | update() | 200 OK |
| DELETE | /livros/:isbn | destroy() | 204 No Content |

## ✅ Checklist

Antes de prosseguir, confirme:

- [ ] 5 controllers criados (Livros, Autores, Exemplares, Reservas, Emprestimos)
- [ ] Todos os métodos CRUD implementados
- [ ] Comando `node ace list` executa sem erros
- [ ] Relacionamentos sendo carregados com `preload()`

## 🎯 Próximo Passo

Controllers prontos! Agora vamos implementar autenticação.

➡️ **[05-autenticacao.md](05-autenticacao.md)**

---

**Tempo estimado:** 50-60 minutos  
**Dificuldade:** ⭐⭐⭐ Intermediário/Avançado
