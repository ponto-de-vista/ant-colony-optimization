Vou explicar o algoritmo por etapas, mostrando a lógica geral do ACO (Ant Colony Optimization) aplicado ao PCV (Problema do Caixeiro Viajante) e, mais importante para a prova, os pontos onde uma mudança nos dados afeta (ou não afeta) o resultado — incluindo alguns bugs reais que aparecem no código.

## 1. Ideia geral do ACO para o PCV

O algoritmo simula formigas que constroem rotas (tours) passando por todas as cidades uma única vez e voltando ao início. Cada formiga escolhe a próxima cidade combinando dois fatores:

- **Heurística** (`distancia_cidades`): quanto mais perto, mais atrativo (o código usa `1/distância`).
- **Feromônio** (`feromonios`): quanto mais feromônio numa aresta, mais formigas passaram por ali em rotas boas no passado.

Depois que todas as formigas completam seus tours, o algoritmo:
- calcula o custo de cada tour,
- identifica o melhor,
- evapora o feromônio de todas as arestas,
- deposita mais feromônio nas arestas usadas pelo melhor tour (e por outros agentes que compartilharam essas arestas).

Isso se repete por várias iterações (aqui, `range(3)`), fazendo o sistema convergir para caminhos mais curtos — na teoria.

## 2. Estruturas de dados principais

| Variável | O que é |
|---|---|
| `distancia_cidades` | matriz 10x10 de distâncias entre cidades (heurística, fixa) |
| `feromonios` | matriz 10x10 de feromônio, **atualizada a cada iteração** |
| `tours` | matriz (10 formigas × 11 posições) com a rota de cada formiga |
| `custos` | custo total de cada tour |
| `qtde_feromonio` | quanto feromônio cada formiga deposita (`Q / custo`) |
| `melhor_agente` | índice da formiga com menor custo na iteração |
| `mapa_cidades` | coordenadas (x,y) só para plotar |

## 3. Fluxo passo a passo (o loop `for i in range(3)`)

**Passo 1 — Sorteio da cidade inicial de cada formiga**
`np.random.shuffle(inicio)` embaralha as 10 cidades e cada formiga começa em uma cidade diferente. É a única fonte de aleatoriedade real do algoritmo.

**Passo 2 — Construção do tour (`prox_cidade`)**
Para a cidade atual, a função calcula, para cada cidade `c` ainda não visitada:
```
valor(c) = (1/distancia[atual][c]) * feromonio[atual][c]
```
normaliza pela soma de todos os valores possíveis, e **escolhe a cidade com maior valor** (não é sorteio por roleta, é sempre o "melhor" segundo essa fórmula — escolha gulosa/determinística).

⚠️ Ponto importante para a prova: um ACO "clássico" normalmente faz uma **escolha probabilística** (roleta) usando essas probabilidades, e também eleva feromônio e heurística a expoentes α (alpha) e β (beta) — que aqui são as constantes `A` e `B`. **No código, `A` e `B` são declaradas mas nunca usadas** em `prox_cidade`. Ou seja, se o professor perguntar "o que acontece se eu mudar A ou B?", a resposta é: **nada**, pois elas não entram em nenhum cálculo.

**Passo 3 — Cálculo de custos (`custos_tours`)**
Soma as distâncias entre cidades consecutivas de cada tour. Também calcula `qtde_feromonio[a] = Q / custos[a]` para todas as formigas (custo maior → menos feromônio depositado) e encontra a formiga com menor custo (`melhor_agente`).

**Passo 4 — Plot**
Desenha todos os tours (linha verde tracejada) e destaca o melhor (linha azul).

**Passo 5 — Atualização de feromônio (`atualiza_feromonio`)**
1. **Evaporação**: toda aresta (exceto diagonal) é multiplicada por `(1 - R)`.
2. **Depósito**: percorre as posições do tour do melhor agente e, para cada aresta dele, verifica se outras formigas usaram a mesma aresta (em qualquer sentido) e soma `qtde_feromonio[a]` a essa aresta.

⚠️ **Bug relevante para a prova**: no depósito, o código usa:
```python
feromonios[m+0][m+1] = feromonios[m+0][m+1] + qtde_feromonio[a]
```
Aqui `m` é a **posição** dentro do tour (0 a 9), não a **cidade** naquela posição. O correto seria usar `tours[melhor_agente][m].astype(int)` como índice da matriz, não `m` diretamente. Ou seja, o depósito de feromônio não está necessariamente incidindo sobre as arestas realmente percorridas pelo melhor agente, e sim sobre células fixas da matriz (0-1, 1-2, 2-3...). Isso é exatamente o tipo de coisa que um professor gosta de perguntar: "rode o código e explique por que o resultado não melhora consistentemente entre iterações" — a resposta passa por esse bug.

## 4. O que acontece se você editar os dados (respostas prováveis de prova)

- **Mudar uma distância em `distancia_cidades`**: afeta diretamente `prox_cidade`, pois `1/distância` muda o "peso" daquela aresta. Cidades mais "baratas" tendem a ser escolhidas antes.
- **Mudar um valor inicial em `feromonios`**: também afeta `prox_cidade` desde a primeira iteração, pois multiplica o termo heurístico. Um feromônio inicial muito alto numa aresta ruim pode fazer o algoritmo escolhê-la mesmo sendo longa.
- **Aumentar `R` (taxa de evaporação)**: feromônio decai mais rápido a cada iteração — o sistema "esquece" mais rápido os caminhos bons anteriores. Como a escolha aqui é determinística (não há sorteio proporcional), isso tem efeito mas não gera "exploração" no sentido clássico do ACO.
- **Aumentar `Q`**: aumenta a quantidade de feromônio depositado por tour (proporcionalmente ao inverso do custo) — reforça mais rapidamente as arestas do melhor tour (ou, por causa do bug acima, as posições fixas m/m+1).
- **Mudar `CIDADES`**: precisa mudar junto `distancia_cidades`, `feromonios` e `mapa_cidades`, senão dá erro de índice — bom ponto para mostrar que entende as dependências entre estruturas.
- **Aumentar o número de iterações (`range(3)` → mais)**: dá mais chances de reforço de feromônio, mas como a escolha é gulosa (sempre pega o "melhor" segundo a fórmula) e não há diversificação real, o algoritmo pode convergir rápido para um mesmo conjunto de tours repetidos, dependendo da cidade inicial sorteada.
- **Mudar `A` ou `B`**: sem efeito nenhum, como explicado — são "pegadinhas" no código, definidas mas não usadas.

Se quiser, posso montar um resumo visual (diagrama do fluxo) ou simular à mão um exemplo pequeno (3-4 cidades) para você testar esse entendimento antes da prova.

ia_aco_01

Introdução à Inteligência dos Enxames

● O feedback positivo (amplificação) é basicamente
definido como a promoção de soluções inteligentes
alocando mais agentes para o mesmo trabalho.

● O feedback negativo (contrapeso) permite que os
enxames de agentes evitem o mesmo comportamento
ou envolvimento no mesmo estado.

● As flutuações são altamente úteis para aleatoriedade,
erros e busca de outras alternativas.

● Múltiplas interações (compartilhamento de
informações) é a condição de compartilhar as
informações entre todos os demais agentes da área de
busca.

---------

Indivíduos em um enxame agem e realizam suas
respectivas tarefas de acordo com as regras e
tarefas atribuídas.

○ As regras às vezes criam um comportamento
complexo, como em bandos de pássaros, onde
cada indivíduo fica a uma certa distância de seu
pássaro vizinho e voa ligeiramente atrás dele.

○ Flexibilidade, Robustez e Auto-organização são as
três razões principais para o sucesso dos enxames

-----------

- O que define convergencia prematura nessa meta-heuristica. (nao deveria mas sofre, por causa de desbalanceamento do grafo ponderado)
- Convergencia minimo local convergencia prematura.
- O que se assemelha na convergencia AG e ACO.

--------
CAOS nivel 1, antecipacao de conhecimento nao modifica o futuro.
--------
a informação heuristica, quando pesa feromonio e distancia de um caminho, com o menor caminho é melhor. A solução pode ter um aresta com mais feromonio mas não seria uma aresta com menor caminho, pois o feromonio guarda o momento. E a solução vai pegar o melhor tour.
