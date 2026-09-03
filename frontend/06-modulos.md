# 6. Organização por Módulos

> **Previsibilidade estrutural:** todo módulo tem exatamente a mesma estrutura de pastas. Um desenvolvedor que nunca viu o módulo de Estoque sabe onde encontrar os hooks, os services, os componentes e os mocks — porque é idêntico ao módulo de Agendamentos, que é idêntico ao módulo de Cadastros. Essa uniformidade não é coincidência, é obrigação.

## Definição de módulo

**Cada página da aplicação é um módulo.** Módulos ficam em `/src/modules`. Cada sessão visual de uma página é um **componente** desse módulo. Um módulo é uma unidade fechada — não importa código de outro módulo, não expõe seus internos, e se comunica com o mundo externo exclusivamente via EventBus.

```
src/modules/Inventory/
├── Inventory.tsx              ← Wrapper do módulo (orquestra os containers)
├── components/
│   ├── InventoryHeader/
│   │   ├── InventoryHeaderComponent.tsx
│   │   ├── InventoryHeaderContainer.tsx
│   │   └── index.ts
│   ├── InventoryAlerts/
│   ├── InventoryFilters/
│   ├── InventoryTable/
│   └── InventoryPagination/
├── constants/
│   └── inventory.constants.ts    ← Constantes do módulo (ver 16)
├── demo/
│   ├── products.mock.ts
│   ├── summary.mock.ts
│   └── factory.ts             ← Factory para gerar mocks dinâmicos
├── docs/
│   └── INVENTORY.md           ← TODOs, bugs, próximas features do módulo
├── hooks/
│   ├── useInventoryState.ts
│   ├── useInventoryActions.ts
│   └── useInventoryApi.ts     ← Hook-facade (único ponto público)
├── i18n/
│   ├── inventory.pt.json      ← Textos PT do módulo (ver 10.6)
│   └── inventory.en.json      ← Textos EN do módulo
├── models/
│   └── inventory.types.ts
├── services/
│   ├── inventory-repository/
│   │   ├── inventoryRepository.ts
│   │   ├── inventoryRepository.adapter.ts
│   │   └── models/
│   │       └── inventoryRepository.types.ts
│   └── inventoryService.ts    ← Service-facade (único ponto público); importa cacheService global
└── utils/
    └── inventoryFilters.util.ts
```

## Hierarquia de tipos

```
/src/models/                              ← Tipos usados em 2+ módulos distintos
/src/modules/Module/models/               ← Entidades do domínio do módulo (dados, enums, unions)
/src/modules/Module/components/Section/models/ ← Props da raiz de uma seção
/src/modules/Module/services/svc/models/  ← Tipos exclusivos de um service
interface Props { } inline no .tsx        ← Props de sub-componentes (1 arquivo só)
```

> **Regra principal:** um tipo vive no nível mais baixo onde é compartilhado. Não suba por precaução.

| Nível | Exemplo | Localização |
|---|---|---|
| Global | `ApiResponse<T>`, `Result<T>` | `src/models/` |
| Módulo — domínio | `AppointmentStatus`, `SchedulingAppointment` | `Module/models/module.types.ts` |
| Seção — props | `SchedulingChartProps` | `Section/models/section.types.ts` |
| Sub-componente | `SchedulingChartBarProps` | `interface Props` inline no `.tsx` |

O arquivo `module.types.ts` tem **duas responsabilidades**:
1. Definir as entidades do domínio do módulo (dados compartilhados por múltiplas seções)
2. Re-exportar as props de seção (`export type { XProps } from '../components/X/models/x.types'`) para acesso conveniente

> Regra de promoção: se um tipo de service passou a ser usado em outro service do mesmo módulo, suba para `/models` do módulo. Se passou a ser usado em outros módulos, suba para `/src/models`.

> Ver detalhes completos em `manual/frontend/15-tipos.md`.

## Tipos globais base (`/src/models`)

Estes tipos devem existir antes de qualquer módulo ser implementado. São a fundação de toda a camada de dados da aplicação.

```ts
// src/models/apiResponse.types.ts
/**
 * @description Estrutura padrão de toda resposta da API backend.
 * O httpClient converte ApiResponse<T> para Result<T> de forma transparente.
 * T representa o tipo do campo data (o payload útil da resposta).
 */
export interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
  timestamp: string;
  severity: 'info' | 'error' | 'warning' | 'success';
}
```

```ts
// src/models/result.types.ts
/**
 * @description Tipo interno de resultado para operações assíncronas.
 * Retornado por services, repositories e hooks — nunca diretamente pela API.
 * A conversão de ApiResponse<T> → Result<T> acontece dentro do httpClient.
 */
export type Result<T> =
  | { ok: true; data: T }
  | { ok: false; error: string; status: number };
```

```ts
// src/models/pagination.types.ts
/**
 * @description Estado de paginação reutilizado por todos os módulos com listas paginadas.
 */
export interface PaginationState {
  page: number;
  pageSize: number;
  total: number;
  totalPages: number;
  setPage: (page: number) => void;
  setPageSize: (size: number) => void;
  setTotal: (total: number) => void;
  reset: () => void;
}
```
