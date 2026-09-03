# 7. Camada de Componentes

## Componentes Globais (`/src/components`)

Componentes atômicos reutilizáveis por toda a aplicação. Devem implementar **Composition Pattern** e ter diretório próprio.

A maior parte deles é um **wrapper sobre o react-bootstrap**: a biblioteca já resolve estrutura, acessibilidade e comportamento base, e o componente global padroniza aparência, densidade e o comportamento próprio da aplicação. Esta camada é o **único ponto do projeto autorizado a importar de `react-bootstrap`** — ver `04`.

> ### ⚠️ Campo de formulário SEMPRE vem do átomo (05-08-2026, ditado do dono)
>
> Nenhum módulo escreve `<input>`/`<select>`/`<textarea>` cru nem `<Form.Control>`: use
> `Input`, `Select`, `Textarea` (e os `*FieldGuard` para o estado inválido). O motivo é
> concreto — enquanto cada tela decidia sozinha, o tamanho dos campos divergiu entre
> módulos e a densidade do app ficou desigual.
>
> - **`controlSize` tem padrão `'sm'`**: essa é a densidade do app. Não repita
>   `controlSize="sm"` no consumidor; só passe `'lg'` se o controle for a peça central
>   da tela.
> - **Dentro de `input-group`, quem dimensiona é o GRUPO**: o átomo encolhe o campo, mas
>   o addon (`R$`, relógio, 🔍) mantém a própria altura e o conjunto continua grande.
>   Todo grupo leva `input-group-sm` (ou `<InputGroup size="sm">`).
> - **Não são campos de texto** e seguem crus: `input type="file"` oculto, `radio`, e o
>   miolo de steppers. A cauda BESPOKE (composers do chat, buscas em pílula do Messages,
>   paleta da busca global) só migra tela a tela, com mockup aprovado antes.

```
src/components/
├── Button/
│   ├── Button.tsx          ← Root + montagem do objeto exportado
│   ├── ButtonIcon.tsx
│   ├── ButtonLabel.tsx
│   ├── ButtonSpinner.tsx
│   ├── models/
│   │   └── button.types.ts
│   └── index.ts            ← Re-export público: export { Button } from './Button'
├── Input/
│   ├── Input.tsx
│   ├── InputLabel.tsx
│   ├── InputError.tsx
│   ├── InputHelper.tsx
│   └── index.ts
├── DataTable/
├── Modal/
├── Alert/
└── Badge/
```

## Componentes de Módulo (`/src/modules/Module/components`)

Sessões da página. Sempre divididos em `Component` (visual puro) + `Container` (composição e dados). Nunca em um arquivo só.

## Regras de componentes

1. **Componentes visuais recebem apenas props e callbacks.** Nunca consomem hooks, services, nem factories. Um componente visual que importa um hook é um erro de arquitetura — mova a lógica para o Container. **Única exceção: `useT`** (i18n, ver `10.6`) — texto é conteúdo, não lógica; Presenters podem consumir `useT` e nenhum outro hook.
2. **Todo componente usa Composition Pattern.** Mesmo que hoje só exista uma variação. Componentes crescem — o padrão já deve estar estabelecido antes que a segunda variação apareça.
3. **`memo()` em todos os componentes visuais.** Sem exceção. O Container decide quando re-renderizar; o Presenter nunca deve re-renderizar por causa de mudanças que não afetam suas props.
4. **Nunca `style={{}}` inline.** Valores estáticos → Bootstrap. Valores dinâmicos de runtime → factory no service-api, resultado chega via props.
5. **Tipos em `/models`, nunca inline.** Tipos inline em props tornam o componente ilegível e impossibilitam reutilização.
6. **Containers importam apenas o hook-api do módulo.** Nenhum sub-hook, nenhum service, nenhuma factory é importada diretamente pelo Container.
7. **Condicionais de renderização pertencem a Guard components.** Nem o Presenter nem o Container decidem se algo é renderizado ou não — essa decisão é de um `*Guard` dedicado.
8. **Nenhuma string de UI literal.** Todo texto visível vem do `useT` (ver `10.6`). Um título, label ou mensagem escrito direto no JSX é um erro — equivale a um `style` inline.
9. **Nenhuma constante mágica no componente.** Números, listas fixas e mapas de configuração vêm de `constants/` (ver `16`).
10. **`react-bootstrap` só é importado dentro de `src/components/`.** Um componente de módulo que importa da biblioteca é um erro de arquitetura — o componente global correspondente precisa ser criado antes, mesmo que no primeiro dia ele só repasse props. Única exceção: os primitivos de layout `Container`, `Row`, `Col` e `Stack` (ver `04`).

## Exemplo completo: componente atômico com Composition Pattern

```tsx
// src/components/Input/Input.tsx

/**
 * @module Input
 * @description Componente atômico de input com composition pattern.
 * Suporta label, mensagem de erro e texto auxiliar.
 * Sem condicionais internas — variações de estado são sub-componentes.
 * @since 2024-01-15
 */

import { memo, forwardRef } from 'react';
import type { InputRootProps } from './models/input.types';
import { InputLabel } from './InputLabel';
import { InputError } from './InputError';
import { InputHelper } from './InputHelper';
import { InputInvalid } from './InputInvalid';

/**
 * Input padrão — sem estado de erro.
 * Nunca recebe prop `isInvalid` — a variação inválida é um sub-componente separado.
 */
const InputRoot = memo(forwardRef<HTMLInputElement, InputRootProps>(function Input(
  { id, type = 'text', placeholder, value, onChange, disabled, className = '' },
  ref
) {
  return (
    <input
      ref={ref}
      id={id}
      type={type}
      value={value}
      placeholder={placeholder}
      disabled={disabled}
      onChange={onChange}
      className={`form-control ${className}`}
    />
  );
}));

function Input(props: InputRootProps) {
  return <InputRoot {...props} />;
}

Input.Label   = InputLabel;
Input.Error   = InputError;
Input.Helper  = InputHelper;
/**
 * Variação inválida do input — adiciona classe `is-invalid` do Bootstrap.
 * Usado pelo Container quando há erro de validação no campo.
 * O Root nunca sabe se está inválido — quem decide é quem compõe.
 */
Input.Invalid = InputInvalid;

export { Input };
```

```tsx
// src/components/Input/InputInvalid.tsx
import { memo, forwardRef } from 'react';
import type { InputRootProps } from './models/input.types';

/**
 * Sub-componente de input no estado inválido.
 * Idêntico ao Root, mas com `is-invalid` fixo no className.
 */
export const InputInvalid = memo(forwardRef<HTMLInputElement, InputRootProps>(function InputInvalid(
  { id, type = 'text', placeholder, value, onChange, disabled, className = '' },
  ref
) {
  return (
    <input
      ref={ref}
      id={id}
      type={type}
      value={value}
      placeholder={placeholder}
      disabled={disabled}
      onChange={onChange}
      className={`form-control is-invalid ${className}`}
    />
  );
}));
```

```tsx
// Componente guardião — encapsula a decisão entre Input normal e Input.Invalid.
// Recebe o valor do erro, decide internamente qual variante renderizar.
// src/components/Input/InputFieldGuard.tsx
import { memo, forwardRef } from 'react';
import { Input } from './Input';
import type { InputRootProps } from './models/input.types';

interface InputFieldGuardProps extends InputRootProps {
  error?: string;
}

export const InputFieldGuard = memo(forwardRef<HTMLInputElement, InputFieldGuardProps>(
  function InputFieldGuard({ error, ...props }, ref) {
    if (error) return <Input.Invalid ref={ref} {...props} />;
    return <Input ref={ref} {...props} />;
  }
));
```

```tsx
// Uso no Container — sem nenhuma condicional. Apenas composição.
<div className="mb-3">
  <Input.Label htmlFor="email">E-mail</Input.Label>
  <InputFieldGuard id="email" type="email" value={email} onChange={handleChange} error={errors.email} />
  <Input.Error>{errors.email}</Input.Error>
</div>
```
