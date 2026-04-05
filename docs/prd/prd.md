# CazorlaHub Lead Capture — Product Requirements Document (PRD)

## Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2026-04-05 | 1.0 | Versão inicial — MVP n8n + Supabase + UazAPI | Morgan (@pm) |

---

## 1. Goals and Background Context

### Goals

- Validar o fluxo completo (captura → banco → WhatsApp) via n8n antes de integrar ao site oficial
- Capturar leads com formulário n8n (nome, telefone, email)
- Persistir dados no Supabase com rastreamento de status de entrega WhatsApp
- Enviar mensagem personalizada via WhatsApp automaticamente após captura
- Após validação, integrar o fluxo ao site `cazorlahub.com.br`
- Garantir conformidade com LGPD no armazenamento

### Background Context

O projeto começa como um ambiente de teste desacoplado do site `cazorlahub.com.br`. Usando n8n (plataforma de automação no-code/low-code), será possível criar e validar rapidamente o fluxo completo: lead preenche formulário → dado salvo no Supabase → WhatsApp enviado automaticamente via UazAPI. Somente após validação o fluxo será migrado para o site oficial, eliminando riscos de impactar páginas existentes.

---

## 2. Requirements

### Functional Requirements

- FR1: O sistema deve exibir formulário de captura com campos: Nome, Telefone (formato BR), Email (opcional)
- FR2: O sistema deve validar o telefone no formato brasileiro antes de aceitar o lead
- FR3: O sistema deve persistir o lead no Supabase com campos: id, nome, telefone, email, status_whatsapp, created_at, deleted_at
- FR4: O sistema deve impedir cadastro duplicado pelo mesmo número de telefone
- FR5: O sistema deve enviar mensagem WhatsApp personalizada com o nome do lead em até 30 segundos após a captura
- FR6: O sistema deve atualizar o `status_whatsapp` para `enviado` ou `erro` após tentativa de envio
- FR7: O sistema deve retentar o envio do WhatsApp automaticamente em caso de falha (máx. 3 tentativas)
- FR8: O sistema deve exibir mensagem de confirmação após submissão do formulário
- FR9: O Supabase deve permitir visualização e exportação dos leads em CSV

### Non-Functional Requirements

- NFR1: O formulário deve responder ao usuário em menos de 3 segundos após submissão
- NFR2: O envio do WhatsApp não deve bloquear a resposta ao usuário (processamento assíncrono)
- NFR3: Dados dos leads devem ser armazenados com soft-delete para conformidade com LGPD
- NFR4: O ambiente de teste (n8n) deve ser replicável para o site oficial sem reescrita completa
- NFR5: O sistema deve funcionar 24/7 sem intervenção manual

---

## 3. User Interface Design Goals

### Overall UX Vision
Formulário limpo, direto, sem distrações. O usuário preenche em menos de 30 segundos e recebe confirmação imediata. Na fase MVP (n8n), a experiência é funcional mas simples — o refinamento visual acontece na integração com `cazorlahub.com.br`.

### Core Screens and Views
- Formulário de captura (nome, telefone, email)
- Mensagem de confirmação pós-submit ("Em instantes você receberá uma mensagem no WhatsApp!")

### Accessibility
Nenhuma (MVP)

### Target Device and Platforms
Web Responsivo

### Branding
Fase 1 sem branding específico — seguirá identidade do `cazorlahub.com.br` na Fase 2.

---

## 4. Technical Assumptions

### Repository Structure
Monorepo — documentação + configurações de workflow n8n exportadas

### Service Architecture
No-code/Low-code (Fase 1 MVP):
- **n8n** — formulário + automação do fluxo
- **Supabase** — banco PostgreSQL gerenciado
- **UazAPI** — envio de mensagens WhatsApp

### Stack

| Componente | Tecnologia |
|-----------|-----------|
| Automação/Formulário | n8n |
| Banco de Dados | Supabase (PostgreSQL) |
| WhatsApp API | UazAPI |
| Fase 2 (site) | A definir conforme tech do cazorlahub.com.br |

### Testing Requirements
Manual (Fase 1 MVP) — mínimo 3 leads reais testados end-to-end.

### Additional Technical Assumptions
- Credenciais UazAPI armazenadas como variáveis de ambiente no n8n
- Soft-delete via campo `deleted_at` no Supabase para LGPD
- Retry de até 3 tentativas com intervalo de 30 segundos para falhas no WhatsApp

---

## 5. Epic List

| Epic | Título | Objetivo |
|------|--------|----------|
| Epic 1 | Infraestrutura & Ambiente de Teste | Configurar Supabase e validar UazAPI independentemente |
| Epic 2 | Fluxo n8n: Captura → Banco → WhatsApp | Criar formulário e automação completa no n8n |
| Epic 3 | Validação & Ajustes MVP | Testar end-to-end, retry automático e exportação de leads |
| Epic 4 | Migração para cazorlahub.com.br *(fase futura)* | Integrar fluxo validado ao site oficial |

---

## 6. Epic Details

### Epic 1 — Infraestrutura & Ambiente de Teste

Configurar todas as dependências externas antes de construir o fluxo. Ao final deste epic, Supabase e UazAPI estarão prontos e testados de forma independente.

#### Story 1.1 — Criar tabela `leads` no Supabase

> Como desenvolvedor, quero criar a tabela de leads no Supabase, para que os dados capturados sejam persistidos com estrutura adequada.

**Acceptance Criteria:**
1. Tabela `leads` criada com campos: `id` (UUID), `nome` (text), `telefone` (text), `email` (text nullable), `status_whatsapp` (text default `pendente`), `created_at` (timestamptz), `deleted_at` (timestamptz nullable)
2. Index único em `telefone` (onde `deleted_at` é null) para evitar duplicatas
3. Row Level Security (RLS) habilitado
4. Exportação CSV disponível pelo dashboard do Supabase

#### Story 1.2 — Conectar e testar UazAPI

> Como desenvolvedor, quero conectar a UazAPI e enviar uma mensagem de teste, para que o canal WhatsApp esteja validado antes de integrar ao fluxo.

**Acceptance Criteria:**
1. Instância UazAPI configurada e conectada a um número WhatsApp
2. Chamada HTTP manual envia mensagem com sucesso
3. Credenciais (token, instanceId) documentadas como variáveis de ambiente
4. Mensagem de teste recebida no WhatsApp destino

---

### Epic 2 — Fluxo n8n: Captura → Banco → WhatsApp

Criar o fluxo automatizado completo no n8n. Ao final, um lead preenchendo o formulário recebe uma mensagem personalizada no WhatsApp e seus dados estão no Supabase.

#### Story 2.1 — Criar formulário de captura no n8n

> Como visitante, quero preencher um formulário simples, para que meus dados sejam registrados e eu receba contato via WhatsApp.

**Acceptance Criteria:**
1. Formulário n8n com campos: Nome (obrigatório), Telefone BR (obrigatório), Email (opcional)
2. Validação de telefone no formato `(99) 99999-9999`
3. Mensagem de confirmação exibida após submissão
4. URL pública do formulário acessível para testes

#### Story 2.2 — Salvar lead no Supabase via n8n

> Como sistema, quero salvar automaticamente cada lead no Supabase, para que nenhum contato seja perdido.

**Acceptance Criteria:**
1. Workflow n8n insere lead na tabela `leads` após submissão
2. Em caso de telefone duplicado, o workflow trata o erro sem quebrar
3. `status_whatsapp` iniciado como `pendente` na inserção
4. Campos nome, telefone, email e created_at preenchidos corretamente

#### Story 2.3 — Enviar WhatsApp personalizado via UazAPI no n8n

> Como sistema, quero enviar mensagem WhatsApp com o nome do lead automaticamente, para que o contato seja imediato e personalizado.

**Acceptance Criteria:**
1. Workflow n8n chama UazAPI via HTTP Request após salvar o lead
2. Mensagem enviada: `"Olá, {nome}! Recebemos seu contato e em breve nossa equipe entrará em touch. 😊"`
3. `status_whatsapp` atualizado para `enviado` em caso de sucesso
4. `status_whatsapp` atualizado para `erro` em caso de falha

---

### Epic 3 — Validação & Ajustes MVP

Testar o fluxo completo end-to-end, tratar casos de erro e garantir que os dados estão íntegros para uso.

#### Story 3.1 — Retry automático em caso de falha no WhatsApp

> Como sistema, quero retentar o envio do WhatsApp em caso de falha, para que erros temporários não resultem em leads sem contato.

**Acceptance Criteria:**
1. Workflow n8n implementa retry de até 3 tentativas com intervalo de 30 segundos
2. Após 3 falhas, `status_whatsapp` marcado como `erro_definitivo`
3. Fluxo não bloqueia novos leads durante o retry

#### Story 3.2 — Teste end-to-end e exportação de leads

> Como administrador, quero testar o fluxo completo e exportar os leads capturados, para que eu possa validar o MVP antes de migrar para o site oficial.

**Acceptance Criteria:**
1. Fluxo testado com mínimo 3 leads reais (incluindo telefone duplicado)
2. Mensagens WhatsApp recebidas com nome personalizado correto
3. Leads exportados do Supabase em formato CSV
4. Documento de validação criado em `docs/` com resultado dos testes

---

### Epic 4 — Migração para cazorlahub.com.br *(fase futura)*

Integrar o fluxo validado ao site oficial. Escopo a definir após validação do MVP.

#### Story 4.1 — Análise da tech stack do cazorlahub.com.br

> Como desenvolvedor, quero entender a tecnologia do site atual, para planejar a integração sem riscos.

**Acceptance Criteria:**
1. Stack do site documentada (CMS, framework, hospedagem)
2. Estratégia de integração definida (embed, nova rota, redirect)
3. Plano de migração aprovado antes de qualquer alteração no site

---

## 7. Next Steps

### Para @architect (Aria)
Com base neste PRD, defina a arquitetura detalhada do fluxo n8n + Supabase + UazAPI. Valide as decisões técnicas e identifique riscos de integração antes do Epic 1.

### Para @sm (River)
Com base neste PRD, crie as stories detalhadas do Epic 1 seguindo o template AIOX. Comece pela Story 1.1 (tabela Supabase).
