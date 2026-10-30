# Better Subs Davinci

Skill per Claude (Claude Code e app): genera, corregge e inserisce i sottotitoli dei reel verticali (1080×1920) in DaVinci Resolve. Versione **1.0**. Le istruzioni sono in italiano e i sottotitoli prodotti sono in italiano.

## Cosa fa
- Controlla che Resolve sia aperto (lo apre se serve) e chiede progetto, timeline e massimo caratteri per sottotitolo se non li indichi.
- Chiede lo script (allegato, Google Docs/Drive o Notion) se non lo indichi. Senza script la trascrizione esterna è obbligatoria e il controllo degli errori è solo sommario.
- Divide in righe brevi, aggancia i tagli, inserisce tutto con un unico SRT e segna i dubbi con marker.

## Requisiti
- DaVinci Resolve con scripting esterno attivo (Studio) e gli strumenti `run_script` / `run_script_unsafe` collegati a Claude.
- Per la trascrizione esterna: un modello Whisper (es. large-v3-turbo) con `whisperx` o `stable-ts`, e `ffmpeg`. La skill spiega quale modello scegliere.
- Facoltativi: connettori Google Drive e Notion per cercare lo script.

## Installare
```bash
git clone https://github.com/Fob-io/better-subs-davinci-skill ~/.claude/skills/better-subs-davinci
```
Riavvia Claude. Per aggiornare: `cd ~/.claude/skills/better-subs-davinci && git pull`.

## Glossario
`glossario.md` è un modello vuoto. Crea il tuo `glossario.<nome>` accanto a `SKILL.md` con clienti, parole chiave ed errori tipici: la skill lo legge e git lo ignora, quindi resta sul tuo Mac.

## Licenza
MIT, vedi `LICENSE`.
