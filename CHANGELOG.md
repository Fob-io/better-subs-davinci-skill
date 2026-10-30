# Changelog

Le versioni seguono `metadata.version` in `SKILL.md`. Più recente in alto.

## 1.2.0 (2026-10-30)
Sottotitoli sincronizzati con la voce. Prove con Resolve vero su due timeline, e sul modello di inserimento con un Resolve simulato.

### Tempi
- `stable-ts` con Whisper large-v3-turbo è ora il predefinito per testo e tempi. Whisper puro poteva anticipare l'inizio delle parole di diversi frame. Su una clip di prova, rispetto a marker messi a mano: scarto medio assoluto 0,8 frame contro 4,1, massimo 2,8 contro 12,8. Su 218 secondi i tempi sono risultati alla pari con quelli di Resolve.
- Nuovo modello "Controllo voce": confronta l'inizio dei sub con gli attacchi della voce. Riporta scarto medio assoluto e massimo, e segnala senza correggere, perché la forma d'onda può scambiare respiri e rumori per attacchi.
- Il motore interno di Resolve resta come ripiego, con l'avviso sui tratti saltati nei video lunghi e il controllo dello stato del lavoro.

### Inserimento
- `RANGES = [(inizio, fine, righe), ...]` gestisce più blocchi con vuoti in mezzo: sostituisce `L`, `RANGE`, `VOICE` e `SONG`.
- `SOSTITUISCI` rigenera tutta la traccia senza ricopiare i sub esistenti. `CONFERMATO` è obbligatorio se la traccia non è vuota.
- Lo script si ferma se la timeline aperta non è quella dichiarata (`TIMELINE`), e non crea o tocca nulla prima dei controlli.
- Con la traccia vuota non si chiama più `DeleteClips` e non si prova a ripristinare un backup vuoto.

### Trascrizione esterna
- Nuovo modello "Export voce": ricostruisce la traccia voce dai clip sorgente, con sorgenti a frame rate diversi, e segnala i clip con velocità cambiata.
- Nuovo modello "Trascrizione" e procedura standard di installazione in un venv (`~/venv-subs`), sempre con consenso.
- Punteggiatura aggiunta prima di suddividere, a parità di parole.

### Flusso
- Leggere il glossario è la prima azione, e il riepilogo dichiara cosa è stato usato. Niente marker Fucsia per ciò che il glossario già copre.
- Notion: con un link rotto o senza un nome, la skill cerca per nome di progetto o timeline, propone il candidato e chiede conferma. Controlla che lo script sia compatibile con la timeline.
- Suddivisione più precisa: niente frasi che finiscono a metà riga né locuzioni spezzate. `MAXC` + 4 anche per le frasi CTA e per evitare righe sotto gli 8 frame.
- Marker: `Fuchsia` è il colore giusto; se rigeneri i sub, i marker "Controlla sub" vengono riposizionati.
- La skill dice subito che lo stile dei sottotitoli resta manuale.

### Glossario
- Il glossario personale si chiama ora `glossario.<nome>.md` (per esempio `glossario.fabio.md`), così si apre e si legge bene. I vecchi nomi senza `.md` funzionano ancora.

## 1.1.5 (2026-10-30)
Correzioni dopo una revisione del codice. Provata solo con un Resolve simulato, non con Resolve vero.

### Sicurezza dei dati
- Prima di sostituire la traccia sottotitoli lo script salva un backup SRT della traccia attuale (`bsd-<timeline>-<data>-backup.srt` nella cartella temporanea) e lo comunica.
- Se l'inserimento fallisce (per esempio l'import dell'SRT torna vuoto), la traccia viene ripristinata dal backup. Prima restava cancellata.
- Se la traccia contiene già sottotitoli e non hai chiesto di rigenerarli, la skill chiede conferma.
- Lo script si ferma senza toccare nulla se `RANGE` taglia a metà un sottotitolo esistente.
- La pulizia del Media Pool riguarda solo gli SRT importati dalla skill (nomi `bsd-...`).

### Trascrizione esterna
- Nuovo parametro `EXT_WORDS = [(parola, frame_inizio, frame_fine), ...]`: il percorso senza script ora ha il codice per ricevere le parole trascritte fuori da Resolve. Prima lo script leggeva solo i sottotitoli già presenti.
- Nuovo parametro `VOICE` per far partire e finire i sottotitoli sulla prima e sull'ultima clip con voce.
- La traccia sottotitoli viene creata se manca anche nel percorso esterno.

### Altro
- Le domande (progetto, timeline, caratteri, script) sono raccolte in un solo messaggio.
- Nome del file SRT unico per esecuzione, adatto anche a timeline con `/` nel nome.
- Controlli in più: righe con zero parole, tagli ordinati, tracce assenti.
- Il riferimento ai file `glossario.*` ora ha il suo passo nella ricognizione.
- Tabella modelli: peso di Parakeet v3 corretto a ~0,6 GB (prima 1,3 GB per un doppio conteggio) e abbinamento tra strumenti e formati dei modelli chiarito.

## 1.1 (2026-10-30)
Indipendente dall'agente.

- I nomi degli strumenti (`run_script`, `get_resolve_status`, `launch_resolve`, ricerca Drive e Notion) sono ora esempi: se il server MCP usa altri nomi, si usano gli equivalenti.
- Tolta l'assunzione di `ToolSearch`, che esiste solo in Claude.
- README: installazione compatta per Claude Code, Codex e OpenCode (`npx skills add` o `git clone`), zip dalla Release per l'app Claude, requisiti e limiti dichiarati.
- Versione spostata in `metadata.version`, per superare la validazione del caricamento.

## 1.0 (2026-10-30)
Prima versione condivisibile.

- Controlla che DaVinci Resolve sia aperto e lo avvia se serve.
- Chiede, solo se non li hai già indicati: progetto, timeline, massimo di caratteri per sottotitolo (`MAXC`, 20 di default) e script (allegato, Google Docs/Drive o Notion).
- Senza script: avvisa che il controllo degli errori è solo sommario e rende obbligatoria la trascrizione esterna (Whisper con `whisperx` o `stable-ts`). Consiglia tre modelli con peso e motivo di scelta: large-v3-turbo (default), large-v3 e Parakeet v3. Se il modello manca, si ferma e chiede il permesso.
- Corregge il testo seguendo audio e script, con marker Fucsia per i dubbi. Non trascrive mai i testi delle canzoni (marker Giallo).
- Divide in righe brevi per il formato verticale 1080×1920, senza spezzare parole né chiudere con articoli o preposizioni, agganciando i sottotitoli ai tagli.
- Inserisce tutto con un unico SRT, controlla il risultato, salva e riassume.
- Glossari separati: `glossario.md` è il modello vuoto condiviso, `glossario.<nome>` è personale e git lo ignora.
