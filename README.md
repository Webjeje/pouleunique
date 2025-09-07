
# Compteur EPS — Tournoi en poule unique (Material • Dark)

Application web **100% front-end** pour gérer un **tournoi en poule unique** : génération automatique des matches (round-robin), saisie des scores, **classement en direct**, **sauvegarde locale automatique**, **import/export** (JSON/CSV), et **version Light** sans scores ni classement.

* **Design** : Material “dark” (pas de mode clair, aucune fenêtre blanche), mobile-first, responsive.
* **Technos** : HTML + CSS + JavaScript vanilla (aucune dépendance).
* **Données** : stockage local via `localStorage` (offline), import/export pour transfert entre machines.

---

## Sommaire

1. [Fonctionnalités](#fonctionnalités)
2. [Mode d’emploi (version complète)](#mode-demploi-version-complète)
3. [Mode d’emploi (version Light)](#mode-demploi-version-light)
4. [Formats d’import/export](#formats-dimportexport)
5. [Règles de classement](#règles-de-classement)
6. [Caractéristiques techniques](#caractéristiques-techniques)
7. [Compatibilité & performances](#compatibilité--performances)
8. [Déploiement (GitHub Pages)](#déploiement-github-pages)
9. [Personnalisation](#personnalisation)
10. [Limites & feuille de route](#limites--feuille-de-route)
11. [Licence](#licence)

---

## Fonctionnalités

### Commun (Full & Light)

* **Génération des matches (round-robin)** avec gestion des participants impairs (`BYE`).
* **3 modes de création de liste** (mêmes boutons “Valider” en haut-droite) :

  1. **Liste rapide** (1…N)
  2. **Import CSV** (une ligne par joueur, accents FR gérés)
  3. **Saisie manuelle** (copier/coller, une ligne par joueur)
* **Dark mode** intégral (pas de switch).
* **Mobile-first / responsive**.
* **Import/Export JSON** de l’état + **Effacer** (réinitialisation).
* **Accents français** robustes à l’import (décode **UTF-8 → Windows-1252 → ISO-8859-1**, normalisation **NFC**).
* **Sauvegarde automatique** dans `localStorage`.

### Version complète (Dark + Classement)

* **Saisie des scores** dans l’onglet *Matches*.
* **Mise en évidence** des matches **complétés** (bordure bleue lorsque les 2 scores sont remplis).
* **Classement en direct** dans l’onglet *Classement* (voir règles ci-dessous).
* **Export CSV (classement)** avec **BOM UTF-8** (compatible Excel/Numbers), séparateur `;`.
* **Alerte anti-écrasement** : si la compétition est commencée (au moins un score saisi) et que vous modifiez la liste (rapide/CSV/manuelle), une **confirmation** est demandée avant de tout regénérer.

### Version Light (Dark)

* **Aucun score, aucun classement**, uniquement la **liste** des rencontres par journée.
* Même UX pour créer la liste et même robustesse d’import.

---

## Mode d’emploi (version complète)

### 1) Paramètres

* **Liste rapide (numéros)**
  Saisir le **nombre de participants (3–50)** → **Valider** → la liste `1,2,3,...,N` est créée.
* **Importer CSV**
  Choisir un fichier texte/CSV **une ligne par joueur**.
  Encodages pris en charge automatiquement : **UTF-8**, **Windows-1252**, **ISO-8859-1**.

  > Exemple de contenu CSV :
  >
  > ```
  > Alice
  > Benoît
  > Chloé
  > Thomas Lefèvre
  > ```
* **Saisie manuelle**
  Coller une liste, **une ligne par joueur** → **Valider**.

> ⚠️ **Si des scores existent déjà**, une **alerte** s’affiche avant d’écraser la compétition (liste + calendrier + scores).

* **Réinitialiser le tournoi (scores uniquement)** : remet tous les scores à vide mais **conserve la liste**.

### 2) Matches

* Les rencontres sont groupées par **Journée** (méthode “cercle”).
* **Match complété** = 2 scores saisis → **bordure bleue** (repère visuel rapide).
* Les **noms d’équipe** sont affichés en **17px** (lisibles sur mobile).

### 3) Classement

* Se met **à jour automatiquement** dès la saisie/modification d’un score.
* Bouton **Exporter CSV (;)** → génère un fichier CSV **UTF-8 avec BOM** pour Excel/Numbers.

### 4) Sauvegarde & transfert

* L’état est **sauvegardé automatiquement** en local (`localStorage`).
* Pour **transférer** vers une autre machine :

  * *Export JSON* sur PC A → récupérer le fichier `.json`
  * *Import JSON* sur PC B → sélectionner le fichier `.json`.

---

## Mode d’emploi (version Light)

* Identique pour **Paramètres** (Liste rapide / Import CSV / Saisie manuelle).
* Onglet **Matches** : affiche **uniquement** l’ordre des matchs par journée (pas de scores).
* **Export/Import JSON** et **Effacer** disponibles dans la topbar.
* Idéal pour **impression** ou **affichage** de l’ordre de passage sans suivi de scoring.

---

## Formats d’import/export

### CSV (import joueurs)

* **Une ligne = un joueur**.
* **Encodage** : détection **UTF-8 → Windows-1252 → ISO-8859-1** puis normalisation **NFC**.
* Aucune colonne supplémentaire, pas d’en-tête.

**Exemple :**

```
Alice
Benoît
Chloé
Thomas Lefèvre
```

### CSV (export classement) — *version complète*

* **Séparateur** : `;`
* **Encodage** : **UTF-8 avec BOM** (Excel/Numbers)
* **Colonnes** : `Rang;Équipe;Points;Pts_plus;Pts_moins;Diff`

**Exemple :**

```
Rang;Équipe;Points;Pts_plus;Pts_moins;Diff
1;Thomas Lefèvre;12;45;21;24
2;Chloé Renaud;10;40;25;15
```

### JSON (export/import état)

* **Version complète** :

  ```json
  {
    "players": ["Alice", "Benoît", "Chloé"],
    "matches": {
      "Alice||Benoît": {"a":"Alice","b":"Benoît","scoreA":5,"scoreB":3},
      "Alice||Chloé":  {"a":"Alice","b":"Chloé","scoreA":2,"scoreB":2}
    },
    "generated": true
  }
  ```
* **Version Light** :

  ```json
  {
    "players": ["Alice", "Benoît", "Chloé"],
    "generated": true
  }
  ```

---

## Règles de classement

* **Victoire** : 3 pts pour le gagnant, **1 pt** pour le perdant
* **Nul** : 2 pts chacun
* **Tiebreakers** (ordre) :

  1. **Points** totaux
  2. **Différence** (Pts+ − Pts−)
  3. **Pts+** (points marqués)

> Le calcul est **rejoué à la volée** à partir de l’état (pas lié à l’affichage).

---

## Caractéristiques techniques

* **Pile front-end** : HTML + CSS + **JavaScript vanilla** (aucun framework).
* **Design system** : Material-like via variables CSS (`:root`) : couleurs, surfaces, rayons, ombres, etc.
* **Dark-only** : pas de switch, aucune surface blanche.
* **Responsive** : grille CSS mobile-first (`grid`, `clamp()`, etc.).
* **Accessibilité** :

  * **Onglets** avec rôles `role="tablist"`, `role="tab"`, `role="tabpanel"`, attributs `aria-selected`.
  * Tables avec `aria-label`.
  * Contrastes adaptés au dark mode.
* **Persistance locale** : `localStorage` (clé `pouleUniqueStateV2` pour la full, `tournoi_light_dark_v1` pour la Light).
  Auto-save supplémentaire lors de `visibilitychange` (onglet quitté).
* **Import CSV** : `FileReader.readAsArrayBuffer` + `TextDecoder` avec **fallback d’encodage**
  (UTF-8 → Windows-1252 → ISO-8859-1) + **normalisation Unicode NFC** (accents corrects).
* **Export CSV** : `Blob` + **BOM** `\uFEFF` en tête pour Excel/Numbers.
* **Génération des matches** : **méthode du cercle** (round-robin) avec insertion d’un `BYE` si nombre impair.
* **Structure d’état (full)** :

  * `players: string[]`
  * `matches: Record<"A||B", {a: string, b: string, scoreA: number|null, scoreB: number|null}>`
  * `generated: boolean`
* **Performance** : DOM minimal, tri en mémoire (petites volumétries ≤ 50 joueurs).

---

## Compatibilité & performances

* **Navigateurs** : versions modernes “evergreen” (Chrome, Edge, Firefox, Safari récents).
  `TextDecoder('windows-1252')` et `Blob` sont requis pour la meilleure expérience.
* **Stockage** : `localStorage` (≈5 Mo). Pour des listes très volumineuses/longues saisons, privilégier l’**export JSON** régulier.
* **Réseau** : **aucun appel réseau** ; l’app fonctionne **hors-ligne** une fois ouverte.

---

## Déploiement (GitHub Pages)

1. Créez un dépôt GitHub (ex. `compteur-eps`).
2. Ajoutez le(s) fichier(s) :

   * `index.html` (version complète)
   * `light-dark.html` (facultatif, variante Light)
3. **Commit/Push**.
4. Dans *Settings → Pages* :

   * *Build and deployment* → *Source*: `Deploy from a branch`
   * *Branch*: `main` / `/root`
5. L’URL `https://<votre-user>.github.io/<repo>/` sert l’app.

---

## Personnalisation

* **Couleurs & thèmes** : modifiez les variables CSS dans `:root` (`--bg`, `--surface`, `--primary`, etc.).
* **Taille des noms** : dans `.team` (actuellement **17px**).
* **Règles de points** : adapter la logique dans la section *classement* (JS) si votre sport a d’autres barèmes.
* **Séparateur CSV** : export par défaut en `;` (adapté à Excel FR). Changez la constante si besoin.

---

## Limites & feuille de route

**Limites actuelles**

* Pas de **multi-poules** / phases finales.
* Pas de **synchro cloud** (utiliser Import/Export JSON pour le transfert).
* Pas de **PWA** / installation sur appareil (possible à ajouter).
* **Doublons de noms** non bloqués (permis mais pas recommandés).

**Idées d’évolution**

* Export **CSV des matches** (toutes les rencontres avec journée).
* **Impression**/PDF de la feuille de matches.
* Support **multi-terrains** / créneaux horaires.
* Mode **“admin + lecteur”** (verrouillage des scores).

---



### Remerciements

Projet conçu pour un usage **simple, offline et sans compte**, avec une attention particulière à l’**ergonomie mobile** et au **respect des accents français** à l’import/export.
Bon tournoi ! 🏆
