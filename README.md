# 📚 Documentação da API de Biblioteca - AdonisJS v6

## Índice de Documentos

Esta pasta contém o guia completo passo a passo da aula prática de criação de uma API REST com AdonisJS v6.

### 📖 Documentos (em ordem de leitura)

1. **[00-introducao.md](00-introducao.md)**  
   Visão geral do projeto, objetivos da aula, tecnologias utilizadas e arquitetura do sistema.

2. **[01-preparacao-ambiente.md](01-preparacao-ambiente.md)**  
   Configuração inicial: verificação de pré-requisitos, criação do banco de dados, configuração do `.env` e Git.

3. **[02-criacao-migrations.md](02-criacao-migrations.md)**  
   Criação de todas as migrations (9 tabelas) com explicações detalhadas sobre foreign keys, índices e relacionamentos.

4. **[03-criacao-models.md](03-criacao-models.md)**  
   Implementação dos 7 models (User, Livro, Autor, Exemplar, Reserva, Emprestimo, ItemEmprestimo) com relacionamentos complexos.

5. **[04-criacao-controllers.md](04-criacao-controllers.md)**  
   Desenvolvimento de 5 controllers CRUD completos com lógica de negócio e preload de relacionamentos.

6. **[05-autenticacao.md](05-autenticacao.md)**  
   Sistema de autenticação via Access Tokens: registro, login, perfil e logout.

7. **[07-validacao-testes.md](07-validacao-testes.md)**  
   Validação da implementação e testes manuais de todos os endpoints com Postman/Insomnia.

## 🎯 Como Usar Esta Documentação

### Para Alunos

1. **Siga em ordem:** Cada documento complementa o anterior
2. **Execute os comandos:** Todos os exemplos são testados e funcionais
3. **Faça commits:** Ao final de cada etapa, faça um commit no Git
4. **Teste sempre:** Valide cada etapa antes de prosseguir

### Para Professores

- Use como roteiro de aula
- Tempo estimado por documento está indicado
- Exemplos podem ser adaptados para outros contextos
- Contém checklistss para validação de aprendizado

## 🛠️ Estrutura do Projeto Final

```
aula-13012026/
├── app/
│   ├── controllers/
│   │   ├── auth_controller.ts
│   │   ├── autores_controller.ts
│   │   ├── emprestimos_controller.ts
│   │   ├── exemplares_controller.ts
│   │   ├── livros_controller.ts
│   │   └── reservas_controller.ts
│   ├── middleware/
│   │   └── auth_middleware.ts
│   └── models/
│       ├── autor.ts
│       ├── emprestimo.ts
│       ├── exemplar.ts
│       ├── item_emprestimo.ts
│       ├── livro.ts
│       ├── reserva.ts
│       └── user.ts
├── config/
│   ├── auth.ts
│   ├── database.ts
│   └── ...
├── database/
│   └── migrations/
│       ├── 1768302505585_create_users_table.ts
│       ├── 1768302505586_create_livros_table.ts
│       ├── 1768302505587_create_autores_table.ts
│       ├── 1768302505588_create_autor_livro_table.ts
│       ├── 1768302505589_create_exemplares_table.ts
│       ├── 1768302505590_create_access_tokens_table.ts
│       ├── 1768302505591_create_reservas_table.ts
│       ├── 1768302505592_create_emprestimos_table.ts
│       └── 1768302505593_create_item_emprestimo_table.ts
├── docs/
│   ├── 00-introducao.md
│   ├── 01-preparacao-ambiente.md
│   ├── 02-criacao-migrations.md
│   ├── 03-criacao-models.md
│   ├── 04-criacao-controllers.md
│   ├── 05-autenticacao.md
│   ├── 06-rotas.md
│   ├── 07-validacao-testes.md
│   └── README.md (este arquivo)
├── start/
│   ├── env.ts
│   ├── kernel.ts
│   └── routes.ts
├── .env
├── adonisrc.ts
├── package.json
└── tsconfig.json
```

## 📊 Resumo de Endpoints

### Autenticação (Públicas)
- `POST /api/auth/register` - Registrar usuário
- `POST /api/auth/login` - Autenticar usuário

### Perfil (Protegidas)
- `GET /api/me` - Dados do usuário autenticado
- `POST /api/logout` - Desautenticar usuário

### Recursos (Protegidas - CRUD)
- Livros: `/api/livros`
- Autores: `/api/autores`
- Exemplares: `/api/exemplares`
- Reservas: `/api/reservas`
- Empréstimos: `/api/emprestimos`

### Rotas Customizadas
- `GET /api/reservas/minhas` - Reservas do usuário
- `GET /api/emprestimos/meus` - Empréstimos do usuário
- `POST /api/emprestimos/itens/:itemId/devolver` - Devolver item

## 🎓 Conceitos Abordados

- ✅ Migrations e schema de banco de dados
- ✅ ORM Lucid (CRUD, queries, relacionamentos)
- ✅ Relacionamentos 1:1, 1:N, N:N
- ✅ Controllers e padrão MVC
- ✅ Autenticação via Access Tokens
- ✅ Middleware de autenticação
- ✅ Rotas públicas e protegidas
- ✅ Tratamento de erros HTTP
- ✅ Preload de relacionamentos (evitar N+1)
- ✅ Validação TypeScript
- ✅ Boas práticas REST

## 🔗 Links Úteis

- [Documentação AdonisJS v6](https://docs.adonisjs.com/guides/introduction)
- [Lucid ORM](https://lucid.adonisjs.com/)
- [Auth Package](https://docs.adonisjs.com/guides/authentication)
- [MySQL Documentation](https://dev.mysql.com/doc/)

## 📝 Licença

Este material é de uso educacional para a disciplina de Tecnologias Empresariais e Programação (TEP).

---

**Autor:** Prof. Luis Vitorino - IFPI  
**Data:** 13 de janeiro de 2026  
**Versão:** 1.0.0
