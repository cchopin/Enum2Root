# Fonctionnalités

Enum2Root tient dans un seul fichier HTML, mais couvre tout le cycle d'une mission : de la
saisie de la cible au suivi de l'avancement, en passant par le carnet de bord et l'export de
rapport. Tour d'horizon.

## Cible & contexte

<div class="grid cards" markdown>

-   :material-form-textbox: __Champs de cible dynamiques__

    IP, Domaine/FQDN, DC, Utilisateur, Mot de passe/hash, **Votre IP (LHOST)**, Interface,
    Wordlist, URL - substitués en direct dans toutes les commandes (jetons surlignés). Modifie
    un champ, tout se met à jour.

-   :material-key-chain: __Prérequis « Ce que je possède »__

    Chaque action déclare ce qu'elle exige (rien, un utilisateur, des identifiants, un hash, un
    admin local, un shell, des privilèges de domaine). Coche ce que tu détiens : les badges
    passent en vert (acquis) ou rouge (manquant).

-   :material-target: __Mode Focus__

    Un clic filtre toute la page selon ton contexte : les commandes faisables sont surlignées,
    celles dont les prérequis manquent se replient, les sections verrouillées disparaissent.

-   :material-eye-off: __Actions indisponibles__

    Choisis l'affichage des actions non réalisables : visibles mais grisées, **réduites** à une
    ligne (par défaut), ou masquées.

</div>

## Filtrage & navigation

<div class="grid cards" markdown>

-   :material-file-import: __Import nmap__

    Charge un scan `.nmap`, `.gnmap` ou `.xml` : la page ne garde que les sections des ports
    ouverts, récupère l'IP/host, et le graphe se filtre aussi.

-   :material-magnify: __Recherche live__

    Filtre instantané sur les outils, ports et techniques ; déplie automatiquement ce qui
    correspond.

-   :material-arrow-collapse-vertical: __Navigation repliable__

    Chevrons par colonne, phases repliables, lien **« + N masquées »** par phase pour révéler
    à la demande.

-   :material-graph: __Vue graphique__

    Graphe force-directed façon Obsidian : cible → phases → sections, coloré selon la
    disponibilité. Molette = zoom, glisser = déplacer, clic = aller à la section.

</div>

## Suivi & rapport

<div class="grid cards" markdown>

-   :material-checkbox-multiple-marked: __Suivi par statut__

    Statut à 4 états par étape (à faire / ✓ validé / ✗ non valide / ∅ non concerné), avec
    progression qui exclut les « non concerné ».

    [:octicons-arrow-right-24: Détails](statuts-et-raz.md)

-   :material-broom: __Bouton RAZ__

    Repartir proprement sur une nouvelle box (vide cible, contexte et filtre nmap) en
    conservant notes et progression.

    [:octicons-arrow-right-24: Détails](statuts-et-raz.md)

-   :material-notebook-edit: __Carnet de bord par commande__

    Colle et enregistre la sortie de chaque commande. Entrées horodatées, en historique,
    cloisonnées **par cible** (IP). Marque une entrée comme **finding** avec ★.

-   :material-file-export: __Rapports & sauvegarde__

    **Rapport .md** (avec section Findings + statut de chaque étape) ; **Export/Import JSON**
    de tout le carnet (notes + findings + progression).

</div>

## Confort

**Liens de référence.** Chaque section pertinente a un bouton **HackTricks ↗** ; les sections
d'attaques web ont en plus un bouton **Payloads ↗** vers PayloadsAllTheThings.

**Thème clair/sombre.** Bascule mémorisée entre les sessions ; les blocs de commande restent
sombres pour la lisibilité « terminal ».

**100 % local.** Notes, progression et réglages vivent uniquement dans le `localStorage` du
navigateur - rien n'est envoyé nulle part.
