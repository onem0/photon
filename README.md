# Photon

Suche für OpenStreetMap-Daten mit [Photon](https://github.com/komoot/photon): Freitext mit Tippfehler-Toleranz, Autovervollständigung und Ranking, dazu Reverse-Geocoding (Koordinate → Adresse).

Läuft als eigener Dienst neben dem [Overpass-Server](https://github.com/onem0/Overpass-API). Overpass kann seine Datenbank nicht mit Photon teilen (eigenes Binärformat, und ohne `META` keine Zeitstempel für Updates). Deshalb lädt Photon einen fertigen Planet-Index herunter (Image [`rtuszik/photon-docker`](https://github.com/rtuszik/photon-docker), `IMPORT_MODE=db`).

## Deploy

- Domain auf Port **2322** zeigen lassen.
- `PHOTON_API_KEY` setzen. Requests ohne Header `X-Photon-Key: <key>` bekommen 403.
- In Coolify unter *Advanced* „Escape special characters in labels“ **ausschalten**. Sonst landet `${PHOTON_API_KEY}` wörtlich im Caddy-Label, Caddy macht daraus `$`, und jeder kommt mit `X-Photon-Key: $` durch.
- Platz auf `/srv/photon`:
  - Erste Installation: ca. 160 GB frei (Download ca. 61 GB, entpackt ca. 100 GB).
  - Update alle 30 Tage mit `UPDATE_STRATEGY=PARALLEL`: ca. 260 GB, die Suche läuft weiter.
  - Mit `SEQUENTIAL` reichen ca. 160 GB, die Suche ist dann aber während des Updates mehrere Stunden weg.
- Erster Download und Entpacken: mit 200 Mbit/s gut eine Stunde. Fortschritt in den Logs von `photon-download`. Bis dahin steht `photon` auf `starting`.

Den ersten Index lädt der Dienst `photon-download` mit aria2c, von `r2.koalasec.org` und GraphHopper zugleich. Er setzt nach Abbrüchen und Neustarts fort, prüft die MD5 und entpackt nach `/srv/photon/data/photon_data`. `photon` wartet so lange und startet dann ohne eigenen Download. Monatliche Updates lädt das Image selbst, mit einer Verbindung von GraphHopper. Dabei kommen Archiv und MD5 von derselben Quelle, deshalb die offizielle statt des Mirrors.

## API

```sh
# Suche
curl -H "X-Photon-Key: $KEY" "https://<domain>/api?q=Bäckerei%20Koblenz&lang=de&limit=10"

# Ergebnisse nahe an einem Punkt zuerst
curl -H "X-Photon-Key: $KEY" "https://<domain>/api?q=Hauptstraße%205&lat=50.36&lon=7.59"

# Nur in einem Kartenausschnitt (minLon,minLat,maxLon,maxLat)
curl -H "X-Photon-Key: $KEY" "https://<domain>/api?q=Pizza&bbox=7.5,50.3,7.7,50.4"

# Koordinate → Adresse
curl -H "X-Photon-Key: $KEY" "https://<domain>/reverse?lat=50.36&lon=7.59"
```

Weitere Parameter (`osm_tag`, `layer`, `countrycode`, …): [Photon-API](https://github.com/komoot/photon/blob/master/docs/api-v1.md).

## Zusammen mit Overpass

Jeder Treffer enthält `osm_type` (`N`, `W`, `R`) und `osm_id`. Die volle Geometrie kommt dann von Overpass:

```
[out:json];
way(191582655);
out geom;
```
