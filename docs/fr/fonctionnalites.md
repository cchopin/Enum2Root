# Fonctionnalités

**Champs de cible dynamiques.** Les champs du header — IP, Domaine/FQDN, DC, Utilisateur,
Mot de passe/hash, **Votre IP (LHOST/listener)**, Interface, Wordlist, URL web — sont substitués
en direct dans toutes les commandes (affichés en jetons surlignés).

**Système de prérequis (« Ce que je possède »).** Chaque action déclare ce dont elle a besoin :
rien, un nom d'utilisateur, des identifiants valides, un hash NT, un admin local, un shell, ou
des privilèges de domaine élevés. Les badges passent en vert (acquis) ou rouge (manquant).

**Actions indisponibles — Tout afficher / Réduire / Masquer.** Décide quoi faire des actions non
encore réalisables : visibles mais grisées, **réduites** à une ligne (par défaut), ou masquées.

**Mode Focus.** Un clic filtre toute la page selon votre contexte : les commandes faisables sont
surlignées, celles dont les prérequis manquent se replient, et les sections verrouillées disparaissent.

**Import nmap.** Chargez un scan `.nmap`, `.gnmap` ou `.xml` : la page ne garde que les sections
des ports ouverts, récupère l'IP/le host, et le graphe se filtre aussi.

**Navigation repliable.** Chevrons par colonne, phases repliables individuellement, lien
**« + N masquées »** par phase pour révéler à la demande.

**Recherche.** Filtre en direct sur les outils, ports et techniques ; déplie ce qui correspond.

**Carnet de bord par commande.** Cliquez le crayon d'une commande pour coller et enregistrer sa
sortie. Entrées horodatées, conservées en historique, cloisonnées **par cible** (l'IP).

**Vues du carnet — Commandes / Timeline / Findings.** Marquez une entrée comme **finding** avec ★.

**Suivi par statut & progression.** Statut à plusieurs états par étape
(à faire / validé / non valide / non concerné). Détails : [Statuts & RAZ](statuts-et-raz.md).

**RAZ (changer de box).** Vide la cible, le contexte et le filtre nmap pour repartir sur une
nouvelle box, en conservant notes et progression. Détails : [Statuts & RAZ](statuts-et-raz.md).

**Rapports & sauvegarde.** **Rapport .md** exporte un rapport Markdown de la cible (avec une
section `## Findings` en tête et le statut de chaque étape documentée). **Export/Import JSON**
sauvegarde et restaure tout le carnet (notes + findings + progression).

**Liens HackTricks.** Chaque section pertinente a un bouton **HackTricks ↗**.

**Vue graphique.** Graphe force-directed façon Obsidian : cible → phases → sections, coloré selon
la disponibilité. Molette = zoom, glisser = déplacer, clic = aller à la section.

**Thème clair/sombre.** Bascule mémorisée entre les sessions.
