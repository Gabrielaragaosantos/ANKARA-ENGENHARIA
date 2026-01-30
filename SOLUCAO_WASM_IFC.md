# SOLUCAO DO ERRO WEBASSEMBLY NO VISUALIZADOR IFC

## Data da Resolucao
**30 de Janeiro de 2026 - 08:32**

## Projeto
**Visualizador IFC 3D - UNILA Campus Arandu**

## Responsavel Tecnico
Gabriel - Coordenador BIM na UNILA

---

## PROBLEMA ORIGINAL

### Erro Encontrado
```
WebAssembly.instantiate(): Import #0 "a": module is not an object or function
```

### Contexto
O projeto utilizava `@thatopen/components` (ThatOpen Engine) com Vite como bundler para criar um visualizador IFC. O erro ocorria na inicializacao do WebAssembly do `web-ifc`, impedindo completamente o carregamento de arquivos IFC.

### Sintomas
- Erro de WASM na inicializacao
- Mensagem "http://localhost:3001/undefined" - Worker nao conseguia resolver path do WASM
- Servidor retornava HTML ao inves do arquivo WASM
- Erro persistia tanto em desenvolvimento quanto em producao

---

## CAUSA RAIZ IDENTIFICADA

### Incompatibilidade Vite + web-ifc
O **Vite** (e seu motor ESBuild) transforma o codigo JavaScript durante o processo de bundling. O problema e que o `web-ifc` e compilado com **Emscripten** (C++ para WebAssembly) e seu arquivo `web-ifc-api.js` contem codigo especifico que NAO pode ser transformado.

### Detalhes Tecnicos
1. **ESBuild pre-bundling**: O Vite usa ESBuild para pre-processar dependencias, o que corrompe o wrapper JavaScript do WASM
2. **Import maps do Emscripten**: O codigo gerado pelo Emscripten usa padroes de import que o Vite nao consegue processar corretamente
3. **Worker resolution**: O web-ifc usa Web Workers que dependem de paths relativos que o Vite resolve incorretamente

### Tentativas que NAO Funcionaram
1. `optimizeDeps.exclude: ['web-ifc']` - Nao suficiente
2. `vite-plugin-wasm` + `vite-plugin-top-level-await` - Nao resolveu
3. Copiar WASM para pasta `public/` - Path ainda era resolvido incorretamente
4. Build de producao (`vite build`) - Mesmo erro
5. Migrar para `web-ifc-three` - Mesmo problema de WASM
6. Diferentes versoes do web-ifc (0.0.44, 0.0.46, 0.0.55, 0.0.57, 0.0.74, 0.0.75) - Todas falharam com Vite

---

## SOLUCAO IMPLEMENTADA

### Abordagem: Bypass Completo do Bundler

A solucao foi **eliminar o Vite do processo de carregamento do web-ifc**, usando CDN diretamente no navegador.

### Arquitetura da Solucao

```
+------------------+     +------------------+     +------------------+
|   index-cdn.html |---->|  Three.js (CDN)  |---->|   Renderizacao   |
|                  |     |   unpkg.com      |     |      3D          |
+------------------+     +------------------+     +------------------+
         |
         v
+------------------+     +------------------+     +------------------+
| Dynamic Import   |---->| web-ifc (CDN)    |---->|  Parsing IFC     |
| await import()   |     | jsdelivr.net     |     |  + Geometrias    |
+------------------+     +------------------+     +------------------+
```

### Codigo da Solucao

#### 1. Import Map para Three.js (ES Modules)
```html
<script type="importmap">
{
    "imports": {
        "three": "https://unpkg.com/three@0.152.0/build/three.module.js",
        "three/addons/": "https://unpkg.com/three@0.152.0/examples/jsm/"
    }
}
</script>
```

#### 2. Dynamic Import do web-ifc
```javascript
async function initWebIFC() {
    // Importar web-ifc como ES Module dinamicamente
    const WebIFC = await import('https://cdn.jsdelivr.net/npm/web-ifc@0.0.46/web-ifc-api.js');

    ifcAPI = new WebIFC.IfcAPI();
    ifcAPI.SetWasmPath('https://cdn.jsdelivr.net/npm/web-ifc@0.0.46/');

    await ifcAPI.Init();
}
```

#### 3. Processamento de Geometrias IFC
```javascript
ifcAPI.StreamAllMeshes(modelID, (mesh) => {
    const geoms = mesh.geometries;
    for (let i = 0; i < geoms.size(); i++) {
        const pg = geoms.get(i);
        const geom = ifcAPI.GetGeometry(modelID, pg.geometryExpressID);

        // Extrair vertices (6 floats por vertice: x,y,z, nx,ny,nz)
        const vData = ifcAPI.GetVertexArray(geom.GetVertexData(), geom.GetVertexDataSize());
        const iData = ifcAPI.GetIndexArray(geom.GetIndexData(), geom.GetIndexDataSize());

        // Criar BufferGeometry do Three.js
        const positions = [];
        const normals = [];
        for (let j = 0; j < vData.length; j += 6) {
            positions.push(vData[j], vData[j+1], vData[j+2]);
            normals.push(vData[j+3], vData[j+4], vData[j+5]);
        }

        // ... criar mesh e adicionar a cena
    }
});
```

### Versoes Utilizadas
- **Three.js**: 0.152.0 (via unpkg CDN)
- **web-ifc**: 0.0.46 (via jsdelivr CDN)
- **Servidor**: Python HTTP Server (porta 5500)

---

## RESULTADO FINAL

### Teste Realizado
- **Arquivo**: ESG_R_0001_PE_MODE_R00.ifc
- **Tamanho**: 104.31 MB
- **Resultado**: 8.002 meshes gerados com sucesso

### Log de Sucesso
```
[08:31:59] Iniciando...
[08:31:59] Inicializando Three.js...
[08:31:59] Three.js inicializado!
[08:31:59] Carregando web-ifc...
[08:32:00] Modulo web-ifc importado
[08:32:00] Inicializando WASM...
[08:32:00] web-ifc pronto!
[08:32:00] VISUALIZADOR PRONTO!
[08:32:21] Selecionado: ESG_R_0001_PE_MODE_R00.ifc
[08:32:21] Carregando: ESG_R_0001_PE_MODE_R00.ifc (104.31 MB)
[08:32:21] Arquivo lido: 109381787 bytes
[08:32:21] Modelo aberto (ID: 0)
[08:32:23] Gerados 8002 meshes
[08:32:23] Modelo carregado com sucesso!
```

### Funcionalidades Operacionais
- [x] Carregamento de arquivos IFC via input file
- [x] Renderizacao 3D com cores corretas do IFC
- [x] Controles de orbita (rotacao, zoom, pan)
- [x] Enquadramento automatico do modelo
- [x] Toggle de grid
- [x] Painel de informacoes (nome, elementos, dimensoes)
- [x] Console de debug integrado
- [x] Liberacao de memoria ao trocar modelos

---

## COMO USAR A SOLUCAO

### Opcao 1: Servidor Python (Recomendado)
```bash
cd "caminho/para/visualizador-ifc-projeto"
python -m http.server 5500
```
Acessar: `http://localhost:5500/index-cdn.html`

### Opcao 2: Live Server (VS Code)
Clicar com botao direito em `index-cdn.html` > "Open with Live Server"

### Opcao 3: Qualquer servidor HTTP
O arquivo `index-cdn.html` funciona com qualquer servidor HTTP estatico.

---

## LICOES APRENDIDAS

1. **Bundlers vs WASM**: Ferramentas como Vite/Webpack podem ser incompativeis com codigo WASM compilado por Emscripten
2. **CDN como alternativa**: Para bibliotecas problematicas, usar CDN diretamente pode ser mais confiavel
3. **Dynamic imports**: `await import()` permite carregar modulos ES de CDNs externos
4. **Cache do navegador**: Sempre usar Ctrl+Shift+N (aba anonima) ou limpar cache ao testar mudancas
5. **Versao especifica**: web-ifc@0.0.46 funcionou bem via CDN com dynamic import

---

## ARQUIVOS RELEVANTES

- `index-cdn.html` - Solucao final funcional (sem bundler)
- `index.html` - Versao original com Vite (NAO FUNCIONA)
- `main.js` - Codigo original com @thatopen/components (NAO FUNCIONA)
- `SOLUCAO_WASM_IFC.md` - Este documento

---

## PROXIMOS PASSOS SUGERIDOS

1. **Otimizacao de performance**: Implementar LOD (Level of Detail) para modelos grandes
2. **Selecao de elementos**: Adicionar raycasting para selecionar elementos IFC
3. **Arvore de propriedades**: Exibir propriedades IFC dos elementos selecionados
4. **Export**: Permitir exportar visualizacoes como imagem
5. **Medidas**: Ferramenta de medicao entre pontos

---

**Documento criado em**: 30/01/2026
**Projeto**: Visualizador IFC UNILA
**Status**: PROBLEMA RESOLVIDO COM SUCESSO
