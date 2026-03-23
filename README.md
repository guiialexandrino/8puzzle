# 8-Puzzle Solver

Aplicação web para resolver o problema clássico do 8-puzzle com busca heurística e visualização dos resultados.

## Preview

https://8puzzle-app.netlify.app/

## Sobre o projeto

Este projeto foi desenvolvido como trabalho da disciplina INE 5633 - Sistemas Inteligentes.

O app permite:

- Escolher diferentes estratégias de busca
- Selecionar desafios prontos
- Informar um desafio personalizado
- Executar a resolução e acompanhar métricas
- Navegar passo a passo no caminho encontrado
- Inspecionar fronteira e visitados

## Estratégias disponíveis

- Custo Uniforme
	- Usa custo acumulado g(n)
- A* Heurística Simples
	- Usa g(n) + h1(n), onde h1 é o número de peças fora do lugar
- Melhor A*
	- Usa g(n) + h2(n), onde h2 é a distância de Manhattan

Atualmente, a resolução está baseada em busca bidirecional para os modos disponíveis.

## Funcionalidades da interface

- Lista de exemplos por tipo de busca
- Modal para criar puzzle personalizado
- Modal de alerta quando não há solução encontrada
- Modal de loading durante a execução
- Exibição de métricas:
	- Máximo de nodos na fronteira
	- Nodos criados
	- Nodos visitados
- Visualização do caminho de solução

## Stack

- Vue 2
- Vuetify
- Vue CLI
- Yarn

## Estrutura principal

- src/components
	- 8Puzzle.vue: orquestração principal da tela e da busca
	- Resolution.vue: exibição da solução passo a passo
	- PathResolution.vue: visão detalhada do caminho
	- ShowInfo.vue: fronteira e histórico de visitados
	- AlertDialog.vue: diálogo de erro de resolução
	- CustomPuzzleDialog.vue: diálogo de puzzle personalizado
- src/utils
	- challenges.js: base de desafios de exemplo

## Requisitos

- Node.js v14.21.3 (projeto antigo)
- Yarn

## Como rodar localmente

1. Instale as dependências:

```bash
yarn install
```

2. Execute em modo desenvolvimento:

```bash
yarn serve
```

3. Acesse no navegador a URL exibida no terminal.

## Build de produção

```bash
yarn build
```

## Scripts úteis

- yarn serve: inicia o servidor de desenvolvimento com hot reload
- yarn build: gera build otimizado para produção

## Observações

- Desafios mais complexos podem demorar mais para convergir.
- Se o navegador indicar lentidão em casos pesados, aguarde o término da busca.
