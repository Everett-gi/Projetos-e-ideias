# EduPath — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
Plataforma de cursos (**LMS**) com módulos, aulas, **progresso do aluno**, quizzes com
correção e certificado.

**Valor de portfólio:** domínio rico (progresso, avaliação) e boa modelagem de dados.

## Stack
Ruby 3.3 · Rails 8 · Devise · Pundit · Active Storage · PostgreSQL 16 · RSpec · RuboCop ·
Brakeman.

## Modelos
- **Course** (instructor) · **Module** (course, posição) · **Lesson** (module, conteúdo)
- **Enrollment** (course, user) · **Progress** (enrollment, lesson, completed_at)
- **Quiz** · **Question** · **Answer** · **Certificate** (enrollment, emitido_em)

## Funcionalidades principais
- Estrutura de curso (módulos/aulas) · matrícula · acompanhamento de progresso
- Quizzes com correção automática · emissão de certificado · painel do instrutor

## Foco de segurança
Autorização **por matrícula** (só quem está matriculado acessa o conteúdo — protege conteúdo
pago, anti-IDOR); **anti-fraude** em quiz (não expor gabarito); validação de uploads.

## Plano de build
1. Estrutura de curso (módulos/aulas)
2. Matrícula + progresso
3. Quizzes + correção
4. Certificado
5. Painel do instrutor + autenticação/autorização
6. Deploy (ver `../../DEPLOY-GERAL.md` — notas de Rails)

## Como começar
Scaffold: `rails new edupath --database=postgresql`. Modele a autorização por matrícula desde
o início — é o ponto de segurança que protege o conteúdo.
