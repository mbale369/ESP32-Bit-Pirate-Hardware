# ESP32 Bit Pirate Hardware — guide de conception et de réutilisation

> Document de travail ajouté au fork `mbale369/ESP32-Bit-Pirate-Hardware`.
> Objectif : identifier les concepts hardware réutilisables dans de futurs PCB et bancs de test.

## 1. Vue globale

Le dépôt contient le matériel qui accompagne le firmware ESP32 Bit Pirate : carte dock, adaptateurs MCU/périphériques, sources KiCad, fichiers de fabrication, BOM et modèles 3D.

Architecture physique :

```text
ESP32-S3 DevKit
      │
      ▼
   Dock PCB
      │
      ├── adaptation de niveaux
      ├── sélection 1.8 / 3.3 / 5 V
      ├── accès aux GPIO
      └── connectique vers périphériques
```

## 2. Parties à récupérer en priorité

| Partie | Dossier | Intérêt |
|---|---|---|
| Dock principal | `dock/` | Très élevé |
| Schéma KiCad | `dock/kicad/` | Très élevé |
| Gerbers | `dock/gerbers/` | Fabrication directe, après adaptation |
| BOM | `dock/BOM.csv` | Très élevé pour étude de coût |
| Modèles 3D | `dock/3d_models/` | Élevé |
| Adaptateurs MCU | `MCU_adapters/` | Élevé |
| Adaptateurs périphériques | `device_adapters/` | Très élevé |
| Documentation | `README.md`, `images/` | Élevé |

## 3. Concept #1 — translation de niveaux

Le dock adapte les niveaux entre l'ESP32-S3 et les périphériques et permet de sélectionner 1,8 V, 3,3 V ou 5 V.

C'est probablement la brique hardware la plus intéressante à récupérer.

### Pourquoi ?

Dans un banc de test universel, les circuits testés n'utilisent pas tous la même tension logique.

```text
ESP32 3.3 V
     │
Level shifter
     │
     ├── 1.8 V device
     ├── 3.3 V device
     └── 5 V device
```

### Applications

- capteurs ;
- mémoires ;
- RFID ;
- GPS ;
- modules industriels ;
- cartes électroniques en test ;
- anciens circuits 5 V.

**Attention :** une translation logique n'est pas automatiquement une alimentation 5 V. Il faut distinguer tension d'alimentation et niveau logique.

## 4. Concept #2 — carte dock universelle

Le dock évite de câbler manuellement le microcontrôleur à chaque test.

Le principe peut être repris pour créer ton propre **Embedded Lab Dock** :

```text
                 ┌── UART
                 ├── SPI
ESP32-S3 ─ Dock ─┼── I2C
                 ├── CAN
                 ├── GPIO
                 └── autres interfaces
```

Pour FIKA, cela pourrait devenir une station où un PCB est branché et testé avant installation sur un bus.

## 5. Concept #3 — adaptateurs interchangeables

Les dossiers `MCU_adapters/` et `device_adapters/` montrent une approche modulaire.

Au lieu de concevoir une énorme carte universelle, créer :

```text
Base Dock
   │
   ├── RFID adapter
   ├── LoRa adapter
   ├── GPS adapter
   ├── CAN adapter
   ├── RS485 adapter
   └── custom FIKA adapter
```

C'est beaucoup plus facile à faire évoluer.

## 6. Concept #4 — KiCad comme source de vérité

Il faut privilégier les fichiers KiCad (`kicad/`) comme source de conception et les Gerbers comme sorties de fabrication.

Workflow recommandé :

```text
Schéma
  ↓
Assignation footprints
  ↓
PCB layout
  ↓
ERC / DRC
  ↓
BOM
  ↓
Gerbers
  ↓
Fabrication
  ↓
Assemblage
  ↓
Test
```

C'est exactement le type de chaîne que tu peux reprendre pour tes propres PCB.

## 7. Concept #5 — BOM exploitable

Le `BOM.csv` doit servir à :
- identifier chaque composant ;
- comparer les prix ;
- préparer l'approvisionnement ;
- trouver des équivalents ;
- vérifier les composants critiques avant production.

Pour une future production, ajouter idéalement :
- fabricant ;
- MPN ;
- fournisseur ;
- quantité par PCB ;
- prix unitaire ;
- alternative validée ;
- statut d'approvisionnement.

## 8. Concept #6 — boîtier intégré au design

Les modèles 3D du dépôt montrent que le PCB est pensé avec son enclosure.

À reprendre dans tes projets :

```text
PCB dimensions
      ↓
Connector placement
      ↓
Mechanical constraints
      ↓
Enclosure
      ↓
Final assembly
```

Cela évite de finir avec un PCB fonctionnel mais impossible à intégrer proprement.

## 9. Application à FIKA

Une adaptation très intéressante serait un **FIKA Production Test Dock**.

Exemple :

```text
              FIKA TEST DOCK
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
      RFID         GPS          LoRa
       │            │            │
       └──────── ESP32-S3 ───────┘
                    │
                 Test CLI
                    │
                 Test report
```

Le banc pourrait automatiquement :
- vérifier les alimentations ;
- détecter les périphériques I²C ;
- tester UART ;
- vérifier le lecteur RFID ;
- tester le LoRa ;
- lire le GPS ;
- enregistrer les résultats ;
- produire un rapport de validation du PCB.

## 10. Application à ton laboratoire embarqué

Créer une base PCB avec :

- ESP32-S3 ;
- USB ;
- alimentation protégée ;
- mesure tension/courant ;
- sélection de niveaux logiques ;
- UART multiples ;
- I²C ;
- SPI ;
- CAN ;
- RS485 ;
- connecteurs modulaires ;
- emplacement pour modules radio.

Le firmware Bit Pirate peut servir de référence logicielle tandis que le dock sert de référence mécanique/électrique.

## 11. Ce qu'il ne faut pas copier aveuglément

Avant de réutiliser le PCB :

1. vérifier le schéma ;
2. comprendre le circuit de translation ;
3. vérifier les courants admissibles ;
4. vérifier les protections ESD ;
5. vérifier les tensions d'alimentation ;
6. vérifier les contraintes USB/GPIO ;
7. refaire ERC/DRC avec ta propre configuration ;
8. adapter les connecteurs à tes besoins.

Les Gerbers sont des fichiers de fabrication, pas une source idéale pour modifier le design.

## 12. Licence

Le dépôt matériel indique une licence **CERN-OHL-W-2.0** pour les fichiers de conception hardware et **CC BY 4.0** pour la documentation/images sauf indication contraire. Avant de fabriquer ou redistribuer une version modifiée, conserver les notices requises et vérifier les obligations exactes de la licence.

## 13. Les concepts à conserver dans ta boîte à outils

### Hardware

- level shifting multi-tension ;
- dock universel ;
- adaptateurs interchangeables ;
- BOM structurée ;
- fabrication à partir de Gerbers ;
- conception mécanique liée au PCB ;
- architecture modulaire.

### Méthodologie

```text
Fonction
  ↓
Module
  ↓
Interface standardisée
  ↓
Adaptateur
  ↓
PCB principal
```

Cette approche est particulièrement adaptée à tes futurs projets ESP32 et aux bancs de test FIKA.
