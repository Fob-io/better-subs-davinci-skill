# Better Subs Davinci

Skill in formato Agent Skills (Claude Code, app Claude e altri agenti): genera, corregge e inserisce i sottotitoli dei reel verticali (1080×1920) in DaVinci Resolve. Versione **1.1**. Le istruzioni sono in italiano e i sottotitoli prodotti sono in italiano.

## Cosa fa
- Controlla che Resolve sia aperto (lo apre se serve) e chiede progetto, timeline e massimo caratteri per sottotitolo se non li indichi.
- Chiede lo script (allegato, Google Docs/Drive o Notion) se non lo indichi. Senza script la trascrizione esterna è obbligatoria e il controllo degli errori è solo sommario.
- Divide in righe brevi, aggancia i tagli, inserisce tutto con un unico SRT e segna i dubbi con marker.

## Requisiti
- DaVinci Resolve con scripting esterno attivo (Studio) e gli strumenti `run_script` / `run_script_unsafe` collegati a Claude.
- Per la trascrizione esterna: un modello Whisper (es. large-v3-turbo) con `whisperx` o `stable-ts`, e `ffmpeg`. La skill spiega quale modello scegliere.
- Facoltativi: connettori Google Drive e Notion per cercare lo script.

## Installare

La cartella della skill deve chiamarsi `better-subs-davinci`. Dopo l'installazione riavvia l'agente.

### Claude Code, Codex, OpenCode
Un solo comando (serve Node.js 22.20 o successivo): l'installer `skills` copia la skill nella cartella giusta di ogni agente.
```bash
npx skills add Fob-io/better-subs-davinci-skill -g -a claude-code -a codex -a opencode
```
Aggiornare: `npx skills update`. Verificare: `npx skills check`.

Senza Node, clona direttamente nella cartella dell'agente:

| Agente | Comando |
|---|---|
| Claude Code | `git clone https://github.com/Fob-io/better-subs-davinci-skill ~/.claude/skills/better-subs-davinci` |
| Codex | `git clone https://github.com/Fob-io/better-subs-davinci-skill ~/.agents/skills/better-subs-davinci` |
| OpenCode | `git clone https://github.com/Fob-io/better-subs-davinci-skill ~/.config/opencode/skills/better-subs-davinci` |

Aggiornare: `git pull` dentro la cartella clonata.

### App Claude (zip)
Scarica `better-subs-davinci-vX.Y.zip` dalla [pagina Release](https://github.com/Fob-io/better-subs-davinci-skill/releases/latest) e caricalo in Impostazioni → Capacità → Skill. Per ogni nuova versione scarica e carica un nuovo zip.

### Altri agenti
Cursor, Gemini CLI, GitHub Copilot e altri che leggono il formato Agent Skills: lancia lo stesso comando senza `-a` e scegli l'agente quando richiesto.

### Prima di usarla
La skill pilota DaVinci Resolve con `run_script` / `run_script_unsafe`: l'agente deve avere un server MCP di Resolve collegato, altrimenti la skill si installa ma non può agire sulla timeline. Con l'app Claude, la trascrizione esterna funziona solo se l'ambiente vede il tuo Mac, Whisper e `ffmpeg`. Provata solo con Claude.

## Glossario
`glossario.md` è un modello vuoto. Crea il tuo `glossario.<nome>` accanto a `SKILL.md` con clienti, parole chiave ed errori tipici: la skill lo legge e git lo ignora, quindi resta sul tuo Mac.

## Licenza
MIT, vedi `LICENSE`.
