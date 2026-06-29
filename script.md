# Récupération des données OSM — Restos & Hôtels à Antananarivo

## 1. Requête Overpass API

Interroger OpenStreetMap pour obtenir les restaurants et hôtels dans la bounding box d'Antananarivo :

```
[out:json][timeout:60];
(
  node["amenity"="restaurant"](-19.07,47.36,-18.75,47.68);
  node["tourism"="hotel"](-19.07,47.36,-18.75,47.68);
);
out center;
```

URL encodée :

```bash
wget -O tana_raw.json 'https://overpass-api.de/api/interpreter?data=%5Bout%3Ajson%5D%5Btimeout%3A60%5D%3B%28node%5B%22amenity%22%3D%22restaurant%22%5D%28-19.07%2C47.36%2C-18.75%2C47.68%29%3Bnode%5B%22tourism%22%3D%22hotel%22%5D%28-19.07%2C47.36%2C-18.75%2C47.68%29%3B%29%3Bout%20center%3B'
```

## 2. Génération du CSV

Script Python utilisé :

```python
import json, random, csv

with open('tana_raw.json') as f:
    d = json.load(f)

random.seed(42)
seen = set()
rows = []

for e in d['elements']:
    tags = e.get('tags', {})
    name = tags.get('name', '')
    if not name:
        continue
    lat = round(e['lat'], 5)
    lon = round(e['lon'], 5)
    key = (name, lat, lon)
    if key in seen:
        continue
    seen.add(key)

    is_hotel = tags.get('tourism') == 'hotel'
    is_resto = tags.get('amenity') == 'restaurant'
    if not is_hotel and not is_resto:
        continue

    typ = 'hotel' if is_hotel else 'restaurant'
    note = round(random.uniform(1.0, 5.0), 1)
    rows.append([name, typ, note, lon, lat])

rows.sort(key=lambda r: (0 if r[1]=='restaurant' else 1, r[0].lower()))

with open('etablissements.csv', 'w', newline='') as f:
    w = csv.writer(f)
    w.writerow(['nom', 'type', 'note', 'x', 'y'])
    w.writerows(rows)
```

## 3. Résultat

- **546 établissements** : 386 restaurants, 160 hôtels
- Colonnes : `nom`, `type`, `note` (aléatoire 1.0–5.0), `x` (longitude), `y` (latitude)
- Projection : WGS 84 (EPSG:4326)
- Source : OpenStreetMap (© contributeurs OSM, licence ODbL)
