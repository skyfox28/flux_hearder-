# Flux Hebdo Dépôt

Application HTML 100 % frontend pour suivre chaque semaine l'activité du dépôt : flux entrants, sortants et préparation.
Aucun serveur : on ouvre `index.html` dans Edge ou Chrome, et les données restent stockées dans le navigateur (IndexedDB).

Gardez ensemble `index.html`, `referentiels.js` et le dossier `lib/`.

## Sources alimentées

| Onglet | Source | Mode |
|---|---|---|
| Préparation | Extraction SAP **VL06O** (`VL06O.XLSX`) | glisser-déposer : l'import remplace la photo précédente |
| Expéditions | **Base affrètement** (Access) | copier-coller des lignes (avec ou sans en-têtes) |
| Intercos | VL06O + affrètement (envois) et planning SST (retours) | automatique : priorité 9, ou mots-clés Plateforme 38 / Compans / McCormick |
| Rapatriements | Classeurs `AAAA_Sxx_<Entrepôt>.xlsx` (FM, Mutual, Tempo One…) | glisser-déposer, plusieurs fichiers à la fois |
| Réceptions SST | `Planning_Reception_SST_Monteux3.xlsb` (onglets « PlanningReception AAAA ») | glisser-déposer, ou saisie manuelle d'un RDV |
| Paramètres | Fichier Article, MLGT, magasins, cadences | `FichierArticle.XLSX`, `MLGT.XLSX` ou directement `Analyse_Activité_Préparation.xlsb` |

## Méthode de calcul de la préparation

Elle reprend la requête Power Query et les formules du classeur *Analyse_Activité_Préparation* :

- les postes VL06O à `Number of Packages = 0` sont exclus ;
- **UQ/PAL** vient du Fichier Article (filtré sur UQA = PAL, hors ROH et ZNVM) ;
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

L'onglet Intercos affiche pour chaque plateforme les palettes envoyées ↗ et reçues ↙, jour par jour.
