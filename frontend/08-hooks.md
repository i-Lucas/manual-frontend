# 8. Camada de Hooks

## Princípios

- **Hooks não implementam regras de negócio.** Orquestram estado e delegam 100% para o service-api. Se há lógica de negócio num hook, ela pertence ao service.
- **Hooks nunca fazem chamadas HTTP.** HTTP é responsabilidade exclusiva do repository. Hooks chamam o service-api; o service-api chama o repository.
- **Estado e ações sempre separados** em hooks distintos. `useModuleState` gerencia o estado local; `useModuleActions` gerencia as operações assíncronas. O hook-facade os agrega.
- **`useMemo` e `useCallback` em todo retorno de hook.** Sem isso, cada render do componente pai cria novas referências, causando re-renders desnecessários em cascata.
- **Um único hook-facade (`useModuleApi`) é o único ponto público da camada de hooks.** Containers nunca importam `useInventoryState`, `useInventoryActions` ou qualquer sub-hook diretamente. Se o Container precisa de algo que o hook-api não expõe, o hook-api é atualizado — não o Container.
- **Hooks globais (`/src/hooks`) são reutilizados por todos os módulos.** `useLoadingState`, `useErrorState`, `usePaginationState` — implementados uma vez, consumidos por todos. Nunca re-implementar a mesma lógica em dois módulos diferentes.

## Hooks Globais (`/src/hooks`)

Lógica reutilizável por todos os módulos. Implementada uma única vez — nunca duplicada. Se dois módulos precisam do mesmo padrão de estado (loading, error, pagination), ele vive aqui.

```ts
// src/hooks/useLoadingState.ts
/**
 * @description Hook global para gerenciamento de estado de carregamento.
 * Reutilizado por todos os hooks de módulos que possuem operações assíncronas.
 */
import { useState, useCallback, useMemo } from 'react';

export function useLoadingState(initialValue = false) {
  const [isLoading, setIsLoading] = useState(initialValue);

  const startLoading = useCallback(() => setIsLoading(true), []);
  const stopLoading = useCallback(() => setIsLoading(false), []);

  return useMemo(() => ({
    isLoading,
    startLoading,
    stopLoading,
    setIsLoading,
  }), [isLoading, startLoading, stopLoading]);
}
```

```ts
// src/hooks/useErrorState.ts
/**
 * @description Hook global para gerenciamento de estado de erro.
 */
import { useState, useCallback, useMemo } from 'react';

export function useErrorState() {
  const [error, setError] = useState<string | null>(null);

  const clearError = useCallback(() => setError(null), []);
  const captureError = useCallback((err: unknown) => {
    setError(err instanceof Error ? err.message : 'Erro inesperado.');
  }, []);

  return useMemo(() => ({
    error,
    setError,
    clearError,
    captureError,
  }), [error, clearError, captureError]);
}
```

```ts
// src/hooks/usePaginationState.ts
/**
 * @description Hook global para gerenciamento de estado de paginação.
 */
import { useState, useCallback, useMemo } from 'react';
import type { PaginationState } from '@/models/pagination.types';

export function usePaginationState(initialPage = 1, initialPageSize = 20) {
  const [page, setPage] = useState(initialPage);
  const [pageSize, setPageSize] = useState(initialPageSize);
  const [total, setTotal] = useState(0);

  const goToPage = useCallback((p: number) => setPage(p), []);
  const reset = useCallback(() => setPage(initialPage), [initialPage]);

  return useMemo((): PaginationState => ({
    page,
    pageSize,
    total,
    totalPages: Math.ceil(total / pageSize),
    setPage: goToPage,
    setPageSize,
    setTotal,
    reset,
  }), [page, pageSize, total, goToPage, reset]);
}
```

## Hooks do Módulo

```ts
// src/modules/Inventory/hooks/useInventoryState.ts
/**
 * @description Estado local do módulo de estoque.
 * Combina hooks globais para garantir padrão uniforme e reaproveitamento de código - Princípio DRY.
 */
import { useState, useMemo } from 'react';
import { useLoadingState } from '@/hooks/useLoadingState';
import { useErrorState } from '@/hooks/useErrorState';
import { usePaginationState } from '@/hooks/usePaginationState';
import type { Product, InventorySummary } from '../models/inventory.types';

export function useInventoryState() {
  const [products, setProducts] = useState<Product[]>([]);
  const [summary, setSummary] = useState<InventorySummary | null>(null);
  const [selectedProduct, setSelectedProduct] = useState<Product | null>(null);
  const [searchTerm, setSearchTerm] = useState('');
  const [activeFilter, setActiveFilter] = useState<string | null>(null);

  const loading = useLoadingState();
  const error = useErrorState();
  const pagination = usePaginationState();

  return useMemo(() => ({
    products,
    summary,
    selectedProduct,
    searchTerm,
    activeFilter,
    ...loading,
    ...error,
    pagination,
    setProducts,
    setSummary,
    setSelectedProduct,
    setSearchTerm,
    setActiveFilter,
  }), [products, summary, selectedProduct, searchTerm, activeFilter, loading, error, pagination]);
}
```

```ts
// src/modules/Inventory/hooks/useInventoryActions.ts
/**
 * @description Actions do módulo de estoque. Toda lógica de orquestração
 * passa por aqui. Delegação total para inventoryService.
 */
import { useCallback, useMemo } from 'react';
import { inventoryService } from '../services/inventoryService';
import type { UseInventoryActionsInput } from '../models/inventory.types';

export function useInventoryActions({
  setProducts,
  setSummary,
  setSelectedProduct,
  startLoading,
  stopLoading,
  captureError,
  pagination,
}: UseInventoryActionsInput) {
  const loadProducts = useCallback(async (page: number, search?: string) => {
    startLoading();
    const result = await inventoryService.fetchProducts({ page, search });
    if (result.ok) {
      setProducts(result.data.items);
      pagination.setTotal(result.data.total);
    } else {
      captureError(result.error);
    }
    stopLoading();
  }, [startLoading, stopLoading, setProducts, captureError, pagination]);

  const loadSummary = useCallback(async () => {
    const result = await inventoryService.fetchSummary();
    if (result.ok) setSummary(result.data);
  }, [setSummary]);

  const updateProductOptimistic = useCallback((updated: Product) => {
    // Atualização otimista: reflete imediatamente sem re-fetch
    setProducts(prev => prev.map(p => p.id === updated.id ? updated : p));
    inventoryService.updateProduct(updated); // dispara em background
  }, [setProducts]);

  return useMemo(() => ({
    loadProducts,
    loadSummary,
    updateProductOptimistic,
  }), [loadProducts, loadSummary, updateProductOptimistic]);
}
```

```ts
// src/modules/Inventory/hooks/useInventoryApi.ts
/**
 * @description Hook-facade do módulo de estoque.
 * Único ponto público. Agrega estado e actions.
 * Consumido pelos containers do módulo.
 */
import { useMemo } from 'react';
import { useInventoryState } from './useInventoryState';
import { useInventoryActions } from './useInventoryActions';

export function useInventoryApi() {
  const state = useInventoryState();
  const actions = useInventoryActions(state);

  return useMemo(() => ({
    ...state,
    ...actions,
  }), [state, actions]);
}
```
