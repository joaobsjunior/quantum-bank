# Quantum Bank Constitution

## Core Principles

### I. Gateway é a única rota (NÃO NEGOCIÁVEL)

Toda comunicação entre app e backend passa pelo KrakenD. O app mobile nunca
chama o backend diretamente. O gateway não é, porém, a única fronteira de
autorização: os serviços de backend continuam validando tokens OAuth2 por
conta própria.

### II. TLS sem atalhos

Nenhum bypass permissivo de TLS é aceitável — não se aceita "confiar em
qualquer certificado". O ciclo de vida de certificados em runtime pertence aos
componentes de PKI e onboarding; o KrakenD consome os certificados e aplica
mTLS. O provider TLS do Terraform não serve como CA de produção.

### III. Segurança da comunicação antes da tela

A prioridade do v1 é a fundação de segurança: OAuth2, onboarding por OTK,
geração de certificado a partir de CSR, mTLS e PKI. Telas iniciais podem ser
simples, mas precisam exercitar o caminho real app → gateway → backend.

### IV. Pix v1 é simulação explícita

O Pix do v1 é simulado por cenários de sucesso e erro controlados pelo backend.
Nenhuma integração com trilhos reais de liquidação Pix entra no v1, e o backend
registra cada tentativa como simulação.

### V. Diferenças de nuvem ficam explícitas

AWS, GCP e Azure têm caminhos de deploy próprios em Terraform. Não se esconde a
diferença entre nuvens atrás de um módulo único de menor denominador comum.

## Stack e Arquitetura

- Flutter 3.41 no app mobile; APIs TLS de `dart:io` para CA confiável e
  material de certificado cliente.
- KrakenD Community Edition 2.13.x para roteamento, validação de JWT e mTLS.
- Spring Boot Kotlin 4.0.6 no backend, com Spring Security OAuth2 Resource
  Server nos endpoints protegidos.
- Spring Authorization Server ou IdP local substituível como OAuth2/OIDC do v1.
- H2 ou persistência em memória para os dados de simulação do v1.
- OpenXPKI ou PKI open source equivalente quando o KrakenD não cobrir todo o
  ciclo de vida de certificados.
- Docker Compose para validação multi-container local; Terraform para os
  caminhos de deploy em AWS, GCP e Azure.
- O repositório é um superprojeto com submodules por camada: `mobile-app`,
  `backend`, `api-gateway`, `infrastructure` e `pki`.

## Fluxo de Desenvolvimento

- O planejamento e o controle de mudança usam Spec Kit. Antes de implementar,
  leia a spec da feature em `specs/NNN-<feature>/spec.md`.
- Para comportamento novo: `/speckit:specify` cria a spec, `/speckit:plan` o
  plano, `/speckit:tasks` as tarefas e `/speckit:implement` executa.
- Commits de implementação ficam no repositório da camada correspondente quando
  os submodules estão inicializados.

## Governance

Esta constituição prevalece sobre outras práticas do repositório. Toda revisão
precisa verificar conformidade com os princípios acima, e complexidade extra
precisa de justificativa registrada no `plan.md` da feature.

**Version**: 1.0.0 | **Ratified**: 2026-09-04 | **Last Amended**: 2026-09-04
