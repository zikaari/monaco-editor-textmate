# Wire `monaco-textmate` with `monaco-editor`

**Note:** *The latest version of this library requires `monaco-editor` version `0.21.2` and up. If you wish to use an older version of `monaco-editor` `0.19.x` or below, you should use an older version `2.2.2` of this library.*

## Install

```sh
npm i monaco-editor-textmate
```

Please install peer dependencies if you haven't already
```sh
npm i monaco-textmate monaco-editor onigasm
```
## Usage

```javascript
import { loadWASM } from 'onigasm' // peer dependency of 'monaco-textmate'
import { Registry } from 'monaco-textmate' // peer dependency
import { wireTmGrammars } from 'monaco-editor-textmate'

export async function liftOff() {
    await loadWASM(`path/to/onigasm.wasm`) // See https://www.npmjs.com/package/onigasm#light-it-up

    const registry = new Registry({
        getGrammarDefinition: async (scopeName) => {
            return {
                format: 'json',
                content: await (await fetch(`static/grammars/css.tmGrammar.json`)).text()
            }
        }
    })

    // map of monaco "language id's" to TextMate scopeNames
    const grammars = new Map()
    grammars.set('css', 'source.css')
    grammars.set('html', 'text.html.basic')
    grammars.set('typescript', 'source.ts')

    // monaco's built-in themes aren't powereful enough to handle TM tokens
    // https://github.com/Nishkalkashyap/monaco-vscode-textmate-theme-converter#monaco-vscode-textmate-theme-converter
    monaco.editor.defineTheme('vs-code-theme-converted', {
        // ... use `monaco-vscode-textmate-theme-converter` to convert vs code theme and pass the parsed object here
    });
    
    var editor = monaco.editor.create(document.getElementById('container'), {
        value: [
            'html, body {',
            '    margin: 0;',
            '}'
        ].join('\n'),
        language: 'css', // this won't work out of the box, see below for more info,
        theme: 'vs-code-theme-converted' // very important, see comment above
    })
    
    await wireTmGrammars(monaco, registry, grammars, editor)
}
```

## Limitation

**Version Issue!**

The latest version of this package requires `monaco-editor` version `0.21.1` and up. Version `2.2.2`
was the last version to support `monaco-editor` version `0.19.x` or below.

`monaco-editor` distribution comes with built-in tokenization support for few languages. Because of this `monaco-editor-textmate` [cannot
be used with `monaco-editor`](https://github.com/Microsoft/monaco-editor/issues/884) without some modification, see explanation of this problem [here](https://github.com/Microsoft/monaco-editor/issues/884#issuecomment-389778611).

### Solution

To get `monaco-editor-textmate` working with `monaco-editor`, you're advised to use Webpack with [`monaco-editor-webpack-plugin`](https://www.npmjs.com/package/monaco-editor-webpack-plugin) which allows you to control which of "built-in" languages should `monaco-editor` use/bundle, leaving the rest.
With that control you must *exclude* any/all languages for which you'd like to use TextMate grammars based tokenization instead.

## License
MIT
