# Changelog Modifiche Scraper & Plugin (Stremio Server)

Questo documento riassume in modo compatto le modifiche apportate ai principali componenti di scraping.

---

## 1. Guardoserie (`guardoserie.js` / `android.js` / `androidtv.js`)
* **Fix Zlib Decompression (`unexpected EOF`):** Corretto l'errore di decompressione Zlib durante la risoluzione dei Captcha/OCR. Implementata una validazione robusta dell'header RFC 1950 per identificare correttamente stream GZIP/Deflate raw prima di darli in pasto all'inflatore.
* **Risoluzione Captcha:** Ripristinato il corretto flusso di bypass e risoluzione dei Captcha su dispositivi Android/AndroidTV.

---

## 2. Eurostreaming Scraper (`es.js`)
* **Allineamento Dominio:** Aggiornato il recupero dei domini dinamici dal repository GitHub.
* **Corrispondenza di Ricerca:** Ottimizzazione preliminare della selezione dell'URL dell'episodio.

---

## 3. Eurostreaming Plugin (`plugin_es.js` / `plugin_es_latest.js`)
* **Fix Match Titoli Brevi:** Risolto il bug di corrispondenza troppo larga (substring loose match) che causava ad esempio la riproduzione di *"Agent from Above"* quando veniva cercata la serie *"FROM"*.
* **Algoritmo di Scoring ed Esclusione:**
  * Introdotta normalizzazione (accenti, minuscolo, rimozione anno ed entità) e tokenizzazione.
  * Per query a parola singola (es. `"from"`), vengono scartati tutti i post con parole extra significative.
  * I risultati vengono pesati, ordinati per accuratezza ed esclusi se sotto la soglia di confidenza ($\ge 50$).
