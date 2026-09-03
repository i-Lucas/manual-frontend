# 4. Regras de Estilo (Bootstrap + react-bootstrap)

## Princípio fundamental

Todo projeto frontend deve utilizar Bootstrap e react-bootstrap como base da interface.

A estilização deve seguir esta ordem de prioridade:

1. Componentes do react-bootstrap
2. Classes utilitárias nativas do Bootstrap
3. Utilitários globais do projeto, somente quando o Bootstrap não oferecer uma solução adequada
4. Composition Pattern para variações estruturais ou visuais que dependam do estado/contexto do componente

> **Regra de ouro:** antes de criar qualquer regra CSS customizada, verifique se o Bootstrap já possui um componente, utility class ou mecanismo nativo que resolva o problema.

O agente pode e deve consultar a documentação do Bootstrap e, quando necessário, inspecionar o pacote instalado em `node_modules/bootstrap` para conhecer as classes e recursos disponíveis.

## Mobile-first e responsividade são obrigatórios

Toda interface deve ser desenvolvida seguindo uma abordagem mobile-first.

Isso significa que:

- a experiência em celulares deve ser considerada desde o início da implementação;
- componentes e layouts devem funcionar corretamente em telas pequenas;
- tablets também devem possuir uma experiência adequada;
- o layout deve escalar corretamente para notebooks, desktops e telas grandes;
- não é aceitável implementar primeiro uma interface exclusivamente desktop e depois aplicar correções improvisadas para dispositivos móveis.

Utilize prioritariamente o sistema responsivo do Bootstrap:

- `container` / `container-fluid`
- `row`
- `col-*`
- `d-*`
- `flex-*`
- `gap-*`
- `w-*`
- `order-*`
- breakpoints `sm`, `md`, `lg`, `xl`, `xxl`
- demais utilities responsivas disponibilizadas pelo Bootstrap

Exemplo:

```tsx
<div className="d-flex flex-column flex-lg-row gap-3">
  ...
</div>
```

Evite criar CSS responsivo manual quando as utilities do Bootstrap forem suficientes.

## Bootstrap deve ser maximizado

Não recrie manualmente funcionalidades que já existem no Bootstrap ou no react-bootstrap.

Sempre prefira componentes do react-bootstrap quando houver um componente adequado.

| Necessidade | Preferir |
|---|---|
| Grid/layout | Container, Row, Col |
| Sidebar/drawer mobile | Offcanvas |
| Navegação | Nav, Nav.Link |
| Barra superior | Navbar |
| Modal | Modal |
| Menu | Dropdown |
| Abas | Tab, Nav |
| Alertas | Alert |
| Notificações | Toast, ToastContainer |
| Botões | Button |
| Formulários | Form |
| Cards | Card |
| Collapse | Collapse / Accordion |
| Indicadores de carregamento | Spinner / Placeholder |

Da mesma forma, propriedades CSS comuns devem ser representadas por classes Bootstrap sempre que possível.

```tsx
// ❌ ERRADO
<div style={{ display: 'flex', alignItems: 'center', gap: '1rem' }}>

// ✅ CERTO
<div className="d-flex align-items-center gap-3">
```

## Valores aproximados devem utilizar a utility Bootstrap mais próxima

Quando o valor definido para uma propriedade não existir exatamente na escala do Bootstrap, o agente deve utilizar a classe Bootstrap com o valor mais próximo.

Uma diferença visual mínima ou imperceptível não justifica a criação de CSS customizado, estilos inline, classes específicas ou novos tokens. Manter a aplicação em conformidade com o sistema de utilities do Bootstrap tem prioridade sobre reproduzir um valor numérico sem impacto visual relevante.

Essa regra vale especialmente (mas não exclusivamente) para valores de espaçamento, como `padding`, `margin` e `gap`.

Exemplo: se o valor pretendido de `padding` for `0.9rem`, utilize `p-3`, que corresponde a `1rem` na escala padrão do Bootstrap, em vez de criar uma regra customizada apenas para preservar a diferença de `0.1rem`.

```tsx
// ❌ ERRADO — CSS customizado para obter exatamente 0.9rem
<div className="custom-padding">

// ✅ CERTO — utility Bootstrap com o valor mais próximo
<div className="p-3">
```

> **Atenção:** o número da utility não representa diretamente o valor em `rem`. Na escala padrão do Bootstrap, `p-1` corresponde a `0.25rem` e `p-3` corresponde a `1rem`. O agente deve consultar a escala da versão instalada e escolher a classe pelo valor efetivo mais próximo, não apenas pelo sufixo numérico.

> **Regra:** a ausência de uma correspondência exata na escala do Bootstrap não autoriza, por si só, a criação de CSS customizado.

## Estilos inline são proibidos

É proibido utilizar a prop style para estilização.

A proibição vale tanto para estilos estáticos quanto para estilos condicionais ou calculados.

```tsx
// ❌ PROIBIDO
<div style={{ padding: '16px' }} />

// ❌ PROIBIDO
<div
  style={{
    borderColor: isSelected ? 'var(--company-primary)' : 'transparent',
  }}
/>

// ❌ PROIBIDO
<button style={getButtonStyle(active)} />

// ❌ PROIBIDO
<div style={styles.container} />
```

Isso inclui:

- objetos CSSProperties;
- factories de estilos;
- funções que retornam objetos CSS;
- objetos de estilos exportados;
- estilos calculados a partir de props;
- estilos calculados a partir de estado;
- estilos calculados a partir de dados da API;
- `<style>` embutido no componente.

Não crie mecanismos alternativos apenas para contornar essa regra.

## `.styles.ts` e factories de estilo são proibidos

Arquivos como:

```text
Component.styles.ts
Card.styles.ts
styles.ts
```

não devem existir com a finalidade de gerar ou armazenar estilos.

Também é proibido este padrão:

```ts
const card = ({ isSelected }: CardStyleParams): CSSProperties => ({
  borderColor: isSelected
    ? 'var(--company-primary)'
    : 'var(--color-border)',
})
```

e seu consumo:

```tsx
<div style={styles.card({ isSelected })} />
```

Factories não são a solução para estilos condicionais.

Quando uma diferença visual representar uma variante real do componente, a solução arquitetural deve ser baseada em Composition Pattern, mantendo os componentes pequenos, explícitos e combináveis.

## Composition Pattern é obrigatório

Os componentes da aplicação devem ser construídos utilizando composição.

Evite componentes monolíticos cuja estrutura interna seja alterada por diversas props, flags e condicionais.

Em vez de criar um componente que tente prever todas as combinações possíveis de conteúdo e aparência, crie peças menores que possam ser compostas.

Exemplo conceitual:

```tsx
<Card>
  <Card.Header>
    ...
  </Card.Header>

  <Card.Body>
    ...
  </Card.Body>

  <Card.Actions>
    <Button>Editar</Button>
    <Button>Excluir</Button>
  </Card.Actions>
</Card>
```

Outra utilização pode simplesmente não compor determinadas partes:

```tsx
<Card>
  <Card.Body>
    ...
  </Card.Body>

  <Card.Actions>
    <Button>Visualizar</Button>
  </Card.Actions>
</Card>
```

A estrutura não deve depender de uma sequência crescente de flags como:

```tsx
// ❌ EVITAR
<Card
  showHeader
  showEditButton
  showDeleteButton
  showFooter
  isSelected={isSelected}
  compact={compact}
/>
```

O objetivo é criar componentes pequenos, reutilizáveis, previsíveis e combináveis.

Composição deve ser a primeira solução considerada quando uma necessidade de UI levar o agente a adicionar condicionais estruturais ou factories de estilos ao componente.

## Componentes devem possuir o mínimo possível de lógica visual condicional

Evite espalhar condicionais pela marcação para transformar um único componente em múltiplas versões diferentes.

Exemplo de padrão a evitar:

```tsx
// ❌ EVITAR
return (
  <div>
    {variant === 'admin' && ...}
    {variant === 'user' && ...}
    {isSelected && ...}
    {showActions && ...}
    {compact && ...}
  </div>
)
```

Quando essas condições representam composições ou variantes distintas de interface, decomponha a solução em componentes menores e componha a interface no nível apropriado.

A regra não significa que JavaScript condicional deixou de existir na aplicação. Condições legítimas de renderização, autorização, carregamento, ausência de dados etc. continuam existindo.

O que deve ser evitado é utilizar condicionais como mecanismo para transformar componentes de apresentação em componentes monolíticos responsáveis por inúmeras variações de UI.

## CSS customizado deve ser excepcional

O CSS customizado da aplicação deve tender ao mínimo.

Não crie uma classe CSS simplesmente porque escrever CSS parece mais rápido.

Antes de adicionar qualquer regra, verifique:

- Existe componente react-bootstrap que resolve isso?
- Existe utility class do Bootstrap?
- Existe combinação de utilities Bootstrap que resolve?
- Já existe um utilitário global equivalente?
- O problema é, na realidade, estrutural e deveria ser resolvido através de composição?

Somente depois dessas verificações uma nova regra global pode ser considerada.

## Existe apenas um arquivo CSS global

A aplicação deve possuir um único arquivo CSS customizado global.

Exemplo:

```text
src/
└── assets/
    └── styles/
        └── Global.css
```

É proibido criar CSS específico para componentes, páginas ou módulos.

```text
❌ src/components/Card/Card.css
❌ src/components/Card/styles.css
❌ src/modules/users/UserList.css
❌ src/pages/Dashboard/Dashboard.css
❌ src/assets/styles/Components.css
```

O Global.css existe somente para regras verdadeiramente globais.

## O que pode existir no Global.css

O arquivo global pode conter:

### Design tokens

```css
:root {
  --company-primary: ...;
  --color-surface: ...;
  --color-text-primary: ...;
  --color-border: ...;
  --radius-md: ...;
  --transition-fast: ...;
}
```

### Overrides globais do Bootstrap

Quando necessário, variáveis ou classes Bootstrap podem ser alinhadas ao design system da aplicação.

```css
:root {
  --bs-primary: var(--company-primary);
  --bs-border-color: var(--color-border);
}
```

### Utilities globais inexistentes no Bootstrap

Se uma regra for necessária em vários pontos e o Bootstrap não possuir equivalente, pode ser criada uma utility global pequena e explícita.

Exemplo:

```css
.cursor-pointer {
  cursor: pointer;
}
```

Uso:

```tsx
<button className="btn btn-primary cursor-pointer">
  Salvar
</button>
```

Utilities globais devem:

- ser reutilizáveis;
- ter apenas uma regra CSS;
- possuir responsabilidade única;
- não conhecer componentes específicos;
- não representar uma página ou módulo;
- existir apenas quando não houver equivalente adequado no Bootstrap.

## Não criar classes CSS específicas de componentes

É proibido criar classes como:

```css
/* ❌ PROIBIDO */

.dashboard-header {
  ...
}

.user-card {
  ...
}

.patient-list-item {
  ...
}

.sidebar-menu-button {
  ...
}
```

Essas classes acoplam o CSS à estrutura específica da aplicação e tendem a produzir folhas de estilo difíceis de manter.

A marcação deve ser construída prioritariamente através de react-bootstrap, utilities Bootstrap e composição.

## Tokens devem centralizar decisões do design system

Valores semânticos compartilhados devem ser definidos como tokens globais.

Exemplos:

```css
:root {
  --company-primary: ...;

  --color-background: ...;
  --color-surface: ...;
  --color-surface-secondary: ...;

  --color-text-primary: ...;
  --color-text-secondary: ...;
  --color-border: ...;

  --radius-sm: ...;
  --radius-md: ...;
  --radius-lg: ...;

  --transition-fast: ...;
  --transition-normal: ...;

  --font-size-sm: ...;
  --font-size-md: ...;
  --font-size-lg: ...;
}
```

Evite espalhar valores arbitrários pela aplicação.

O design system deve possuir uma fonte central para decisões visuais compartilhadas.

## Dark/light theme

Quando o projeto possuir suporte aos temas do Bootstrap, utilize o mecanismo nativo baseado em data-bs-theme.

Exemplo:

```html
<html lang="pt-BR" data-bs-theme="dark">
```

Para alternar:

```html
<html lang="pt-BR" data-bs-theme="light">
```

Não crie uma segunda arquitetura de tema paralela ao Bootstrap sem necessidade.

Tokens próprios podem complementar o tema quando o design system exigir, mas devem permanecer centralizados no Global.css.

## Exemplos de conversão para Bootstrap

| CSS/propriedade | Preferir |
|---|---|
| display: flex | d-flex |
| display: grid | d-grid |
| align-items: center | align-items-center |
| justify-content: space-between | justify-content-between |
| width: 100% | w-100 |
| height: 100% | h-100 |
| padding | p-*, px-*, py-* |
| margin | m-*, mx-*, my-* |
| gap | gap-* |
| font-weight | fw-* |
| text-align | text-start, text-center, text-end |
| white-space: nowrap | text-nowrap |
| border: 0 | border-0 |
| bordas padrão | border, border-top, border-bottom, etc. |
| arredondamento | rounded-*, rounded-circle, rounded-pill |
| sombras padrão | shadow-sm, shadow, shadow-lg |
| opacidade | opacity-* |
| cores semânticas | text-*, bg-* |
| flex shrink/grow | flex-shrink-*, flex-grow-* |
| posição | position-* |
| overflow | overflow-* |
| visibilidade responsiva | d-none, d-md-block, etc. |
| cursor: pointer | cursor-pointer global, caso não exista na versão adotada |

Essa tabela não é exaustiva.

A ausência de uma classe nesta tabela não significa que ela não exista no Bootstrap.

O agente deve verificar a biblioteca antes de criar CSS customizado.

De preferência pesquisar na pasta node_modules/bootstrap ou na documentação oficial do Bootstrap.

## Proibições absolutas

```tsx
// ❌ style inline
<div style={{ padding: 16 }} />

// ❌ style inline condicional
<div style={{ borderColor: selected ? 'blue' : 'gray' }} />

// ❌ factory
<div style={styles.card({ selected })} />

// ❌ objeto CSSProperties
const styles: CSSProperties = { ... }

// ❌ arquivo .styles.ts
import { cardStyles } from './Card.styles'

// ❌ <style> dentro do componente
<style>{`...`}</style>

// ❌ classe específica criada para o componente
<div className="dashboard-header">

// ❌ CSS específico do componente
import './Dashboard.css'
```

Também é proibido:

- criar CSS Modules;
- utilizar styled-components ou solução equivalente sem decisão arquitetural explícita do projeto;
- criar múltiplos arquivos CSS;
- criar factories de estilo;
- usar style como escape para valores condicionais;
- duplicar utilities que já existem no Bootstrap;
- criar classes globais específicas para um único componente;
- ignorar responsividade;
- implementar desktop-first e corrigir mobile posteriormente;
- criar componentes monolíticos controlados por inúmeras flags quando composição resolver o problema.

## Checklist obrigatório antes de finalizar uma implementação

Antes de considerar uma tela ou componente concluído, o agente deve verificar:

- [ ] A interface foi construída mobile-first.
- [ ] O comportamento foi considerado para celular, tablet e desktop.
- [ ] react-bootstrap foi utilizado quando havia componente apropriado.
- [ ] As utilities Bootstrap foram utilizadas ao máximo.
- [ ] Valores sem correspondência exata utilizaram a utility Bootstrap de valor efetivo mais próximo.
- [ ] Não existe style={{ ... }}.
- [ ] Não existe uso da prop style para estilização.
- [ ] Não existe .styles.ts.
- [ ] Não existe factory de estilos.
- [ ] Não existe CSS específico do componente.
- [ ] Não foi criado nenhum arquivo .css adicional.
- [ ] Qualquer CSS customizado necessário está no único Global.css.
- [ ] Utilities globais adicionadas são realmente reutilizáveis e inexistentes no Bootstrap.
- [ ] Tokens compartilhados estão centralizados.
- [ ] Componentes complexos foram decompostos.
- [ ] Composition Pattern foi utilizado para compor variações estruturais da UI.
- [ ] Não existem componentes monolíticos controlados por uma coleção crescente de flags.
- [ ] O resultado mantém boa experiência em telas grandes, apesar da abordagem mobile-first.

## Regra final

Ao implementar qualquer interface, siga esta sequência mental:

```text
react-bootstrap
      ↓
Bootstrap utilities
      ↓
Utilities/tokens globais já existentes
      ↓
Composition Pattern
      ↓
Somente se inevitável:
nova utility global mínima no Global.css
```

Nunca:

```text
Problema visual
      ↓
style={{ ... }}
```

Nunca:

```text
Problema condicional
      ↓
.styles.ts / factory
```

Para variações de interface, prefira:

```text
Necessidade de variação
      ↓
decompor o componente
      ↓
criar peças menores
      ↓
compor explicitamente a UI
```

O objetivo é manter o frontend responsivo, previsível, reutilizável e fácil de manter, utilizando o ecossistema Bootstrap como base e evitando a proliferação de CSS e lógica visual ad hoc.
