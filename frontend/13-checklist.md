# 13. Checklist de Qualidade

> Este checklist não é opcional. Todo item deve ser verificado antes de qualquer commit. Um item que "não se aplica a esse caso específico" é sinal de que o padrão não foi seguido — revise antes de pular.

Antes de qualquer commit, verificar:

**Uniformidade e arquitetura**
- [ ] O arquivo que estou commitando segue exatamente o mesmo padrão dos arquivos equivalentes já existentes no projeto
- [ ] Nenhuma camada foi pulada na cadeia `Container → hook-api → service-api → sub-services`
- [ ] Nenhum módulo importa código de outro módulo diretamente (toda comunicação inter-modular passa pelo EventBus)
- [ ] Nenhum arquivo tem mais de uma responsabilidade

**TypeScript**
- [ ] Sem uso de `any` em nenhum arquivo
- [ ] Todos os tipos definidos em `/models` (não inline)
- [ ] Todas as funções com tipos de parâmetro e retorno explícitos
- [ ] Nenhum `as NomeDoTipo` desnecessário — se precisar de cast, o tipo está modelado incorretamente

**Componentes**
- [ ] Nenhum componente visual consome hooks, services ou factories
- [ ] Composition Pattern aplicado — sem props booleanas que mudam comportamento (`isLoading`, `hasError`, `showBadge`)
- [ ] `memo()` em todos os componentes visuais
- [ ] Nenhum `style={{}}` inline
- [ ] Condicionais de renderização encapsuladas em Guard components (`*Guard`), não em Containers ou Presenters
- [ ] Container importa apenas o hook-api do módulo — nenhum sub-hook, service ou factory diretamente

**Estilo**
- [ ] 100% classes Bootstrap para valores estáticos
- [ ] Nenhuma classe CSS criada para estilização (exceto `Global.css`)
- [ ] Variáveis CSS definidas apenas em `Global.css` — nunca em arquivos `.css` por módulo

**Hooks**
- [ ] Estado e actions separados em hooks distintos (`useModuleState` / `useModuleActions`)
- [ ] Hooks globais (`useLoadingState`, `useErrorState`, `usePaginationState`) reutilizados — nunca re-implementados
- [ ] `useCallback` e `useMemo` em todo retorno de hook
- [ ] Nenhum hook faz chamada HTTP diretamente — delega para o service-api
- [ ] Nenhum hook implementa lógica de negócio — delega para o service-api
- [ ] `useModuleApi` é o único arquivo de hook exportado para fora do módulo

**Services**
- [ ] Cada service tem uma única responsabilidade e um único motivo para mudar
- [ ] Repository é o único ponto de acesso HTTP — nenhum `fetch` fora dele
- [ ] Adapter normaliza a resposta da API antes de qualquer dado chegar ao restante do sistema
- [ ] Strategy Pattern isola a decisão demo/real em um único ponto (`getStrategy()`)
- [ ] Cache consultado antes de qualquer fetch; resultado da API persistido no cache após fetch bem-sucedido
- [ ] Rollback de cache implementado em toda operação otimista (original salvo antes do update)
- [ ] EventBus usado para comunicação entre módulos — nenhum service importa service de outro módulo
- [ ] Factory de estilos chamada pelo service-api — nunca pelo hook, nunca pelo Container
- [ ] `moduleService` (service-facade) é o único arquivo de service exportado para fora da pasta de services

**Internacionalização (ver 10.6)**
- [ ] Nenhuma string de UI literal em `.tsx`, hook ou service — todo texto via `useT`
- [ ] Módulo tem `i18n/module.pt.json` e `i18n/module.en.json` com **as mesmas chaves**, organizadas por componente
- [ ] `useT` é o único hook consumido por um Presenter — nenhum outro hook/service/factory
- [ ] Mapas de label para módulos legados derivados do `.pt.json` (sem duplicar texto)

**Constantes (ver 16)**
- [ ] Nenhuma constante "mágica" solta em componente/hook/service — todas em `constants/`
- [ ] Texto NÃO está em `constants/` (está no i18n); tipos NÃO estão em `constants/` (estão em `models/`)
- [ ] Constantes nomeadas `SCREAMING_SNAKE_CASE` e prefixadas pelo módulo quando exportadas

**Documentação**
- [ ] JSDoc no topo de cada arquivo (módulo, descrição, responsabilidade)
- [ ] CHANGELOG atualizado
- [ ] `/docs` do módulo atualizado se necessário
- [ ] Tipos exportados e documentados em `/models`
- [ ] Novos eventos do EventBus registrados em `eventBus.types.ts` antes do uso

---

> **Cada arquivo tem uma responsabilidade. Cada função tem uma responsabilidade. Quando em dúvida entre fazer em um arquivo só ou separar em dois, sempre separe. O custo de um arquivo a mais é zero. O custo de misturar responsabilidades cresce com o tempo.**
