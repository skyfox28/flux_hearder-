# Flux Hebdo Dépôt

Application HTML 100 % frontend pour suivre chaque semaine l'activité du dépôt : flux entrants, sortants et préparation.
Aucun serveur : on ouvre `index.html` dans Edge ou Chrome, et les données restent stockées dans le navigateur (IndexedDB).

Gardez ensemble `index.html` et le dossier `lib/`. Le référentiel articles (Fichier Article, MLGT, magasins) est intégré à `index.html`.

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
