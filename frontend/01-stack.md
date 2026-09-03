# 1. Stack & Tecnologias

## Dependências Obrigatórias

| Pacote | Uso |
|---|---|
| `react` + `react-dom` | UI |
| `react-router-dom` | Roteamento |
| `bootstrap` | Estilo base, tema, responsividade |
| `react-bootstrap` | Componentes bootstrap prontos para React |
| `bootstrap-icons` | Ícones |
| `lucide-react` | Ícones vetoriais |
| `vite` | Bundler / Dev server |

## Dependências Opcionais (somente se necessário)

| Pacote | Uso |
|---|---|
| `chart.js` | Gráficos e dashboards |
| `socket.io-client` | Comunicação em tempo real |

## Regras absolutas

- **100% TypeScript.** Nenhum arquivo `.js` ou `.jsx`. Todo código é `.ts` ou `.tsx`.
- **Proibido usar `any`.** Sem exceções. Use `unknown` + type guard, generics, ou modele corretamente o tipo.
- **Mobile-first obrigatório.** Toda UI é projetada primeiro para telas pequenas e expandida para telas maiores usando as classes responsivas do Bootstrap (`sm`, `md`, `lg`, `xl`, `xxl`). Responsividade máxima e adaptabilidade a diferentes tamanhos de tela são obrigatórias desde o primeiro componente.
- **PWA — fase final.** A configuração de PWA (service worker, manifest) é implementada na **fase final** do desenvolvimento, quando a aplicação está estável. `vite-plugin-pwa` é a solução recomendada quando necessário. A estratégia de service worker (network-first, cache-first, stale-while-revalidate) varia por projeto de acordo com a necessidade. O service worker de PWA e o sistema de cache em memória deste guia são camadas completamente independentes.
