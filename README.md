ï»¿# YouTube bombando ê¢â¬â radar do que bomba

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


**ê°Å¸â¡Â§ê°Å¸â¡Â· [PortuguêÂªs](README.md) êÂ· ê°Å¸â¡Âºê°Å¸â¡Â¸ [English](README.en.md) êÂ· ê°Å¸â¡Âªê°Å¸â¡Â¸ [EspaêÂ±ol](README.es.md)**

[![Radar do que bomba](guia/assets/banner.jpg)](https://linnoff88-dev.github.io/arckhe-/guia/)

## O que êÂ©

O **youtube-bombando** êÂ© um pequeno sistema que roda sozinho no seu computador, toda manhêÂ£. Ele procura no YouTube os vêÂ­deos da semana nos assuntos que vocêÂª escolher, anota as views de cada um por 7 dias e monta um painel com o que estêÂ¡ crescendo mais rêÂ¡pido. Para cada vêÂ­deo, ele compara as views com o normal do prêÂ³prio canal e diz se o sinal êÂ© de **conteêÂºdo**, de **quem posta** (canal grande) ou de **quem fala** (convidado). êâ° para quem grava vêÂ­deo ou quer comeêÂ§ar um canal. Precisa sêÂ³ de Python e de uma ferramenta grêÂ¡tis que lêÂª o YouTube; nêÂ£o usa chave nem API paga.

## ê°Å¸ââ Guia de uso

Guia completo (landing + passo a passo): **https://linnoff88-dev.github.io/arckhe-/guia/**

## Requisitos

Python 3.9+ e biblioteca padrêÂ£o, sem pip, API ou chaves. A coleta real usa o executêÂ¡vel `yt-dlp` disponêÂ­vel no PATH; os testes nêÂ£o o executam. Linux com systemd de usuêÂ¡rio êÂ© necessêÂ¡rio apenas para a instalaêÂ§êÂ£o do timer. A pesquisa real requer internet; o modo fixtures e o painel funcionam offline.
## ComeêÂ§ar localmente

```bash
python3 -m unittest discover -s tests -v
python3 bombando/rodada.py --fixtures tests/fixtures
# Abra dados/teste/painel/index.html no navegador, inclusive via file://.
python3 bombando/rodada.py --so-painel
```

As fixtures sêÂ£o exemplos fixos, identificados no painel; a idade cresce conforme o tempo passa. Elas gravam em `dados/teste/`, com banco e painel separados da produêÂ§êÂ£o. Repetir a rodada no mesmo dia preserva a foto êÂºnica diêÂ¡ria e o cache dos canais. Testes unitêÂ¡rios usam diretêÂ³rios temporêÂ¡rios isolados.

Coleta real (execute quando quiser consultar o YouTube):

```bash
python3 bombando/rodada.py
python3 bombando/rodada.py --nicho 'claude code'
```

## InstalaêÂ§êÂ£o diêÂ¡ria

```bash
bash deploy/instalar.sh
systemctl --user list-timers youtube-bombando.timer
journalctl --user -u youtube-bombando.service
```

O instalador substitui `@REPO@` nas units, copia para `~/.config/systemd/user/`, recarrega o systemd, ativa o timer e inicia/reinicia o servidor de painel. Pode ser repetido. A coleta ocorre êÂ s **07:15 no fuso da mêÂ¡quina**, com `Persistent=true` (recupera execuêÂ§êÂ£o perdida), teto de 45 minutos, memêÂ³ria de 1 GB e Nice 10. O timer depende do gerenciador de usuêÂ¡rio estar ativo; execuêÂ§êÂ£o apêÂ³s logout requer linger configurado pelo administrador.

Painel em `http://localhost:8795`; o servidor escuta `0.0.0.0` conforme a especificaêÂ§êÂ£o e tambêÂ©m fica acessêÂ­vel na rede local. Para parar:

```bash
systemctl --user disable --now youtube-bombando.timer youtube-bombando-painel.service
```

A instalaêÂ§êÂ£o e a coleta real nêÂ£o fazem parte dos testes offline.

## ConfiguraêÂ§êÂ£o

Edite `config.json`. Cada item de `nichos` contêÂ©m `nome`, `busca` e `ativo` (booleano). Um vêÂ­deo pode pertencer a vêÂ¡rios nichos sem duplicar suas fotos. Desativar um nicho impede novas buscas, mas preserva seu histêÂ³rico; vêÂ­deos jêÂ¡ descobertos continuam sendo acompanhados atêÂ© o fim da janela.

| Campo | PadrêÂ£o | FunêÂ§êÂ£o |
|---|---:|---|
| `por_nicho_busca` | 40 | MêÂ¡ximo de resultados por busca semanal |
| `dias_acompanhar` | 7 | Janela desde a primeira descoberta |
| `max_detalhes_por_rodada` | 250 | Teto de consultas de detalhes |
| `max_canais_por_rodada` | 120 (este repo usa 200) | Teto de tentativas de consulta de canais por rodada |
| `pausa_s` | 1.5 | Intervalo mêÂ­nimo entre fim e inêÂ­cio das chamadas |
| `idioma` | pt | Reservado; nêÂ£o êÂ© passado ao yt-dlp (com `lang=pt` ele lêÂª "3,4 mil" como 34) |
| `regiao` | BR | PaêÂ­s informado ao mecanismo geo-bypass do yt-dlp |

A regiêÂ£o nêÂ£o garante resultados geolocalizados; o YouTube tambêÂ©m considera a conexêÂ£o. O filtro semanal êÂ© o parêÂ¢metro de pesquisa solicitado ao YouTube. O sistema preserva os resultados devolvidos pelo extrator.

Cada subprocesso tem timeout de 120 segundos, socket de 20 segundos e retries limitados. Erros sêÂ£o registrados por nicho/vêÂ­deo e a rodada continua. VêÂ­deos explicitamente privados/removidos ficam indisponêÂ­veis; timeout e 429 nêÂ£o provocam essa marcaêÂ§êÂ£o. Fotos jêÂ¡ coletadas hoje sêÂ£o puladas. O teto prioriza a velocidade da êÂºltima foto; como ela sêÂ³ guarda o dia, usa meio-dia UTC como referêÂªncia dessa prioridade, com desempate por descoberta e ID. VêÂ­deos ainda sem fotos têÂªm prioridade de velocidade zero.

## MêÂ©tricas e nota

Datas sêÂ£o armazenadas em UTC. `timestamp` tem preferêÂªncia sobre `upload_date`; este êÂºltimo representa meia-noite UTC, uma aproximaêÂ§êÂ£o quando a hora nêÂ£o êÂ© fornecida.

- **Idade (h):** horas desde a publicaêÂ§êÂ£o atêÂ© a geraêÂ§êÂ£o do painel, limitada a zero para datas futuras.
- **Velocidade:** views da foto mais recente / `max(idade_h, 1)`; êÂ© a mêÂ©dia desde a publicaêÂ§êÂ£o, nêÂ£o a velocidade instantêÂ¢nea.
- **Ganho 24h:** `(views de hoje ê¢Ëâ views da foto anterior) / dias entre fotos`. Sem foto de hoje ou sem anterior, fica ausente. Quedas de contagem sêÂ£o preservadas.
- **Multiplicador:** views / inscritos. Inscritos ausentes ou zero deixam a mêÂ©trica ausente.
- **Nota:** `z-score(velocidade dentro do nicho) + ln(1 + multiplicador)`. O z-score usa desvio padrêÂ£o populacional; com desvio zero ou velocidade ausente, vale zero. Multiplicador ausente contribui zero. A nota pode ser negativa. Em ê¢â¬ÅTodosê¢â¬Â, usa a maior nota entre os nichos do vêÂ­deo; ao selecionar um nicho, usa a nota daquele nicho. IndisponêÂ­veis ficam no fim e nêÂ£o entram nas estatêÂ­sticas nem nos destaques.

Zero e ausêÂªncia sêÂ£o distintos: `ê¢â¬â` indica dado desconhecido. Views da busca ficam no banco como estimativa inicial, separadas das fotos. A tabela exibe as fotos confirmadas. O **grêÂ¡fico de crescimento** (coluna ê¢â¬ÅCrescimentoê¢â¬Â, clique para ampliar, e no vêÂ­deo mais quente de cada nicho) pêÂµe as views no eixo vertical e as horas desde a publicaêÂ§êÂ£o no horizontal. ComeêÂ§a em 0 views na publicaêÂ§êÂ£o; o trecho tracejado vai atêÂ© a 1êÂª mediêÂ§êÂ£o (sêÂ³ a mêÂ©dia, nêÂ£o hêÂ¡ pontos no meio) e a linha cheia liga as mediêÂ§êÂµes diêÂ¡rias das 07:15, cada uma na idade que o vêÂ­deo tinha naquele dia.

**Ordenar por** (o painel explica a opêÂ§êÂ£o escolhida embaixo do seletor):

| OpêÂ§êÂ£o | O que mede |
|---|---|
| Nota | z-score das views/h dentro do nicho + ln(1 + multiplicador): o mais quente agora |
| Velocidade | views êÂ· idade em horas; favorece vêÂ­deos novos |
| Ganho em 24h | views ganhas desde a mediêÂ§êÂ£o anterior (precisa de 2 dias de coleta) |
| Multiplicador | views êÂ· inscritos; acima de 1êâ passou o tamanho do canal |
| êâ canal | views êÂ· mediana dos êÂºltimos vêÂ­deos do canal; ê¢â°Â¥ 3êâ furou a bolha |
| Views | total de views, sem olhar idade nem canal |
| Mais novos | menor idade primeiro |

**Idade** = tempo desde que o vêÂ­deo foi publicado no YouTube (nêÂ£o desde que o sistema o achou); aparece em horas atêÂ© 48 h e depois em dias + horas.

**Assuntos em alta:** top 30 vêÂ­deos por nota em cada nicho (e top 30 geral sem duplicar IDs). Com pelo menos 5 vêÂ­deos, um termo precisa aparecer em dois têÂ­tulos; em grupos menores, basta um. Extrai palavras e bigramas adjacentes, remove stopwords PT/EN, palavras de uma letra e nêÂºmeros isolados. Cada termo recebe uma contribuiêÂ§êÂ£o por têÂ­tulo, igual êÂ  velocidade do vêÂ­deo. Exibe os 15 maiores pesos. êâ° uma heurêÂ­stica lexical, sem IA ou interpretaêÂ§êÂ£o semêÂ¢ntica.

## Por que bombou? ê¢â¬â mêÂ©todo e limites

O sinal principal êÂ© **fator_canal = views / mediana de views do prêÂ³prio canal**.
Consulta os êÂºltimos 20 vêÂ­deos do canal uma vez e guarda a amostra em SQLite.
A mediana exclui o prêÂ³prio vêÂ­deo analisado e publicaêÂ§êÂµes com menos de 3 dias.
Quando a data de uma entrada nêÂ£o estêÂ¡ disponêÂ­vel, exclui essa entrada se estiver
entre as duas primeiras da lista. Views desconhecidas nêÂ£o entram; zero êÂ© vêÂ¡lido.
Cada vêÂ­deo tem sua mediana e `n_amostra` recalculados a partir do cache, inclusive
quando hêÂ¡ vêÂ¡rios vêÂ­deos do mesmo canal. Na tabela `canais`, mediana e tamanho
representam o vêÂ­deo prioritêÂ¡rio na coleta; `amostra` preserva os dados para os demais.

O cache vale atêÂ© 7 dias inclusive; sêÂ³ êÂ© renovado apêÂ³s esse prazo. Os canais dos
vêÂ­deos acompanhados (disponêÂ­veis, dentro de `dias_acompanhar`) de maior nota têÂªm
prioridade. O teto conta tentativas, inclusive falhas. Erros preservam o cache
anterior e ficam no log. A data do cache aparece em `canal_atualizado` nos dados.
NêÂ£o hêÂ¡ chamada extra para descriêÂ§êÂ£o: ela êÂ© guardada quando chega no detalhe.

`pessoas` em `config.json` êÂ© a lista editêÂ¡vel de nomes conhecidos. O detector
procura nomes completos sem diferenciar maiêÂºsculas e minêÂºsculas, no têÂ­tulo e na
descriêÂ§êÂ£o disponêÂ­vel, alêÂ©m de sinais de formato: `com Nome Sobrenome`, `ft.`,
`feat.`, entrevista, podcast, palestra, `| Nome Sobrenome`, `- NOME SOBRENOME`
e `#123`. O trecho encontrado acompanha o motivo. Uma menêÂ§êÂ£o nêÂ£o comprova
participaêÂ§êÂ£o; o mêÂ©todo êÂ© uma heurêÂ­stica, nêÂ£o uma demonstraêÂ§êÂ£o de causalidade.

Porte por inscritos da foto mais recente (ou do canal, se ausentes): pequeno
abaixo de 10 mil; mêÂ©dio de 10 mil a 500 mil inclusive; grande acima de 500 mil.
Sem inscritos, o porte êÂ© desconhecido. Regras, nesta ordem:

| CondiêÂ§êÂ£o | Categoria |
|---|---|
| Sem views confirmadas ou mediana ausente/zero | Sem dados |
| Fator ê¢â°Â¥ 3 e pessoa/formato de convidado | ConteêÂºdo + quem fala |
| Fator ê¢â°Â¥ 3 sem pessoa/formato de convidado | ConteêÂºdo |
| Fator ê¢â°Â¥ 1,5 e pessoa/formato de convidado | Quem fala |
| Fator < 1,5 e canal grande | Quem posta |
| Demais casos | Misto |

A regra combinada vem primeiro para nêÂ£o ser absorvida por ê¢â¬ÅQuem falaê¢â¬Â. Os limiares
sêÂ£o constantes no topo de `metricas.py`. AusêÂªncia nunca êÂ© estimada por inscritos.
Os motivos mostram fator, mediana, porte e evidêÂªncia de convidado.

O resumo geral e por nicho considera vêÂ­deos disponêÂ­veis com views confirmadas,
sem duplicaêÂ§êÂ£o dentro de cada grupo. A legenda mostra contagens e percentuais de
vêÂ­deos e views nas **seis categorias**, incluindo a combinada separadamente para
nêÂ£o contar suas views duas vezes. Sem views totais, percentuais sêÂ£o zero. A
conclusêÂ£o prioriza insuficiêÂªncia de dados (ê¢â°Â¥ 50% das views sem base); depois,
maioria estrita (> 50%) de conteêÂºdo (puro + combinado), quem posta ou quem fala;
sem maioria, informa sinais mistos. ê¢â¬ÅFuraram a bolhaê¢â¬Â lista atêÂ© 5 vêÂ­deos com
fator ê¢â°Â¥ 3 de canais pequenos/mêÂ©dios, ordenados pelo fator.

Para preencher apenas as linhas de base jêÂ¡ existentes, mantendo descoberta e
fotos intactas (consulta real de canais):

```bash
python3 bombando/rodada.py --so-canais
python3 bombando/rodada.py --so-canais --nicho 'claude code'
# Variante inteiramente offline, usando o banco de demonstraêÂ§êÂ£o:
python3 bombando/rodada.py --so-canais --fixtures tests/fixtures
```

`--so-canais` e `--so-painel` sêÂ£o mutuamente exclusivos. No modo canais, nenhuma
consulta necessêÂ¡ria (cache vêÂ¡lido ou seleêÂ§êÂ£o vazia) êÂ© sucesso; falha em todas as
consultas tentadas retorna 1, e sucesso parcial retorna 0. O painel mostra erros.

## Arquivos e operaêÂ§êÂ£o

`dados/bombando.db` contêÂ©m vêÂ­deos, nichos, ligaêÂ§êÂµes, fotos êÂºnicas por `(id, dia)`, rodadas e canais com suas amostras. A abertura do banco migra o esquema antigo sem apagar dados. `dados/.lock` usa `flock` nêÂ£o bloqueante para evitar concorrêÂªncia. `dados/logs/AAAA-MM-DD.log` guarda falhas e resumo. `painel/index.html` contêÂ©m CSS, JS, dados e SVG inline; `painel/dados.json` tem os mesmos dados e `painel/historico/AAAA-MM-DD.html` êÂ© a êÂºltima cêÂ³pia daquele dia. Miniaturas sêÂ³ carregam apêÂ³s clicar em ê¢â¬ÅCarregar miniaturas (internet)ê¢â¬Â; fontes usam fallback de sistema, sem CDN.

`--so-painel` mantêÂ©m o status e horêÂ¡rio da êÂºltima coleta e regenera as mêÂ©tricas. `--nicho NOME` restringe busca, acompanhamento e canais, preservando os outros dados no painel. O cêÂ³digo de saêÂ­da êÂ© zero quando algum nicho teve busca vêÂ¡lida, mesmo com erros parciais; nenhum nicho bem-sucedido retorna 1. Falhas estruturais (configuraêÂ§êÂ£o, banco ou escrita) tambêÂ©m retornam 1. Lock ocupado êÂ© execuêÂ§êÂ£o ignorada com cêÂ³digo zero.

As fixtures ficam em `tests/fixtures/buscas.json` (mapa por termo de busca, com `entries`), `detalhes.json` (mapa por ID) e `canais.json` (mapa por channel_id com entries). `{"erro":"mensagem"}` simula falha; `"indisponivel":true` simula remoêÂ§êÂ£o. NêÂ£o hêÂ¡ fallback de fixtures para rede.

## O processo, do inêÂ­cio ao fim

1. **ê°Å¸Â§Â  sessêÂ£o com o Nei ê¢â¬â escolher os nichos.** Comando: `python3 -m json.tool config.json`. Conferir nomes, buscas, limites e campos `ativo` depois de editar o arquivo.
2. **ê¢Å¡â¢ê¯Â¸Â script/timer ê¢â¬â verificar sem rede.** Comando: `python3 -m unittest discover -s tests -v`. Testa mêÂ©tricas, persistêÂªncia, falhas e painel em diretêÂ³rios temporêÂ¡rios.
3. **ê¢Å¡â¢ê¯Â¸Â script/timer ê¢â¬â demonstrar a rodada.** Comando: `python3 bombando/rodada.py --fixtures tests/fixtures`. Descobre exemplos, grava fotos, consulta fixtures dos canais, calcula mêÂ©tricas e gera HTML em `dados/teste/painel/`.
4. **ê°Å¸Â§Â  sessêÂ£o com o Nei ê¢â¬â revisar o resultado.** Comando: `xdg-open dados/teste/painel/index.html`. Comparar nichos, notas, histêÂ³rico, assuntos e ê¢â¬ÅPor que bombou?ê¢â¬Â.
5. **ê¢Å¡â¢ê¯Â¸Â script/timer ê¢â¬â consultar os vêÂ­deos reais.** Comando: `python3 bombando/rodada.py`. Executa descoberta semanal ê¢â â acompanhamento diêÂ¡rio ê¢â â linha de base dos canais ê¢â â mêÂ©tricas ê¢â â painel, com lock e logs.
6. **ê¢Å¡â¢ê¯Â¸Â script ê¢â¬â preencher canais de vêÂ­deos acompanhados.** Comando: `python3 bombando/rodada.py --so-canais`. Respeita cache, teto, janela e filtro sem descoberta ou fotos. Para testar offline, acrescentar `--fixtures tests/fixtures`.
7. **ê°Å¸Â§Â  sessêÂ£o com o Nei ê¢â¬â ativar a rotina quando desejar.** Comando: `bash deploy/instalar.sh`. Instala o agendamento e o servidor.
8. **ê¢Å¡â¢ê¯Â¸Â script/timer ê¢â¬â repetir diariamente.** Comando executado pela unit: `python3 bombando/rodada.py`. êâ¬s 07:15, atualiza as fotos dentro da janela e a cêÂ³pia diêÂ¡ria.
9. **ê°Å¸Â§Â  sessêÂ£o com o Nei ê¢â¬â ajustar e investigar.** Comandos: `journalctl --user -u youtube-bombando.service` e `python3 bombando/rodada.py --so-painel`. Revisar erros, mudar nichos e regenerar a visualizaêÂ§êÂ£o sem nova coleta.

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>