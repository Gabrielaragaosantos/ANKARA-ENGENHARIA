# Visualizador IFC 3D - UNILA Campus Arandu

Visualizador web de arquivos IFC (Industry Foundation Classes) desenvolvido com Three.js e web-ifc.

![Status](https://img.shields.io/badge/status-production-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

## Demo Online

**[Acessar Visualizador](https://SEU-USUARIO.github.io/visualizador-ifc-unila/)**

> Substitua `SEU-USUARIO` pelo seu nome de usuario do GitHub apos publicar.

## Funcionalidades

- Carregar arquivos IFC locais
- Renderizacao 3D com cores originais do modelo
- Controles de orbita (rotacao, zoom, pan)
- Enquadramento automatico do modelo
- Toggle de grid e eixos
- Painel de informacoes (nome, elementos, dimensoes)
- Console de debug integrado
- Responsivo para mobile

## Tecnologias

- **Three.js** 0.152.0 - Renderizacao 3D via CDN
- **web-ifc** 0.0.46 - Parser de arquivos IFC via CDN
- **HTML5/CSS3/ES6** - Interface web moderna

## Como Usar

### Opcao 1: GitHub Pages (Recomendado)

Acesse a demo online e carregue seus arquivos IFC diretamente no navegador.

### Opcao 2: Localmente

```bash
# Clone o repositorio
git clone https://github.com/SEU-USUARIO/visualizador-ifc-unila.git
cd visualizador-ifc-unila

# Inicie um servidor HTTP
python -m http.server 5500

# Acesse
# http://localhost:5500/docs/
```

## Estrutura do Projeto

```
visualizador-ifc-unila/
├── docs/                    # GitHub Pages (producao)
│   └── index.html          # Visualizador para web
├── index-cdn.html          # Versao de desenvolvimento
├── SOLUCAO_WASM_IFC.md     # Documentacao tecnica da solucao WASM
└── README.md               # Este arquivo
```

## Configurar GitHub Pages

1. Faca push do repositorio para o GitHub
2. Va em **Settings** > **Pages**
3. Em **Source**, selecione **Deploy from a branch**
4. Selecione a branch `main` e a pasta `/docs`
5. Clique em **Save**
6. Aguarde alguns minutos e acesse a URL gerada

## Navegacao 3D

- **Rotacionar:** Botao esquerdo do mouse + arrastar
- **Pan:** Botao direito do mouse + arrastar
- **Zoom:** Scroll do mouse
- **Enquadrar:** Botao "Enquadrar Modelo"

## Limitacoes

- Arquivos IFC muito grandes (>200MB) podem demorar para carregar
- Requer navegador moderno com suporte a WebAssembly
- O processamento e feito no navegador do usuario

## Testado Com

- Arquivo: ESG_R_0001_PE_MODE_R00.ifc (104.31 MB)
- Resultado: 8.002 meshes gerados com sucesso

## Solucao Tecnica

Este projeto usa uma abordagem CDN para contornar incompatibilidades entre Vite/ESBuild e o WebAssembly compilado pelo Emscripten. Veja `SOLUCAO_WASM_IFC.md` para detalhes tecnicos.

## Creditos

Desenvolvido por **Gabriel** - Coordenador BIM na UNILA
Projeto: UNILA Campus Arandu

Powered by:
- [Three.js](https://threejs.org/)
- [web-ifc](https://github.com/ThatOpen/engine_web-ifc)

## Licenca

MIT License - Livre para uso e modificacao.
