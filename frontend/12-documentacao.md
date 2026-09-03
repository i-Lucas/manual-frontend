# 12. Documentação & Commits

## JSDoc obrigatório

Todo arquivo deve ter um bloco de comentário no topo descrevendo sua missão. Funções críticas devem ter JSDoc completo.

```ts
/**
 * @module inventoryService
 * @description Service-facade do módulo de estoque. Único ponto público
 * para hooks e containers. Orquestra repository, cache, validator e strategy.
 * 
 * @changelog
 * 2024-03-10 - Adicionado updateProductOptimistic com rollback de cache
 * 2024-01-15 - Criação inicial
 */
```

## Docs por módulo (`/src/modules/module/docs/`)

```markdown
# Inventory — Notas de Desenvolvimento

## TODOs
- [ ] Implementar exportação de relatório CSV
- [ ] Adicionar filtro por data de validade

## Bugs conhecidos
- [ ] Paginação não reseta ao alterar filtro ativo (#123)

## Próximas features
- Integração com sistema de reservas
```

## Padrão de commits

```
feat: add optimistic update to inventory product
fix: reset pagination on filter change
refactor: extract cache TTL to constants
chore: update inventory docs with new cache keys
docs: add jsdocs to inventoryService facade
```

> **Nunca fazer push sem autorização explícita.** Commitar após cada feature ou correção significativa. Sempre em inglês. Sempre atualizar CHANGELOG.md antes do commit — sempre atualizar as docs com informações importantes/mudanças significativas.

## Atualização do CHANGELOG

```markdown
# CHANGELOG

## [Unreleased]

## [1.2.0] - 2024-03-10
### Added
- Optimistic update for inventory product editing
- Granular cache invalidation per module section

## [1.1.0] - 2024-01-20
### Added
- Demo mode with factory-generated mock data
### Fixed
- Pagination not resetting on filter change
```
