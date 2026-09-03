# 16. Constantes

> **Princípio central:** nenhuma constante "mágica" vive solta dentro de um componente, hook ou service. Toda constante de um módulo mora num arquivo dedicado de constantes. Ao abrir qualquer arquivo de lógica/visual, não há números, strings de configuração ou listas fixas declaradas no meio do código — elas estão num único lugar, fáceis de achar e de ajustar.

---

## O problema que isto resolve

Constantes espalhadas pelo código são o tipo de dívida técnica que cresce em silêncio: um `PAGE_SIZE = 5` num hook, um `[300, 500]` de delay noutro, um `2` de janela de paginação num componente, um TTL embutido num service. Quando o produto precisa mudar "5 por página" para "10 por página", a mudança vira uma caçada por arquivos. Pior: a mesma constante acaba duplicada com valores diferentes em lugares diferentes.

A solução é a mesma filosofia de uniformidade do resto do guia: **um lugar previsível para cada coisa.** Constantes de um módulo vivem em `constants/`, exatamente como tipos vivem em `models/`.

## Onde vivem (hierarquia, igual à de tipos)

| Nível | Onde fica | Quando usar |
|---|---|---|
| **Global** | `src/constants/` | Constante usada em 2+ módulos diferentes |
| **Módulo** | `src/modules/Module/constants/module.constants.ts` | Constantes do domínio/configuração do módulo |
| **Seção** | `components/Module/Components/Section/constants/section.constants.ts` | Constantes usadas só por uma seção (cresceu o bastante p/ separar) |

> **Regra de decisão:** a constante vive no nível mais baixo onde é usada. Não suba por precaução. Um arquivo só de uma seção pode manter a constante inline **somente** se ela não for "mágica" — ou seja, se o nome da variável já explica tudo e ela é usada num único ponto. Na dúvida, extraia.

Quando o arquivo de constantes do módulo cresce demais, ele vira uma pasta `constants/` com arquivos por tema (`module.cache.constants.ts`, `module.config.constants.ts`, etc.) — sempre reexportados por um `index.ts`.

## O que é constante (vai para `constants/`)

- **Números mágicos:** tamanho de página, janela de paginação, limites, thresholds, durações.
- **Configuração de cache:** TTLs, prefixos de chave legados, nomes de chave fixos.
- **Delays de demo:** intervalos `[min, max]` de `simulateDelay`.
- **Listas/opções fixas:** ordem de filtros, abas, colunas, status disponíveis na UI.
- **Mapas código → valor não-textual:** código de status → cor, código → ícone.
- **Enums-como-const (`as const`)** de códigos de domínio que não são `type` puro.
- **Regex, formatos, separadores** reutilizados.

## O que NÃO vai em `constants/`

- **Texto de UI** (labels, títulos, placeholders, mensagens) → vai para o **i18n** (`10.6-i18n.md`). Texto nunca é constante de código — é conteúdo, e conteúdo é traduzível.
- **Tipos, interfaces, unions** → vão para `models/` (`15-tipos.md`).
- **Dados mockados** → vão para `demo/` (`11-modo-demo.md`).

> Mapa código→cor fica em `constants/` (é valor, não texto). Mapa código→label fica no **i18n** (é texto). Quando ambos existem para o mesmo enum, eles vivem em arquivos diferentes — cor em constants, label em i18n — pela mesma razão que tipo e estilo vivem separados.

## Convenções

- Nomenclatura `SCREAMING_SNAKE_CASE`, prefixada pelo módulo quando exportada para fora do arquivo: `APPROVALS_PAGE_SIZE`, não `PAGE_SIZE`.
- `as const` em objetos e tuplas de configuração, para preservar os literais no tipo.
- Sem lógica: o arquivo de constantes só **declara** valores. Funções (mesmo puras, como construtores de chave de cache) ficam no seu próprio arquivo (`cacheKeys`, `utils`).
- JSDoc no topo descrevendo o escopo, como todo arquivo do projeto.

## Exemplo

```ts
// src/modules/Approvals/constants/approvals.constants.ts
/**
 * @module modules/Approvals/constants/approvals.constants
 * @description Constantes do módulo de Casos/Aprovações: configuração de página,
 * cache, demo, opções de filtro, cores de status e códigos de tipo.
 * Texto de UI NÃO vive aqui — vive no i18n (`../i18n/approvals.*.json`).
 */
import { CACHE_TTL } from '@/services/cache/cacheService'
import type { ApprovalRequestStatus } from '../models/approvals.types'

/** Itens por página da lista de casos. */
export const APPROVALS_PAGE_SIZE = 5

/** Quantos números mostrar de cada lado da página atual na paginação. */
export const APPROVALS_PAGE_WINDOW = 2

/** TTL do cache de listagem (dado volátil — fila de revisão). */
export const APPROVALS_LIST_TTL = CACHE_TTL.SHORT

/** Intervalos de latência simulada no modo demo (ms). */
export const APPROVALS_READ_DELAY: [number, number]  = [300, 500]
export const APPROVALS_WRITE_DELAY: [number, number] = [400, 700]

/** Ordem das pílulas de filtro ('' = Todos). 'APPROVED' não tem pílula. */
export const APPROVALS_FILTERS: readonly (ApprovalRequestStatus | '')[] =
  ['', 'PENDING', 'REJECTED', 'APPLIED', 'CANCELLED'] as const

/** Código de status → cor de fundo do badge (valor, não texto). */
export const APPROVAL_STATUS_COLORS: Record<ApprovalRequestStatus, string> = {
  PENDING:   'rgba(234,179,8,0.12)',
  APPROVED:  'rgba(34,197,94,0.12)',
  REJECTED:  'rgba(239,68,68,0.12)',
  APPLIED:   'rgba(99,102,241,0.12)',
  CANCELLED: 'rgba(100,116,139,0.12)',
}
```

```ts
// ❌ ERRADO — constante mágica solta no hook
const PAGE_SIZE = 5
export function useApprovalsLoader() { /* ... usa PAGE_SIZE ... */ }

// ✅ CERTO — importa do arquivo dedicado
import { APPROVALS_PAGE_SIZE } from '../constants/approvals.constants'
```

## Por quê sempre

O custo de um arquivo de constantes a mais é zero. O custo de uma constante duplicada — ou de uma caçada por dezenas de arquivos quando o valor muda — cresce com o tempo. Centralizar desde o início é o mesmo investimento antecipado que justifica cada outra camada deste guia: previsibilidade hoje, manutenção barata amanhã.
