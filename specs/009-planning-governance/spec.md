# Feature Specification: Planning Governance

**Feature Branch**: `009-planning-governance`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `planning-governance`

## Overview

Estabelecer o Spec Kit como fonte única de verdade para planejamento, specs e
controle de mudança no Quantum Bank, garantindo que pessoas contribuidoras e
agentes trabalhem a partir dos artefatos em `.specify/` e `specs/`, sem árvores
de planejamento paralelas.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - Spec Kit é a fonte de verdade do planejamento (Priority: P1)

O projeto SHALL usar o Spec Kit como fonte de verdade para planejamento, specs
e controle de mudança.

**Acceptance Scenarios**:

1. **When** alguém contribuindo ou um agente inicia trabalho planejado, **Then** inspeciona ou cria artefatos em `specs/NNN-<feature>/`, **And** os princípios aplicáveis vêm de `.specify/memory/constitution.md`
2. **When** o contexto de planejamento do projeto é necessário, **Then** `.specify/memory/constitution.md` e `specs/` fornecem o estado ativo

---

### User Story 2 - Não existem artefatos de planejamento paralelos (Priority: P1)

O repositório SHALL NOT manter árvores paralelas de artefatos de planejamento
para o planejamento ativo do projeto.

**Acceptance Scenarios**:

1. **When** os arquivos de planejamento do repositório são inspecionados, **Then** os artefatos ativos estão sob `specs/` e `.specify/`, **And** nenhuma outra árvore de planejamento ativa coexiste
2. **When** a documentação para contribuidores referencia o fluxo de planejamento, **Then** ela aponta para os comandos `/speckit:*` e para os artefatos em `specs/`

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: O projeto SHALL usar o Spec Kit como fonte de verdade para planejamento, specs e controle de mudança.
- **FR-002**: O repositório SHALL NOT manter árvores paralelas de artefatos de planejamento para o planejamento ativo do projeto.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por inspeção do repositório.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
