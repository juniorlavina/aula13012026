# Introdução - API de Biblioteca com AdonisJS v6

## 📚 Sobre este Projeto

Este guia documenta passo a passo a criação de uma **API REST completa** para um sistema de biblioteca utilizando **AdonisJS v6**, **TypeScript** e **MySQL**.

## 🎯 Objetivos da Aula

Ao final desta aula prática, você será capaz de:

- ✅ Configurar um projeto AdonisJS v6 do zero
- ✅ Criar e gerenciar migrations de banco de dados
- ✅ Implementar models com relacionamentos complexos (1:1, 1:N, N:N)
- ✅ Desenvolver controllers com CRUD completo
- ✅ Implementar autenticação via tokens (JWT)
- ✅ Configurar rotas públicas e protegidas
- ✅ Validar e corrigir erros TypeScript/ESLint
- ✅ Testar endpoints com ferramentas REST

## 🏗️ Arquitetura do Sistema

### Entidades Principais

O sistema de biblioteca gerencia as seguintes entidades:

1. **Usuários (Users)** - Pessoas que utilizam a biblioteca
2. **Livros (Livros)** - Catálogo de obras disponíveis
3. **Autores (Autores)** - Escritores das obras
4. **Exemplares (Exemplares)** - Cópias físicas dos livros
5. **Reservas (Reservas)** - Reservas de livros pelos usuários
6. **Empréstimos (Emprestimos)** - Registro de empréstimos
7. **Itens de Empréstimo (ItemEmprestimo)** - Exemplares emprestados

### Relacionamentos

```
User (1) ──┬── (N) Reserva ── (1) Livro
           │
           └── (N) Emprestimo ── (N) ItemEmprestimo ── (1) Exemplar ── (1) Livro

Livro (N) ──── (N) Autor (relacionamento N:N via autor_livro)

Livro (1) ──── (N) Exemplar
```

## 🛠️ Tecnologias Utilizadas

- **Node.js** - Runtime JavaScript
- **AdonisJS v6** - Framework web full-stack
- **TypeScript** - Superset tipado do JavaScript
- **Lucid ORM** - ORM nativo do AdonisJS
- **MySQL** - Banco de dados relacional
- **OAT (Opaque Access Token)** - Autenticação via tokens do próprio AdonisJS

## 📋 Pré-requisitos

Antes de iniciar, certifique-se de ter instalado:

- Node.js (v20 ou superior)
- MySQL (v8 ou superior)
- Git
- Editor de código (VS Code recomendado)

## 📖 Estrutura da Documentação

Esta documentação está dividida nos seguintes módulos:

1. **[00-introducao.md](00-introducao.md)** - Visão geral do projeto (você está aqui)
2. **[01-preparacao-ambiente.md](01-preparacao-ambiente.md)** - Configuração inicial do ambiente
3. **[02-criacao-migrations.md](02-criacao-migrations.md)** - Estrutura do banco de dados
4. **[03-criacao-models.md](03-criacao-models.md)** - Modelos e relacionamentos
5. **[04-criacao-controllers.md](04-criacao-controllers.md)** - Lógica de negócio e CRUD
6. **[05-autenticacao.md](05-autenticacao.md)** - Sistema de autenticação
7. **[06-rotas.md](06-rotas.md)** - Configuração de endpoints
8. **[07-validacao-testes.md](07-validacao-testes.md)** - Testes e validação

## 🚀 Começando

Pronto para começar? Siga para o próximo documento:

➡️ **[01-preparacao-ambiente.md](01-preparacao-ambiente.md)**

---

**Data da Aula:** 13 de janeiro de 2026  
**Versão AdonisJS:** 6.x  
**Autor:** Sistema de Biblioteca - Aula Prática TEP
