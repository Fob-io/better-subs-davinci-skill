---
name: "better-subs-davinci"
metadata:
  version: "1.1"
description: "Better Subs Davinci: sottotitoli per reel verticali in DaVinci Resolve (trascrizione, correzione su script, suddivisione, SRT, marker). Usala per \"fai i subs\" o \"rigenera i subs in questo punto\"."
---

# Better Subs Davinci — v1.1
Sottotitoli in DaVinci Resolve per reel verticali.

Lavora sulla timeline aperta in Resolve tramite gli strumenti `run_script` / `run_script_unsafe` (se l'agente li tiene da attivare o differiti, caricali prima con il suo meccanismo; i nomi possono cambiare a seconda del server MCP di Resolve collegato, in tal caso usa gli equivalenti). Ogni chiamata è stateless: rileggi sempre lo stato dalla timeline. Rispondi in italiano.

## 0. Resolve, progetto e timeline
- Verifica che DaVinci Resolve sia aperto (es. `get_resolve_status`). Se non lo è, aprilo con lo strumento di avvio (es. `launch_resolve`) e aspetta che risponda (il primo avvio può richiedere fino a un minuto). Se l'avvio fallisce, dillo all'utente e fermati.
- Leggi progetto e timeline attualmente aperti.
- **Se l'utente non ha già indicato progetto e timeline** nella richiesta, chiedigli su quali devi generare i sottotitoli, proponendo come default quelli aperti ora (es. "Genero su *NN - Titolo V1* nel progetto *X*, giusto?"). Se ha già indicato progetto e/o timeline, usa quelli: se diversi da quelli aperti, aprili (`LoadProject`, `SetCurrentTimeline`) e verifica di essere sulla timeline giusta prima di toccare qualcosa.
- Se progetto o timeline indicati non esistono, mostra i nomi disponibili e chiedi quale.
- **Se l'utente non ha già indicato quanti caratteri deve contenere al massimo ogni sottotitolo**, chiedilo (nella stessa domanda di progetto e timeline, se manca anche quello), proponendo 20 come valore tipico per i reel verticali 1080×1920. Chiamalo `MAXC` e usalo ovunque sotto: generazione (sezione 3), suddivisione (sezione 5), riepilogo (sezione 8) e controllo `long` dello script. Se l'utente l'ha già detto, usa quel valore senza richiedere.

## 1. Ricognizione
- Leggi: progetto, timeline, fps, `GetStartFrame()` e TC di partenza (può essere 01:00:00:00 = 86400 a 24 fps), In/Out (`GetMarkInOut`), marker esistenti con colori, tracce audio (quale è la voce, di solito A1), tagli su V1 (`GetStart` degli item V1), sottotitoli già presenti.
- Range di lavoro, in ordine di priorità: indicazione in chat (es. "ignora da 00:00:45:26") → In/Out dell'utente → tutta la timeline.
- **Modalità parziale** ("rigenera nel punto segnato"): tocca solo quel tratto; i testi del resto della traccia (anche modificati a mano dall'utente) devono restare identici.
- Annota i colori marker già usati: non riusarli (Fucsia e Giallo di solito liberi).

## 2. Script
**Se l'utente non ha già indicato lo script nella richiesta** (allegato, testo incollato, nome o link di un file su Google Docs/Drive o su Notion, oppure "senza script"), **chiediglielo, a ogni richiesta, prima di generare i sottotitoli.** Se l'ha già indicato, usa quello e non chiedere di nuovo. Non indovinarlo dal nome del progetto o della timeline e non riusare quello di una richiesta precedente. Fai la domanda subito dopo la ricognizione (sezione 1) e aspetta la risposta prima di passare alla sezione 3. Formula (adattala, resta breve):

> Mi serve lo script del video. Come me lo dai?
> 1. Allegato o incollato in chat
> 2. Nome (o link) del file su Google Docs / Drive
> 3. Nome (o link) della pagina su Notion
> 4. Nessuno script: sottotitoli solo dall'audio, con trascrizione esterna obbligatoria (attenzione: senza script il controllo degli errori è solo sommario)

Poi, in base alla risposta:
- **Allegato o incollato**: usa direttamente quel testo.
- **Google Docs / Drive**: lo strumento di ricerca file del connettore (es. `search_files` con `title contains '<nome>'`), poi quello di lettura (es. `read_file_content`). Se i risultati sono più d'uno o nessuno, mostra i candidati e chiedi quale; non scegliere a caso.
- **Notion**: cerca la pagina per titolo con lo strumento di ricerca Notion (es. `notion-search`) oppure apri il link dato. Se ci sono più corrispondenze, chiedi quale. Leggi anche Note/proprietà della pagina (CTA, parole chiave, avvertenze).
- **Nessuno script**: **insisti sul fatto che senza script il controllo è solo sommario.** Senza un testo di riferimento non puoi sapere come sono scritti nomi, marchi, titoli e parole chiave, né se la trascrizione automatica ha saltato o storpiato dei pezzi: puoi correggere solo gli errori evidenti (grammatica, parole senza senso). Prima di procedere dillo chiaramente e invita l'utente a darti lo script (anche solo incollato in chat). Se conferma di voler andare avanti senza, **la trascrizione esterna (sezione 3A) è un passaggio obbligatorio**: il motore interno di Resolve non permette di scegliere il modello e da solo, senza script a correggerlo, non è affidabile. Marca Fucsia ogni dubbio su nomi e termini.
- Se il connettore richiesto (Drive o Notion) non è disponibile o la ricerca fallisce, dillo e chiedi di allegare o incollare il testo. Non proseguire senza script a meno che l'utente scelga l'opzione 4.

## 3A. Trascrizione esterna (obbligatoria se non c'è uno script)
Resolve non permette di scegliere il modello di trascrizione. Per una generazione migliore serve un modello installato **fuori** da Resolve, e **se non c'è uno script la trascrizione va fatta con quello, non con `CreateSubtitlesFromAudio`**. Con uno script puoi usare il motore interno (sezione 3), ma il percorso esterno resta consigliato.
- **Quale modello consigliare all'utente.** Spiega la scelta in due righe, con peso su disco e motivo. Peso = spazio occupato su disco (approssimato, può variare con la versione). Tempi per parola con `whisperx` o `stable-ts`.

  | Modello | Peso | Scegli se | Contro |
  |---|---|---|---|
  | **Whisper large-v3-turbo** (`openai-whisper`, file `large-v3-turbo.pt`) | ~1,5 GB | È il default: ottimo italiano, veloce, poco spazio. Va bene per quasi tutti i reel. | Su audio molto sporco (musica sotto, più voci) è un filo meno preciso di large-v3. |
  | **Whisper large-v3** (`faster-whisper`, cartella `faster-whisper-large-v3`) | ~3 GB | Massima accuratezza su parlato difficile: accenti, nomi propri, musica sotto la voce. È il modello che usa `whisperx`. | Più lento (su Mac gira su CPU), occupa più spazio, può inventare testo nei silenzi. |
  | **Parakeet v3** (`sherpa-onnx`, int8) | ~1,3 GB | Quando contano velocità e precisione dei tempi per parola, su parlato pulito. | Italiano meno collaudato, richiede `sherpa-onnx`, niente allineamento `whisperx`. Usalo solo se l'utente lo chiede. |

  Regola breve: **turbo di default; large-v3 se turbo sbaglia nomi o l'audio è sporco; Parakeet solo per velocità su audio pulito.**
- **Verifica prima di trascrivere** (senza eseguire nulla): modulo Python e file del modello esistono (es. `~/.cache/whisper/large-v3-turbo.pt`, `~/.cache/huggingface/hub/models--Systran--faster-whisper-large-v3`), e `ffmpeg` è nel PATH. Cerca i moduli in tutti i Python presenti (di sistema, Homebrew, venv): possono stare in uno solo e non nel `python3` del PATH.
- **Se il modello non è installato**, fermati: dillo all'utente, indica quale scarichi e quanto pesa, e aspetta il suo ok prima di installare qualsiasi cosa. Non ripiegare in silenzio sul motore interno.
- **Procedura:**
  1. Esporta dalla timeline il solo audio della voce (tracce musica/SFX/canzoni escluse), nel range di lavoro.
  2. Trascrivi in italiano con timestamp **per parola** e riporta i tempi a frame della timeline (offset = inizio del range − `GetStartFrame()`).
  3. Passa le parole alle sezioni 4 e 5 (correzione e suddivisione) e inserisci tutto con l'unico SRT della sezione 6: non serve `CreateSubtitlesFromAudio`.
- Canzoni: vale la regola della sezione 4 (non trascrivere i testi cantati; tratto vuoto + marker Giallo).

## 3. Generazione (motore interno di Resolve)
1. Se `GetTrackCount("subtitle") == 0` → `tl.AddTrack("subtitle")` (senza traccia la generazione fallisce in silenzio).
2. Disattiva tutte le tracce audio tranne la voce (musica, SFX, canzoni), genera, riattivale nello stesso script (try/finally) allo stato originale.
3. `tl.CreateSubtitlesFromAudio({resolve.SUBTITLE_LANGUAGE: resolve.AUTO_CAPTION_ITALIAN, resolve.SUBTITLE_CAPTION_PRESET: resolve.AUTO_CAPTION_SUBTITLE_DEFAULT, resolve.SUBTITLE_CHARS_PER_LINE: MAXC + 4, resolve.SUBTITLE_LINE_BREAK: resolve.AUTO_CAPTION_LINE_SINGLE, resolve.SUBTITLE_GAP: 0})`. Rispetta l'In/Out e aggiunge alla traccia esistente.
4. La funzione ritorna prima che i sottotitoli compaiano: in una chiamata successiva fai `time.sleep(6-10)` e rileggi.
5. Range da linea in chat: `tl.SetMarkInOut(in_rel, out_rel)` (frame relativi a inizio timeline), poi `ClearMarkInOut()` alla fine. Se l'utente aveva un suo In/Out, rimettilo come prima a fine lavoro.
6. Se la generazione salta dei tratti o produce righe senza senso: rigenera solo quel tratto con In/Out temporaneo (cancella prima i sub spazzatura). Fallback: `MediaPoolItem.TranscribeAudio()` + `GetTranscription()`, ma spesso è incompleta.
7. Chiamate > 60 s vanno in timeout ma il lavoro prosegue. Se Resolve non risponde (pagina `None`), c'è un dialogo aperto: chiedi all'utente di chiuderlo.

## 4. Correzione del testo
- Segui l'audio, non la lettera dello script: riformulazioni, aggiunte e inversioni vanno bene se il senso regge. Correggi errori di trascrizione veri e grammatica.
- Nomi, marchi, titoli, parole chiave: grafia dello script / del glossario.
- Se dubbio forte: ipotesi più probabile + marker Fucsia.
- Intercalari tipo "e" sospesi possono essere tolti (segnalalo nel marker).
- Testi citati da slide o messaggi: tra virgolette.
- **Canzoni**: mai trascrivere o scrivere testi cantati (nemmeno citati). Lascia il tratto vuoto e metti un marker Giallo "Testo canzone" lungo quanto il verso. Titoli di canzoni sì.

## 5. Suddivisione (1080×1920)
- Una unità di senso per riga, max `MAXC` caratteri (fino a `MAXC` + 4 solo se un nome/titolo non si spezza).
- Spezza dopo la punteggiatura o prima di congiunzioni/preposizioni. Mai riga che finisce con articolo, preposizione, congiunzione o possessivo ("il", "di", "e", "che", "tua"...). Mai spezzare una parola.
- Tempi parole: dentro ogni sub automatico distribuisci la durata in proporzione ai caratteri (len(parola)+1).
- Definisci le righe come lista `(testo_nuovo, n_parole_originali)`. Prima di inserire stampa per ogni riga `testo nuovo <= parole originali` e verifica che la somma degli n = numero parole trascritte. Un errore di conteggio si propaga a cascata.
- Confini:
  - inizio riga = inizio della sua prima parola; fine = inizio della riga successiva (nessuno spazio);
  - se dentro la pausa tra fine ultima parola precedente e inizio riga (±5 frame) cade un taglio, il confine va esattamente sul taglio;
  - altrimenti, se entro ±5 frame da un taglio, aggancia al taglio;
  - primo sub dall'inizio della prima clip con voce, ultimo fino alla fine dell'ultima;
  - restano vuoti solo tratti senza voce (solo musica/grafica) e canzoni.
- Durata minima 8 frame: se una riga è più corta, accorpala alla vicina e rilancia (lo script deve fermarsi senza inserire se trova righe corte).
- Attraversamenti di taglio a metà frase o su clip < ~1 s sono accettabili: elencali.

## 6. Inserimento (il testo dei sub non è modificabile via API)
- **Ricostruisci sempre l'intera traccia con un unico SRT**: sub attuali da tenere (con i testi correnti) + tratto nuovo. Un SRT parziale accodato finisce in coda alla timeline, non al suo posto.
- Tempi SRT relativi a `GetStartFrame()` (frame - start) / fps.
- Nella stessa chiamata `run_script_unsafe`: scrivi SRT in tempfile, `DeleteClips(subs)`, `SetTrackEnable("subtitle",1,True)`, `SetCurrentTimecode(tl.GetStartTimecode())`, `MediaPool.ImportMedia([srt])`, `MediaPool.AppendToTimeline([item])` passando **solo il MediaPoolItem** (mai dict con mediaType: crasha).
- Rileggi: numero sub, tempi e testi identici ai previsti, nessuno spazio, `GetEndFrame()` invariato. `AppendToTimeline` può dire successo anche se non ha inserito o ha inserito altrove.
- L'append cancella l'In/Out: ripristinalo se era dell'utente.
- Salva (`SaveProject`). Elimina dal Media Pool gli SRT intermedi (non quello della traccia finale finché non è verificata). Avvisa che lo stile personalizzato di singoli sub va riapplicato se la traccia è stata ricostruita.

## 7. Marker
- Fucsia "Controlla sub": nota `Trascritto: "...". Ho scritto "..."`, durata pari al sub (o ai sub coinvolti).
- Non toccare i marker dell'utente; se il frame è occupato sposta di +1.
- In modalità parziale aggiorna la nota dei tuoi marker già presenti nel tratto.
- Marker in frame relativi a inizio timeline; converti in TC per il riepilogo.

## 8. Riepilogo finale (breve, italiano)
- Numero sub, range coperto, spazi (dove e perché), righe oltre `MAXC` caratteri, attraversamenti accettati.
- Tabella correzioni principali (trascritto → corretto).
- Marker con timecode e cosa verificare.
- Differenze rilevanti dallo script (frasi improvvisate, parti tagliate, CTA diverso).
- Se si è lavorato senza script: ripeti in cima al riepilogo che il controllo è stato solo sommario (nessun testo di riferimento per nomi, marchi e parole chiave) e che l'utente deve rileggere i sottotitoli, a partire dai marker Fucsia.

## Glossario (opzionale)
I glossari stanno in file a parte, nella stessa cartella di questa skill. Non sono codice della skill: contengono clienti e progetti di chi la usa.
- `glossario.md` è il modello condiviso: struttura vuota con voci segnaposto. Ignora le voci segnaposto; chi vuole può compilarlo.
- `glossario.<nome>` (es. `glossario.fabio`) è il glossario personale di una persona, pensato per restare privato.
- All'inizio (sezione 1) cerca nella cartella della skill tutti i file `glossario.*`. Leggili e usa le sezioni che corrispondono al cliente o al progetto in corso (parole chiave CTA, frasi tipiche, errori ricorrenti, grafia di nomi e brani). Se più file hanno la stessa voce, vale quello personale.
- Se non trovi voci utili, procedi senza e non inventare. Puoi proporre all'utente di compilare `glossario.md` o di creare il proprio `glossario.<nome>`.
- Quando emergono nuove forme (un nome scritto in modo diverso, un errore che si ripete, una nuova parola chiave), proponi di aggiungerle al glossario dell'utente e scrivile solo dopo il suo ok.

## Modello script (adatta L, MAXC, range, SONG)

```python
import tempfile, os
MAXC = 20            # massimo caratteri per sottotitolo, come scelto dall'utente
L = [("Prima riga", 2), ("seconda riga", 3)]  # (testo, n parole originali)
SONG = None          # es. (76, 162) tratto cantato da lasciare vuoto
RANGE = None         # es. (86660, 86748) per modalità parziale, frame assoluti
tl = project.GetCurrentTimeline(); mp = project.GetMediaPool()
fps = float(tl.GetSetting("timelineFrameRate"))
S, E = tl.GetStartFrame(), tl.GetEndFrame()
subs = sorted(tl.GetItemListInTrack("subtitle", 1), key=lambda x: x.GetStart())
zone = [x for x in subs if RANGE is None or RANGE[0] <= x.GetStart() < RANGE[1]]
keep = [(x.GetStart(), x.GetEnd(), x.GetName()) for x in subs if x not in zone]
W, names = [], []
for it in zone:
    ws = it.GetName().split(); s, e = it.GetStart(), it.GetEnd()
    tot = sum(len(w)+1 for w in ws); acc = 0
    for w in ws:
        a = s + (e-s)*acc/tot; acc += len(w)+1
        W.append((a, s + (e-s)*acc/tot)); names.append(w)
align, i = [], 0
for t, n in L:
    align.append(f"{t} <= {' '.join(names[i:i+n])}"); i += n
if sum(n for _, n in L) != len(W):
    result = {"abort": "count", "need": sum(n for _, n in L), "have": len(W), "align": align}
else:
    cuts = [it.GetStart() for it in tl.GetItemListInTrack("video", 1)][1:]
    lo = RANGE[0] if RANGE else S; hi = RANGE[1] if RANGE else E
    st, i = [], 0
    for k, (t, n) in enumerate(L):
        if k == 0: st.append(lo)
        else:
            a, prev_end = W[i][0], W[i-1][1]
            inside = [c for c in cuts if prev_end - 5 <= c <= a + 5]
            st.append(min(inside, key=lambda c: abs(c-a)) if inside else round(a))
        i += n
    spans = []
    for k in range(len(L)):
        a = st[k]; e = st[k+1] if k+1 < len(L) else hi
        if SONG and a < SONG[0] < e: e = SONG[0]
        spans.append((a, e, L[k][0]))
    short = [(t, a, e-a) for a, e, t in spans if e - a < 8]
    if short:
        result = {"abort": "short", "short": short, "align": align}
    else:
        data = sorted(keep + spans)
        def tc(f):
            ms = round((f - S) * 1000 / fps)
            h, ms = divmod(ms, 3600000); m, ms = divmod(ms, 60000); s, ms = divmod(ms, 1000)
            return f"{h:02}:{m:02}:{s:02},{ms:03}"
        srt = "\n".join(f"{k+1}\n{tc(a)} --> {tc(e)}\n{t}\n" for k, (a, e, t) in enumerate(data))
        path = os.path.join(tempfile.gettempdir(), f"{tl.GetName()} sub.srt")
        open(path, "w", encoding="utf-8").write(srt)
        inout = tl.GetMarkInOut()
        tl.DeleteClips(subs, False)
        tl.SetTrackEnable("subtitle", 1, True)
        tl.SetCurrentTimecode(tl.GetStartTimecode())
        imp = mp.ImportMedia([path]); mp.AppendToTimeline([imp[0]])
        new = [(x.GetStart(), x.GetEnd(), x.GetName()) for x in sorted(tl.GetItemListInTrack("subtitle", 1) or [], key=lambda x: x.GetStart())]
        gaps = [(new[k][1], new[k+1][0]) for k in range(len(new)-1) if new[k][1] != new[k+1][0]]
        cross = [(t, c) for a, e, t in new for c in cuts if a < c < e]
        v = inout.get("video", {})
        if "in" in v: tl.SetMarkInOut(v["in"], v["out"])
        result = {"align": align, "match": new == data, "n": len(new), "gaps": gaps, "cross": cross,
                  "end": tl.GetEndFrame(), "long": [t for _, t in [(0, x[2]) for x in data] if len(t) > MAXC],
                  "saved": resolve.GetProjectManager().SaveProject()}
```

## Versioni
Aggiorna `metadata.version` nel blocco iniziale e aggiungi una riga qui a ogni modifica.
- **1.0** — prima versione condivisibile: richiesta di script, progetto, timeline e `MAXC` se non indicati; trascrizione esterna obbligatoria senza script; glossari in file separati (`glossario.md` modello, `glossario.<nome>` personali).
- **1.1** — indipendente dall'agente: i nomi degli strumenti (Resolve, Drive, Notion) sono esempi e non si assume più `ToolSearch`.
