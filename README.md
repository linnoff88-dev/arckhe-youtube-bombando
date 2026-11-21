# YouTube bombando â€” radar do que bomba

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

[![Radar do que bomba](guia/assets/banner.jpg)](https://inematds.github.io/youtube-bombando/guia/)

## O que Ã©

O **youtube-bombando** Ã© um pequeno sistema que roda sozinho no seu computador, toda manhÃ£. Ele procura no YouTube os vÃ­deos da semana nos assuntos que vocÃª escolher, anota as views de cada um por 7 dias e monta um painel com o que estÃ¡ crescendo mais rÃ¡pido. Para cada vÃ­deo, ele compara as views com o normal do prÃ³prio canal e diz se o sinal Ã© de **conteÃºdo**, de **quem posta** (canal grande) ou de **quem fala** (convidado). Ã‰ para quem grava vÃ­deo ou quer comeÃ§ar um canal. Precisa sÃ³ de Python e de uma ferramenta grÃ¡tis que lÃª o YouTube; nÃ£o usa chave nem API paga.

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/youtube-bombando/guia/**

## Requisitos

Python 3.9+ e biblioteca padrÃ£o, sem pip, API ou chaves. A coleta real usa o executÃ¡vel `yt-dlp` disponÃ­vel no PATH; os testes nÃ£o o executam. Linux com systemd de usuÃ¡rio Ã© necessÃ¡rio apenas para a instalaÃ§Ã£o do timer. A pesquisa real requer internet; o modo fixtures e o painel funcionam offline.
## ComeÃ§ar localmente

```bash
python3 -m unittest discover -s tests -v
python3 bombando/rodada.py --fixtures tests/fixtures
# Abra dados/teste/painel/index.html no navegador, inclusive via file://.
python3 bombando/rodada.py --so-painel
```

As fixtures sÃ£o exemplos fixos, identificados no painel; a idade cresce conforme o tempo passa. Elas gravam em `dados/teste/`, com banco e painel separados da produÃ§Ã£o. Repetir a rodada no mesmo dia preserva a foto Ãºnica diÃ¡ria e o cache dos canais. Testes unitÃ¡rios usam diretÃ³rios temporÃ¡rios isolados.

Coleta real (execute quando quiser consultar o YouTube):

```bash
python3 bombando/rodada.py
python3 bombando/rodada.py --nicho 'claude code'
```

## InstalaÃ§Ã£o diÃ¡ria

```bash
bash deploy/instalar.sh
systemctl --user list-timers youtube-bombando.timer
journalctl --user -u youtube-bombando.service
```

O instalador substitui `@REPO@` nas units, copia para `~/.config/systemd/user/`, recarrega o systemd, ativa o timer e inicia/reinicia o servidor de painel. Pode ser repetido. A coleta ocorre Ã s **07:15 no fuso da mÃ¡quina**, com `Persistent=true` (recupera execuÃ§Ã£o perdida), teto de 45 minutos, memÃ³ria de 1 GB e Nice 10. O timer depende do gerenciador de usuÃ¡rio estar ativo; execuÃ§Ã£o apÃ³s logout requer linger configurado pelo administrador.

Painel em `http://localhost:8795`; o servidor escuta `0.0.0.0` conforme a especificaÃ§Ã£o e tambÃ©m fica acessÃ­vel na rede local. Para parar:

```bash
systemctl --user disable --now youtube-bombando.timer youtube-bombando-painel.service
```

A instalaÃ§Ã£o e a coleta real nÃ£o fazem parte dos testes offline.

## ConfiguraÃ§Ã£o

Edite `config.json`. Cada item de `nichos` contÃ©m `nome`, `busca` e `ativo` (booleano). Um vÃ­deo pode pertencer a vÃ¡rios nichos sem duplicar suas fotos. Desativar um nicho impede novas buscas, mas preserva seu histÃ³rico; vÃ­deos jÃ¡ descobertos continuam sendo acompanhados atÃ© o fim da janela.

| Campo | PadrÃ£o | FunÃ§Ã£o |
|---|---:|---|
| `por_nicho_busca` | 40 | MÃ¡ximo de resultados por busca semanal |
| `dias_acompanhar` | 7 | Janela desde a primeira descoberta |
| `max_detalhes_por_rodada` | 250 | Teto de consultas de detalhes |
| `max_canais_por_rodada` | 120 (este repo usa 200) | Teto de tentativas de consulta de canais por rodada |
| `pausa_s` | 1.5 | Intervalo mÃ­nimo entre fim e inÃ­cio das chamadas |
| `idioma` | pt | Reservado; nÃ£o Ã© passado ao yt-dlp (com `lang=pt` ele lÃª "3,4 mil" como 34) |
| `regiao` | BR | PaÃ­s informado ao mecanismo geo-bypass do yt-dlp |

A regiÃ£o nÃ£o garante resultados geolocalizados; o YouTube tambÃ©m considera a conexÃ£o. O filtro semanal Ã© o parÃ¢metro de pesquisa solicitado ao YouTube. O sistema preserva os resultados devolvidos pelo extrator.

Cada subprocesso tem timeout de 120 segundos, socket de 20 segundos e retries limitados. Erros sÃ£o registrados por nicho/vÃ­deo e a rodada continua. VÃ­deos explicitamente privados/removidos ficam indisponÃ­veis; timeout e 429 nÃ£o provocam essa marcaÃ§Ã£o. Fotos jÃ¡ coletadas hoje sÃ£o puladas. O teto prioriza a velocidade da Ãºltima foto; como ela sÃ³ guarda o dia, usa meio-dia UTC como referÃªncia dessa prioridade, com desempate por descoberta e ID. VÃ­deos ainda sem fotos tÃªm prioridade de velocidade zero.

## MÃ©tricas e nota

Datas sÃ£o armazenadas em UTC. `timestamp` tem preferÃªncia sobre `upload_date`; este Ãºltimo representa meia-noite UTC, uma aproximaÃ§Ã£o quando a hora nÃ£o Ã© fornecida.

- **Idade (h):** horas desde a publicaÃ§Ã£o atÃ© a geraÃ§Ã£o do painel, limitada a zero para datas futuras.
- **Velocidade:** views da foto mais recente / `max(idade_h, 1)`; Ã© a mÃ©dia desde a publicaÃ§Ã£o, nÃ£o a velocidade instantÃ¢nea.
- **Ganho 24h:** `(views de hoje âˆ’ views da foto anterior) / dias entre fotos`. Sem foto de hoje ou sem anterior, fica ausente. Quedas de contagem sÃ£o preservadas.
- **Multiplicador:** views / inscritos. Inscritos ausentes ou zero deixam a mÃ©trica ausente.
- **Nota:** `z-score(velocidade dentro do nicho) + ln(1 + multiplicador)`. O z-score usa desvio padrÃ£o populacional; com desvio zero ou velocidade ausente, vale zero. Multiplicador ausente contribui zero. A nota pode ser negativa. Em â€œTodosâ€, usa a maior nota entre os nichos do vÃ­deo; ao selecionar um nicho, usa a nota daquele nicho. IndisponÃ­veis ficam no fim e nÃ£o entram nas estatÃ­sticas nem nos destaques.

Zero e ausÃªncia sÃ£o distintos: `â€”` indica dado desconhecido. Views da busca ficam no banco como estimativa inicial, separadas das fotos. A tabela exibe as fotos confirmadas. O **grÃ¡fico de crescimento** (coluna â€œCrescimentoâ€, clique para ampliar, e no vÃ­deo mais quente de cada nicho) pÃµe as views no eixo vertical e as horas desde a publicaÃ§Ã£o no horizontal. ComeÃ§a em 0 views na publicaÃ§Ã£o; o trecho tracejado vai atÃ© a 1Âª mediÃ§Ã£o (sÃ³ a mÃ©dia, nÃ£o hÃ¡ pontos no meio) e a linha cheia liga as mediÃ§Ãµes diÃ¡rias das 07:15, cada uma na idade que o vÃ­deo tinha naquele dia.

**Ordenar por** (o painel explica a opÃ§Ã£o escolhida embaixo do seletor):

| OpÃ§Ã£o | O que mede |
|---|---|
| Nota | z-score das views/h dentro do nicho + ln(1 + multiplicador): o mais quente agora |
| Velocidade | views Ã· idade em horas; favorece vÃ­deos novos |
| Ganho em 24h | views ganhas desde a mediÃ§Ã£o anterior (precisa de 2 dias de coleta) |
| Multiplicador | views Ã· inscritos; acima de 1Ã— passou o tamanho do canal |
| Ã— canal | views Ã· mediana dos Ãºltimos vÃ­deos do canal; â‰¥ 3Ã— furou a bolha |
| Views | total de views, sem olhar idade nem canal |
| Mais novos | menor idade primeiro |

**Idade** = tempo desde que o vÃ­deo foi publicado no YouTube (nÃ£o desde que o sistema o achou); aparece em horas atÃ© 48 h e depois em dias + horas.

**Assuntos em alta:** top 30 vÃ­deos por nota em cada nicho (e top 30 geral sem duplicar IDs). Com pelo menos 5 vÃ­deos, um termo precisa aparecer em dois tÃ­tulos; em grupos menores, basta um. Extrai palavras e bigramas adjacentes, remove stopwords PT/EN, palavras de uma letra e nÃºmeros isolados. Cada termo recebe uma contribuiÃ§Ã£o por tÃ­tulo, igual Ã  velocidade do vÃ­deo. Exibe os 15 maiores pesos. Ã‰ uma heurÃ­stica lexical, sem IA ou interpretaÃ§Ã£o semÃ¢ntica.

## Por que bombou? â€” mÃ©todo e limites

O sinal principal Ã© **fator_canal = views / mediana de views do prÃ³prio canal**.
Consulta os Ãºltimos 20 vÃ­deos do canal uma vez e guarda a amostra em SQLite.
A mediana exclui o prÃ³prio vÃ­deo analisado e publicaÃ§Ãµes com menos de 3 dias.
Quando a data de uma entrada nÃ£o estÃ¡ disponÃ­vel, exclui essa entrada se estiver
entre as duas primeiras da lista. Views desconhecidas nÃ£o entram; zero Ã© vÃ¡lido.
Cada vÃ­deo tem sua mediana e `n_amostra` recalculados a partir do cache, inclusive
quando hÃ¡ vÃ¡rios vÃ­deos do mesmo canal. Na tabela `canais`, mediana e tamanho
representam o vÃ­deo prioritÃ¡rio na coleta; `amostra` preserva os dados para os demais.

O cache vale atÃ© 7 dias inclusive; sÃ³ Ã© renovado apÃ³s esse prazo. Os canais dos
vÃ­deos acompanhados (disponÃ­veis, dentro de `dias_acompanhar`) de maior nota tÃªm
prioridade. O teto conta tentativas, inclusive falhas. Erros preservam o cache
anterior e ficam no log. A data do cache aparece em `canal_atualizado` nos dados.
NÃ£o hÃ¡ chamada extra para descriÃ§Ã£o: ela Ã© guardada quando chega no detalhe.

`pessoas` em `config.json` Ã© a lista editÃ¡vel de nomes conhecidos. O detector
procura nomes completos sem diferenciar maiÃºsculas e minÃºsculas, no tÃ­tulo e na
descriÃ§Ã£o disponÃ­vel, alÃ©m de sinais de formato: `com Nome Sobrenome`, `ft.`,
`feat.`, entrevista, podcast, palestra, `| Nome Sobrenome`, `- NOME SOBRENOME`
e `#123`. O trecho encontrado acompanha o motivo. Uma menÃ§Ã£o nÃ£o comprova
participaÃ§Ã£o; o mÃ©todo Ã© uma heurÃ­stica, nÃ£o uma demonstraÃ§Ã£o de causalidade.

Porte por inscritos da foto mais recente (ou do canal, se ausentes): pequeno
abaixo de 10 mil; mÃ©dio de 10 mil a 500 mil inclusive; grande acima de 500 mil.
Sem inscritos, o porte Ã© desconhecido. Regras, nesta ordem:

| CondiÃ§Ã£o | Categoria |
|---|---|
| Sem views confirmadas ou mediana ausente/zero | Sem dados |
| Fator â‰¥ 3 e pessoa/formato de convidado | ConteÃºdo + quem fala |
| Fator â‰¥ 3 sem pessoa/formato de convidado | ConteÃºdo |
| Fator â‰¥ 1,5 e pessoa/formato de convidado | Quem fala |
| Fator < 1,5 e canal grande | Quem posta |
| Demais casos | Misto |

A regra combinada vem primeiro para nÃ£o ser absorvida por â€œQuem falaâ€. Os limiares
sÃ£o constantes no topo de `metricas.py`. AusÃªncia nunca Ã© estimada por inscritos.
Os motivos mostram fator, mediana, porte e evidÃªncia de convidado.

O resumo geral e por nicho considera vÃ­deos disponÃ­veis com views confirmadas,
sem duplicaÃ§Ã£o dentro de cada grupo. A legenda mostra contagens e percentuais de
vÃ­deos e views nas **seis categorias**, incluindo a combinada separadamente para
nÃ£o contar suas views duas vezes. Sem views totais, percentuais sÃ£o zero. A
conclusÃ£o prioriza insuficiÃªncia de dados (â‰¥ 50% das views sem base); depois,
maioria estrita (> 50%) de conteÃºdo (puro + combinado), quem posta ou quem fala;
sem maioria, informa sinais mistos. â€œFuraram a bolhaâ€ lista atÃ© 5 vÃ­deos com
fator â‰¥ 3 de canais pequenos/mÃ©dios, ordenados pelo fator.

Para preencher apenas as linhas de base jÃ¡ existentes, mantendo descoberta e
fotos intactas (consulta real de canais):

```bash
python3 bombando/rodada.py --so-canais
python3 bombando/rodada.py --so-canais --nicho 'claude code'
# Variante inteiramente offline, usando o banco de demonstraÃ§Ã£o:
python3 bombando/rodada.py --so-canais --fixtures tests/fixtures
```

`--so-canais` e `--so-painel` sÃ£o mutuamente exclusivos. No modo canais, nenhuma
consulta necessÃ¡ria (cache vÃ¡lido ou seleÃ§Ã£o vazia) Ã© sucesso; falha em todas as
consultas tentadas retorna 1, e sucesso parcial retorna 0. O painel mostra erros.

## Arquivos e operaÃ§Ã£o

`dados/bombando.db` contÃ©m vÃ­deos, nichos, ligaÃ§Ãµes, fotos Ãºnicas por `(id, dia)`, rodadas e canais com suas amostras. A abertura do banco migra o esquema antigo sem apagar dados. `dados/.lock` usa `flock` nÃ£o bloqueante para evitar concorrÃªncia. `dados/logs/AAAA-MM-DD.log` guarda falhas e resumo. `painel/index.html` contÃ©m CSS, JS, dados e SVG inline; `painel/dados.json` tem os mesmos dados e `painel/historico/AAAA-MM-DD.html` Ã© a Ãºltima cÃ³pia daquele dia. Miniaturas sÃ³ carregam apÃ³s clicar em â€œCarregar miniaturas (internet)â€; fontes usam fallback de sistema, sem CDN.

`--so-painel` mantÃ©m o status e horÃ¡rio da Ãºltima coleta e regenera as mÃ©tricas. `--nicho NOME` restringe busca, acompanhamento e canais, preservando os outros dados no painel. O cÃ³digo de saÃ­da Ã© zero quando algum nicho teve busca vÃ¡lida, mesmo com erros parciais; nenhum nicho bem-sucedido retorna 1. Falhas estruturais (configuraÃ§Ã£o, banco ou escrita) tambÃ©m retornam 1. Lock ocupado Ã© execuÃ§Ã£o ignorada com cÃ³digo zero.

As fixtures ficam em `tests/fixtures/buscas.json` (mapa por termo de busca, com `entries`), `detalhes.json` (mapa por ID) e `canais.json` (mapa por channel_id com entries). `{"erro":"mensagem"}` simula falha; `"indisponivel":true` simula remoÃ§Ã£o. NÃ£o hÃ¡ fallback de fixtures para rede.

## O processo, do inÃ­cio ao fim

1. **ðŸ§  sessÃ£o com o Nei â€” escolher os nichos.** Comando: `python3 -m json.tool config.json`. Conferir nomes, buscas, limites e campos `ativo` depois de editar o arquivo.
2. **âš™ï¸ script/timer â€” verificar sem rede.** Comando: `python3 -m unittest discover -s tests -v`. Testa mÃ©tricas, persistÃªncia, falhas e painel em diretÃ³rios temporÃ¡rios.
3. **âš™ï¸ script/timer â€” demonstrar a rodada.** Comando: `python3 bombando/rodada.py --fixtures tests/fixtures`. Descobre exemplos, grava fotos, consulta fixtures dos canais, calcula mÃ©tricas e gera HTML em `dados/teste/painel/`.
4. **ðŸ§  sessÃ£o com o Nei â€” revisar o resultado.** Comando: `xdg-open dados/teste/painel/index.html`. Comparar nichos, notas, histÃ³rico, assuntos e â€œPor que bombou?â€.
5. **âš™ï¸ script/timer â€” consultar os vÃ­deos reais.** Comando: `python3 bombando/rodada.py`. Executa descoberta semanal â†’ acompanhamento diÃ¡rio â†’ linha de base dos canais â†’ mÃ©tricas â†’ painel, com lock e logs.
6. **âš™ï¸ script â€” preencher canais de vÃ­deos acompanhados.** Comando: `python3 bombando/rodada.py --so-canais`. Respeita cache, teto, janela e filtro sem descoberta ou fotos. Para testar offline, acrescentar `--fixtures tests/fixtures`.
7. **ðŸ§  sessÃ£o com o Nei â€” ativar a rotina quando desejar.** Comando: `bash deploy/instalar.sh`. Instala o agendamento e o servidor.
8. **âš™ï¸ script/timer â€” repetir diariamente.** Comando executado pela unit: `python3 bombando/rodada.py`. Ã€s 07:15, atualiza as fotos dentro da janela e a cÃ³pia diÃ¡ria.
9. **ðŸ§  sessÃ£o com o Nei â€” ajustar e investigar.** Comandos: `journalctl --user -u youtube-bombando.service` e `python3 bombando/rodada.py --so-painel`. Revisar erros, mudar nichos e regenerar a visualizaÃ§Ã£o sem nova coleta.

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>