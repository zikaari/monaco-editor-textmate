Module to wire `monaco-textmate` with `monaco-editor`

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

    const grammars = new Map()
    grammars.set('css', 'CSS')

    await wireTmGrammars(monaco, registry, grammars)

    var editor = monaco.editor.create(document.getElementById('container'), {
        value: [
            'html, body {',
            '    margin: 0;',
            '}'
        ].join('\n'),
        language: 'css'
    })
}
```
## License
MIT