# Riassunto sessione - StremioServer patch

**Data salvataggio:** 2026-06-17 circa 00:35
**Stato:** modifiche applicate, test parziali OK (NetMirror + Vidxgo con audio), in attesa di conferma StreamingCommunity e CinemaCity.

## Problemi affrontati

1. **NetMirror** - audio "disabilitato" / senza ITA
2. **StreamingCommunity** - non partiva con Exo, proxy engineFS in 500
3. **Vidxgo** - audio non si sentiva, immagine stretchata su Android TV
4. **CinemaCity** - stream crasha l'app su Android TV

## Modifiche applicate

File modificati:
- `C:\Users\emanu\Downloads\stremioserver\server.js`
- `C:\Users\emanu\Downloads\stremioserver\tv.js`
- `C:\Users\emanu\Downloads\stremioserver\mobile.js`

Backup creati automaticamente:
- `server.js.bak_20260617_003452`
- `tv.js.bak_20260617_003452`
- `mobile.js.bak_20260617_003452`

### 1. NetMirror
- Nella mappa degli stream plugin, se l'URL contiene `tv.imgcdn.kim` o `s.provider === 'netmirror'` e non c'e' lingua italiana (`!s.language`), lo stream viene scartato.
- NetMirror viene fatto passare da `/plugin/m3u8-proxy` per evitare il bug del proxy engineFS che appendeva `null` alle URI audio/sottotitoli.

### 2. StreamingCommunity
- Nella stessa mappa, StreamingCommunity (`streamingcommunity` o `vixsrc.to`) viene fatto passare da `/plugin/m3u8-proxy` con URL e headers codificati, evitando il 500 del proxy engineFS sugli URL con query string.

### 3. Vidxgo /clone (semplificato)
- `rewriteVidxgoMaster` semplificato: pass-through del master originale, riscrivendo SOLO URI delle varianti index in `/clone/index...`.
- Rimosso: selezione "best" variant, forzatura `DEFAULT=YES/AUTOSELECT=YES` su audio ITA.
- Master originale preservato (tag, resolution, codecs restano originali).
- Riscrittura segmenti in `/clone/seg...` mantenuta per refresh token automatico.

### 4. /plugin/m3u8-proxy robusto
- Riconosciuti come segmenti anche i file `.js` e `.jpg` dentro `/files/` o `/hls/` (NetMirror).
- Forzato `Content-Type: video/mp2t` per questi segmenti, in modo che Exo li accetti.

### 5. CinemaCity
- CinemaCity (`cinemacity` nell'URL o `s.provider === 'cinemacity'`) viene fatto passare da `/plugin/m3u8-proxy` con URL e headers codificati, bypassando il proxy engineFS che crasha l'app su Android TV.

### 6. Fix behaviorHints
- Quando URL punta a `/plugin/m3u8-proxy`, NON impostare `behaviorHints.proxyHeaders`. Header già codificati nell'URL, doppio-proxy causava conflitti.

## Test effettuati

- Link master NetMirror fornito dall'utente: confermato problema `null` nel proxy engineFS.
- Dopo patch (riavvio server da parte dell'utente): **NetMirror parte con audio ITA**, **Vidxgo parte con audio**.
- StreamingCommunity: non testato direttamente dall'utente dopo la patch; il server di test locale era in ascolto su 11471 ma e' stato terminato.

## Domande aperte per proseguire

1. **StreamingCommunity** adesso parte con Exo?
2. **CinemaCity** adesso non crasha più l'app su Android TV?
3. **Vidxgo** su Android TV: audio funziona? Stretch risolto?

## Come ripristinare i backup

Se qualcosa non va, sostituire il file modificato con il corrispondente `.bak_20260617_003452`.
