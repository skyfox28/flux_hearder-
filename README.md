# Flux Hebdo Dépôt

Version **1.25.0** · Développé par **Galaad Poivey**

Application HTML 100 % frontend pour suivre chaque semaine l'activité du dépôt : flux entrants, sortants et préparation.
Aucun serveur : on ouvre le fichier de l'app dans Edge ou Chrome, et les données restent stockées dans le navigateur (IndexedDB).

Dans le zip livré, le fichier de l'app porte le nom de l'app et sa version, par exemple `Flux_Hebdo_v1.23.0.html` (dans ce dépôt, il s'appelle `index.html`). Gardez-le avec le dossier `lib/`. Le référentiel articles (Fichier Article, MLGT, magasins) est intégré au fichier. Changer de version ne fait rien perdre : les données sont liées au navigateur, pas au nom du fichier.

## Sources alimentées

| Onglet | Source | Mode |
|---|---|---|
| Préparation | Extraction SAP **VL06O** (`VL06O.XLSX`) | glisser-déposer : l'import remplace la photo précédente |
| Expéditions | **Base affrètement** (Access) + livraisons VL06O non affrétées | copier-coller des lignes (avec ou sans en-têtes) |
| Intercos | VL06O + affrètement (envois) et planning SST (retours) | automatique : priorité 9, ou mots-clés Plateforme 38 / Compans / McCormick |
| Rapatriements | Classeurs `AAAA_Sxx_<Entrepôt>.xlsx` (FM, Mutual, Tempo One…) | glisser-déposer, plusieurs fichiers à la fois |
| Réceptions SST | `Planning_Reception_SST_Monteux3.xlsb` (onglets « PlanningReception AAAA ») | glisser-déposer, ou saisie manuelle d'un RDV |
| Paramètres | Fichier Article, MLGT, magasins, cadences | `FichierArticle.XLSX`, `MLGT.XLSX` ou directement `Analyse_Activité_Préparation.xlsb` |

## Méthode de calcul de la préparation

Elle reprend la requête Power Query et les formules du classeur *Analyse_Activité_Préparation* :

- les postes VL06O à `Number of Packages = 0` sont exclus ;
- **UQ/PAL** vient du Fichier Article (filtré sur UQA = PAL, hors ROH et ZNVM). Les codes articles sont comparés sans leurs zéros de tête (`000000000000278710` = `278710`). Un poste sans fiche article n'est compté ni en SILO ni en picking, comme dans le fichier Excel ; l'app le signale ;
- **Pal SILO** = arrondi inférieur de (colis ÷ UQ/PAL), **Colis SILO** = Pal SILO × UQ/PAL, **Colis picking** = colis − Colis SILO ;
- **Circuit** selon la priorité de livraison : 1 Entrepôt, 2 GMS, 3 Export, 4 Allotie, 6 MDD, 9 Interco ;
- **Hr SILO** = Pal SILO ÷ 18 ; **Hr Pick** = colis picking ÷ cadence (Export 750, MDD 1300, autres circuits 400). Les cadences se modifient dans Paramètres ;
- **Nb Pick** = 1 si le poste a du picking ; l'emplacement picking vient de MLGT ;
- **Fait** = postes au statut global de prélèvement C, **A faire** = postes au statut A ou B.

Sur l'extraction VL06O fournie, les totaux ont été vérifiés contre un recalcul indépendant : 1 994 postes, 145 livraisons, 1 060 pal SILO, 58,89 h SILO, 64 419 colis picking et 140,51 h picking.

## Rapatriements

Chaque onglet placé après un séparateur `LUNDI>>`, `MARDI>>`, etc. est lu comme un camion. La date vient de la colonne « A LIVRER LE » ou « Date Détournement ». À défaut, elle est déduite du jour du séparateur et de la semaine indiquée dans le nom du fichier.
Les palettes sont prises dans la colonne « Pal » / « Nb Pal » ; sans cette colonne, chaque ligne compte pour une palette.
Sens des flux :
- **DT** (onglets `F…`, `M…`, `T…` vers le destinataire 2530) : rapatriement, **entrée** au dépôt ;
- **DESTO** (déstockage) et **NAV** (navette détournée) : **sortie** de chez nous vers l'entrepôt externe.

Un clic sur le sens d'un camion l'inverse en cas d'exception.

## Réceptions sous-traitants

Seuls les créneaux qui ont un RDV sous-traitant ou un nombre de palettes sont repris. Les onglets « Cumul » et « Ne pas toucher » sont ignorés.
Le pointage « Reçu » se fait dans l'app, et il est conservé quand on ré-importe le planning.
Un créneau passé sans pointage est signalé en orange.
Les totaux sont calculés sur les lignes du planning, pas sur le tableau croisé « Cumul » : si celui-ci n'a pas été actualisé, les deux peuvent différer.

## Intercos : envois et retours

Certaines plateformes sont dans les deux flux : Plateforme 38, par exemple, reçoit nos produits, les retravaille et nous les renvoie.
- Les **envois** sont les livraisons VL06O ou affrètement de priorité 9, ou dont le réceptionnaire contient un mot-clé interco.
- Les **retours** sont les RDV du planning SST dont le sous-traitant contient ce même mot-clé. La comparaison ignore les espaces : « Plateforme38 » = « PLATEFORME 38 ».

Les deux flux sont suivis **séparément**, sans rapprochement : un camion envoyé un jour ne revient pas forcément la même semaine. L'onglet Intercos affiche pour chaque plateforme ce qui est envoyé et ce qui est reçu sur la semaine affichée, jour par jour.

## Indicateurs cliquables

Chaque indicateur (tuiles et chiffres du tableau « Semaine jour par jour ») ouvre un détail en surimpression : répartition par jour, par transporteur, par entrepôt ou par circuit, et la liste des livraisons, camions ou RDV concernés.
Un chiffre de la matrice ouvre le détail du jour ; le bouton « Toute la semaine » élargit à la semaine. Pour fermer : `Échap`, la croix, ou un clic à côté.

## Transporteurs

La correspondance code → nom de la base affrètement est intégrée (3 STEF PARIS ATHIS-MONS … 440 EXPORT DDE). Elle peut être surchargée dans Paramètres.

## Expéditions : base affrètement + VL06O

Le collage accepte l'export complet de la base affrètement, avec ou sans en-têtes :
`affret · Prep · Code Depot · Liv · Date enlevement · code transporteur · Nom transporteur · CodeClient · nom · ville · rue · rue 4 · cp · NbPalettes_Ent · NbCouches_Ent · NbColis_Detail · NbCartons_PK · poids taxable · Pds brut · blocs · Palettes · commentaires`.
L'ancien format (`Date Enl · Livraison · …`) reste accepté. Le nom du transporteur de l'export est retenu, et l'adresse s'affiche au survol de la ville.

Les livraisons de la VL06O **absentes** de la base affrètement sont ajoutées à la liste, sans doublon : une livraison affrétée n'est jamais reprise depuis la VL06O.
- Elles sont datées à leur date de chargement et marquées « VL06O · non affrété ».
- Leurs palettes et leurs blocs ne sont connus qu'une fois l'affrètement collé ; en attendant, l'app affiche leurs Pal SILO.

## Capacité de picking (équipes)

Les heures de picking du fichier Excel (colis ÷ cadence) sont des **heures de travail pour un préparateur**, pas une durée réelle.
L'app les compare à la capacité des équipes, qui se règle dans Paramètres. Par défaut :
- 2 préparateurs de 5h00 à 12h30 et 2 préparateurs de 12h30 à 20h00 ;
- soit 4 × 7,5 h = **30 h de picking par jour travaillé** (du lundi au vendredi).

Pour chaque jour de chargement, l'app affiche :
- le **taux de charge** = heures de picking ÷ capacité ;
- les **préparateurs nécessaires** = heures ÷ 7,5 h de poste ;
- la **durée avec l'équipe** = heures ÷ 4 préparateurs.

Le SILO (palettes complètes, 18 pal/h) est suivi à part.

## Livraisons expédiées et départs hors Monteux3

- Une livraison de la base affrètement **absente de la VL06O** est considérée comme **expédiée** : elle est marquée « Expédiée » et comptée dans « Prêtes ou expédiées ».
- Les livraisons au **code dépôt 540** ou pour **COLRUYT** partent directement de **Compans**, pas de Monteux3. Elles ne sont pas dans la VL06O, et c'est normal. Elles sont listées à part (« Départs directs Compans ») et exclues des totaux du dépôt.
- Le site, les codes dépôt et les clients concernés se règlent dans Paramètres.

## Purger les données

Le bouton **Purger** en haut de l'écran (également dans Paramètres) ouvre une fenêtre de purge. On y choisit :
- **ce qu'on supprime** : VL06O, base affrètement, rapatriements, réceptions SST, et éventuellement le référentiel importé et les paramètres ;
- **la période** : tout, ou seulement les données antérieures à la semaine affichée, pour garder l'historique récent et alléger le navigateur.

Une confirmation est demandée avant de supprimer. En mode « avant la semaine », la VL06O est décochée d'office : c'est une photo unique, qu'on ne supprime qu'en entier.
Pensez à exporter une sauvegarde avant de purger.

## Grands détails : flux palettes et charge SILO / picking

Sur le tableau de bord, un clic sur les graphiques « Flux palettes par jour » et « Charge SILO / picking » ouvre un détail en quasi plein écran.

- **Flux palettes :**
  - graphique entrées | sorties par jour, avec l'origine de chaque palette (rapatriements, réceptions SST, expéditions affrétées, transferts vers les externes) ;
  - tableau détaillé ligne par ligne (entrepôt, sous-traitant, transporteur) avec le solde entrées − sorties ;
  - pour information : les livraisons VL06O non affrétées et les départs Compans ;
  - un journal des flux jour par jour.
- **Charge SILO / picking :**
  - heures de picking par circuit face à la capacité des préparateurs ;
  - palettes SILO faites / à faire ;
  - tableau par jour : reste à faire, taux de charge, préparateurs nécessaires, durée avec l'équipe ;
  - répartition par circuit, statut des livraisons, et livraisons non terminées triées par heures restantes.

Chaque chiffre ouvre son propre détail, et le bouton « ← Retour » ramène au grand détail.

## Versions

Le numéro de version (`APP_VERSION` dans `index.html`) augmente à chaque modification de l'app. L'historique complet est consultable en cliquant sur le numéro de version, en bas de l'écran ou dans Paramètres.

## Capacités (quai, SILO, picking)

- **Quai :** 2 250 palettes par semaine en entrée et 3 000 en sortie, réparties sur les jours travaillés (450 et 600 palettes par jour).
  - Entrées = rapatriements + réceptions SST.
  - Sorties = expéditions affrétées + transferts vers les externes.
  - L'app affiche le taux par jour et par semaine, sur le graphique « Flux palettes » (lignes pointillées) et dans le tableau de bord.
- **SILO et picking : taux de charge calculés séparément.**
  - SILO : heures SILO ÷ capacité des équipes SILO. Par défaut, 1 cariste de 5h00 à 12h30 et 1 de 12h30 à 20h00, soit 15 h/jour ou 270 palettes à 18 pal/h.
  - Picking : heures de picking ÷ capacité des préparateurs (30 h/jour).
- Toutes ces capacités se règlent dans Paramètres.

## Réel : la journée passe-t-elle ?

À chaque import de la VL06O, l'app fait un point « réel ». L'heure de référence est celle de l'**extraction du fichier** (sa date d'enregistrement).

**Le calcul :**
- **Reste à faire :** les postes pas encore au statut C, convertis en heures SILO et picking avec les cadences. Le reste est cumulé dans l'ordre des chargements, en comptant aussi les chargements en retard.
- **Capacité restante :** les heures d'équipe encore disponibles entre l'heure d'extraction et l'échéance de préparation. Par défaut, l'échéance est la fin de la veille travaillée ; on peut la mettre au jour même dans Paramètres.
- **Verdict :** « passe » si le reste cumulé tient dans la capacité, sinon « ne passe pas » avec les heures manquantes. L'app donne aussi une heure de fin estimée, et calcule tout séparément pour le SILO et le picking.
- **Limite réelle :** sur les graphiques, un trait rouge pointillé par jour = heures déjà faites + capacité encore disponible. Si le haut de la barre dépasse le trait, la journée ne passe pas.
- **Rendement réel :** entre deux imports successifs (par exemple le matin et l'après-midi), l'app mesure les heures théoriques réalisées. Ce sont les postes passés au statut C ou sortis de la VL06O. Elle les divise par les heures d'équipe écoulées. Ce rendement ajuste la capacité restante et s'affine à chaque actualisation ; l'ajustement peut être désactivé dans Paramètres.

## Créneaux de chargement (table LIKP)

Dans l'onglet Préparation, section « Heures de chargement (table LIKP) », on colle l'extraction LIKP ou on dépose le fichier Excel.
- **Collage sans en-têtes :** l'app repère le n° de livraison, puis la première date suivie d'une heure non nulle (`01.10.2026 · 06:00:00`).
- **Fichier avec en-têtes :** les colonnes `VBELN / Livraison`, `LDDAT / Date de chargement`, `LDUHR / Heure de chargement` sont reconnues ; à défaut, `KODAT / KOUHR`.
- Les imports successifs se complètent.

L'heure LIKP est le **début d'un créneau de chargement de 2 h** (8h → 8h–10h).

**Effet sur la projection « réel » :**
- chaque livraison a pour échéance le début de son créneau moins 30 minutes ;
- le reste à faire est ordonné par échéance, livraison par livraison ;
- l'app donne pour chacune sa fin de préparation estimée et son éventuel retard, dans le tableau « Livraisons en retard prévu ».

La durée du créneau, le calage (début ou fin du créneau) et la marge se règlent dans Paramètres. Sans heure LIKP, l'échéance est **estimée le jour même avant 12h00**. L'heure et la règle se règlent dans Paramètres : jour même avant une heure, veille au soir ou fin de journée.

## Planning de chargement (onglet Préparation)

C'est la vue par défaut de l'onglet Préparation. Elle se lit **jour par jour**.

- **En-tête du jour :** volume, avancement, verdict picking et SILO (passe / ne passe pas, marge ou manque) et nombre de livraisons en retard.
- **Une ligne par créneau de chargement LIKP** (6h–8h, 8h–10h… puis « Sans créneau LIKP ») :
  - livraisons prêtes, palettes SILO et colis de picking (reste compris) ;
  - reste à faire SILO et picking, échéance de préparation et fin estimée ;
  - statut : ✓ tout est prêt, ✓ passe (avec la marge) ou ✗ en retard (avec le nombre de livraisons et le retard maximal).
- **Frise horaire du jour (0h–24h) :**
  - plages des équipes picking et SILO, y compris la nuit SILO 20h–3h30 ;
  - créneaux de chargement en blocs (vert = passe ou prêt, orange = en cours, rouge = retard), avec les livraisons au survol ;
  - repères de fin estimée du picking et du SILO, et heure d'extraction de la VL06O si elle tombe ce jour-là.
- **Un clic sur un créneau** déplie ses livraisons : reste à faire, fin estimée, statut prépa / OT et verdict.
- **« Détail des N livraisons du jour »** affiche toutes les livraisons du jour, triées par créneau. On peut y **saisir l'heure de chargement** d'une livraison sans créneau ; elle est enregistrée comme une heure LIKP. Un bandeau signale les livraisons à préparer sans créneau et donne leurs n° pour les extraire de la LIKP.
- Les filtres de l'onglet (date, circuit, statuts, recherche) s'appliquent.

## Créneaux fixes par client

Certains clients ont un créneau de chargement fixe. Il **prime sur la LIKP** :
- **Plateforme 38 : 6h–8h.** Prête au début du créneau, soit 5h30 avec la marge de 30 min.
- **Compans : 6h–15h.** Chargement possible sur toute la fenêtre, donc prête pour la fin, soit 14h30.

Les autres livraisons prennent leur créneau dans la LIKP. Liste modifiable dans Paramètres, section « Réel » : client (mot-clé), début, fin et « prêt pour le début / la fin du créneau ».

## Équipes par défaut

- **Picking :** 2 préparateurs de 5h00 à 12h30 et 2 de 12h30 à 20h00, soit 30 h par jour.
- **SILO :** 1 cariste de 5h00 à 12h30, 1 de 12h30 à 20h00 et **1 de nuit de 20h00 à 3h30**, soit 22,5 h par jour. La nuit est prise en compte dans le calcul « réel ».

## Interface

- **Couleurs sobres :** palette atténuée d'environ 20 %, fond et effet verre plus calmes. Les couleurs de séries sont vérifiées pour la lisibilité, y compris pour les daltoniens, en thème clair comme sombre.
- **État des sources** sous le titre : VL06O, LIKP, affrètement, rapatriements, SST.
  - Pastille verte = mise à jour depuis moins de 12 h, orange = plus ancienne, grise = absente.
  - Un clic ouvre l'onglet correspondant.
- **Pastilles dans le menu :** livraisons en retard prévu pour Préparation, RDV passés non pointés pour Réceptions SST.
- **Points d'attention** affichés juste après les indicateurs, seulement s'il y en a.
- **Tableaux zébrés** pour faciliter la lecture.

## Écran d'ouverture

À l'ouverture, une scène 3D animée (en CSS, sans bibliothèque) montre l'entrepôt Monteux3 : camions qui reculent à quai puis repartent, camions sur la route, chariot élévateur qui sort des palettes, palettes préparées qui apparaissent au quai. Sous la scène, les vraies étapes du démarrage s'affichent :

1. **Données locales** : lecture de la base du navigateur.
2. **Calcul des flux de la semaine.**
3. **Sauvegarde partagée** : lecture du dossier et chargement de la sauvegarde la plus récente. Si le navigateur demande de reconfirmer l'accès au dossier, l'écran propose « Charger la dernière sauvegarde » (il faut un clic) ou « Continuer sans ».

**Connexion (temporaire, le temps des tests) :** l'écran ne se ferme pas tout seul. On entre dans l'app en se connectant avec l'identifiant `admin` et le mot de passe de test (communiqué à part). Le nom de l'utilisateur connecté s'affiche en bas du menu, avec un lien « déconnexion ». Il s'agit d'un simple contrôle dans le navigateur, **pas d'une vraie sécurité** : les données restent lisibles par qui ouvre le fichier. Un panneau dans la scène et un badge à côté du nom de l'app indiquent « Coming soon · en développement » : c'est l'app elle-même qui est en cours de développement.

Paramètres → À propos → « Animer l'écran d'ouverture » : décoché, la scène reste figée, mais la connexion reste demandée. L'animation est aussi réduite si le système demande de limiter les animations.

## Onglet Sources

Toutes les mises à jour se font au même endroit : l'onglet **Sources**, juste sous le tableau de bord. Une carte par source, chacune avec son état (date, volume) :

| Source | Mise à jour | Dossier SharePoint possible |
|---|---|---|
| VL06O | dépôt du fichier | oui (VL06O le plus récent) |
| LIKP | collage ou fichier Excel | oui (LIKP le plus récent) |
| Base affrètement | collage depuis Access | – |
| Navettes usines (VL06I) | collage | – |
| Rapatriements | dépôt des fichiers | oui, **un dossier par entrepôt** : FM, Mutual, Tempo One |
| Planning réception SST | dépôt du fichier | oui (planning le plus récent) |

Les dossiers reliés sont relus automatiquement à chaque ouverture (étape « Fichiers SharePoint » de l'écran d'ouverture), et à la demande avec « Tout actualiser depuis SharePoint ». Les autres onglets n'affichent plus qu'une barre d'état avec un bouton « Mettre à jour › ». Les pastilles en haut de page mènent aussi à Sources. Le référentiel articles et la sauvegarde partagée restent dans Paramètres.

## Navettes usines (VL06I)

- On colle la VL06I dans Sources. Comme la VL06O, chaque collage **remplace** le précédent : une navette absente du nouveau collage a été déchargée.
- Seuls les sites usines **2560** (Carpentras) et **2510** (usine épices) sont retenus. La liste est réglable dans Paramètres → Quai de réception.
- **Palettes = nombre de colis.** Si le nombre de colis est vide, il est estimé au poids, à partir du poids moyen par palette des autres navettes (signalé par ≈).
- **Date dépassée :** une navette encore présente compte le jour de la VL06I. Par exemple, une navette du 29/09 dans une VL06I du 01/10 compte le 01/10.
- Pas d'horaire : les chauffeurs de parc les ramènent au fil de la journée. Pour le quai de réception, elles sont réparties de 05:00 à 20:00 (réglable).
- Les navettes s'ajoutent aux **entrées** : tableau de bord, overlay flux palettes, matrice jour par jour, quai de réception. L'onglet **Navettes usines** liste simplement les navettes en cours (date, usine, livraison, palettes).

## Affichage épuré

Les longues explications sont repliées derrière une petite icône **ⓘ** à côté du titre (survol = bulle, clic = déplier). Les points d'attention tiennent sur une ligne (clic pour voir la liste). Dans l'overlay flux palettes, le solde et les retours interco ont été retirés, et les navettes usines ajoutées.

## Quai de réception

Simulation, jour par jour, de la réception de la semaine affichée :

1. **Arrivée des camions :** heure d'arrivée pointée, sinon heure du RDV du planning SST, sinon (rapatriements sans heure) répartie régulièrement sur une plage réglable (06:00–14:00 par défaut).
2. **Portes :** chaque camion prend la première porte libre (3 portes par défaut) et décharge pendant 45 min (réglable). Si toutes les portes sont prises, il attend : l'attente est calculée.
3. **Sol :** à la fin du déchargement, les palettes sont posées au sol (1 bloc par palette par défaut). Capacité du sol : 150 blocs par défaut.
4. **Rangement :** dans l'ordre d'arrivée, à 25 palettes/h par cariste réception présent. Les équipes de rangement se règlent comme les autres équipes (par défaut, 1 cariste 05:00–12:30 et 1 cariste 12:30–20:00).

Verdict rouge pour un jour si le sol dépasse sa capacité ou si un camion attend 30 min ou plus. Une tuile « Quai réception (blocs) » est ajoutée au tableau de bord, dans le groupe Entrées. Son overlay contient :
- la courbe des palettes au sol ;
- le tableau par jour : pic, attente maximale, heure de fin de rangement ;
- le calcul camion par camion : porte, déchargement, attente, heure de rangement.

**Les valeurs par défaut sont à ajuster** dans Paramètres → « Quai de réception » et « Équipes de rangement ».

## Dossiers sources SharePoint

Onglet Sources → « Relier un dossier SharePoint ». On choisit une fois, pour chaque source, le dossier SharePoint synchronisé par OneDrive où les fichiers sont déposés. Ensuite :
- **À chaque ouverture**, l'app relit automatiquement les fichiers nouveaux ou modifiés depuis le dernier import (étape « Fichiers SharePoint » de l'écran d'ouverture).
- **Rapatriements :** un dossier par entrepôt (FM, Mutual, Tempo One) ; fichiers `AAAA_Sxx_<Entrepôt>.xlsx` des semaines proches (± 3 semaines), sous-dossiers compris.
- **VL06O et LIKP :** le fichier le plus récent du dossier.
- **Planning SST :** le fichier « Planning…Réception… » le plus récent.
- Les boutons **Actualiser** (par dossier) et **Tout actualiser depuis SharePoint** de l'onglet Sources relisent sans fermer l'app.
- Si le navigateur demande de reconfirmer l'accès, une seule question à l'ouverture couvre tous les dossiers : sauvegarde et sources.

**Fermeture de l'onglet :** tant que la dernière modification n'est pas enregistrée dans le dossier de sauvegarde partagé, le navigateur demande confirmation avant de fermer, et l'enregistrement démarre pendant ce temps. Un navigateur ne permet pas d'empêcher totalement la fermeture : c'est le maximum autorisé.

## Sauvegarde partagée automatique

Dossier prévu : `MC CORMICK & COMPANY INC\WeDeliver - Documents\101_Sandbox GP\Flux_hebdo\Sauvegarde`. C'est le dossier SharePoint synchronisé en local par OneDrive.

- **Mise en place, une fois par poste :** ouvrez le fichier de l'app dans **Edge ou Chrome**, puis Paramètres → « Choisir le dossier Sauvegarde… » (ou cliquez sur le bouton nuage en haut). Sélectionnez le dossier `Sauvegarde` et indiquez votre nom. Un navigateur ne peut pas ouvrir un chemin tout seul : l'accès au dossier est donné une fois, puis mémorisé.
- **À chaque ouverture :** l'app lit le dossier et charge la sauvegarde **la plus récente** si elle est plus récente que les données du navigateur. Si le navigateur redemande l'autorisation, un clic sur « Sauvegarde : reconnecter » suffit ; Chrome/Edge proposent aussi « Autoriser à chaque visite ».
- **Après chaque modification** (import, collage, réglage), l'app enregistre une copie horodatée au bout de 4 secondes, puis de nouveau quand on ferme ou quitte l'onglet. S'il reste des modifications non enregistrées à la fermeture, le navigateur demande confirmation, le temps d'écrire le fichier.
- **Plusieurs utilisateurs :** chaque copie a son propre nom, `Flux_Hebdo_AAAA-MM-JJ_HHhMMmSS_<nom>.json`, donc OneDrive ne crée pas de conflit. La sauvegarde la plus récente l'emporte : à l'ouverture et au retour sur l'onglet, l'app charge celle d'un collègue si elle est plus récente que la vôtre. Les modifications ne sont pas fusionnées : deux personnes qui modifient en même temps, c'est la dernière enregistrée qui compte.
- Les **30 dernières copies** sont conservées ; les plus anciennes sont supprimées automatiquement.
- Boutons dans Paramètres : Enregistrer maintenant, Recharger la plus récente, Changer de dossier, Déconnecter. Les 5 dernières copies sont listées.
- Non disponible dans Firefox ni dans l'aperçu en ligne (la page intégrée n'a pas accès aux dossiers). L'export / restauration manuel reste disponible partout.

## Saturation du quai d'expédition

Le quai se compte en **blocs** : 1 bloc = 1 place au sol. Les livraisons préparées occupent le quai jusqu'au départ du camion ; si le quai est plein, on ne peut plus préparer. L'app simule l'occupation du quai heure par heure, à partir de la dernière VL06O :

1. **Au départ :** les blocs des livraisons déjà préparées et pas encore parties. Une livraison terminée compte en entier, une livraison en cours au prorata des heures faites.
2. **Arrivées :** chaque livraison pose ses blocs restants à sa **fin de préparation estimée** (projection « réel »).
3. **Départs :** elle libère ses blocs au **début, au milieu ou à la fin de son créneau de chargement** (milieu par défaut). Si la préparation finit après le créneau, la livraison part dès qu'elle est prête (signalée « après créneau »).
4. **Blocs d'une livraison :**
   - colonne `blocs` de la base affrètement ;
   - à défaut, palettes (affrètement, ou Pal SILO + ⌈colis picking ÷ 80⌉) × **ratio blocs/palettes observé** dans la base affrètement (0,65 par défaut s'il y a moins de 5 livraisons affrétées).

**Où le voir :**
- **Réglages :** la capacité (**450 blocs** par défaut), le ratio blocs/palettes par défaut, les colis par palette de picking et le moment de départ du camion se règlent dans Paramètres, section « Quai d'expédition ».
- **Overlay « Quai d'expédition » :** s'ouvre depuis la tuile « Quai expédition (blocs) » du tableau de bord, ou depuis l'encart du détail SILO / picking. Il regroupe les chiffres clés (blocs au quai à l'extraction, pic prévu, plages saturées, ratio), la courbe d'occupation heure par heure sur tous les jours projetés (week-ends masqués, survol pour le détail de chaque heure), le verdict par jour et le calcul par livraison.
- **Planning :** chaque jour a un verdict « Quai exp. », fluide ou saturé avec les plages horaires. La frise montre la courbe d'occupation, la ligne de capacité et les plages saturées en rouge.
- **Tableau de bord :** une tuile « Quai expédition (blocs) » donne l'occupation actuelle et la prochaine saturation ; un clic ouvre l'overlay du quai.
- **Calcul livraison par livraison** (dans l'overlay du quai) : blocs, origine du nombre, déjà au quai, à poser, heure de pose, heure de départ. Le grand détail Charge SILO / picking n'affiche plus qu'un encart résumé qui renvoie vers cet overlay.
