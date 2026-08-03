solo una domanda, vedendo questi file js grandi, sono di stremio un app... ho scompattato gli apk, preso il server.js e moificato... ma si potrebbe fare le modifiche in un file esterno? tipo prisonmike.js e mettere in server.js un richiamo a prisonmike.js cosi che funzionano le modifiche del proxu come un modulo esterno






10:22 PM






Sì, si può fare ed è una soluzione molto più mantenibile.
L’idea migliore sarebbe lasciare nei tre server.js soltanto un piccolo caricatore:
var path = require("path");
var installPrisonMike = require(
  path.join(__dirname, "prisonmike.js")
);

installPrisonMike({
  router: enginefs.router,
  needle: __webpack_require__(926)
});
In windows.js cambierebbe solo il modulo interno di Needle:
needle: __webpack_require__(74)
Poi prisonmike.js conterrebbe tutta la gestione /clone, refresh token e proxy:
module.exports = function installPrisonMike(options) {
  var router = options.router;
  var needle = options.needle;

  router.get("/clone/manifest.m3u8", function (req, res) {
    // ...
  });

  router.get("/clone/index:suffix", function (req, res) {
    // ...
  });

  router.get("/clone/seg:suffix", function (req, res) {
    // ...
  });
};
I vantaggi sarebbero:
modifiche future concentrate in un solo file;
server.js quasi intatto;
stesso prisonmike.js utilizzabile su Android, Android TV e Windows;
aggiornamenti e confronti molto più semplici.
La condizione importante è che prisonmike.js venga collocato nella stessa directory runtime di server.js. Nell’APK non basta aggiungerlo casualmente agli asset: bisogna verificare dove Stremio estrae/esegue realmente server.js e assicurarsi che copi anche il modulo esterno.
Inoltre bisogna rimuovere dal server.js il vecchio blocco /clone, altrimenti si registrerebbero due volte le stesse rotte.
Quindi sì: fattibile e consigliabile. Nei tre file resterebbe soltanto il richiamo, con la piccola differenza 926 per Android e 74 per Windows.
