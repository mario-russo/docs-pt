# Construindo para Produção {#building-for-production}

Quando for o momento de implementar a nossa aplicação em produção, simplesmente executamos o comando `vite build`. Por padrão, este usa `<root>/index.html` como o ponto de entrada da construção, e produz um pacote de aplicação que é adequado para ser servido sobre um serviço de hospedagem estática. Consultar o [Implementando uma Aplicação Estática](./static-deploy) por orientações sobre os serviços populares.

## Compatibilidade do Navegador {#browser-compatibility}

O pacote de produção pressupõe suporte para JavaScript moderno. Por padrão, a Vite dirige-se aos navegadores que suportam os [módulos de ECMAScript nativo](https://caniuse.com/es6-module), [importação dinâmica de Módulo de ECMAScript nativo](https://caniuse.com/es6-module-dynamic-import), e [`import.meta`](https://caniuse.com/mdn-javascript_operators_import_meta):

- Chrome >=87
- Firefox >=78
- Safari >=14
- Edge >=88

Nós podemos especificar alvos personalizados através da [opção de configuração `build.target`](/config/build-options#build-target), onde o alvo inferior é `es2015`.

Nota que por padrão, a Vite apenas manipula os transformações de sintaxe e **não cobre os tapa-buracos de funcionalidades**. Nós podemos consultar [Polyfill.io](https://polyfill.io/) que é um serviço que gera automaticamente pacotes de tapa-buracos baseado na sequência de caracteres do `UserAgent` do navegador do utilizador.

Os navegadores antigos podem ser suportados através do [`@vitejs/plugin-legacy`](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy), que gerará os pedaços para os navegadores antigos e tapa-buracos de funcionalidade da linguagem do ECMAScript correspondente. Os pedaços para os navegadores antigos são carregados condicionalmente apenas nos navegadores que não tem suporte ao Módulo de ECMAScript nativo.

## Caminho de Base Pública {#public-base-path}

- Relacionado com: [Manipulação de Recurso](assets)

Se estivermos implementando o nosso projeto em produção sob um caminho público encaixado, simplesmente especificamos a [opção de configuração `base`](/config/shared-options#base) e todos os caminhos do recurso serão reescritos em conformidade. Esta opção também pode ser especificada como um sinal da linha de comando, por exemplo, `vite build --base=/my/public/path/`.

As URLs do recurso importado da JavaScript, as referências da `url()` da CSS, e as referências do recurso nos nossos ficheiros `.html` são todos automaticamente ajustados para respeitarem esta opção durante a construção.

A exceção é quando precisamos concatenar dinamicamente as URLs em tempo real. Neste caso, podemos usar a variável `import.meta.env.BASE_URL` injetado globalmente que será o caminho de base pública. Nota que esta variável é substituída estaticamente durante a construção, então deve aparecer exatamente como está (isto é, `import.meta.env['BASE_URL']` não funcionará).

Para controlo avançado do caminho de base, consultar as [Opções Avançadas da Base](#advanced-base-options).

## Personalizando a Construção {#customizing-the-build}

A construção pode ser personalizada através de várias [opções de configuração da construção](/config/build-options). Especificamente, podemos ajustar diretamente as [opções de Rollup](https://rollupjs.org/configuration-options/) subjacentes através da `build.rollupOptions`:

```js
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      // https://rollupjs.org/configuration-options/
    },
  },
})
```

Por exemplo, podemos especificar várias saídas da Rollup com as extensões que apenas são aplicadas durante a construção.

## Estratégia de Fragmentação {#chunking-strategy}

Nós podemos configurar como os pedaços são separados usando a `build.rollupOptions.output.manualChunks` (consultar a [documentação da Rollup](https://rollupjs.org/configuration-options/#output-manualchunks)). Até a Vite 2.8, a estratégia de fragmentação padrão dividia os pedaços em `index` e `vendor`. É uma boa estratégia para algumas aplicações de página única, mas é difícil fornecer uma solução geral para todo caso de uso do alvo da Vite. Desde a Vite 2.9, `manualChunks` já não é modificada por padrão. Nós podemos continuar a usar a estratégia de pedaço de fornecedor dividido adicionando a `splitVendorChunkPlugin` no ficheiro de configuração:

```js
// vite.config.js
import { splitVendorChunkPlugin } from 'vite'
export default defineConfig({
  plugins: [splitVendorChunkPlugin()],
})
```

Está estratégia também é fornecida como uma fábrica `splitVendorChunk({ cache: SplitVendorChunkCache })`, no caso de ser necessária uma composição com lógica personalizada. `cache.reset()` precisa ser chamada no `buildStart` para modo de observação da construção para funcionar corretamente neste caso.

:::warning AVISO
Nós devemos usar a forma de função `build.rollupOptions.output.manualChunks` quando usamos esta extensão. Se forma de objeto for usada, a extensão não surtirá nenhum efeito.
:::

## Manipulação do Erro de Carregamento {#load-error-handling}

A Vite emite o evento `vite:preloadError` quando não consegue carregar as importações dinâmicas. `event.payload` contém o erro de importação original. Se chamarmos `event.preventDefault()`, o erro não será lançado:

```js
window.addEventListener('vite:preloadError', (event) => {
  window.reload() // por exemplo, atualizar a página
})
```

Quando uma nova implementação de produção ocorre, o serviço de hospedagem pode eliminar os recursos das implementações anteriores de produção. Como resultado, um utilizador que visitou a nossa aplicação antes da nova implementação de produção pode encontrar um erro de importação. Este erro acontece porque os recursos executando sobre o dispositivo do utilizador estão desatualizados e esta tenta importar o pedaço antigo correspondente, que foi eliminado. Este evento é útil para solucionar esta situação.

## Reconstruir Sobre as Mudanças dos Ficheiros {#rebuild-on-files-changes}

Nós podemos ativar o observador da Rollup com `vite build --watch`. Ou, podemos ajustar diretamente a [`WatcherOptions`](https://rollupjs.org/configuration-options/#watch) subjacente através da `build.watch`:

```js
// vite.config.js
export default defineConfig({
  build: {
    watch: {
      // https://rollupjs.org/configuration-options/#watch
    },
  },
})
```

Com a opção `--watch` ativada, as mudanças ao `vite.config.js`, bem como às quaisquer ficheiros à serem empacotados, acionarão uma reconstrução.

## Aplicação de Várias Páginas {#multi-page-app}

Suponhamos que temos a seguinte estrutura de código-fonte:

```
├── package.json
├── vite.config.js
├── index.html
├── main.js
└── nested
    ├── index.html
    └── nested.js
```

Durante o desenvolvimento, simplesmente navegamos a ou ligamos ao `/nested/` - este funciona como esperado, tal como para um servidor de ficheiro estático normal.

Durante a construção, tudo o que precisamos fazer é especificar vários ficheiros `.html` como pontos de entrada:

```js
// vite.config.js
import { resolve } from 'path'
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
        nested: resolve(__dirname, 'nested/index.html'),
      },
    },
  },
})
```

Se especificarmos uma raiz diferente, devemos lembrar de que `__dirname` ainda será a pasta do nosso ficheiro `vite.config.js` quando resolvermos os caminhos de entrada. Portanto, precisaremos adicionar a nossa entrada `root` aos argumentos para `resolve`.

Nota que para os ficheiros HTML, a Vite ignora o nome dado à entrada no objeto `rollupOptions.input` e ao invés disto respeita o identificador único resolvido do ficheiro quando geramos o recurso de HTML na pasta de distribuição (`dist`). Isto garante uma estrutura consistente com a maneira que o servidor de desenvolvimento funciona.

## Modo de Biblioteca {#library-mode}

Quando estivermos desenvolvimento uma biblioteca orienta ao navegador, estaremos provavelmente gastando a maior parte do tempo numa página de teste ou demonstração que importa de fato a nossa biblioteca. Com a Vite, podemos usar o nosso `index.html` para este propósito para obter uma experiência de desenvolvimento mais suave.

Na hora de empacotar a nossa biblioteca para distribuição, usamos a [ opção de configuração `build.lib`](/config/build-options#build-lib). Temos que certificar-nos de também expomos quaisquer dependências que não queremos empacotar na nossa biblioteca, por exemplo, `vue` ou `react`:

```js
// vite.config.js
import { resolve } from 'path'
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    lib: {
      // Também poderia ser um diretório ou
      // vetor de vários pontos de entrada
      entry: resolve(__dirname, 'lib/main.js'),
      name: 'MyLib',
      // as extensões apropriadas serão adicionadas
      fileName: 'my-lib',
    },
    rollupOptions: {
      // certificar de expor as dependências que não
      //  devem ser empacotadas na nossa biblioteca
      external: ['vue'],
      output: {
        // fornecer as variáveis globais para usar na
        // construção UMD para as dependências expostas
        globals: {
          vue: 'Vue',
        },
      },
    },
  },
})
```

O ficheiro de entrada conteria as exportações que poderiam ser importadas pelos utilizadores do nosso pacote:

```js
// lib/main.js
import Foo from './Foo.vue'
import Bar from './Bar.vue'
export { Foo, Bar }
```

Executar `vite build` com esta configuração usa uma predefinição de Rollup que é orientada para bibliotecas de entrega e produz dois formatos de pacote: `es` e `umd` (configurável através de `build.lib`):

```
$ vite build
building for production...
dist/my-lib.js      0.08 kB / gzip: 0.07 kB
dist/my-lib.umd.cjs 0.30 kB / gzip: 0.16 kB
```

O `package.json` recomendado para a nossa biblioteca:

```json
{
  "name": "my-lib",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-lib.umd.cjs",
  "module": "./dist/my-lib.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.js",
      "require": "./dist/my-lib.umd.cjs"
    }
  }
}
```

Ou, se estivermos expondo vários pontos de entrada:

```json
{
  "name": "my-lib",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-lib.cjs",
  "module": "./dist/my-lib.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.js",
      "require": "./dist/my-lib.cjs"
    },
    "./secondary": {
      "import": "./dist/secondary.js",
      "require": "./dist/secondary.cjs"
    }
  }
}
```

:::tip EXTENSÕES DE FICHEIRO
Se o `package.json` não contiver `"type": "module"`, a Vite gerará extensões de ficheiro diferentes para fins de compatibilidade com a Node.js. A `.js` tornar-se-á em `.mjs` e `.cjs` tornar-se-á em `.js`.
:::

:::tip VARIÁVEIS DE AMBIENTE
No modo de biblioteca, todos os usos de [`import.meta.env.*`](./env-and-mode) serão substituídos estaticamente quando construirmos para produção. No entanto, os usos de `process.env.*` não são, então para que os consumidores da nossa biblioteca possam mudá-lo dinamicamente. Se isto for indesejável, podemos usar `define: { 'process.env.NODE_ENV': '"production"' }` por exemplo, os substituir estaticamente, ou usamos [`esm-env`](https://github.com/benmccann/esm-env) para melhor compatibilidade com os empacotadores e executores.
:::

:::warning USO AVANÇADO
O modo de biblioteca inclui uma simples e opiniosa configuração para as bibliotecas orientadas ao navegador e para a abstração de JavaScript. Se estivermos a construir bibliotecas que não destinadas ao navegador, ou exigem fluxos de construção avançados, podemos usar diretamente a [Rollup](https://rollupjs.org) ou [esbuild](https://esbuild.github.io).
:::

## Opções de Base Avançadas {#advanced-base-options}

:::warning AVISO
Esta funcionalidade é experimental. [Comentar](https://github.com/vitejs/vite/discussions/13834).
:::

Para os casos de uso avançados, os recursos e ficheiros públicos implementados em produção podem estar em caminhos diferentes, por exemplo para usarmos diferentes estratégias de armazenamento de consulta imediata. Um utilizador pode escolher implementar em produção nestes três caminhos diferentes:

- Os ficheiros de HTML de entrada gerados (os quais podem ser processados durante a interpretação do lado do servidor)
- Os recursos de nome embaralhado gerados (JavaScript, CSS, e outros tipos de ficheiro como imagens)
- Os [ficheiros públicos](assets#the-public-directory) copiados

Uma única [base](#public-base-path) estática não é o suficiente nestes cenários. A Vite fornece suporte experimental para opções de base avançadas durante a construção, usando a `experimental.renderBuiltUrl`;

```ts
experimental: {
  renderBuiltUrl(filename: string, { hostType }: { hostType: 'js' | 'css' | 'html' }) {
    if (hostType === 'js') {
      return { runtime: `window.__toCdnUrl(${JSON.stringify(filename)})` }
    } else {
      return { relative: true }
    }
  }
}
```

Se os recursos de nome embaralhado e os ficheiros públicos são forem implementados em produção em conjunto, as opções para grupo podem ser definidas de maneira independente usando a `type` de recurso incluída no segundo parâmetro `context` dado à função:

```ts
experimental: {
  renderBuiltUrl(filename: string, { hostId, hostType, type }: { hostId: string, hostType: 'js' | 'css' | 'html', type: 'public' | 'asset' }) {
    if (type === 'public') {
      return 'https://www.domain.com/' + filename
    }
    else if (path.extname(hostId) === '.js') {
      return { runtime: `window.__assetsPath(${JSON.stringify(filename)})` }
    }
    else {
      return 'https://cdn.domain.com/assets/' + filename
    }
  }
}
```
