# desafio_teste_sudoku

Implementação das regras do **Sudoku binário 6 x 6** em JavaScript (Node.js, CommonJS), acompanhada de testes automatizados com Jest. O projeto não possui interface, concentrando-se na lógica do jogo e em sua validação.

## Colaborador

- João Gabriel Caldeira Milare (R.A: 24113672-2)

## Especificação do desafio

Enunciado do professor:

> **Sudoku binário (by Zulli)**
> - casos de teste para testar Sudoku binário de tamanho 6 x 6;
> - casos de teste em Javascript para testar Javascript;

Autor da especificação: **Zulli**.

## Regras implementadas

As regras identificadas como **(decisão adotada)** não aparecem no enunciado original. Elas foram definidas ao longo do desenvolvimento, sem confirmação específica.

### Tabuleiro

- O tabuleiro é fixo em **6 x 6**. Não há outros tamanhos **(decisão adotada)**.
- Cada célula vale `0`, `1` ou está vazia (`null`) **(decisão adotada)**.
- Linhas e colunas são indexadas de `0` a `5` **(decisão adotada)**.

### Regras de solução (Sudoku binário)

1. Nunca pode haver **três valores iguais seguidos** numa linha ou numa coluna.
2. Cada linha e cada coluna tem **exatamente 3 zeros e 3 uns**. Durante o jogo, isso significa que nenhuma linha ou coluna pode passar de 3 valores iguais.
3. Não pode haver **duas linhas completas iguais** nem **duas colunas completas iguais**. Linhas e colunas incompletas não entram nessa comparação.

### Jogadas

- Qualquer jogada que desrespeite uma regra de solução é **rejeitada imediatamente** por meio de `ErroDeRegra`, mantendo o tabuleiro inalterado **(decisão adotada)**.
- As células que já vêm preenchidas no quebra-cabeça são consideradas **fixas**, portanto não podem ser modificadas ou removidas **(decisão adotada)**.
- Jogar numa célula já preenchida pelo jogador **substitui** o valor, se o novo valor respeitar as regras **(decisão adotada)**.
- Apagar uma célula já vazia é proibido (`ErroDeRegra`). Nenhuma ação é ignorada em silêncio **(decisão adotada)**.

### Fim de jogo

- O jogador **vence a partida** quando todas as células do tabuleiro estão preenchidas. Como toda jogada inválida é recusada, um tabuleiro cheio é sempre uma solução válida. Vale **qualquer solução válida**, sem comparação com gabarito **(decisão adotada)**.
- Não existe derrota **(decisão adotada)**.
- Depois da vitória, qualquer jogada ou apagamento é recusado com `ErroDeRegra` **(decisão adotada)**.

### Quebra-cabeças

- O quebra-cabeça vem de um **catálogo fixo** de 3 quebra-cabeças, sorteado com `Math.random` por padrão **(decisão adotada)**. Gerar quebra-cabeças por algoritmo seria overengineering para o desafio.
- Cada quebra-cabeça do catálogo foi conferido com um solver descartável e tem **solução única**. As soluções ficam em `tests/helpers/solucoesDoCatalogo.js`.
- A partida realiza a validação do quebra-cabeça fornecido pela função injetada. Se não for uma grade 6 x 6 com `0`, `1` ou `null`, gera `TypeError` ou `RangeError`. Se já violar alguma regra ou não tiver nenhuma célula vazia, gera `RangeError` **(decisão adotada)**.
- Se o gerador de números aleatórios devolver algo fora de `[0, 1)`, o sorteio é recusado com `RangeError` **(decisão adotada)**.

### Ordem das validações

A validação ocorre primeiro sobre o argumento (`TypeError` ou `RangeError`), em seguida sobre as regras (`ErroDeRegra`) e, por último, é feita a alteração. Por exemplo, um valor inválido numa partida já vencida gera `TypeError` ou `RangeError`, e não `ErroDeRegra`.

## Estrutura de pastas

```
src/
  constantes.js            tamanho, limites, valores e estados da partida (congelados)
  erros.js                 ErroDeRegra e as mensagens das regras
  validacoes.js            validação de argumentos: posição, valor e formato da grade
  regras.js                funções puras que encontram violações das regras de solução
  tabuleiro.js             grade com células fixas; preencher e apagar respeitando as regras
  catalogo.js              catálogo congelado de quebra-cabeças com solução única
  sorteio.js               sorteio de um quebra-cabeça com gerador aleatório injetável
  partida.js               classe principal: estado, jogadas, vitória e bloqueio após o fim
tests/
  constantes.test.js       valores e congelamento das constantes
  erros.test.js            classe ErroDeRegra e mensagens
  validacoes.test.js       TypeError/RangeError de posição, valor e formato da grade
  regras.test.js           cada regra de solução, em linhas e em colunas
  tabuleiro.test.js        preencher, apagar, células fixas, recusas sem mudança de estado e encapsulamento
  catalogo.test.js         cada quebra-cabeça do catálogo é válido e pode ser resolvido
  sorteio.test.js          sorteio determinístico, Math.random e validação do gerador
  partida.test.js          ciclo da partida, injeção, vitória, fim de jogo e encapsulamento
  helpers/
    grades.js              grades montadas à mão (congeladas) e utilitários congelarGrade/transpor
    geradores.js           geradores de números previsíveis
    solucoesDoCatalogo.js  soluções do catálogo e a lista de jogadas que completam cada uma
```

## Como executar

```bash
npm install
npm test
npm run test:coverage
```

Versões usadas: **Node.js 26.1.0**, **npm 11.13.0** e **Jest 30.5.2**. O Jest 30.5.2 declara suporte a `^18.14.0 || ^20.0.0 || ^22.0.0 || >=24.0.0`.

Resultado atual:

- **198 testes** em **8 suítes**, todos passando.
- Cobertura de `src/`: **100%** de statements, branches, funções e linhas.
- Teste de mutação (Stryker): **99,64%** de mutantes mortos (278 de 279; 1 mutante equivalente restante em `src/regras.js`).

`npm audit` aponta vulnerabilidades moderadas em dependências transitivas do Jest (`glob`, `minimatch`, `deepmerge`). Não foi aplicado `npm audit fix --force`, porque ele faria downgrade do Jest para a versão 25.

## Como funcionam os casos de teste

### Controle da aleatoriedade

O comportamento aleatório é controlado por meio de injeção de dependências. Em produção, `sortearQuebraCabeca(catalogo, gerarNumeroAleatorio = Math.random)` usa `Math.random`, e a `Partida` usa `prepararQuebraCabeca`, que por padrão sorteia do catálogo. Nos testes, entram cenários montados à mão:

| Cenário | Configuração | Para que serve |
|---|---|---|
| `GRADE_VAZIA` | 36 células vazias | jogadas livres, apagar célula vazia, encapsulamento |
| `SOLUCAO_VALIDA` | solução completa e válida | grade sem violação; quebra-cabeça sem célula vazia (recusado) |
| `SOLUCAO_VALIDA_SEM_O_CANTO_INFERIOR_DIREITO` | solução sem a célula (5, 5) | vitória numa única jogada; valor errado recusado; células fixas |
| `LINHA_COM_TRES_UNS_SEGUIDOS_NO_INICIO` / `..._TRES_ZEROS_SEGUIDOS_NO_FIM` | trio na borda esquerda ou direita | regra dos três seguidos, nas bordas |
| `LINHA_COM_QUATRO_UNS_SEM_TRIO` / `..._QUATRO_ZEROS_SEM_TRIO` | 4 valores iguais sem trio | regra da quantidade, isolada da regra do trio |
| `LINHAS_COMPLETAS_IGUAIS_E_VIZINHAS` / `..._E_DISTANTES` | duas linhas completas idênticas | regra de linhas repetidas |
| `LINHAS_INCOMPLETAS_COM_O_MESMO_COMECO` | duas linhas iguais com uma célula vazia | linhas incompletas não contam; completar a segunda é proibido |
| `DOIS_VALORES_IGUAIS_SEGUIDOS_EM_LINHA_E_COLUNA` | pares iguais permitidos | dois seguidos é permitido; o quarto valor numa coluna é proibido |
| `transpor(...)` | o mesmo cenário girado | toda regra de linha também é verificada em coluna |
| `GERADOR_QUE_SEMPRE_ESCOLHE_O_PRIMEIRO` | devolve `0` | sorteio do primeiro item |
| `GERADOR_QUE_SEMPRE_ESCOLHE_O_ULTIMO` | devolve `1 - Number.EPSILON` | limite superior do sorteio |
| `criarGeradorQueRepeteASequencia` | repete uma sequência fixa | a mesma sequência reproduz os mesmos sorteios |
| `SOLUCOES_DO_CATALOGO` + `listarJogadasQueCompletam` | solução de cada quebra-cabeça | jogar uma partida inteira até a vitória |

Também são realizados testes sem injeção (`Math.random`), considerando apenas as propriedades asseguradas pelas regras: o quebra-cabeça sorteado sempre pertence ao catálogo, e a partida começa em andamento. `jest.fn` prova quantas vezes o gerador e a função de preparo são chamados. Todos os fixtures são congelados com `Object.freeze`.

### Categorias de erro

| Categoria | Quando | Exemplos |
|---|---|---|
| `ErroDeRegra` | ação válida no formato, mas proibida pelas regras do jogo | três valores iguais seguidos; quarto valor igual na linha; linha completa repetida; alterar ou apagar célula inicial; apagar célula vazia; jogar depois da vitória |
| `TypeError` | argumento do tipo errado | linha `'1'` ou `1.5`; valor `'1'` ou `null`; grade que não é lista; `prepararQuebraCabeca` que não é função; gerador que devolve texto |
| `RangeError` | argumento do tipo certo, mas fora do permitido | linha `6` ou `-1`; valor `2`; grade 5 x 6; gerador que devolve `1` ou `NaN`; catálogo vazio; quebra-cabeça inicial inválido ou sem células vazias |

A validação das exceções é feita utilizando `toThrow(new Classe('mensagem'))`. No Jest 30 esse formato confere **a classe e a mensagem**, o que foi testado à parte antes de ser adotado. Sempre que uma ação é rejeitada, os testes conferem se o estado permaneceu exatamente igual.

### Detalhamento dos testes por arquivo

| Arquivo | Nº de testes | O que verifica |
|---|---|---|
| `constantes.test.js` | 8 | tamanho, limites, chaves e valores exatos das enumerações e congelamento |
| `erros.test.js` | 5 | `ErroDeRegra` é um `Error` próprio; todas as mensagens exatas, congeladas e distintas |
| `validacoes.test.js` | 49 | posições nas bordas e fora delas, valores e formato da grade, com `TypeError`/`RangeError` |
| `regras.test.js` | 27 | cada regra de solução em linha e em coluna (bordas e meio), prioridade entre regras e grades válidas |
| `tabuleiro.test.js` | 33 | preencher, substituir, apagar, células fixas, ordem das validações, recusas sem mudança de estado, cópias defensivas |
| `catalogo.test.js` | 12 | catálogo congelado com 3 quebra-cabeças distintos e com pistas; cada um é aceito, tem solução válida e pode ser completado |
| `sorteio.test.js` | 26 | escolha por faixa do número sorteado, reprodutibilidade, `Math.random`, validação do gerador e do catálogo |
| `partida.test.js` | 38 | criação e validação da injeção, jogadas, vitória com qualquer solução válida, bloqueio após o fim, ordem das validações, encapsulamento |
| **Total** | **198** | **8 suítes, 100% de cobertura** |
