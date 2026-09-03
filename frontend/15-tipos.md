# 15. Organização de Tipos

> **Princípio central:** um tipo vive no nível mais baixo onde é compartilhado. Não existe "subir por precaução" — se um tipo é usado em um único arquivo, ele fica naquele arquivo.

---

## Os quatro níveis

| Nível | Onde fica | Quando usar |
|---|---|---|
| **Global** | `src/models/` | Tipo usado em 2+ módulos diferentes |
| **Módulo** | `src/modules/Module/models/module.types.ts` | Entidades do domínio do módulo (dados, enums, unions) |
| **Seção** | `components/Section/models/section.types.ts` | Props da raiz de uma seção (usadas pelo Container na Fase 3) |
| **Inline** | Dentro do próprio arquivo `.tsx` | Props de sub-componentes usadas em um único arquivo |

---

## Regra de decisão

```
Usado em 2+ módulos?         → src/models/
Usado em todo o módulo?      → Module/models/module.types.ts
Props de raiz de seção?      → Section/models/section.types.ts
Usado em 1 arquivo só?       → interface Props { } inline no .tsx
```

> **Nunca suba um tipo antes de precisar.** Entidades do domínio chegam ao `module.types.ts` porque o dado atravessa múltiplas seções — não porque "talvez" seja usado em outro lugar.

---

## Estrutura de um módulo com tipos organizados

```
src/modules/Scheduling/
├── models/
│   └── scheduling.types.ts        ← entidades do domínio + barrel de re-exports
├── components/
│   ├── SchedulingChart/
│   │   ├── models/
│   │   │   └── schedulingChart.types.ts   ← SchedulingChartProps
│   │   ├── SchedulingChartBar/            ← sub-componente com diretório
│   │   │   ├── SchedulingChartBarComponent.tsx  ← interface Props inline
│   │   │   └── index.ts
│   │   ├── SchedulingChartSummary.tsx     ← interface Props inline
│   │   ├── SchedulingChartBars.tsx        ← interface Props inline
│   │   └── SchedulingChartComponent.tsx
│   └── SchedulingAppointmentList/
│       ├── models/
│       │   └── schedulingAppointmentList.types.ts  ← SchedulingAppointmentListProps
│       ├── SchedulingAppointmentListHeader.tsx      ← interface Props inline
│       ├── SchedulingAppointmentListEmpty.tsx       ← sem props
│       └── SchedulingAppointmentCard/               ← sub-componente com diretório
│           ├── SchedulingAppointmentCard.tsx        ← interface local (usada apenas aqui)
│           ├── SchedulingAppointmentCardAvatar.tsx  ← interface Props inline
│           └── index.ts
```

---

## O arquivo `module.types.ts` como barrel

O arquivo de tipos do módulo tem **duas responsabilidades** claramente separadas:

```ts
// scheduling.types.ts

// ── 1. Entidades do domínio ──────────────────────────────────────────────────
// Dados compartilhados por múltiplas seções do módulo.
export type AppointmentStatus = 'confirmed' | 'pending' | ...
export interface SchedulingAppointment { id: number; patient: string; ... }
export interface DayStatus { confirmed: number; pending: number; ... }

// ── 2. Barrel de props de seção ──────────────────────────────────────────────
// Props das raízes de seção vivem em seus próprios models/,
// mas são re-exportadas aqui para acesso conveniente (root do módulo, containers).
export type { SchedulingChartProps }    from '../components/SchedulingChart/models/schedulingChart.types'
export type { SchedulingFiltersProps }  from '../components/SchedulingFilters/models/schedulingFilters.types'
```

---

## Props de seção vs. entidades

| Tipo | Categoria | Localização |
|---|---|---|
| `SchedulingAppointment` | Entidade (dado do domínio) | `module.types.ts` |
| `AppointmentStatus` | Union/enum do domínio | `module.types.ts` |
| `SchedulingChartProps` | Props da raiz de seção | `SchedulingChart/models/` |
| `SchedulingChartBarProps` | Props de sub-componente (1 arquivo) | inline no `.tsx` |

---

## Props de seção importam entidades do módulo

Os arquivos de tipos de cada seção importam as entidades que precisam do arquivo de tipos do módulo:

```ts
// components/SchedulingChart/models/schedulingChart.types.ts
import type { SchedulingChartEntry } from '../../../models/scheduling.types'

export interface SchedulingChartProps {
  chartData: SchedulingChartEntry[]
}
```

> O sentido da dependência é sempre **de baixo para cima**: seção → módulo → global. Nunca o contrário.

---

## Candidatos a promoção (`@candidate-global`)

Quando um sub-componente é identificado como repetição de padrão em outros módulos, marcar com `@candidate-global` no JSDoc:

```tsx
/**
 * @candidate-global Avatar com iniciais — mesmo padrão em Scheduling, Approvals e SchedulingDay.
 *   Promover para /src/components/ quando confirmado em 3+ módulos distintos.
 */
```

Para listar todos os candidatos do projeto:
```bash
grep -r "@candidate-global" src/
```
