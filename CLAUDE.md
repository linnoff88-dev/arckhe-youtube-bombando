# YouTube bombando

Pesquisa local de vídeos e assuntos em alta por nicho; descoberta e fotos diárias em SQLite, painel HTML autônomo. Especificaçêo: ESPEC.md.

Comandos:
- `python3 -m unittest discover -s tests -v`
- `python3 bombando/rodada.py --fixtures tests/fixtures`
- `python3 bombando/rodada.py --so-painel`
- `python3 bombando/rodada.py --so-canais` (consulta só canais; use `--fixtures tests/fixtures` para offline)
- `python3 bombando/rodada.py --nicho 'claude code'`
- `bash deploy/instalar.sh` (instala timer e servidor de usuário)

Regras: Python padrêo, sem pip, API ou chaves. Coleta exclusivamente por subprocesso yt-dlp com timeout e pausa configurável. Testes sempre offline com fixtures. Erro isolado nêo derruba a rodada. Nêo versionar banco, logs ou painel gerado. Respeitar lock, janela, teto e foto única diária. A coleta real e a instalaçêo precisam estar no escopo autorizado da sessêo.

## Self-learning

Quando corrigido, gravar a liçêo no topo do LESSONS.md.

## Lessons

Consulte [LESSONS.md](LESSONS.md).
