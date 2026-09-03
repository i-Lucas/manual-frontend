# 5. Padrões Arquiteturais (Design Patterns)

> **Estes padrões não são opcionais e não são aplicados "quando fizer sentido". São aplicados sempre, em todo arquivo, em todo módulo, sem exceção.**

Os padrões descritos nesta seção funcionam como um **sistema**: cada um resolve um problema específico de uma camada específica, e juntos eles garantem que toda a aplicação seja uniforme, previsível e navegável. Um desenvolvedor que entende o sistema sabe exatamente o que vai encontrar em qualquer arquivo antes mesmo de abri-lo.

- **Composition Pattern** resolve o problema de variações em componentes visuais — sem condicionais, sem props booleanas que mudam comportamento.
- **Container/Presenter** resolve o problema de mistura entre lógica e renderização — um arquivo nunca faz as duas coisas.
- **Guard Components** resolvem o problema de condicionais de renderização — a decisão de exibir ou não algo pertence a um componente dedicado, nunca ao Container nem ao Presenter.
- **Facade Pattern** resolve o problema de acoplamento entre camadas — hooks falam com um único service-api, containers falam com um único hook-api.
- **Observer Pattern (EventBus)** resolve o problema de comunicação entre módulos — módulos nunca se importam diretamente.
- **Strategy Pattern** resolve o problema de comportamento variável por ambiente ou modo — a lógica condicional fica isolada em um único ponto de decisão.
- **Repository Pattern** resolve o problema de chamadas HTTP espalhadas — todo acesso à API passa por um único arquivo de repository.
- **Adapter Pattern** resolve o problema de acoplamento ao contrato da API — o modelo interno nunca depende do formato que o backend retorna.

> Aplicar um padrão somente quando o problema já ficou óbvio é tarde demais. O custo de refatorar código que cresceu sem estrutura é sempre maior do que o custo de aplicar o padrão desde o início. **A consistência do projeto inteiro vale mais do que a conveniência de um arquivo específico.**

## Composition Pattern

Todos os componentes devem adotar este padrão. Elimina condicionais de renderização, torna o componente extensível sem modificação, e encapsula cada variação em um sub-componente com responsabilidade própria.

> **Por quê sempre?** Componentes começam simples e crescem. Quando crescem sem Composition Pattern, acumulam props booleanas (`isLoading`, `hasError`, `isDisabled`, `showBadge`...) e condicionais dentro do JSX que tornam o componente ilegível e frágil. O Composition Pattern elimina esse acúmulo na raiz — cada variação é um sub-componente independente, sem interferência nos outros. - Cada componente é composto por sub-componentes com responsabilidades únicas -> são pequenos e previsíveis.

> **Obs:** Conceito de Componente Atômico Global vs Componente Atômico Local: cada módulo pode ter seus próprios componentes atômicos, que não são exportados para fora do módulo. Componentes atômicos globais são aqueles que são exportados para uso em outros módulos. - Componentes atômicos globais ficam em /src/components e devem ser reutilizados por todos os módulos. Exemplo: Button, Input, Modal, Table, etc. - Componentes atômicos locais ficam dentro do módulo e são usados apenas naquele módulo. Exemplo: InventoryHeader, ProductCard, etc.


```tsx
// src/components/Button/Button.tsx
/**
 * @module Button
 * @description Componente atômico (global) de botão com composition pattern.
 * Sub-componentes: Button.Icon, Button.Label, Button.Spinner
 */
import { memo } from 'react';
import type { ButtonProps } from './models/button.types';
import { ButtonIcon } from './ButtonIcon';
import { ButtonLabel } from './ButtonLabel';
import { ButtonSpinner } from './ButtonSpinner';

const ButtonRoot = memo(function ButtonRoot({ children, onClick, variant = 'primary', disabled = false, className = '' }: ButtonProps) {
  return (
    <button
      type="button"
      className={`btn btn-${variant} d-inline-flex align-items-center gap-2 ${className}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
});

// Preset opinativo — sub-componentes acessíveis via Button.Icon, Button.Label, etc.
// Atribuição direta: TypeScript infere as propriedades adicionadas à função.
function Button(props: ButtonProps) {
  return <ButtonRoot {...props} />;
}

Button.Icon    = ButtonIcon;
Button.Label   = ButtonLabel;
Button.Spinner = ButtonSpinner;

export { Button };
```

```tsx
// src/components/Button/ButtonIcon.tsx
import { memo } from 'react';
import type { ButtonIconProps } from './models/button.types';

/**
 * Tamanhos mapeados para classes Bootstrap de font-size.
 * Nunca usar style inline — tamanho controlado via variante tipada.
 */
const SIZE_CLASS: Record<NonNullable<ButtonIconProps['size']>, string> = {
  sm: 'fs-6',
  md: 'fs-5',
  lg: 'fs-4',
};

export const ButtonIcon = memo(function ButtonIcon({ icon, size = 'md' }: ButtonIconProps) {
  return <i className={`bi bi-${icon} ${SIZE_CLASS[size]}`} />;
});
```

```tsx
// Uso no módulo
<Button onClick={handleSave} variant="success">
  <Button.Spinner /> {/* exibido quando isLoading */}
  <Button.Icon icon="floppy-disk" />
  <Button.Label>Salvar</Button.Label>
</Button>

{/* Variação mínima */}
<Button onClick={handleCancel} variant="outline-secondary">
  <Button.Label>Cancelar</Button.Label>
</Button>
```

## Container / Presenter Pattern

Separação obrigatória entre **lógica** e **renderização**. Nenhum componente que retorna TSX deve consumir hooks, services ou possuir lógica de negócio diretamente.

| Arquivo | Responsabilidade |
|---|---|
| `SectionComponent.tsx` | Somente props + renderização visual |
| `SectionContainer.tsx` | Consome hook-api, orquestra dados, renderiza o Component |

```tsx
// ✅ Componente puramente visual — com Composition Pattern
// src/modules/Inventory/components/InventoryHeader/InventoryHeaderComponent.tsx
import { memo } from 'react';
import type {
  InventoryHeaderProps,
  InventoryHeaderCriticalBadgeProps,
  InventoryHeaderInfoProps,
  InventoryHeaderActionsProps,
} from '../../models/inventory.types';
import { Button } from '@/components/Button';

/**
 * Sub-componente: informações do lado esquerdo do header.
 * Recebe apenas o total de produtos — sem lógica condicional.
 */
const InventoryHeaderInfo = memo(function InventoryHeaderInfo({ totalProducts, children }: InventoryHeaderInfoProps) {
  return (
    <div className="d-flex align-items-center gap-2">
      <div>
        <h1 className="h4 mb-0 fw-bold">Estoque</h1>
        <small className="text-secondary">{totalProducts} produtos cadastrados</small>
      </div>
      {children}
    </div>
  );
});

/**
 * Sub-componente: badge de produtos em nível crítico.
 * Renderizado pelo Container somente quando criticalCount > 0.
 * O componente visual nunca decide se renderiza ou não — isso é responsabilidade do Container.
 */
const InventoryHeaderCriticalBadge = memo(function InventoryHeaderCriticalBadge({ count }: InventoryHeaderCriticalBadgeProps) {
  return <span className="badge bg-danger">{count} críticos</span>;
});

/**
 * Sub-componente: botões de ação do lado direito do header.
 */
const InventoryHeaderActions = memo(function InventoryHeaderActions({ onAddProduct, onManageStock }: InventoryHeaderActionsProps) {
  return (
    <div className="d-flex gap-2">
      <Button onClick={onAddProduct} variant="primary">
        <Button.Icon icon="plus-lg" />
        <Button.Label>Novo Produto</Button.Label>
      </Button>
      <Button onClick={onManageStock} variant="outline-secondary">
        <Button.Label>Gerenciar</Button.Label>
      </Button>
    </div>
  );
});

/**
 * Root do componente de header do módulo de estoque.
 * Sem condicionais. Sem lógica. Apenas estrutura e composição.
 */
const InventoryHeaderRoot = memo(function InventoryHeaderComponent({ children }: InventoryHeaderProps) {
  return (
    <header className="d-flex justify-content-between align-items-center py-3 border-bottom">
      {children}
    </header>
  );
});

function InventoryHeaderComponent(props: InventoryHeaderProps) {
  return <InventoryHeaderRoot {...props} />;
}

InventoryHeaderComponent.Info          = InventoryHeaderInfo;
InventoryHeaderComponent.CriticalBadge = InventoryHeaderCriticalBadge;
InventoryHeaderComponent.Actions       = InventoryHeaderActions;

export { InventoryHeaderComponent };
```

```tsx
// Componente guardião — encapsula a lógica condicional de exibição do badge.
// Recebe o dado bruto, decide internamente se renderiza ou retorna null.
// O Container apenas o instancia — sem nenhuma condicional.
// src/modules/Inventory/components/InventoryHeader/InventoryHeaderCriticalBadgeGuard.tsx
import { memo } from 'react';
import { InventoryHeaderComponent } from './InventoryHeaderComponent';

interface Props { count: number; }

export const InventoryHeaderCriticalBadgeGuard = memo(function InventoryHeaderCriticalBadgeGuard({ count }: Props) {
  if (count === 0) return null;
  return <InventoryHeaderComponent.CriticalBadge count={count} />;
});
```

```tsx
// ✅ Container — sem nenhuma condicional. Apenas composição e dados.
// src/modules/Inventory/components/InventoryHeader/InventoryHeaderContainer.tsx
import { useCallback } from 'react';
import { useInventoryApi } from '../../hooks/useInventoryApi';
import { InventoryHeaderComponent } from './InventoryHeaderComponent';
import { InventoryHeaderCriticalBadgeGuard } from './InventoryHeaderCriticalBadgeGuard';

export function InventoryHeaderContainer() {
  const { summary, openAddModal, openManageModal } = useInventoryApi();

  const handleAddProduct = useCallback(() => openAddModal(), [openAddModal]);

  return (
    <InventoryHeaderComponent>
      <InventoryHeaderComponent.Info totalProducts={summary.total}>
        <InventoryHeaderCriticalBadgeGuard count={summary.critical} />
      </InventoryHeaderComponent.Info>
      <InventoryHeaderComponent.Actions
        onAddProduct={handleAddProduct}
        onManageStock={openManageModal}
      />
    </InventoryHeaderComponent>
  );
}
```

> **Regra fundamental do Composition Pattern:** nem o componente visual nem o container contêm condicionais de renderização. Condicionais pertencem a **componentes guardiões** dedicados (`*Guard`), que recebem o dado bruto, decidem internamente se renderizam ou retornam `null`, e são simplesmente instanciados por quem precisar — sem nenhuma lógica no ponto de uso.

## Facade Pattern

Agrega múltiplas partes internas e expõe uma interface única. Aplicado em duas camadas obrigatórias:

- **`useModuleApi`** — hook-facade: agrega todos os sub-hooks do módulo. É o **único** arquivo de hook que o Container importa. Containers nunca importam sub-hooks diretamente.
- **`moduleService`** — service-facade: agrega todos os sub-services do módulo. É o **único** arquivo de service que o hook-api importa. Hooks nunca importam sub-services (repository, cache, validator) diretamente.

> **Por quê?** Sem o facade, qualquer mudança interna (renomear um sub-hook, dividir um service em dois, adicionar um cache service) exige alterar todos os arquivos que os importavam diretamente. Com o facade, a mudança fica encapsulada — nenhum arquivo externo precisa ser tocado.

## Observer Pattern (EventBus)

Comunicação desacoplada entre módulos via `EventBus` global (detalhado em `10.3-event-bus.md`). **Módulos nunca se importam diretamente** — nenhum módulo importa código de outro módulo. Toda comunicação inter-modular passa obrigatoriamente pelo EventBus.

> **Por quê?** Importação direta entre módulos cria acoplamento: mudar o módulo A quebra o módulo B. Com o EventBus, cada módulo é completamente independente — pode ser reescrito, renomeado ou removido sem afetar nenhum outro.

## Strategy Pattern

Comportamento diferente de acordo com o ambiente ou modo. O ponto de decisão é **único e isolado** — nenhuma condicional `if (isDemoMode)` espalhada pelo código. A troca de strategy acontece em um único lugar, e o restante do sistema não sabe qual strategy está ativa.

Aplicado em:
- Services: alternar entre dados reais (API) e dados mockados (demo)
- HTTP Client: alternar entre fetch real e mock de demo
- WebSocket: alternar entre conexão real e simulação de demo

```ts
// A decisão de qual strategy usar fica SOMENTE aqui — em nenhum outro lugar
function getStrategy() {
  if (isDemoMode()) return createDemoInventoryStrategy();
  return {
    fetchProducts: inventoryRepository.getAll.bind(inventoryRepository),
    fetchSummary: inventoryRepository.getSummary?.bind(inventoryRepository),
    updateProduct: inventoryRepository.update.bind(inventoryRepository),
  };
}
```

> **Por quê sempre?** Sem Strategy, condicionais de `isDemoMode()` aparecem espalhadas em hooks, services, components — impossível auditar ou trocar o comportamento de forma segura. Com Strategy, uma única função decide, e o resto do código é agnóstico.

## Repository Pattern

Centraliza todas as chamadas HTTP de um módulo em um único arquivo. **Nenhum `fetch` ou `axios` existe fora do repository.** Nenhum hook, nenhum service-facade, nenhum component faz chamada HTTP diretamente.

> **Por quê?** Se a URL muda, se o método HTTP muda, se os headers mudam — a mudança acontece em um único arquivo. Sem repository, essas mudanças ficam espalhadas em dezenas de lugares.

## Adapter Pattern

Transforma a estrutura de dados da API no modelo interno da aplicação. O adapter fica dentro do repository e é aplicado **antes** de qualquer dado chegar ao resto do sistema. O modelo interno nunca depende do formato que o backend retorna.

> **Por quê?** APIs mudam. Sem adapter, uma mudança de `produto_id` para `product_id` no backend exige alterar dezenas de arquivos. Com adapter, a mudança fica em uma única função de conversão.

## Factory Pattern

Exemplo de uso no projeto:

1. **Factory de mocks** (`demo/factory.ts`): geram dados mockados para o modo demonstração. Implementam a mesma interface do service real — nenhum outro arquivo sabe se o dado é real ou mockado.

## Module Pattern

Cada página é um módulo autocontido em `/src/modules/NomeModulo`. **Nada vaza para fora do módulo sem ser explicitamente exportado.** Módulos não se importam entre si — se precisam de algo compartilhado, esse algo sobe para `/src/models`, `/src/hooks` ou `/src/services` globais, ou é comunicado via EventBus.

## Provider Pattern

Compartilhamento de estado global via React Context. Usado para auth, sessão, tema e dados verdadeiramente globais. **Não usar Context para estado de módulo** — estado de módulo pertence ao hook do módulo. Context é para o que é transversal a toda a aplicação.

## Hook Pattern (Custom Hooks)

Lógica de estado e comportamento reutilizável extraída dos componentes. Cada hook tem **uma única responsabilidade**: `useInventoryState` gerencia estado, `useInventoryActions` gerencia ações, `usePaginationState` gerencia paginação. O hook-facade (`useInventoryApi`) agrega tudo e é o único ponto público.

> **Regra de ouro dos hooks:** se um hook está importando de mais de uma fonte (dois services diferentes, um service e um context), ele provavelmente está fazendo mais do que deveria. Quebre em hooks menores.
