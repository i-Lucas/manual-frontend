# 11. Modo Demonstração

Toda aplicação frontend deve suportar modo demonstração completo. O usuário — ou o desenvolvedor testando a UI — deve conseguir navegar pelo sistema inteiro sem nenhum backend ativo, nem localmente.

## Regras do modo demo

- **Ativação em runtime**, não via `.env`. O modo demo é ativado quando o usuário clica em "ver demonstração" (ou similar) na landing page do produto. O botão chama `activateDemoMode()` e redireciona para a aplicação. Desativar ao fazer logout/clicar explicitamente em sair modo demo/retornar à landing page.
- **Nenhum `fetch` real é disparado** — o Strategy seleciona a implementação mock antes de qualquer chamada HTTP.
- **Delays aleatórios são simulados** — para exibir estados de loading e dar imersão realista ao usuário.
- **Dados são gerados via Factory Pattern** em `/src/modules/module/demo/`.
- **A estratégia é selecionada via Strategy Pattern** nos services — sem condicionais espalhadas pelo código. É responsabilidade do service decidir qual estratégia usar, não do `httpClient`.
- **A detecção do modo é centralizada** em um único utilitário global (`environment.util.ts`).
- **Operações de escrita são simuladas** — o mock retorna sucesso com o dado enviado, e o cache é atualizado normalmente, como se a API tivesse respondido.
- **A experiência deve ser idêntica ao modo real** — o usuário não deve perceber diferença no comportamento da aplicação.

## Por que o modo demo é importante

Ele serve dois propósitos:

1. **Produto:** Usuários em fase de avaliação podem explorar o sistema completo sem precisar criar conta ou configurar nada.
2. **Desenvolvimento:** O frontend pode ser desenvolvido e testado independentemente do backend — sem depender de API rodando localmente, sem dados de banco, sem autenticação.

Por isso, o modo demo não é opcional. **Todo módulo deve ter sua pasta `/demo`** com mocks suficientes para cobrir todos os fluxos da página.

## Implementação

```ts
// src/utils/environment.util.ts
/**
 * @description Utilitários de detecção de ambiente e modo de execução.
 * O modo demo é ativado em runtime (clique em "ver demonstração" na landing page),
 * nunca via variável de ambiente.
 */

/** Lê o estado de demo mode em runtime (persiste durante a sessão do browser) */
export function isDemoMode(): boolean {
  return sessionStorage.getItem('demo_mode') === 'true';
}

/** Chamado pelo botão "ver demonstração" na landing page antes de redirecionar */
export function activateDemoMode(): void {
  sessionStorage.setItem('demo_mode', 'true');
}

/** Chamado no logout ou ao retornar para a landing page */
export function deactivateDemoMode(): void {
  sessionStorage.removeItem('demo_mode');
}

export function isDevMode(): boolean {
  return import.meta.env.DEV;
}

export function isProdMode(): boolean {
  return import.meta.env.PROD;
}
```

```ts
// src/utils/demo.util.ts
/**
 * @description Utilitários para o modo demonstração.
 * Simula delays realistas para imersão do usuário.
 */

/** Simula um delay aleatório dentro de um intervalo (ms) */
export function simulateDelay(minMs = 300, maxMs = 800): Promise<void> {
  const delay = Math.floor(Math.random() * (maxMs - minMs + 1)) + minMs;
  return new Promise(resolve => setTimeout(resolve, delay));
}
```
