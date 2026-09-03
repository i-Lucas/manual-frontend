# 3. Estrutura de Pastas

```
/
├── public/
│   ├── manifest.json
│   └── icons/
├── docs/                          ← Documentação do projeto (fora de src)
│   ├── ARCHITECTURE.md
│   ├── CHANGELOG.md
│   └── FRONTEND.md                ← Este arquivo (ou link para ele)
├── src/
│   ├── components/                ← Componentes atômicos globais
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── ButtonIcon.tsx
│   │   │   ├── ButtonLabel.tsx
│   │   │   ├── ButtonSpinner.tsx
│   │   │   └── index.ts
│   │   ├── Input/
│   │   ├── Alert/
│   │   ├── Badge/
│   │   ├── Modal/
│   │   └── Table/
│   ├── modules/                   ← Páginas (cada página = 1 módulo)
│   │   └── Dashboard/
│   │       ├── Dashboard.tsx
│   │       ├── components/        ← Sessões da página
│   │       ├── constants/         ← Constantes do módulo (ver 16)
│   │       │   └── dashboard.constants.ts
│   │       ├── demo/              ← Dados mockados para modo demo (demonstração)
│   │       ├── docs/              ← Docs específicas do módulo
│   │       ├── hooks/
│   │       ├── i18n/              ← Textos por idioma (ver 10.6)
│   │       │   ├── dashboard.pt.json
│   │       │   └── dashboard.en.json
│   │       ├── models/
│   │       ├── services/
│   │       └── utils/
│   ├── services/                  ← Services globais
│   │   ├── cache/
│   │   ├── http/
│   │   ├── eventBus/
│   │   ├── ws/                    ← WebSocket Client
│   │   ├── i18n/                  ← Motor de internacionalização (ver 10.6)
│   │   └── auth/
│   ├── hooks/                     ← Hooks globais reutilizáveis (inclui useT)
│   ├── models/                    ← Tipos globais compartilhados
│   ├── constants/                 ← Constantes globais (usadas em 2+ módulos)
│   ├── i18n/                      ← Textos globais compartilhados (common.{locale}.json)
│   ├── utils/                     ← Utilitários globais
│   ├── contexts/                  ← Contextos globais (Provider pattern)
│   ├── assets/
│   │   └── styles/
│   │       └── Global.css         ← ÚNICO arquivo CSS global (ver 04)
│   ├── router/
│   │   ├── AppRouter.tsx          ← tabela de rotas
│   │   ├── routes.ts              ← as páginas, declaradas como chunks sob demanda
│   │   ├── lazyRoute.tsx          ← React.lazy + recuperação de chunk + namespace de i18n
│   │   ├── routePreload.ts        ← aquece o chunk da rota antes do clique
│   │   └── RouteFallback.tsx      ← vão exibido enquanto o chunk chega
│   ├── App.tsx
│   └── main.tsx                   ← bootstrap: *Reactivity + i18n do idioma ativo + render
├── .env
├── .env.dev
├── index.html
├── tsconfig.json                 ← Somente se/quando necessário -> vite já traz configuração default
└── vite.config.ts                ← Somente se/quando necessário (plugins, proxy, ws) -> vite já traz configuração default
```

## Arquivos de documentação obrigatórios (`/docs`)

| Arquivo | Conteúdo |
|---|---|
| `ARCHITECTURE.md` | Estrutura de pastas, design patterns adotados, princípios de organização, decisões arquiteturais |
| `CHANGELOG.md` | Registro cronológico de features e correções. Novas entradas **sempre no topo** |
| `FRONTEND.md` | Regras específicas de desenvolvimento (este guia, ou link para ele) |
