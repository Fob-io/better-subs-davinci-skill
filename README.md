# Better Subs Davinci

Skill in formato Agent Skills (Claude Code, app Claude e altri agenti) che genera, corregge e inserisce in DaVinci Resolve i sottotitoli dei reel verticali (1080×1920). Versione 1.2. Le istruzioni e i sottotitoli sono in italiano.

## Cosa fa
- Controlla che Resolve sia aperto, lo apre se serve, e chiede progetto, timeline e massimo di caratteri per sottotitolo se non li indichi.
- Chiede lo script (allegato, Google Docs/Drive o Notion) se non lo indichi. Senza script la trascrizione esterna è obbligatoria e il controllo degli errori è solo sommario.
- Divide in righe brevi, aggancia i tagli, inserisce tutto con un unico SRT e segna i dubbi con marker.

## Requisiti
- DaVinci Resolve Studio con lo scripting esterno attivo, e un server MCP di Resolve collegato all'agente che offra `run_script` / `run_script_unsafe`. Senza, la skill si installa ma non può agire sulla timeline.
- Per la trascrizione esterna: un modello Whisper (per esempio large-v3-turbo) con `whisperx` o `stable-ts`, più `ffmpeg`. La skill spiega quale modello scegliere.
- Facoltativi: i connettori Google Drive e Notion, per cercare lo script.
- Provata finora solo con Claude.

## Installare

La cartella della skill deve chiamarsi `better-subs-davinci`. Dopo l'installazione riavvia l'agente.

### Claude Code, Codex, OpenCode
Basta un comando, con Node.js 22.20 o successivo. L'installer `skills` installa la skill nella cartella giusta di ogni agente.
```bash
npx skills add Fob-io/better-subs-davinci-skill -g -a claude-code -a codex -a opencode
```
Per aggiornare: `npx skills update`. Per controllare: `npx skills check`. Un aggiornamento potrebbe non rispettare un tuo `glossario.<nome>` dentro la cartella installata: tienine una copia fuori, oppure usa `git clone`, dove git lo ignora.

Senza Node, clona direttamente nella cartella dell'agente:

| Agente | Comando |
|---|---|
| Claude Code | `git clone https://github.com/Fob-io/better-subs-davinci-skill ~/.claude/skills/better-subs-davinci` |
| Codex | `git clone https://github.com/Fob-io/better-subs-davinci-skill ~/.agents/skills/better-subs-davinci` |
| OpenCode | `git clone https://github.com/Fob-io/better-subs-davinci-skill ~/.config/opencode/skills/better-subs-davinci` |

Per aggiornare: `git pull` dentro la cartella clonata.

### App Claude (zip)
Scarica `better-subs-davinci-vX.Y.zip` dalla [pagina Release](https://github.com/Fob-io/better-subs-davinci-skill/releases/latest) e caricalo in Impostazioni → Capacità → Skill. A ogni nuova versione devi scaricare e caricare un nuovo zip. Nell'app la trascrizione esterna funziona solo se l'ambiente vede il tuo Mac, Whisper e `ffmpeg`.

### Altri agenti
Cursor, Gemini CLI, GitHub Copilot e gli altri che leggono il formato Agent Skills: lancia lo stesso comando senza `-a` e scegli l'agente quando te lo chiede.

## Glossario
`glossario.md` è un modello vuoto. Crea il tuo `glossario.<nome>` accanto a `SKILL.md` con clienti, parole chiave ed errori tipici. La skill lo legge, git lo ignora e quindi resta sul tuo Mac.

## Licenza
MIT, vedi `LICENSE`.
