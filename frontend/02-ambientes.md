# 2. Ambientes (Environments)

Cada projeto frontend possui dois ambientes com dois arquivos `.env`:

| Arquivo | Ambiente | Uso |
|---|---|---|
| `.env` | Produção (`PROD`) | URL da API de produção |
| `.env.dev` | Desenvolvimento (`DEV`) | URL da API local |

```env
# .env (produção)
VITE_API_URL=https://api.meusite.com
VITE_WS_URL=wss://api.meusite.com

# .env.dev (desenvolvimento)
VITE_API_URL=http://localhost:3000
VITE_WS_URL=ws://localhost:3000
```

> O ambiente `TEST` existe apenas no backend. No frontend não há testes automatizados — o modo de demonstração cobre a validação visual.
