# StreamingCommunity + ExoPlayer su Android TV

## Scoperta

StreamingCommunity (vixsrc.to) su Android TV NON parte con ExoPlayer.
Stremio fa fallback a MPV (player esterno). Con MPV funziona perfettamente.

## Perché?

Tutti gli stream passano dal processore generico in `server.js` (~linea 87200)
e gemelli in `mobile.js` / `tv.js` (~linea 66400).

Il codice costruisce `behaviorHints` così:

```javascript
var behaviorHints = {
    notWebReady: true,             // ← SEMPRE true per TUTTI i plugin
    bingeGroup: "plugin-" + scraper.id
};
if (hasHeaders && finalUrl.indexOf('/plugin/m3u8-proxy') === -1) {
    behaviorHints.proxyHeaders = { request: proxyHeaders };
}
```

Su Android TV, `notWebReady: true` fa 2 cose:
1. Impedisce al web player di prenderlo (OK)
2. **Forza engineFS** (proxy nativo di Stremio) a wrappare l'URL

engineFS wrappa l'URL di TUTTI gli stream plugin:
```
Player → engineFS → /plugin/m3u8-proxy.m3u8?url=CDN_URL&headers=H
```

## Perché NetMirror e CinemaCity funzionano?

Anche loro passano da engineFS + m3u8-proxy, ma funzionano.
Differenza probabile: il formato della **risposta** di `reorderPlaylist`.

EngineFS potrebbe riscrivere gli URL SEGMENT nel playlist ritornato da
reorderPlaylist per streamingcommunity, rompendoli. Per cinemaacity/no
lo stesso non accade (formato CDN diverso).

## Perché Vidxgo funziona?

Vidxgo NON passa da `/plugin/m3u8-proxy.m3u8`. Usa `/clone/manifest.m3u8`
(rotta separata). Il suo URL non matcha `/plugin/m3u8-proxy`, quindi
`notWebReady: true` resta — ma engineFS non fa conflitto col clone route.

## Perché MPV funziona?

MPV ignora engineFS. Usa l'URL direttamente. L'URL va a
`/plugin/m3u8-proxy.m3u8` che risponde correttamente (200 OK,
playlist HLS valida). Il CDN risponde, i segmenti arrivano.

## Fix proposto

Quando l'URL contiene già `/plugin/m3u8-proxy`, togliere `notWebReady`.
Il player nativo (ExoPlayer) usa l'URL direttamente, SENZA engineFS proxy:

```javascript
var behaviorHints = {
    bingeGroup: "plugin-" + scraper.id
};
if (finalUrl.indexOf('/plugin/m3u8-proxy') === -1) {
    behaviorHints.notWebReady = true;
}
```

### Effetto su ogni provider:

| Provider | URL | notWebReady | engineFS | Risultato |
|----------|-----|-------------|----------|-----------|
| NetMirror | `/plugin/m3u8-proxy` | NO (rimosso) | No | Diretto, OK |
| CinemaCity | `/plugin/m3u8-proxy` | NO (rimosso) | No | Diretto, OK |
| StreamingCommunity | `/plugin/m3u8-proxy` | NO (rimosso) | No | Diretto → ExoPlayer? |
| Vidxgo | `/clone/manifest.m3u8` | SÌ (non matcha) | SÌ | Come prima, OK |
| Altri plugin senza proxy | URL normale | SÌ | SÌ | Come prima |

### File da modificare

- `server.js` (~linea 87231)
- `mobile.js` (~linea 66441)
- `tv.js` (~linea 66441)

## Verifica necessaria

Testare su Android TV dopo il fix:
1. StreamingCommunity → parte con ExoPlayer?
2. NetMirror → ancora OK?
3. CinemaCity → ancora OK?
4. Vidxgo → ancora OK?
