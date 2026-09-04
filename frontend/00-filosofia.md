# 0. Filosofia e Visão

> **O objetivo central de toda aplicação frontend construída com este guia é ser extremamente reativa, inteligente e previsível.**

**Reativa** significa: qualquer mudança — seja feita pelo próprio usuário, recebida via API, ou emitida por outro módulo — reflete na interface **imediatamente**, sem recarregamentos desnecessários.

**Inteligente** significa: a aplicação **sabe o que mudou**, **sabe o que está cacheado**, e **sabe exatamente o que precisa ser atualizado**. Navegar entre páginas, mudar filtros, paginar, acessar detalhes — tudo isso é fluido porque o dado já está disponível localmente. O carregamento acontece somente quando necessário.

**Previsível** significa: qualquer desenvolvedor que abrir qualquer arquivo do projeto já sabe o que vai encontrar antes mesmo de ler o código. Cada camada tem sempre a mesma estrutura, os mesmos padrões, as mesmas responsabilidades. Não existe surpresa. Não existe exceção.

## Os três princípios fundamentais

**1. Cache-first:** Toda informação que vem da API deve ser cacheada. Antes de qualquer fetch, consulte o cache. O servidor é acionado somente quando o cache não existe ou expirou.

**2. O cliente sabe o que mudou:** Quando o próprio usuário realiza uma mudança (edita um produto, cria um registro, etc.), não faz sentido invalidar o cache e refazer o fetch. A aplicação já conhece o que mudou — basta atualizar o cache localmente com o dado retornado pela API, de forma cirúrgica. A tela atualiza antes de qualquer re-fetch.

**3. Granularidade máxima:** Não existe "recarregar a página". Existe "recarregar a sessão X". Ou ainda "atualizar o recurso Y dentro da sessão X". Cada sessão tem seu próprio estado de carregamento. Cada recurso pode ser atualizado de forma isolada, sem afetar o resto da página.

> Estes três princípios devem guiar todas as decisões arquiteturais, desde a estrutura do cache até a forma como os módulos se comunicam via EventBus.

## Uniformidade é inegociável

Este guia não é um conjunto de sugestões — é um contrato. Todo módulo, todo componente, todo hook, todo service implementa exatamente os mesmos padrões, sem exceção e sem adaptações criativas.

O objetivo é que navegar pelo código seja **intuitivo por uniformidade**: ao abrir qualquer arquivo do projeto, o desenvolvedor já sabe o que vai encontrar. Um módulo novo tem a mesma estrutura que o primeiro módulo criado. Um service novo segue exatamente as mesmas camadas do service ao lado. Um componente novo implementa o mesmo Composition Pattern que todos os outros. Não há surpresa. Não há "esse aqui foi feito diferente porque parecia mais simples".

**Por que isso importa:**

Projetos crescem. O que parece simples demais para justificar uma camada extra hoje, em seis meses terá cinco variações de comportamento, três exceções de estado e lógica espalhada por lugares que ninguém esperava. A separação em camadas pequenas e padronizadas é o custo pago antecipadamente para evitar esse caos. Cada camada com uma única responsabilidade significa que, quando algo precisa mudar, a mudança fica isolada — um arquivo, um service, uma função — sem efeito colateral no restante do sistema.

**Sobre parecer "over-engineering":**

Pode parecer excessivo criar um repository, um adapter, integrar o cache global, um service-facade, um hook e um hook-facade para algo que "poderia ser um fetch numa linha". Não é excessivo. É o padrão correto. A complexidade não está no tamanho do problema hoje — está no tamanho que ele vai ter amanhã. Implementar os patterns sempre, mesmo quando parece desnecessário, é o que garante que o código escrito no primeiro dia seja tão fácil de manter quanto o código escrito no centésimo dia.

## A cadeia de responsabilidades

Cada camada da aplicação conhece apenas a camada imediatamente abaixo dela. Nenhuma camada pula outra. Nenhuma camada importa código de uma camada que não seja a sua vizinha direta.

```
Presenter (visual puro)
    ↑ props
Container (composição, sem lógica)
    ↑ useModuleApi()
Hook-api facade (único ponto público de hooks)
    ↑ moduleService
Service-api facade (único ponto público de services)
    ↑
Sub-services do módulo: repository · adapter · validator
    ↑
Infraestrutura global: cache · HTTP · EventBus
```

| Camada | Responsabilidade única | Importa de |
|---|---|---|
| Presenter | Renderizar props recebidas. Nenhuma lógica. | Nada além de tipos e sub-componentes |
| Container | Compor o Presenter com dados do hook. Nenhuma lógica condicional. | Somente o hook-api |
| Hook-api | Agregar sub-hooks. Único ponto de entrada para containers. | Somente o service-api |
| Sub-hooks | Estado local, actions, paginação — cada um com responsabilidade única. | Somente o service-api |
| Service-api | Orquestrar sub-services. Único ponto de entrada para hooks. | Sub-services do módulo |
| Repository | Chamadas HTTP. Nada mais. | httpClient global |
| Cache global | Fonte única de leitura e escrita dos dados em memória. Nunca é reimplementado por módulo. | cacheStore global |
| Adapter | Converter resposta da API para modelo interno. Nada mais. | Tipos do módulo |
| Factory (mocks) | Gerar dados mockados para modo demonstração. Nada mais. | Tipos do módulo |

> **Regra de ouro:** se um arquivo está importando algo de duas camadas diferentes ou de outro módulo que não seja o EventBus, é sinal de que algo está errado. Revise antes de continuar.
