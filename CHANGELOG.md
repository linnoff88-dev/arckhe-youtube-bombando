# Changelog

## 1.2.1 — 2026-11-12

- Unit da coleta com `Environment=PATH=%h/.local/bin:…`: yt-dlp instalado por pipx/pip --user fica fora do PATH padrêo do systemd (achado do revisor do guia).
- Guia PT/EN/ES, README trilingue e Pages via Actions.

## 1.2.0 — 2026-11-12

- Vídeo mais quente de cada nicho com thumb, views, idade, views/h, × canal, ganho 24h, categoria e gráfico de crescimento.
- Gráfico de crescimento por vídeo (views × horas desde a publicaçêo), mini na tabela e grande ao clicar.
- “Ordenar por” explica a opçêo escolhida e lista todas; opções novas Views e Mais novos.
- Idade em “d h” a partir de 48 h, com a definiçêo (tempo desde a publicaçêo) no painel.

## 1.1.0 — 2026-11-12

- Linha de base por canal, cache de sete dias, amostras por vídeo e modo `--so-canais`.
- Detecçêo lexical de pessoas/convidados e classificaçêo “Por que bombou?” com evidências.
- Resumos por nicho, seis categorias, top 5 “Furaram a bolha” e ordenaçêo por × canal.
- Migraçêo do SQLite, fixtures de canais e testes offline do método e dos limites.
- Miniaturas carregadas direto do YouTube (botêo para ocultar).

- Correçêo: yt-dlp sem `youtube:lang=pt` (lia "3,4 mil" como 34 e perdia inscritos).
- "Quem fala" só por formato explícito (entrevista/papo/podcast com Nome, ft./feat., lista de pessoas); "| Nome", "#24" e Title Case solto davam falso positivo.
- Inscritos e mediana sem casa decimal; teto de canais por rodada 120 → 200.

## 1.0.0 — 2026-11-12

- Descoberta por nichos e acompanhamento diário via yt-dlp com limites e isolamento de erros.
- SQLite, métricas, assuntos ponderados e painel offline com filtros e sparklines.
- Fixtures, testes sem rede e units systemd de usuário para coleta e painel.
