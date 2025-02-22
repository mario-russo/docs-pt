# Opções Partilhadas {#shared-options}

## `root` {#root}

- **Tipo:** `string`
- **Predefinido como:** `process.cwd()`

O diretório raiz do projeto (onde o `index.html` está localizado). Pode ser um caminho absoluto, ou um caminho relativo ao diretório de trabalho atual.

Consulte [Raiz do Projeto](/guide/#index-html-and-project-root) por mais detalhes.

## `base` {#base}

- **Tipo:** `string`
- **Predefinido como:** `/`
- **Relacionado ao:** [`server.origin`](/config/server-options#server-origin)

O caminho público de base quando servido em desenvolvimento ou produção. Os valores válidos incluem:

- Nome do caminho da URL absoluta, por exemplo `/foo/`
- URL completa, por exemplo `https://foo.com/` (A parte da origem não será usada em desenvolvimento)
- Sequência de caracteres vazia ou `./` (para o implementação de produção embutida)

Consulte [Caminho de Base Pública](/guide/build#public-base-path) por mais detalhes.

## `mode` {#mode}

- **Tipo:** `string`
- **Predefinido como:** `'development'` para servir, `'production'` para construir

A especificação disto na configuração sobreporá o modo padrão para **tanto servir e construir**. Este valor também pode ser sobreposto através da opção `--mode` da linha de comando.

Consulte [Variáveis de Ambiente e Modos](/guide/env-and-mode) por mais detalhes.

## `define` {#define}

- **Tipo:** `Record<string, string>`

Define as substituições de constante global. As entradas serão definidas como globais durante o desenvolvimento e substituídos estaticamente durante a construção.

A Vite usa as [definições da `esbuild`](https://esbuild.github.io/api/#define) para realizar substituições, assim as expressões de valor devem ser uma sequência de caracteres que contém um valor serializável de JSON (`null`, `boolean`, `number`, `string`, `array`, ou `object`) ou um único identificador. Para valores que não são sequência de caracteres, a Vite converterá automaticamente para uma sequência de caracteres com `JSON.stringify`.

**Exemplo:**

```js
export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify('v1.0.0'),
    __API_URL__: 'window.__backend_api_url',
  }
})
```

:::tip NOTA
Para os utilizadores de TypeScript, certifiquem-se de adicionar as declarações de tipo no ficheiro `env.d.ts` ou `vite-env.d.ts` para obterem as verificações de tipo e o sensor inteligente.

Exemplo:

```ts
// vite-env.d.ts
declare const __APP_VERSION__: string
```

:::

## `plugins` {#plugins}

- **Tipo:** `(Plugin | Plugin[] | Promise<Plugin | Plugin[]>)[]`

Vetor de extensões à usar. As extensões falsas são ignoradas e os vetores de extensões são aplanados. Se uma promessa for retornada, seria resolvida antes da execução. Consulte a [API de Extensão](/guide/api-plugin) por mais detalhes sobre as extensões da Vite.

## `publicDir` {#publicdir}

- **Tipo:** `string | false`
- **Predefinido como:** `"public"`

Diretório para servir como recursos estáticos simples. Os ficheiros neste diretório são servidos no `/` durante o desenvolvimento e copiados para a raiz da `outDir` durante a construção, e são sempre servidos ou copiados como estão sem transformação. O valor pode ser ou um caminho absoluto do sistema de ficheiro ou um caminho relativo à raiz do projeto.

A definição de `publicDir` como `false` desativa esta funcionalidade.

Consulte [O Diretório `public`](/guide/assets#the-public-directory) por mais detalhes.

## `cacheDir` {#cachedir}

- **Tipo:** `string`
- **Predefinido como:** `"node_modules/.vite"`

Diretório para guardar ficheiros de consulta imediata. Os ficheiros neste diretório são dependências pré-empacotadas ou outros ficheiros de consulta imediata gerados pela Vite, os quais podem melhorar o desempenho. Nós podemos usar a opção `--force` ou eliminar manualmente o diretório para regenerar os ficheiros de consulta imediata. O valor pode ser ou um caminho absoluto do sistema de ficheiro ou um caminho relativo a raiz do projeto. Predefinida para `.vite` quando nenhum `package.json` for detetado.

## `resolve.alias` {#resolve-alias}

- **Tipo:**
  `Record<string, string> | Array<{ find: string | RegExp, replacement: string, customResolver?: ResolverFunction | ResolverObject }>`

Será passada ao `@rollup/plugin-alias` como sua [opções de entradas](https://github.com/rollup/plugins/tree/master/packages/alias#entries). Pode ser ou um objeto, ou um vetor de pares `{ find, replacement, customResolver }`.

Quando definirmos pseudónimos aos caminhos do sistema de ficheiro, sempre usamos os caminhos absolutos. Os valores de pseudónimos relativos serão usados como estão e não serão resolvidos nos caminhos do sistema de ficheiro.

Resolução personalizada mais avançada pode ser alcançada através da [extensões](/guide/api-plugin).

:::warning Usando com a Interpretação do Lado do Servidor
Se tivermos configurado pseudónimos para [dependências expostas da interpretação do lado do servidor](/guide/ssr#ssr-externals), podemos querer atribuir pseudónimo os pacotes do `node_modules` de verdade. Ambos [Yarn](https://classic.yarnpkg.com/en/docs/cli/add/#toc-yarn-add-alias) e [pnpm](https://pnpm.io/aliases/) suportam a atribuição pseudónimos através do prefixo `npm:`.
:::

## `resolve.dedupe` {#resolve-dedupe}

- **Tipo:** `string[]`

Se tivermos cópias duplicadas da mesma dependência na nossa aplicação (provavelmente devido ao içamento ou pacotes ligados nos mono-repositórios), usamos esta opção para forçar a Vite à resolver sempre as dependências listadas à mesma cópia (a partir da raiz do projeto).

:::warning Interpretação do Lado do Servidor + Módulo de ECMAScript
Para construções da interpretação do lado do servidor, a eliminação de duplicações não funciona para saídas da construção de módulo de ECMAScript configuradas a partir de `build.rollupOptions.output`. Uma solução é usar as saídas da construção de CJS até o módulo de ECMAScript tiver suporte de extensão melhor para o carregamento de módulo.
:::

## `resolve.conditions` {#resolve-conditions}

- **Tipo:** `string[]`

Além das condições permitidas quando resolvemos as [Exportações Condicionais](https://nodejs.org/api/packages.html#packages_conditional_exports) dum pacote.

Um pacote com exportações condicionais pode ter o seguinte campo `exports` no seu `package.json`:

```json
{
  "exports": {
    ".": {
      "import": "./index.mjs",
      "require": "./index.js"
    }
  }
}
```

Aqui, `import` e `require` são "condições". As condições podem ser encaixados e devem ser especificados a partir do mais específico ao menos específico.

A Vite tem uma lista de "condições permitidas" e corresponderá a primeira condição que está na lista permitida. As condições permitidas padrão são: `import`, `module`, `browser`, `default`, e `production/development` baseado no modo atual. A opção de configuração `resolve.conditions` permite especificar condições permitidas adicionais.

:::warning Resolução das Exportações do Sub-caminho
As chaves de exportação que terminam com "/" está depreciada pela Node e podem não funcionar bem. Contacte o autor do pacote para usar [padrões de sub-caminho `*`](https://nodejs.org/api/packages.html#package-entry-points).
:::

## `resolve.mainFields` {#resolve-mainfields}

- **Tipo:** `string[]`
- **Predefinido como:** `['browser', 'module', 'jsnext:main', 'jsnext']`

A lista de campos no `package.json` para experimentar quando resolvemos um ponto de entrada do pacote. Nota que isto recebe menor prioridade  do que as exportações condicionais resolvidas a partir do campo `exports`: se um ponto de entrada for resolvido com sucesso a partir do `exports`, o campo principal será ignorado.

## `resolve.extensions` {#resolve-extensions}

- **Tipo:** `string[]`
- **Predefinido como:** `['.mjs', '.js', '.mts', '.ts', '.jsx', '.tsx', '.json']`

A lista de extensões de ficheiro para experimentar em importações que omitem as extensões. Nota que **NÃO** é recomendado omitir as extensões em tipos de importação personalizadas (por exemplo, `.vue`), já que isto pode interferir com o ambiente de desenvolvimento integrado e o suporte de tipo.

## `resolve.preserveSymlinks` {#resolve-preservesymlinks}

- **Tipo:** `boolean`
- **Predefinido como:** `false`

A ativação desta definição faz com que a Vite determine a identidade do ficheiro de acordo com o caminho do ficheiro original (isto é, o caminho sem as seguintes ligações simbólicas) ao invés do caminho do ficheiro verdadeiro (isto é, o caminho depois das seguintes ligações simbólicas).

- **Relacionado ao:** [esbuild#preserve-symlinks](https://esbuild.github.io/api/#preserve-symlinks), [webpack#resolve.symlinks](https://webpack.js.org/configuration/resolve/#resolvesymlinks)

## `css.modules` {#css-modules}

- **Tipo:**

  ```ts
  interface CSSModulesOptions {
    scopeBehaviour?: 'global' | 'local'
    globalModulePaths?: RegExp[]
    generateScopedName?:
      | string
      | ((name: string, filename: string, css: string) => string)
    hashPrefix?: string
    /**
     * default: null
     */
    localsConvention?:
      | 'camelCase'
      | 'camelCaseOnly'
      | 'dashes'
      | 'dashesOnly'
      | null
  }
  ```

Configura o comportamento dos módulos de CSS. As opções são passadas ao [`postcss-modules`](https://github.com/css-modules/postcss-modules).

Esta opção não surte qualquer feito quando usamos [CSS Relâmpago](../guide/features.md#lightning-css). Se ativada, [`css.lightningcss.cssModules`](https://lightningcss.dev/css-modules.html) deve ser usado.

## `css.postcss` {#css-postcss}

- **Tipo:** `string | (postcss.ProcessOptions & { plugins?: postcss.AcceptedPlugin[] })`

Incorpora a configuração de PostCSS ou um diretório personalizado a partir do qual procurar a configuração do PostCSS (o padrão é a raiz do projeto).

Para a configuração de PostCSS em linha, espera o mesmo formato que o `postcss.config.js`. Mas para a propriedade `plugins`, apenas o [formato de vetor](https://github.com/postcss/postcss-load-config/blob/main/README.md#array) pode ser usado.

A procura é feita usando o [`postcss-load-config`](https://github.com/postcss/postcss-load-config) e apenas os nomes de ficheiro de configuração suportados são carregados.

Nota que se uma configuração em linha for fornecida, a Vite são procurará por outras fontes de configuração de PostCSS.

## `css.preprocessorOptions` {#css-preprocessoroptions}

- **Tipo:** `Record<string, object>`

Especifica opções à passar aos pré-processadores de CSS. As extensões de ficheiro são usadas como chaves para as opções. As opções suportadas para cada pré-processador pode ser encontrada na sua respetiva documentação:

- `sass`/`scss` - [Opções](https://sass-lang.com/documentation/js-api/interfaces/LegacyStringOptions).
- `less` - [Opções](https://lesscss.org/usage/#less-options).
- `styl`/`stylus` - Apenas a opção [`define`](https://stylus-lang.com/docs/js#define-name-node) é suportada, a qual pode ser passada como um objeto.

Todos as opções dos pré-processadores também suportam a opção `additionalData`, a qual pode ser usada para injetar código adicional para cada conteúdo de estilo. Nota que se incluirmos os estilos verdadeiros e não apenas variáveis, estes estilos serão duplicados no pacote final.

Por exemplo:

```js
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `$injectedColor: orange;`,
      },
      less: {
        math: 'parens-division',
      },
      styl: {
        define: {
          $specialColor: new stylus.nodes.RGBA(51, 197, 255, 1),
        },
      },
    },
  },
})
```

## `css.devSourcemap` {#css-devsourcemap}

- **Experimental:** [Comente](https://github.com/vitejs/vite/discussions/13845)
- **Tipo:** `boolean`
- **Predefinido como:** `false`

Se for verdadeiro ativa os mapas de código-fonte durante o desenvolvimento.

## `css.transformer` {#css-transformer}

- **Experimental:** [Comente](https://github.com/vitejs/vite/discussions/13835)
- **Tipo:** `'postcss' | 'lightingcss'`
- **Predefinido como:** `'postcss'`

Seleciona o motor usado para o processamento de CSS. Consulte a [CSS Relâmpago](../guide/features#lightning-css) por mais informação.

## `css.lightningcss` {#css-lightningcss}

- **Experimental:** [Comente](https://github.com/vitejs/vite/discussions/13835)
- **Tipo:**

```js
import type {
  CSSModulesConfig,
  Drafts,
  Features,
  NonStandard,
  PseudoClasses,
  Targets,
} from 'lightningcss'
```

```js
{
  targets?: Targets
  include?: Features
  exclude?: Features
  drafts?: Drafts
  nonStandard?: NonStandard
  pseudoClasses?: PseudoClasses
  unusedSymbols?: string[]
  cssModules?: CSSModulesConfig,
  // ...
}
```

Configura a CSS Relâmpago. As opções de transformação completa podem ser encontradas no [repositório da CSS Relâmpago](https://github.com/parcel-bundler/lightningcss/blob/master/node/index.d.ts).

## `json.namedExports` {#json-namedexports}

- **Tipo:** `boolean`
- **Predefinido como:** `true`

Se for verdadeiro suporta as importações nomeadas a partir de ficheiros `.json`.

## `json.stringify` {#json-stringify}

- **Tipo:** `boolean`
- **Predefinido como:** `false`

Se definido para `true`, o JSON importado será transformado em `export default JSON.parse("...")` o que é significativamente mais otimizado do que literais de `Object`, especialmente quando o ficheiro JSON for grande.

A ativação disto desativa as importações nomeadas.

## `esbuild` {#esbuild}

- **Tipo:** `ESBuildOptions | false`

`ESBuildOptions` estende as [opções de transformação do próprio `esbuild`](https://esbuild.github.io/api/#transform). O caso de uso mais comum é a personalização de JSX:

```js
export default defineConfig({
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment'
  }
})
```

Por padrão, a `esbuild` é aplicada aos ficheiros `ts`, `jsx`, `tsx`. Nós podemos personalizar isto com `esbuild.include` e `esbuild.exclude`, as quais pode, ser uma expressão regular, um padrão [`picomatch`](https://github.com/micromatch/picomatch#globbing-features), ou um vetor de ambos.

Além disto, também podemos usar `esbuild.jsxInject` para injetar automaticamente importações auxiliares de JSX para cada ficheiro transformado pela `esbuild`:

```js
export default defineConfig({
  esbuild: {
    jsxInject: `import React from 'react'`
  }
})
```

Quando [`build.minify`](./build-options#build-minify) for `true`, todas otimizações de compactação são aplicadas por padrão. Para desativar [certos aspetos](https://esbuild.github.io/api/#minify) disto, definimos quaisquer uma das opções `esbuild.minifyIdentifiers`, `esbuild.minifySyntax`, ou `esbuild.minifyWhitespace` para `false`. Nota que a opção `esbuild.minify` não pode ser usada para sobrepor a `build.minify`.

Definimos para `false` para desativar as transformações da `esbuild`.

## `assetsInclude` {#assetsinclude}

- **Tipo:** `string | RegExp | (string | RegExp)[]`
- **Related:** [Manipulação de Recurso Estático](/guide/assets)

Especifica [padrões de `picomatch`](https://github.com/micromatch/picomatch#globbing-features) adicionais à serem tratado como recursos estáticos para:

- Eles sejam excluídos a partir da conduta de transformação de extensão quando referenciados a partir do HTML ou diretamente requisitados sobre `fetch` ou XHR.

- A importação deles a partir da JavaScript retornará as suas sequências de caracteres de URL resolvidas (isto pode ser sobrescrito se tivermos uma extensão `enforce: 'pre'` para manipular o tipo de recurso de maneira diferente).

A lista de tipo de recurso embutido pode ser encontrada no [`node/constants.ts`](https://github.com/vitejs/vite/blob/main/packages/vite/src/node/constants.ts).

**Exemplo:**

```js
export default defineConfig({
  assetsInclude: ['**/*.gltf'],
})
```

## `logLevel` {#loglevel}

- **Tipo:** `'info' | 'warn' | 'error' | 'silent'`

Ajusta a verbosidade da saída da consola. O padrão é `'info'`.

## `customLogger` {#customlogger}

- **Tipo:**

  ```ts
  interface Logger {
    info(msg: string, options?: LogOptions): void
    warn(msg: string, options?: LogOptions): void
    warnOnce(msg: string, options?: LogOptions): void
    error(msg: string, options?: LogErrorOptions): void
    clearScreen(type: LogType): void
    hasErrorLogged(error: Error | RollupError): boolean
    hasWarned: boolean
  }
  ```

Usa um registador personalizado para registar mensagens. Nós podemos usar a API `createLogger` da Vite para obter o registador padrão e personalizá-lo para, por exemplo, mudar a mensagem ou filtrar certos avisos.

```js
import { createLogger, defineConfig } from 'vite'

const logger = createLogger()
const loggerWarn = logger.warn

logger.warn = (msg, options) => {
  // Ignorar o aviso de ficheiros de CSS vazios
  if (msg.includes('vite:css') && msg.includes(' is empty')) return
  loggerWarn(msg, options)
}

export default defineConfig({
  customLogger: logger,
})
```

## `clearScreen` {#clearscreen}

- **Tipo:** `boolean`
- **Predefinido como:** `true`

Definimos para `false` para impedir a Vite de limpar o tela do terminal quando registamos certas mensagens. Através da linha de comando, usamos `--clearScreen false`.

## `envDir` {#envdir}

- **Tipo:** `string`
- **Predefinido como:** `root`

O diretório a partir do qual os ficheiros `.env` são carregados. Pode ser um caminho absoluto, ou um caminho relativo a raiz do projeto.

Consulte [Ficheiros de Configuração de Ambiente](/guide/env-and-mode#env-files) por mais detalhes sobre ficheiros de ambiente.

## `envPrefix` {#envprefix}

- **Tipo:** `string | string[]`
- **Predefinido como:** `VITE_`

As variáveis de ambiente começando com `envPrefix` serão expostas ao código-fonte do nosso cliente através de `import.meta.env`.

:::warning NOTAS DE SEGURANÇA
O `envPrefix` não deve ser definido como `''`, o que exporá todas as nossas variáveis de ambiente e causará vazamentos inesperados de informações sensíveis. A Vite lançará um erro quando detetar `''`.

Se gostaríamos de expor uma variável não prefixada, podemos usar [`define`](#define) para a expor:

```js
define: {
  'import.meta.env.ENV_VARIABLE': JSON.stringify(process.env.ENV_VARIABLE)
}
```

:::

## `appType` {#apptype}

- **Tipo:** `'spa' | 'mpa' | 'custom'`
- **Predefinido como:** `'spa'`

Se a nossa aplicação for uma aplicação duma única página, uma [aplicação de várias páginas](../guide/build#multi-page-app), ou uma aplicação personalizada (interpretação do lado do servidor e abstrações com a manipulação de HTML personalizada):


- `'spa'`: inclui intermediário de retrocesso de aplicação duma única página e configura [`sirv`](https://github.com/lukeed/sirv) com `single: true` na pré-visualização
- `'mpa'`: inclui intermediários de HTML
- `'custom'`: não inclui intermediários de HTML

Saiba mais no [Guia de Interpretação do Lado do Servidor](/guide/ssr#vite-cli) da Vite. Relacionado ao [`server.middlewareMode`](./server-options#server-middlewaremode).
