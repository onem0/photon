# Photon

Suche für OpenStreetMap-Daten mit [Photon](https://github.com/komoot/photon): Freitext mit Tippfehler-Toleranz, Autovervollständigung und Ranking, dazu Reverse-Geocoding (Koordinate → Adresse).

Läuft als eigener Dienst neben dem [Overpass-Server](https://github.com/onem0/Overpass-API). Overpass kann seine Datenbank nicht mit Photon teilen (eigenes Binärformat, und ohne `META` keine Zeitstempel für Updates). Deshalb lädt Photon einen fertigen Planet-Index herunter (Image [`rtuszik/photon-docker`](https://github.com/rtuszik/photon-docker), `IMPORT_MODE=db`).

## Deploy

- Domain auf Port **2322** zeigen lassen.
- `PHOTON_API_KEY` setzen. Requests ohne Header `X-Photon-Key: <key>` bekommen 403.
- Platz auf `/srv/photon`:
  - Erste Installation: ca. 160 GB frei (Download ca. 61 GB, entpackt ca. 100 GB).
  - Update alle 30 Tage mit `UPDATE_STRATEGY=PARALLEL`: ca. 260 GB, die Suche läuft weiter.
  - Mit `SEQUENTIAL` reichen ca. 160 GB, die Suche ist dann aber während des Updates mehrere Stunden weg.
- Erster Download und Entpacken dauern mehrere Stunden. Bis dahin steht der Container auf `starting`.

Der Index kommt standardmäßig von einem Community-Mirror des Image-Autors. Offizielle Quelle: `BASE_URL=https://download1.graphhopper.com/public`.

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
