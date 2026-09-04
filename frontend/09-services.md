# 9. Camada de Services

## Princípios

- **Cada service tem uma única responsabilidade e um único motivo para mudar.** Repository faz HTTP. Adapter normaliza dados. Validator valida dados de negócio. Factories são admitidas para gerar mocks do modo demo, nunca para produzir estilos. Nenhum desses faz mais do que isso.
- **Cache é infraestrutura global.** Existe um único `cacheService` em `src/services/cache/`, importado diretamente pelo service-facade de cada módulo. É proibido criar `inventoryCacheService`, pastas `*-cache/` ou qualquer wrapper de cache por módulo.
- **Services são objetos literais ou funções puras — não classes.** Classes com estado interno criam acoplamento oculto e dificultam testes. Objects literals com métodos são suficientes e mais explícitos.
- **Toda lógica de negócio vive nos services, nunca nos hooks.** Se há uma regra de negócio num hook, ela pertence ao service-facade ou a um sub-service. Hooks apenas orquestram estado e delegam.
- **Um único service-facade (`moduleService`) é o único ponto público da camada de services.** Hooks nunca importam `inventoryRepository`, `inventoryValidator` ou o `cacheService` global diretamente. Tudo passa pelo service-facade. Se o hook precisa de algo que o service-facade não expõe, o service-facade é atualizado — não o hook.
- **A comunicação entre módulos é exclusivamente via EventBus.** Um service nunca importa código de outro módulo. Se o service de Estoque precisa notificar o módulo de Pedidos, ele emite um evento no EventBus. O módulo de Pedidos reage à sua própria subscription — sem saber a origem.
- **Strategy Pattern para comportamento variável.** A decisão entre demo e real, entre DEV e PROD, acontece em um único ponto — a função `getStrategy()` do service-facade. O restante do código é completamente agnóstico.

## Estrutura de um service de módulo

```
src/modules/Inventory/services/
├── inventory-repository/            ← Camada HTTP
│   ├── inventoryRepository.ts
│   ├── inventoryRepository.adapter.ts
│   └── models/
│       └── inventoryRepository.types.ts
├── inventory-validator/             ← Validações de negócio
│   └── inventoryValidator.service.ts
└── inventoryService.ts              ← Service-facade (único ponto público)
```

## Repository (camada HTTP)

```ts
// src/modules/Inventory/services/inventory-repository/inventoryRepository.ts
/**
 * @description Repositório HTTP do módulo de estoque.
 * Único lugar que faz chamadas à API de inventário.
 * Usa o httpClient global para injeção de token e tratamento de erros.
 * @since 2024-01-15
 */
import { httpClient } from '@/services/http/httpClient';
import { adaptProductList, adaptProduct } from './inventoryRepository.adapter';
import type {
  RawProductListResponse,
  RawProductResponse,
  FetchProductsParams,
} from './models/inventoryRepository.types';
import type { ProductList, Product } from '../../models/inventory.types';
import type { Result } from '@/models/result.types';

export const inventoryRepository = {
  async getAll(params: FetchProductsParams): Promise<Result<ProductList>> {
    const response = await httpClient.get<RawProductListResponse>('/inventory/products', { params });
    if (!response.ok) return response;
    return { ok: true, data: adaptProductList(response.data) };
  },

  async getById(id: string): Promise<Result<Product>> {
    const response = await httpClient.get<RawProductResponse>(`/inventory/products/${id}`);
    if (!response.ok) return response;
    return { ok: true, data: adaptProduct(response.data) };
  },

  async update(product: Product): Promise<Result<Product>> {
    const response = await httpClient.put<RawProductResponse>(`/inventory/products/${product.id}`, product);
    if (!response.ok) return response;
    return { ok: true, data: adaptProduct(response.data) };
  },
};
```

## Adapter (normalização de dados da API)

```ts
// src/modules/Inventory/services/inventory-repository/inventoryRepository.adapter.ts
/**
 * @description Adapta as respostas da API para o modelo interno da aplicação.
 * Garante que mudanças na API não propaguem para o restante do código.
 */
import type { RawProductResponse, RawProductListResponse } from './models/inventoryRepository.types';
import type { Product, ProductList } from '../../models/inventory.types';

/** Converte um produto bruto da API para o modelo interno */
export function adaptProduct(raw: RawProductResponse): Product {
  return {
    id: raw.produto_id,
    name: raw.nome_produto,
    sku: raw.sku,
    quantity: raw.quantidade_atual,
    minQuantity: raw.quantidade_minima,
    category: raw.categoria,
    expiresAt: raw.data_validade ? new Date(raw.data_validade) : null,
    status: resolveProductStatus(raw.quantidade_atual, raw.quantidade_minima),
    updatedAt: new Date(raw.updated_at),
  };
}

/** Converte a resposta paginada de produtos */
export function adaptProductList(raw: RawProductListResponse): ProductList {
  return {
    items: raw.produtos.map(adaptProduct),
    total: raw.total,
    page: raw.pagina,
  };
}

function resolveProductStatus(quantity: number, minQuantity: number): Product['status'] {
  if (quantity === 0) return 'out_of_stock';
  if (quantity <= minQuantity * 0.5) return 'critical';
  if (quantity <= minQuantity) return 'warning';
  return 'normal';
}
```

## Consumo do cache global

O módulo **não possui uma camada ou um service de cache próprio**. Seu service-facade importa o `cacheService` global e executa nele as operações necessárias. A construção genérica de chaves pertence ao `cacheKeys` global; TTLs específicos do domínio podem ser declarados nas constantes do módulo. Nenhum desses elementos encapsula o cache em outro service.

```ts
// ✅ O service-facade consome a infraestrutura global diretamente
import { cacheService } from '@/services/cache/cacheService';
import { cacheKeys } from '@/services/cache/cacheKeys';
import { INVENTORY_CACHE_TTL } from '../constants/inventory.constants';

// ❌ Não existe integração local, wrapper ou facade intermediário de cache
import { inventoryCacheService } from './inventory-cache/inventoryCache.service';
```

## Service-facade (único ponto público do módulo)

```ts
// src/modules/Inventory/services/inventoryService.ts
/**
 * @description Service-facade do módulo de estoque.
 * Único ponto público. Orquestra repository, cache global, validator e strategy.
 * Os hooks apenas consomem este service — nunca os sub-services diretamente.
 * @since 2024-01-15
 */
import { inventoryRepository } from './inventory-repository/inventoryRepository';
import { inventoryValidator } from './inventory-validator/inventoryValidator.service';
import { cacheService } from '@/services/cache/cacheService';
import { cacheKeys } from '@/services/cache/cacheKeys';
import { eventBus } from '@/services/eventBus/eventBus';
import { isDemoMode } from '@/utils/environment.util';
import { createDemoInventoryStrategy } from '../demo/factory';
import { INVENTORY_CACHE_TTL } from '../constants/inventory.constants';
import type { FetchProductsParams, Product, InventorySummary, ProductList } from '../models/inventory.types';
import type { Result } from '@/models/result.types';

/**
 * Seleciona a estratégia de fetch de acordo com o modo atual (avaliado em runtime).
 * isDemoMode() lê sessionStorage — o modo pode ser ativado em qualquer momento.
 */
function getStrategy() {
  if (isDemoMode()) return createDemoInventoryStrategy();
  return {
    fetchProducts: inventoryRepository.getAll.bind(inventoryRepository),
    fetchSummary: inventoryRepository.getSummary?.bind(inventoryRepository),
    updateProduct: inventoryRepository.update.bind(inventoryRepository),
  };
}

export const inventoryService = {
  /**
   * Busca lista de produtos com cache granular por página/filtro.
   * Cache hit → retorna imediatamente. Cache miss → fetch + armazena.
   */
  async fetchProducts(params: FetchProductsParams): Promise<Result<ProductList>> {
    const listKey = cacheKeys.build('inventory', 'list', params.page, params.search, params.filter);
    const cached = cacheService.get<ProductList>(listKey);
    if (cached) return { ok: true, data: cached };

    const result = await getStrategy().fetchProducts(params);
    if (result.ok) cacheService.set(listKey, result.data, INVENTORY_CACHE_TTL.PRODUCT_LIST);
    return result;
  },

  /** Busca resumo do header. Cache de curta duração por ser dado crítico. */
  async fetchSummary(): Promise<Result<InventorySummary>> {
    const summaryKey = cacheKeys.build('inventory', 'header', '_', 'summary', '_');
    const cached = cacheService.get<InventorySummary>(summaryKey);
    if (cached) return { ok: true, data: cached };

    const result = await getStrategy().fetchSummary();
    if (result.ok) cacheService.set(summaryKey, result.data, INVENTORY_CACHE_TTL.SUMMARY);
    return result;
  },

  /**
   * Atualiza produto com atualização otimista no cache.
   * Não invalida listas — apenas atualiza o produto individualmente.
   * Emite evento para outros módulos reagirem se necessário.
   */
  async updateProduct(product: Product): Promise<Result<Product>> {
    const validation = inventoryValidator.validateProduct(product);
    if (!validation.ok) return validation;

    // Salva o dado original antes do update otimista (necessário para rollback)
    const productKey = cacheKeys.build('inventory', 'product', '_', 'detail', product.id);
    const original = cacheService.get<Product>(productKey);

    // Atualização otimista imediata
    cacheService.set(productKey, product, INVENTORY_CACHE_TTL.PRODUCT_DETAIL);

    const result = await getStrategy().updateProduct(product);
    if (result.ok) {
      cacheService.set(productKey, result.data, INVENTORY_CACHE_TTL.PRODUCT_DETAIL);
      // Notifica outros módulos via EventBus (ex: módulo de relatórios)
      eventBus.emit('inventory:product:updated', { productId: result.data.id });
    } else if (original) {
      // Rollback: restaura o estado anterior ao update otimista
      cacheService.set(productKey, original, INVENTORY_CACHE_TTL.PRODUCT_DETAIL);
    }
    return result;
  },
};
```

## Factory de dados mock (modo demonstração)

```ts
// src/modules/Inventory/demo/factory.ts
/**
 * @description Factory de dados mock para o modo demonstração.
 * Simula delays realistas para dar imersão ao usuário.
 * Implementa a mesma interface do strategy real.
 */
import { demoProducts } from './products.mock';
import { demoSummary } from './summary.mock';
import type { FetchProductsParams, ProductList, InventorySummary } from '../models/inventory.types';
import type { Result } from '@/models/result.types';
import { simulateDelay } from '@/utils/demo.util';

export function createDemoInventoryStrategy() {
  return {
    async fetchProducts(params: FetchProductsParams): Promise<Result<ProductList>> {
      await simulateDelay(400, 900);
      const filtered = demoProducts.filter(p =>
        !params.search || p.name.toLowerCase().includes(params.search.toLowerCase())
      );
      const start = (params.page - 1) * 20;
      return {
        ok: true,
        data: {
          items: filtered.slice(start, start + 20),
          total: filtered.length,
          page: params.page,
        },
      };
    },

    async fetchSummary(): Promise<Result<InventorySummary>> {
      await simulateDelay(200, 500);
      return { ok: true, data: demoSummary };
    },

    async updateProduct(product: Product): Promise<Result<Product>> {
      await simulateDelay(300, 700);
      return { ok: true, data: product };
    },
  };
}
```
