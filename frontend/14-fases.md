# 14. Fases de Desenvolvimento

> As fases definem **a ordem em que o código é construído**, não o que será construído. Cada fase tem um foco único, proibições explícitas e critérios de saída claros. Avançar para a próxima fase antes de concluir a atual é a principal causa de retrabalho.

---

## Visão geral

| Fase | Nome | Foco | Padrões do manual? |
|------|------|------|--------------------|
| 1 | Protótipo visual | O que vai ter na tela | Não — arquivo único |
| 2 | Estrutura de módulo | Como vai ser organizado | Parcial — composition |
| 3 | Containers | Separação lógica/visual | Sim — container/presenter, guards |
| 4 | Serviços e hooks | Como os dados chegam | Sim — tudo |

---

## Fase 1 — Protótipo Visual

### Objetivo

Definir **tudo que vai existir na tela** — cada página, cada modal, cada seção — sem se preocupar com lógica, arquitetura ou padrões. O resultado é uma representação fiel do produto, navegável, visual e validável com qualquer pessoa (produto, design, cliente).

### Regras

- **Um arquivo por página.** Toda a página em um único `.tsx`, incluindo modais e seções.
- **Dados como constantes inline.** Arrays e objetos fake declarados no topo do arquivo — sem services, sem fetch, sem mocks externos.
- **Estado mínimo e justificado.** Apenas o `useState` necessário para testar a UI: abrir/fechar modal, trocar de aba, selecionar filtro. Sem `useEffect`, sem lógica de negócio.
- **Sem types complexos.** Tipos simples inline se necessário; nada em `/models`.
- **Sem imports de outros módulos.** Componentes globais (`Modal`, `Pagination`, `SearchInput`) são permitidos — tudo do módulo fica inline.
- **Não aplicar os padrões do manual** (composition pattern, container/presenter, facade, strategy, etc.). Essa fase é propositalmente simples para permitir iteração rápida.

### Por que não aplicar os padrões?

A Fase 1 é volátil por natureza. A estrutura visual muda com frequência até ser validada. Aplicar arquitetura antes da validação visual significa refatorar código bem estruturado toda vez que o layout muda — desperdício garantido. A estrutura vem depois, na Fase 2.

### Proibições explícitas

- `useEffect` de qualquer tipo
- Custom hooks
- Services, repositories, adapters
- Imports de `/modules` ou de outros arquivos de página
- Tipos em `/models`

### Critério de saída

> Todas as páginas e modais foram revisados e aprovados visualmente. Não há pendências de layout conhecidas. Nenhuma seção ou funcionalidade visual está faltando.

---

## Fase 2 — Estrutura de Módulo

### Objetivo

Transformar cada arquivo único da Fase 1 em um **módulo estruturado** — sem adicionar lógica. O código visual já validado é reorganizado em componentes atômicos, compostos via Composition Pattern. Os diretórios do módulo são criados, mas a maioria ainda estará vazia.

### O que acontece

1. **Identificar as seções** da página (header, filtros, tabela, cards, modais, etc.).
2. **Criar o diretório do módulo** em `/src/modules/NomeModulo/` com a estrutura completa de pastas — mesmo as que ficam vazias por ora (`hooks/`, `services/`, `demo/`, `docs/`).
3. **Extrair cada seção** para seu próprio arquivo em `/components/NomeSecao/`.
4. **Decompor recursivamente** cada seção até que seus sub-componentes sejam atômicos — ou seja, não possam ser divididos sem perder identidade ou responsabilidade única.
5. **Aplicar Composition Pattern** em cada componente: sub-componentes acessíveis via `Componente.SubComponente`.
6. **Mover os tipos** para `/models` do módulo.
7. **Extrair todo texto de UI** para `i18n/module.{pt,en}.json` e consumir via `useT` (ver `10.6`).
8. **Extrair toda constante** (números mágicos, opções fixas) para `constants/module.constants.ts` (ver `16`).

### "Atômico" — o que significa

Um sub-componente é atômico quando:
- Tem **uma única responsabilidade visual** (exibir um badge, renderizar uma linha de tabela, mostrar um título com ícone)
- **Não pode ser dividido** sem que o resultado seja fragmentos sem identidade própria
- **Aceita apenas props simples** — dados tipados, sem lógica interna

### O que NÃO fazer nesta fase

- Não conectar nenhum componente a hooks ou services
- Não criar `useModuleState`, `useModuleApi` nem nenhum service
- Não implementar lógica de negócio de nenhuma forma
- Containers ainda não existem — os componentes ainda recebem dados fake diretamente

### Estrutura esperada ao final da Fase 2

```
src/modules/Inventory/
├── Inventory.tsx                  ← monta as seções (ainda com dados fake)
├── components/
│   ├── InventoryHeader/
│   │   ├── InventoryHeaderComponent.tsx   ← composition pattern aplicado
│   │   └── index.ts
│   ├── InventoryTable/
│   │   ├── InventoryTableComponent.tsx
│   │   └── index.ts
│   └── ...
├── demo/                          ← vazio
├── docs/
│   └── INVENTORY.md               ← TODOs, pendências, decisões
├── hooks/                         ← vazio
├── models/
│   └── inventory.types.ts         ← tipos extraídos dos componentes
├── services/                      ← vazio
└── utils/                         ← vazio (se necessário)
```

### Critério de saída

> Nenhuma seção da página está inline em `Módulo.tsx`. Cada componente tem responsabilidade única e não pode ser decomposto mais. Composition Pattern aplicado em todos os componentes. Todos os tipos estão em `/models`. A página ainda funciona visualmente igual à Fase 1.

---

## Fase 3 — Containers

### Objetivo

Separar a **lógica de orquestração** da **renderização visual**. Nesta fase, os componentes visuais param de receber dados fake diretamente e passam a ser alimentados por containers — que por ora ainda usam os mesmos dados fake, mas já na estrutura definitiva.

### O que acontece

1. **Criar um `Container` para cada seção** em `/components/NomeSecao/NomeSecaoContainer.tsx`.
2. O container **importa o componente visual** e fornece dados e callbacks via props — sem lógica de negócio ainda.
3. **Criar Guard components** para toda condicional de renderização (`*Guard.tsx`). Nenhuma condicional no container ou no componente visual.
4. O arquivo raiz `Módulo.tsx` passa a **importar apenas containers** — não componentes visuais diretamente.

### Regras

- Componentes visuais nunca importam hooks, services ou qualquer fonte de dados
- Containers nunca contêm JSX de apresentação além do necessário para orquestrar
- Toda lógica condicional de renderização fica em Guard components
- Containers ainda podem usar dados fake — a conexão real vem na Fase 4

### Estrutura esperada ao final da Fase 3

```
InventoryHeader/
├── InventoryHeaderComponent.tsx      ← visual puro, sem hooks
├── InventoryHeaderContainer.tsx      ← orquestra, passa props
├── InventoryHeaderCriticalBadgeGuard.tsx ← decide se renderiza o badge
└── index.ts
```

### Critério de saída

> Nenhum componente visual consome hooks ou tem lógica interna. Todos os Guard components estão criados. O arquivo raiz do módulo importa apenas containers. A página ainda funciona visualmente igual à Fase 2.

---

## Fase 4 — Serviços e Hooks

### Objetivo

Construir toda a camada de dados: hooks, services, repository, adapter, cache e strategy. Ao final desta fase, o módulo está completamente funcional — conectado à API real, com demo mode, cache e rollback implementados.

### Ordem de construção

1. **Models** — revisar e completar os tipos em `/models` para refletir o contrato real da API
2. **Repository** — único ponto de acesso HTTP do módulo (`inventoryRepository.ts` + adapter)
3. **Cache** — não criar serviço de cache por módulo. O `cacheService` global (`src/services/cache/`) é importado diretamente pelo service-facade. Ver `10.1-cache.md`.
4. **Strategy** — isolamento do ponto de decisão demo/real (`getStrategy()`)
5. **Demo factory** — mocks dinâmicos em `/demo/factory.ts`, implementando a mesma interface do service real
6. **Service-facade** — `inventoryService.ts` orquestra repository, cache e strategy; único ponto público da camada de services
7. **Hooks de estado** — `useInventoryState.ts` e `useInventoryActions.ts`
8. **Hook-facade** — `useInventoryApi.ts` agrega os hooks; único ponto público da camada de hooks
9. **Containers** — substituir os dados fake pelos retornos do hook-api

### Regras

Todas as regras do manual se aplicam integralmente a partir desta fase. Ver especialmente:
- `05-design-patterns.md` — Strategy, Repository, Adapter, Facade
- `08-hooks.md` — separação state/actions, useCallback/useMemo
- `09-services.md` — responsabilidades de cada service
- `10.1-cache.md` — rollback de cache em operações otimistas
- `11-modo-demo.md` — demo mode via Strategy, sem condicionais espalhadas

### Critério de saída

> O módulo funciona com dados reais da API. Demo mode funcional sem nenhum fetch real. Cache implementado com rollback em operações de escrita. Checklist `13-checklist.md` aprovado integralmente.

---

## Transição entre fases

Nunca iniciar uma nova fase em um módulo enquanto a fase atual não tiver sido concluída — mesmo que outro módulo esteja mais avançado. A consistência dentro de cada módulo é mais importante do que avançar rapidamente.

A ordem de módulos deve seguir prioridade de produto: implementar completamente um módulo antes de avançar para o próximo.
