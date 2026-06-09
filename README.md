# PROJECT-CG

Application web de prévision des ventes mensuelles pour **PAPETIS DISTRIBUTION SARL** (Casablanca), développée pour la Direction Administrative et Financière.

## Présentation

PAPETIS est une application monopage (**SPA monofichier**) entièrement côté client, sans dépendance externe à l'exécution. Elle analyse l'historique de ventes 2021–2025 (60 mois, 7 familles de produits) et génère des prévisions glissantes à 12 mois avec intervalles de confiance.

> **Aucune installation requise.** Aucun serveur. Aucune connexion Internet. Ouvrez `PAPETIS_dashboard_final.html` dans un navigateur moderne et l'outil fonctionne immédiatement.

## Lancement

1. Ouvrir `PAPETIS_dashboard_final.html` dans Chrome, Firefox ou Edge (version ≥ 2021).
2. Les données historiques intégrées (2021–2025) se chargent automatiquement au démarrage.
3. Pour utiliser vos propres données, importez un fichier CSV via le bouton d'import de la sidebar.
4. Les graphiques, indicateurs et prévisions se recalculent instantanément à chaque changement de paramètre.

## Fonctionnalités

| Fonctionnalité | Statut |
|---|---|
| Données historiques 2021–2025 intégrées (60 mois × 7 familles) | ✅ |
| Import CSV personnalisé (minimum 24 mois) | ✅ |
| Tendance par MCO (moindres carrés ordinaires) | ✅ |
| Tendance par moyennes mobiles centrées d'ordre 12 (MMC) | ✅ |
| Lissage exponentiel avec saisonnalité | ✅ |
| Coefficients saisonniers — modèles multiplicatif et additif | ✅ |
| Prévisions 12 mois glissants avec intervalles de confiance à 90 % | ✅ |
| Graphique historique + prévisions (Canvas 2D natif) | ✅ |
| Export CSV des prévisions | ✅ |
| Export PDF (rapport synthétique via `window.print`) | ✅ |
| Comparaison des méthodes — RMSE / MAPE | ✅ |
| Recommandation automatique additif / multiplicatif | ✅ |
| Analyse de scénarios pessimiste / central / optimiste (±20 %) | ✅ |
| Détection automatique des observations atypiques (Z-score > 2) | ✅ |
| Ventilation par famille de produits (7 familles) | ✅ |
| KPI synthétiques : CA 2025, prévision 2026, croissance, fourchette | ✅ |

## Familles de produits

Le dashboard couvre les 7 familles de PAPETIS : **Cahiers**, **Classeurs**, **Écriture**, **Technique**, **Manuels**, **Bureau**, et l'agrégat **Ventes totales**. Chaque famille peut être analysée indépendamment ; le modèle saisonnier (multiplicatif ou additif) est réévalué pour chaque sélection.

## Architecture

Le dashboard est un **fichier HTML unique** contenant l'intégralité du code (HTML, CSS, JavaScript). Il n'y a aucun dossier, aucun fichier séparé, aucune bibliothèque externe chargée à l'exécution.

```
PAPETIS_dashboard_final.html   ← l'unique fichier du projet
```

Le code JavaScript interne est organisé en blocs fonctionnels distincts au sein de la balise `<script>` :

| Bloc logique | Rôle | Fonctions principales |
|---|---|---|
| Forecasting | Algorithmes statistiques | `ols()`, `centeredMovingAverage()`, `computeSeasonalCoeffs()`, `decomposeAndForecast()`, `computeErrors()`, `detectOutliers()`, `adjustedForecast()` |
| Charts | Rendu graphique | `drawChart()` — Canvas 2D natif, redimensionnement automatique |
| Export | Génération des fichiers | `exportCsv()` via Blob natif · `exportPdf()` via `canvas.toDataURL` |
| Contrôleur App | État global et orchestration | `state {family, method, modelType, scenario}` · `render()` centrale |

### Flux de données

1. **Initialisation** — `buildRows(rawRows)` construit les 60 observations intégrées puis `render()` s'exécute automatiquement.
2. **Import optionnel** — un CSV utilisateur remplace `rows` et déclenche un nouveau `render()`.
3. **Calcul** — `decomposeAndForecast()` + `computeErrors()` pour la méthode active et les deux modèles en parallèle.
4. **Scénario** — `adjustedForecast()` multiplie les prévisions par `(1 + scenario/100)`.
5. **Rendu graphique** — `drawChart(canvas, hist, adjustedFc)` via Canvas 2D.
6. **Rendu DOM** — KPI, tableau comparatif, tableau mensuel, tags atypiques mis à jour.
7. **Export** — à la demande : CSV ou PDF.

## Technologies

- **Canvas API 2D** — Graphiques interactifs sans dépendance externe
- **Blob / URL.createObjectURL** — Export CSV natif
- **window.print / canvas.toDataURL** — Export PDF
- **HTML5 / CSS Grid / Flexbox** — Interface responsive

Aucune dépendance NPM, aucun CDN, aucun framework.

## Méthodes statistiques

### Modèle multiplicatif vs additif

Le modèle **multiplicatif** est recommandé lorsque l'amplitude des fluctuations saisonnières croît proportionnellement avec le niveau de la série. Pour PAPETIS, le pic de rentrée (juillet–septembre) représente systématiquement 3× à 4× la vente mensuelle moyenne quelle que soit l'année, ce qui justifie ce choix. Le dashboard teste les deux modèles en parallèle et affiche une recommandation automatique fondée sur le RMSE.

### Décomposition (MCO)

1. Tendance par moyenne mobile centrée d'ordre 12
2. Ratios saisonniers : ventes / MMC, moyennés par mois et normalisés
3. Série désaisonnalisée : ventes / coefficient saisonnier
4. Ajustement de la tendance par MCO sur la série désaisonnalisée
5. Prévision = tendance extrapolée × coefficient saisonnier du mois cible

### Lissage exponentiel

Lissage triple avec paramètres α = 0,3 (niveau), β = 0,1 (tendance), γ = 0,2 (saisonnalité).

### Intervalles de confiance

Calculés analytiquement : ±1,645 × écart-type des résidus in-sample (intervalle à 90 %).

### Détection des atypiques

Basée sur le Z-score des résidus — seuil d'alerte : |Z| > 2,0. Les mois concernés sont signalés par des tags visuels dans l'interface.

## Format CSV attendu

Si vous importez vos propres données, le fichier doit respecter le format suivant :

```
annee,mois,Cahiers,Classeurs,Ecriture,Technique,Manuels,Bureau,Total
2021,1,252,200,262,133,165,181,1193
...
```

- Minimum requis : **24 mois** (2 cycles saisonniers complets)
- Les 7 colonnes de familles doivent être présentes, ou la colonne `Total` seule peut suffire
- Les valeurs sont en **kMAD** (milliers de dirhams marocains)

## Glossaire

| Terme | Définition |
|---|---|
| MCO | Moindres Carrés Ordinaires — régression linéaire de la série désaisonnalisée |
| MMC | Moyenne Mobile Centrée d'ordre 12 — lissage de la tendance |
| RMSE | Racine de l'erreur quadratique moyenne (indicateur de comparaison des méthodes) |
| MAPE | Erreur absolue moyenne en pourcentage (cible < 15 %) |
| IC 90 % | Intervalle de confiance à 90 % : ±1,645·σ autour de la prévision centrale |
| Z-score | Nombre d'écarts-types d'un résidu — seuil d'alerte |Z| > 2,0 |
| kMAD | Milliers de dirhams marocains — unité monétaire du dashboard |
| Scénario | Ajustement multiplicatif des prévisions via le curseur ±20 % de la sidebar |
| `state` | Objet JS global `{family, method, modelType, scenario}` |
| `render()` | Fonction centrale recalculant et redessinant l'intégralité du dashboard |

## Compatibilité

Testé sur Chrome, Firefox et Edge ≥ 2021. Aucune dépendance à Internet Explorer.

---

*PAPETIS DISTRIBUTION SARL — Document confidentiel — Usage interne uniquement — Version 1.0 — Juin 2026*
