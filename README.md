# Quais temas cada bancada mais propõe

Análise temática dos **projetos de lei apresentados na Câmara dos Deputados na 57ª
legislatura** (2023 até agosto de 2026), a partir dos dados abertos da própria
Câmara. São 21.457 projetos de lei e 49.738 classificações temáticas.

A pergunta é uma só: **de tudo que um partido apresenta, quanto é de cada tema?**
De cada 100 propostas do PSOL, quantas são de Direitos Humanos; do NOVO, quantas
são de Finanças Públicas. O denominador é sempre o próprio partido — é isso que
torna as bancadas comparáveis: uma de 3.690 projetos e outra de 128 aparecem na
mesma escala, porque o que se compara é a composição da pauta, não o volume.

Tudo está em [`analise_temas_propostas_documentado.ipynb`](analise_temas_propostas_documentado.ipynb),
comentado passo a passo e com as saídas salvas — dá para ler direto aqui pelo
GitHub, sem rodar nada.

## Como rodar

```bash
pip install -r requirements.txt
jupyter lab analise_temas_propostas_documentado.ipynb
```

Os dados brutos não são versionados (~365 MB). Crie uma pasta `dados/` ao lado do
notebook e baixe nela os 12 arquivos — três assuntos, quatro anos:

```
https://dadosabertos.camara.leg.br/arquivos/{assunto}/csv/{assunto}-{ano}.csv
```

com `assunto` em `proposicoes`, `proposicoesTemas` e `proposicoesAutores`, e `ano`
de 2023 a 2026. Os caminhos no notebook são relativos, então abra o Jupyter a
partir desta pasta.

| arquivo | uma linha é | chave |
|---|---|---|
| `proposicoes-{ano}.csv` | uma proposição (PL, PEC, requerimento...) | `id` |
| `proposicoesTemas-{ano}.csv` | uma classificação temática de uma proposição | `uriProposicao` |
| `proposicoesAutores-{ano}.csv` | uma assinatura de autoria | `idProposicao` |

Os arquivos do ano corrente são reescritos pela Câmara conforme novas proposições
entram: a data do download é, na prática, a data de corte da análise.

## Método

Depois de juntar as três bases, uma linha é a combinação (assinatura × proposição
× tema). Três recortes desfazem essa multiplicação e definem o universo:

- **`siglaTipo` em PL e PLP** — só projetos de lei, ordinária e complementar. Fora
  ficam requerimentos, indicações, PECs e recursos, que são instrumentos de
  tramitação e não proposta de política pública. Requerimento é o tipo mais
  numeroso da base; sem esse corte, a análise viraria uma medida de atividade
  parlamentar.
- **`tipoAutor == "Deputado(a)"`** — exclui comissões, o Executivo e órgãos
  externos. O caso menos óbvio é o Senado: projetos que chegam de lá trazem sigla
  de partido e seriam creditados à bancada da Câmara.
- **`ordemAssinatura == 1`** — só o autor principal. A unidade é o projeto, não a
  assinatura: um PL com 40 apoiadores conta uma vez, para quem o apresentou.

As siglas de partido são consolidadas antes da contagem. A base escreve a mesma
sigla de várias formas (`REP`, `REPUBLIC`, `REPUBLICA`; `SOLIDARI` pelo
SOLIDARIEDADE) e mantém siglas extintas de partidos que se fundiram — DEM e PSL
viraram UNIÃO, PPS virou CIDADANIA, PMDB virou MDB. Sem consolidar, a mesma
bancada aparece repartida em várias linhas.

O resultado é `partidos_temas`: uma linha por (partido, tema), com quantas
propostas a bancada apresentou naquele tema (`total_tema`), quantas apresentou no
total (`total_geral`) e a fatia que isso representa (`% da pauta`).

### Uma advertência sobre o denominador

`total_geral` soma **pares (projeto, tema)**, não projetos. A base de temas repete
a proposição uma vez por tema, e cada PL tem 2,3 temas em média: o PL aparece com
8.529 pares e 3.690 projetos. Para percentuais de pauta isso está correto —
numerador e denominador contam a mesma coisa e as fatias fecham em 100% —, mas
para dizer "a bancada apresentou N projetos" é preciso contar `id` único.

## Ressalvas

- **2026 é ano parcial.** Os arquivos vão até a data do download, então o último
  ano da legislatura tem menos projetos. Como as leituras são proporcionais dentro
  da própria bancada, isso não distorce a comparação entre partidos — mas
  inviabiliza série temporal.
- **Tema é classificação da Câmara**, atribuída pelo serviço de documentação da
  Casa, e uma proposição pode ter mais de um. Não é leitura do conteúdo do projeto.
- **A análise mede o que foi apresentado, não o que foi aprovado.** Apresentar
  projeto é barato; nada aqui diz respeito a tramitação ou resultado.
- **Autoria é do primeiro assinante.** Coautoria e apoiamento não entram.
- **Partido é o do momento do registro** na base de autores, não a filiação atual
  do parlamentar. Trocas de partido no meio da legislatura ficam onde estavam.
- **Bancadas pequenas oscilam.** Num partido com 21 propostas, um único projeto
  vale cinco pontos percentuais.

## Dados

Públicos, sob os termos de uso do portal de
[Dados Abertos da Câmara dos Deputados](https://dadosabertos.camara.leg.br/).
