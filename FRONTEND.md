# FRONTEND DEVELOPMENT GUIDE — Índice

> **Para o Agente:** Este é o índice do guia de desenvolvimento frontend. Identifique a seção relevante para sua tarefa e leia o arquivo correspondente. Leia sempre as seções **0 — Filosofia** e **13 — Checklist** em toda nova tarefa.

---

## Seções

| # | Arquivo | Quando ler |
|---|---|---|
| 0 | [Filosofia e Visão](frontend/00-filosofia.md) | **Sempre — leia antes de qualquer tarefa** |
| 1 | [Stack & Tecnologias](frontend/01-stack.md) | Ao iniciar o projeto ou adicionar dependências |
| 2 | [Ambientes](frontend/02-ambientes.md) | Ao configurar `.env` ou variáveis de ambiente |
| 3 | [Estrutura de Pastas](frontend/03-estrutura-pastas.md) | Ao criar arquivos ou módulos novos |
| 4 | [Regras de Estilo](frontend/04-estilos.md) | Ao trabalhar em qualquer componente visual |
| 5 | [Design Patterns](frontend/05-design-patterns.md) | Ao criar componentes, hooks ou services |
| 6 | [Organização por Módulos](frontend/06-modulos.md) | Ao criar ou modificar um módulo de página |
| 7 | [Camada de Componentes](frontend/07-componentes.md) | Ao criar componentes globais ou de módulo |
| 8 | [Camada de Hooks](frontend/08-hooks.md) | Ao criar ou modificar hooks |
| 9 | [Camada de Services](frontend/09-services.md) | Ao criar ou modificar services |
| 10.1 | [Cache Client](frontend/10.1-cache.md) | Ao implementar cache em qualquer módulo |
| 10.2 | [HTTP Client](frontend/10.2-http-client.md) | Ao implementar ou ajustar chamadas HTTP |
| 10.3 | [Event Bus](frontend/10.3-event-bus.md) | Ao implementar comunicação entre módulos |
| 10.4 | [WebSocket Client](frontend/10.4-websocket.md) | Ao implementar features em tempo real |
| 10.5 | [Contexto Global](frontend/10.5-contexto-global.md) | Ao trabalhar com Auth, tema ou dados globais |
| 10.6 | [Internacionalização (i18n)](frontend/10.6-i18n.md) | Ao adicionar/alterar qualquer texto de UI ou suportar idiomas |
| 10.7 | [Camada LIVE (dados vivos)](frontend/10.7-live-data.md) | Ao implementar QUALQUER dado reativo/cacheado (o motor da Filosofia §0) |
| 11 | [Modo Demonstração](frontend/11-modo-demo.md) | Ao criar mocks ou o modo demo de um módulo |
| 12 | [Documentação & Commits](frontend/12-documentacao.md) | Antes de commitar qualquer mudança |
| 13 | [Checklist de Qualidade](frontend/13-checklist.md) | **Sempre — antes de qualquer commit** |
| 14 | [Fases de Desenvolvimento](frontend/14-fases.md) | Ao iniciar ou avançar uma fase em qualquer módulo |
| 15 | [Organização de Tipos](frontend/15-tipos.md) | Ao criar tipos, props ou reorganizar models de qualquer módulo |
| 16 | [Constantes](frontend/16-constantes.md) | Ao criar qualquer constante, número mágico ou opção fixa |

---

## Guia rápido por tipo de tarefa

**Criar um novo módulo (página):**
Ler `00`, `03`, `05`, `06` — depois `07`, `08`, `09` conforme for implementando cada camada.

**Criar ou modificar um componente:**
Ler `00`, `04`, `05`, `07` — e `10.6` (todo texto via `useT`) + `16` (toda constante em `constants/`).

**Adicionar ou alterar texto de UI / suportar idiomas:**
Ler `10.6`.

**Criar uma constante (número mágico, opção fixa, TTL):**
Ler `16`.

**Implementar cache em um módulo:**
Ler `00`, `09`, `10.1`, `10.7`.

**Implementar dado reativo (histórico vivo, lista sincronizada, projeção):**
Ler `10.7` — declarar um recurso vivo; NÃO religar cache+eventos à mão.

**Comunicação entre módulos:**
Ler `10.3`.

**Adicionar eventos em tempo real:**
Ler `10.3`, `10.4`.

**Implementar modo demonstração de um módulo:**
Ler `11`, `09`.

**Antes de qualquer commit:**
Ler `12`, `13`.
