<div align="center">

# CartoSnap

*(anciennement LambertSnap)*

<img src="assets/icon/app_icon.png" width="80" alt="CartoSnap icon"/>

**Appareil photo de terrain qui nomme, annote et géoréférence chaque cliché en Lambert 93, puis l'exporte prêt à ouvrir dans QGIS**

[![Flutter](https://img.shields.io/badge/Flutter-Dart%20%5E3.9.2-02569B?logo=flutter&logoColor=white)](https://flutter.dev)
[![Version](https://img.shields.io/badge/version-0.95-blue)](pubspec.yaml)
[![Platform](https://img.shields.io/badge/plateforme-Android-3DDC84?logo=android&logoColor=white)](android/)
[![CRS](https://img.shields.io/badge/CRS-EPSG%3A2154%20Lambert%2093-orange)](lib/services/coordinate_converter.dart)
[![Docs](https://img.shields.io/badge/docs-FR%20%7C%20EN%20%7C%20ES%20%7C%20PT%20%7C%20DE-lightgrey)](#français)

</div>

---

## Français

### Description

CartoSnap est une application mobile Flutter qui prend des photos géolocalisées et calcule leurs coordonnées en **Lambert 93 (EPSG:2154)** directement sur le téléphone, sans service en ligne ni bibliothèque de projection externe. Chaque photo reçoit un nom normalisé, un bandeau de coordonnées incrusté, et peut ensuite être exportée en CSV, GeoJSON ou projet QGIS.

Fini les photos de chantier à relocaliser à la main : visez, déclenchez, et le fichier porte déjà dans son nom la date et la position X/Y au mètre près. De retour au bureau, un ZIP « QGIS » s'ouvre directement sur un fond OpenStreetMap, avec un point par photo.

### Fonctionnalités

- **Conversion Lambert 93 embarquée** : projection conique conforme RGF93 / GRS80 (algorithmes IGN), aller et retour WGS84 ↔ Lambert 93, écart nul au millimètre près avec PROJ.
- **Nommage automatique** : `AAAAMMJJHHMMSS_X652469_Y6862035[_prefixe].jpg`, avec un préfixe de chantier facultatif (espaces et accents nettoyés).
- **Bandeau GPS incrusté** : adresse, `X … Y … | EPSG:2154`, altitude et précision GPS, ajoutés automatiquement quand l'annotation est désactivée.
- **Viseur informatif** : position en Lambert 93 ou en WGS84 décimal, altitude, précision (vert ≤ 10 m, orange ≤ 30 m, rouge au-delà), heure, mini-carte OSM.
- **Boussole** : cap de prise de vue affiché dans le viseur et enregistré dans l'EXIF (`GPSImgDirection`, nord magnétique sous Android, nord vrai sous iOS). Sans magnétomètre, le cap GPS est utilisé.
- **Choix d'objectif** : sélecteur 0.5x / 1x / téléobjectif quand le téléphone expose plusieurs caméras arrière, en plus du zoom par pincement et de la bascule avant/arrière.
- **Annotation après capture** : stylo (12 couleurs, 6 épaisseurs, annuler), commentaire ajouté sous la photo, canevas GPS sur la photo ou bandeau GPS sous la photo.
- **Géocodage inverse** : adresse via Nominatim (OSM), limitée à une requête par seconde et déclenchée après 3 s de position stable.
- **Galerie** : grille ou liste, sélection multiple, partage, suppression, vue plein écran avec coordonnées Lambert 93.
- **Carte des photos** : vignettes regroupées par proximité, échelle affichée et forçable (1/250 à 1/25 000), export JPEG au format A4 (portrait ou paysage, 100 dpi) avec titre et commentaire.
- **Exports** : CSV (`;`, UTF-8 avec BOM pour Excel), ZIP photos + CSV, ZIP QGIS (GeoJSON Lambert 93 + projet `.qgz` et `.qgs` + `photos/`), ZIP complet.
- **Projet QGIS prêt à l'emploi** : chaque photo affichée **en vignette** sur la carte (point bleu = position exacte), dispersion des photos superposées, étiquette = nom du fichier, info-bulle, action « Ouvrir la photo », chemins relatifs, fond de carte du réglage. Vérifié en chargeant le projet généré dans QGIS 3.44.
- **Fond de carte** : OpenStreetMap ou OpenStreetMap France (réglage), appliqué à la mini-carte, à la carte des photos, à l'export JPEG A4 et au projet QGIS.
- **Réparation des anciennes photos** (Galerie → ⋮) : reconversion en JPEG des PNG enregistrés en `.jpg`, EXIF GPS réécrit, noms de fichiers renommés avec les X/Y corrigés, position factice de Paris effacée.
- **EXIF GPS** : latitude, longitude, altitude, cap, date et coordonnées Lambert 93 (description) écrits dans chaque photo géolocalisée, y compris après bandeau ou annotation (réencodage JPEG qualité 92).
- **Pas de position inventée** : si le GPS est coupé ou refusé, la photo est enregistrée sans coordonnées (`CartoSnap_<date>.jpg`), le témoin passe au rouge, et les exports laissent ses colonnes X/Y vides (géométrie nulle dans le GeoJSON). Le suivi reprend seul quand le GPS est réactivé.
- **Dossier de sauvegarde** : `DCIM/CartoSnap` par défaut, ou un dossier choisi par l'utilisateur.

### Prérequis

| Élément | Version |
|---|---|
| Flutter SDK | compatible Dart `^3.9.2` |
| Android | téléphone avec GPS et caméra |
| Réseau | facultatif : nécessaire pour l'adresse (Nominatim) et les fonds de carte OSM |

> **Permissions Android** déclarées : caméra, localisation précise et approximative, stockage, Internet.
> Sans localisation, les photos restent possibles mais sans coordonnées. Une position de démonstration (Paris) n'est utilisée que dans l'aperçu web.

### Installation

```bash
git clone https://github.com/Cartoyoyo/CartoSnap.git
cd CartoSnap
flutter pub get
flutter run            # téléphone branché en mode développeur
flutter build apk      # APK de release dans build/app/outputs/flutter-apk/
```

Lancer les tests (conversion Lambert 93, encodage JPEG, EXIF, exports, réparation, écran de démarrage) :

```bash
flutter test
```

### Utilisation

1. **Ouvrir l'application** : l'écran de démarrage laisse place au viseur, la position GPS s'affiche dès le premier fix.
2. **Régler** (icône engrenage) : système de coordonnées, préfixe de nom, dossier de sauvegarde, options d'annotation.
3. **Photographier** : le bouton central enregistre la photo. Le pincement zoome, le double-tap bascule entre 1x et 2x, les pastilles au-dessus du bouton changent d'objectif, le bouton de droite bascule caméra arrière/frontale. La photo apparaît aussitôt dans la galerie Android.
4. **Annoter** (bouton « Annoter » du viseur) : après chaque capture, dessinez, commentez, puis « Enregistrer ».
5. **Exporter** : Galerie → menu ⋮ → *Export CSV / ZIP*, ou depuis la carte des photos. Le fichier est ensuite proposé au partage Android.
6. **Ouvrir dans QGIS** : décompressez le ZIP QGIS et ouvrez le fichier `.qgz` (ou `.qgs`).
7. **Anciennes photos** : une fois après la mise à jour, Galerie → ⋮ → *Réparer les anciennes photos*.

### Contenu des exports

| Colonne | Contenu |
|---|---|
| `Nom fichier` | nom de la photo |
| `Date`, `Heure` | horodatage de la capture |
| `X_L93`, `Y_L93` | coordonnées Lambert 93 en mètres, 2 décimales |
| `Systeme` | `EPSG:2154` |
| `Altitude` | altitude GPS (m) |
| `Adresse`, `Code postal`, `Ville`, `Pays` | géocodage Nominatim |
| `Google Maps` | lien vers la position |
| `photo_path` | GeoJSON seulement : `photos/<nom>` |

Les coordonnées des exports sont recalculées à partir de la latitude et de la longitude stockées, au moment de l'export.

### Précision de la conversion Lambert 93

La conversion suit les algorithmes IGN (ALG0054 pour les constantes, ALG0001/0003/0004 pour la projection). Contrôle sur 8 villes de métropole et de Corse (Brest, Bastia, Bonifacio, Dunkerque…) : écart inférieur au millimètre par rapport à PROJ, aller-retour inférieur à 0,01 mm.

Le GPS fournit des coordonnées WGS84, traitées ici comme RGF93. L'écart entre les deux référentiels est inférieur au mètre (dérive d'environ 2,5 cm par an depuis 1989), bien en dessous de la précision d'un GPS de téléphone (3 à 10 m).

> **Correctif du 04/10/2026.** Les versions précédentes calculaient mal l'exposant `n` de la projection (0,7302 au lieu de 0,7256). L'erreur était nulle à l'origine de la projection (46,5° N, 3° E, près de Moulins) et grandissait avec la distance : environ 14 m à Vichy, 113 m à Paris, 245 m à Lille et 300 m à Brest.
> Pour les photos prises avant ce correctif, *Réparer les anciennes photos* corrige le nom de fichier et l'EXIF. **Le bandeau incrusté dans l'image garde les anciennes coordonnées.** Les exports CSV, GeoJSON et QGIS sont justes dès le prochain export, car ils repartent de la latitude et de la longitude.

### Limitations connues

| Sujet | État actuel du code |
|---|---|
| Bandeau des photos antérieures à 0.94 | Les coordonnées incrustées dans l'image ne peuvent pas être recalculées. La réparation corrige le nom, l'EXIF et le format, pas les pixels. Les photos prises GPS coupé gardent le bandeau de Paris. |
| Objectifs sous Android | CameraX ne précise pas le type des caméras : l'étiquette 0.5x / téléobjectif est déduite de l'ordre des caméras et peut être fausse. Beaucoup de téléphones n'exposent qu'une caméra arrière, et le sélecteur est alors masqué. |

---

## English

### Description

CartoSnap is a Flutter field camera app that geotags photos and computes their **Lambert 93 (EPSG:2154)** coordinates on the device, with no online service and no external projection library. Each photo gets a standardised file name and a burned-in coordinate banner, and can be exported to CSV, GeoJSON or a ready-to-open QGIS project.

### Features

- **Built-in Lambert 93 conversion**: RGF93 / GRS80 conformal conic projection (IGN algorithms), WGS84 ↔ Lambert 93 both ways, sub-millimetre agreement with PROJ.
- **Automatic naming**: `YYYYMMDDHHMMSS_X652469_Y6862035[_prefix].jpg`.
- **GPS banner**: address, `X … Y … | EPSG:2154`, altitude and accuracy burned into the photo.
- **Annotation**: pen, comment below the photo, GPS canvas or GPS banner.
- **Reverse geocoding** through Nominatim (OSM), 1 request per second.
- **Gallery and photo map**: clustering, forced scale, A4 JPEG map export at 100 dpi.
- **Compass heading** stored in EXIF (magnetic north on Android, true north on iOS) and **lens selector** (0.5x / 1x / tele) when several rear cameras are exposed.
- **Exports**: CSV (`;`, UTF-8 BOM), ZIP photos + CSV, QGIS ZIP (GeoJSON + `.qgz`/`.qgs` + `photos/`, photos shown as thumbnails), full ZIP.
- **Basemap setting**: OpenStreetMap or OpenStreetMap France, used by every map and the QGIS project.
- **Old photo repair** (Gallery → ⋮): PNG-in-`.jpg` re-encoded, GPS EXIF rewritten, file names fixed, Paris demo positions cleared.
- **GPS EXIF** in every geotagged photo, banner and annotation included (JPEG re-encoding, quality 92).
- **No made-up position**: with GPS off or denied, photos are saved without coordinates (empty X/Y in exports, null geometry in GeoJSON); tracking resumes when GPS is turned back on.

### Installation

```bash
git clone https://github.com/Cartoyoyo/CartoSnap.git
cd CartoSnap
flutter pub get
flutter run
flutter test
```

### Accuracy and fix of 2026-10-04

Earlier versions computed the projection exponent `n` incorrectly (0.7302 instead of 0.7256), causing errors from zero at the projection origin (near Moulins) up to about 300 m in Brittany. Photos taken before the fix keep wrong coordinates in their file name, banner and EXIF description. CSV, GeoJSON and QGIS exports are correct on the next export.

### Known limitations

Burned-in banners of photos taken before 0.94 keep their old coordinates (the repair tool fixes names, EXIF and format, not pixels). Android lens labels are guessed. See the French table above for details.

---

## Español

### Descripción

CartoSnap es una aplicación Flutter de cámara de campo que georreferencia cada foto y calcula sus coordenadas **Lambert 93 (EPSG:2154)** en el propio teléfono, con nombre de archivo normalizado, banda de coordenadas y exportación a CSV, GeoJSON y proyecto QGIS.

### Funcionalidades

- Conversión WGS84 ↔ Lambert 93 integrada (algoritmos IGN)
- Nombre de archivo con fecha y coordenadas X/Y
- Banda GPS, anotación con lápiz y comentario
- Galería, mapa de fotos y exportación JPEG A4
- Exportación CSV, GeoJSON y proyecto QGIS `.qgs`

### Instalación

```bash
git clone https://github.com/Cartoyoyo/CartoSnap.git
cd CartoSnap && flutter pub get && flutter run
```

---

## Português

### Descrição

CartoSnap é uma aplicação Flutter de câmara de campo que georreferencia cada fotografia e calcula as suas coordenadas **Lambert 93 (EPSG:2154)** no próprio telemóvel, com nome de ficheiro normalizado, faixa de coordenadas e exportação para CSV, GeoJSON e projeto QGIS.

### Funcionalidades

- Conversão WGS84 ↔ Lambert 93 integrada (algoritmos IGN)
- Nome de ficheiro com data e coordenadas X/Y
- Faixa GPS, anotação com caneta e comentário
- Galeria, mapa de fotografias e exportação JPEG A4
- Exportação CSV, GeoJSON e projeto QGIS `.qgs`

### Instalação

```bash
git clone https://github.com/Cartoyoyo/CartoSnap.git
cd CartoSnap && flutter pub get && flutter run
```

---

## Deutsch

### Beschreibung

CartoSnap ist eine Flutter-Feldkamera-App, die jedes Foto georeferenziert und seine **Lambert-93-Koordinaten (EPSG:2154)** direkt auf dem Gerät berechnet – mit normiertem Dateinamen, eingeblendetem Koordinatenband und Export nach CSV, GeoJSON und als QGIS-Projekt.

### Funktionen

- Integrierte Umrechnung WGS84 ↔ Lambert 93 (IGN-Algorithmen)
- Dateiname mit Datum und X/Y-Koordinaten
- GPS-Band, Annotation mit Stift und Kommentar
- Galerie, Fotokarte und A4-JPEG-Export
- Export als CSV, GeoJSON und QGIS-Projekt `.qgs`

### Installation

```bash
git clone https://github.com/Cartoyoyo/CartoSnap.git
cd CartoSnap && flutter pub get && flutter run
```

---

## Architecture

L'architecture complète (écrans, providers, services, caméra multi-objectifs, boussole, stockage, galerie Android et services externes) est documentée dans [`architecture.d2`](architecture.d2).

```bash
d2 --layout elk architecture.d2 architecture.svg
```

---

## Changelog

|---------|-------|
| **0.95 — 04/10/2026 (branche `web`)** | Renommage LambertSnap → CartoSnap (nom affiché, EXIF, exports, dossier `DCIM/CartoSnap` ; la réparation des anciennes photos LambertSnap est conservée) — version web utilisable sur iPhone via Safari : photos stockées dans le navigateur (IndexedDB), partage iOS, export ZIP/CSV/QGIS, installation sur l'écran d'accueil |
| **0.94 — 04/10/2026** | Correction de l'exposant `n` de la projection Lambert 93 (erreur jusqu'à ~300 m) — photos annotées ou avec bandeau enregistrées en vrai JPEG avec EXIF GPS — suppression de la position factice de Paris quand le GPS est coupé, exports sans coordonnées pour ces photos, reprise automatique du suivi — chemin `assets/icon/` corrigé dans `pubspec.yaml` — tests unitaires (conversion, JPEG, EXIF, exports sans GPS) — README réécrit — diagramme `architecture.d2` — boussole branchée (cap EXIF) — sélecteur d'objectif — canal MediaScanner Android — suffixe Portrait/Paysage de l'export carte — fuite du dialogue d'échelle corrigée — version unique `appVersion` — réglage « Horodatage » appliqué au bandeau et au canevas — fond de carte OSM / OSM France branché partout — projet QGIS en `.qgz` avec photos en vignettes — réparation des anciennes photos — GPSTimeStamp EXIF en UTC — test de l'écran de démarrage corrigé |
| **0.93** | Version affichée dans l'application au moment de l'analyse |

---

## Auteur · Author

<div align="center">

Développé par / Developed by **Yoan Laloux**

Technicien SIG — Vichy Communauté · GIS Technician — Vichy Communauté

[![LinkedIn](https://img.shields.io/badge/LinkedIn-ylaloux-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/ylaloux/)
[![GitHub](https://img.shields.io/badge/GitHub-Cartoyoyo-black?logo=github)](https://github.com/Cartoyoyo)

*Concept et idée originale par Yoan Laloux — développé avec l'assistance d'outils d'IA générative.*
*Concept and original idea by Yoan Laloux — developed with the assistance of generative AI tools.*

</div>

---

## Licence · License

Aucun fichier de licence n'est présent dans le dépôt : la licence reste à définir.
No license file is included in the repository yet.

Signaler un problème · Report an issue : [github.com/Cartoyoyo/CartoSnap/issues](https://github.com/Cartoyoyo/CartoSnap/issues)

Données cartographiques © contributeurs OpenStreetMap (ODbL). Géocodage : Nominatim.
