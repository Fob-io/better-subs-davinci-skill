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

### Consigliato: Claude Code
È il modo più comodo: la skill si aggiorna con un comando e Claude Code ha accesso al tuo Mac, quindi trova Whisper e `ffmpeg` per la trascrizione esterna.
```bash
git clone https://github.com/Fob-io/better-subs-davinci-skill ~/.claude/skills/better-subs-davinci
```
Riavvia Claude. Per aggiornare:
```bash
cd ~/.claude/skills/better-subs-davinci && git pull
```

### Alternativa: app Claude (zip)
1. Scarica la repo come zip (Code → Download ZIP) oppure crea lo zip da Terminale. La cartella dentro lo zip deve chiamarsi `better-subs-davinci` e contenere `SKILL.md`:
```bash
git clone https://github.com/Fob-io/better-subs-davinci-skill /tmp/better-subs-davinci && cd /tmp && zip -r better-subs-davinci.zip better-subs-davinci -x "*/.git/*"
```
2. In Claude: Impostazioni → Capacità → Skill → carica `better-subs-davinci.zip`.

Limiti dell'app: servono gli strumenti di Resolve collegati a Claude (connettore DaVinci Resolve), e la trascrizione esterna funziona solo se l'ambiente vede il tuo Mac, Whisper e `ffmpeg`. Gli aggiornamenti vanno rifatti a mano con un nuovo zip.

### Altri agenti (Codex, Cursor, Gemini CLI, Copilot, OpenCode e simili)
`SKILL.md` è il formato Agent Skills, letto da più agenti. Il modo più semplice è l'installer `skills` (serve Node.js 22.20 o successivo), che riconosce gli agenti installati e copia la skill nella cartella giusta di ciascuno:
```bash
npx skills add Fob-io/better-subs-davinci-skill -g
```
`-g` la rende disponibile in tutti i progetti; `-a <agente>` ne sceglie uno (es. `-a claude-code`). Aggiornamenti: `npx skills check` e `npx skills update`.

A mano, copia la cartella `better-subs-davinci` nella cartella skill dell'agente. Percorsi riportati da guide della community, **non verificati sulla documentazione ufficiale**, controllali prima:

| Agente | Cartella utente |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Codex | `~/.codex/skills/` o `~/.agents/skills/` |
| Cursor | `~/.cursor/skills/` o `~/.agents/skills/` |
| Gemini CLI | `~/.gemini/skills/` o `~/.agents/skills/` |
| GitHub Copilot | `~/.copilot/skills/` |
| OpenCode | `~/.config/opencode/skills/` |

**Importante per tutti gli agenti:** la skill pilota DaVinci Resolve con `run_script` / `run_script_unsafe`. L'agente deve avere un server MCP di Resolve collegato che offra quegli strumenti (o equivalenti); senza, la skill si installa ma non può agire sulla timeline. Questa skill è testata solo con Claude.

## Glossario
`glossario.md` è un modello vuoto. Crea il tuo `glossario.<nome>` accanto a `SKILL.md` con clienti, parole chiave ed errori tipici: la skill lo legge e git lo ignora, quindi resta sul tuo Mac.

## Licenza
MIT, vedi `LICENSE`.
