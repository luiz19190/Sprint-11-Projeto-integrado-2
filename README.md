# Sprint 11 — Funil de vendas e teste A/A/B no aplicativo

Projeto do curso de Data Analytics da [TripleTen](https://tripleten.com/), por **Luiz Trajano**.

Análise do log de eventos de um aplicativo de venda de produtos alimentícios, com duas frentes que
saem do mesmo arquivo. A primeira é o **funil de vendas**: por onde o usuário passa até a compra,
quantos chegam lá e em que etapa eles ficam presos. A segunda é um **teste A/A/B**: os designers
querem trocar a fonte do aplicativo inteiro e os gerentes acham que o desenho novo vai intimidar o
usuário. Dois grupos de controle ficaram com a fonte antiga (246 e 247) e um grupo de teste recebeu
a nova (248).

Ter **dois** grupos de controle não é redundância, é controle de qualidade. Se os dois braços que
receberam exatamente o mesmo tratamento já diferem entre si, o mecanismo de divisão está torto e
qualquer diferença encontrada contra o grupo de teste não prova nada.

> **Status:** análise concluída e revisada. Seções 1 a 5 escritas, com conclusão em cada etapa e
> recomendação final na seção 5.

## Estrutura do projeto

```
├── Sprint 11 - Funil e Teste A-A-B.ipynb   # notebook principal
├── Data/
│   └── logs_exp_us.csv         # log de eventos do aplicativo (TSV)
├── images/                     # gráficos exportados
├── presentation/
├── .gitignore
├── LICENSE                     # MIT
└── README.md
```

## Os dados

`logs_exp_us.csv` — 244.126 registros de evento, separados por tabulação.

| coluna | descrição |
|---|---|
| `EventName` | nome do evento disparado |
| `DeviceIDHash` | identificador do usuário |
| `EventTimestamp` | carimbo de tempo em segundos desde 01/01/1970 UTC |
| `ExpId` | grupo do experimento: 246 e 247 são controle, 248 é teste |

Depois da preparação sobram **240.887 eventos e 7.534 usuários**, cobrindo de 01/08 a 07/08/2019.

## Roteiro da análise

**Seção 1 — Preparação.** Separador do arquivo, ausentes declarados e disfarçados, duplicados,
tipos e conversão do carimbo de tempo. Num log de evento "duplicado" precisa de definição antes de
virar contagem: repetição costuma ser comportamento real, não erro.

**Seção 2 — Estudo dos dados.** Tamanho da base, período coberto, corte dos dias com coleta
incompleta e verificação dos três grupos experimentais, com qui-quadrado de aderência para o
equilíbrio entre eles.

**Seção 3 — O funil.** Frequência e alcance de cada evento, ordem real das etapas testada contra o
relógio, conversão de etapa para etapa e localização do gargalo.

**Seção 4 — O experimento.** Teste A/A entre os controles, comparação do grupo de teste contra cada
controle e contra os dois combinados, e correção do nível de significância para testes múltiplos.

**Seção 5 — Conclusão geral e recomendações.**

## O que a análise encontrou

- **O funil perde quase tudo num degrau só.** Dos 7.419 usuários que abrem o aplicativo, 4.482
  chegam às ofertas (60,41%), 3.580 ao carrinho (79,88%) e 3.429 ao pagamento (95,78%). São 2.937
  dos 3.990 usuários perdidos concentrados na primeira passagem, e esse degrau é o pior nas duas
  réguas ao mesmo tempo, tanto em taxa quanto em gente.
- **O problema não está no pagamento.** Quem chega ao carrinho quase sempre paga: 151 pessoas se
  perdem na última etapa.
- **Do primeiro evento até o pagamento chegam 45,51%** dos usuários da base.
- **A ordem das etapas do meio não se estabelece.** A tela principal vem primeiro em 93,73% a
  94,97% dos casos, mas ofertas e carrinho são intercambiáveis, e carrinho e pagamento disparam no
  mesmo segundo em 49,77% das vezes, o que um carimbo em segundos não separa.
- **O `Tutorial` não é etapa do funil.** Alcança 840 usuários (11,15%) e é o primeiro evento
  registrado de 785 deles (93,45%): é tela de entrada.
- **O teste A/A passou.** Nenhum dos cinco eventos diferiu entre os dois controles, com menor
  p-valor de 0,1146, o que autoriza o resto do experimento.
- **A fonte nova não mudou nada mensurável.** Foram 20 testes de hipótese ao todo, nenhum
  significativo, nem com alfa de 5% nem com o corrigido por Bonferroni para 0,0025. O menor
  p-valor de toda a seção é 0,0784.

## Recomendação

**Não implementar a fonte nova, e o motivo é de custo e não de estatística.** O teste não encontrou
efeito em direção nenhuma, então nem o ganho que justificaria a troca aparece, nem o receio dos
gerentes se confirma. Como o dado não decide, decide a assimetria: trocar a fonte do aplicativo
inteiro tem custo certo e benefício não medido.

**Direcionar o esforço para a passagem da tela principal para as ofertas**, onde estão 2.937 dos
3.990 usuários perdidos no funil inteiro.

O log não tem campo nenhum que diga por que o usuário desistiu, então a causa dessa queda continua
em aberto. As hipóteses estão registradas na seção 5, cada uma com o teste que a confirmaria.

## Limites declarados

O corte descartou tudo antes de 01/08, o que representa 2.826 eventos (1,16%) e 17 usuários
(0,23%), então a análise fala de sete dias e não de catorze. A remoção de duplicatas tratou como
suspeita apenas a linha idêntica nas quatro colunas ao mesmo tempo, 413 registros (0,17%), dentro
do limite de 1% da base fixado antes do resultado. Os braços do experimento ficaram com 1,09% de
desvio, acima da régua de 1% definida antes do teste, embora o qui-quadrado de aderência não
encontre evidência de divisão desigual (p-valor de 0,7554). A folga custa um pouco de poder:
ausência de significância aqui não prova igualdade, apenas que não há evidência de diferença.

## Tecnologias utilizadas

- Python (pandas)
- seaborn e matplotlib — gráficos da análise
- plotly — funil de conversão, onde a forma do gráfico é o próprio resultado
- kaleido — exporta o gráfico plotly como PNG
- scipy — distribuições normal e qui-quadrado, usadas no z-test de proporções e no teste de
  aderência, os dois escritos à mão
- Jupyter Notebook

## Como executar

1. Clone o repositório.
2. Instale as dependências: `pip install pandas matplotlib seaborn plotly kaleido scipy`
   O `kaleido` só é necessário para salvar o funil como PNG. Sem ele o notebook roda normalmente e
   o gráfico aparece; apenas o arquivo não é gravado.
3. Abra `Sprint 11 - Funil e Teste A-A-B.ipynb` e execute as células em ordem.

A célula de setup procura o arquivo de dados nos caminhos possíveis e usa o primeiro que existir,
então o notebook roda localmente, na plataforma da TripleTen e no Colab sem edição.
