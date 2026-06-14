# Statuts & RAZ

## Suivi par statut

Chaque étape de la méthodologie possède une **case de statut** (à gauche du titre).
Un clic fait défiler quatre états :

| Icône | État | Sens | Rendu visuel |
|:-----:|------|------|--------------|
| `·` | **À faire** | état par défaut, non traité | neutre |
| `✓` | **Validé** | testé avec succès | bordure gauche **verte**, titre vert |
| `✗` | **Non valide** | testé sans succès / non exploitable | bordure gauche **rouge**, titre rouge |
| `∅` | **Non concerné** | sans objet pour cette box (pas besoin / hors périmètre) | ligne **grisée et barrée** |

Le cycle au clic est : **à faire → validé → non valide → non concerné → à faire**.
Le libellé du prochain état apparaît dans l'infobulle de la case.

### Pourquoi quatre états ?

Une simple case « fait / pas fait » ne distingue pas *« j'ai réussi »* de *« j'ai essayé,
ça ne marche pas »* ou *« ça ne me concerne pas »*. Ces trois situations n'ont pas le même
sens dans un rapport ni dans le suivi d'avancement.

### Impact sur la progression

- Les compteurs par section et le pourcentage global comptent les étapes **validé**.
- Les étapes **non concerné** sont **exclues du dénominateur** : le pourcentage reflète donc
  l'avancement sur ce qui est réellement pertinent pour la box.
- Un suffixe `· N n/c` indique combien d'étapes ont été marquées « non concerné ».

### Dans le rapport et la sauvegarde

- Le **Rapport .md** préfixe chaque étape documentée par son statut
  (`[✓ validé]`, `[✗ non valide]`, `[∅ n/c]`).
- Les statuts sont inclus dans l'**Export JSON** et cloisonnés **par cible** (IP).
- **Compatibilité** : un ancien carnet (case unique « fait ») est migré automatiquement -
  chaque case cochée devient **validé**.

## RAZ - changer de box

Le bouton **RAZ** (barre d'outils, à côté de *Thème*) prépare le passage à une nouvelle cible.
Après confirmation, il :

- **vide** les champs de la cible : IP, Domaine, DC, Utilisateur, Mot de passe, URL, et la recherche ;
- **réinitialise** le contexte « Ce que je possède » ;
- **retire** le filtre nmap (ports ouverts importés).

Il **conserve** :

- les champs côté attaquant : **LHOST, Interface, Wordlist** (identiques d'une box à l'autre) ;
- **toutes les notes et la progression** - elles sont stockées **par IP cible**, donc l'ancienne
  box reste accessible : il suffit de ressaisir son IP pour retrouver son carnet.

!!! tip "Astuce"
    La RAZ ne supprime rien d'enregistré : c'est un simple « repartir propre » sur le formulaire.
    Pour repartir totalement de zéro sur tout l'outil, utilisez plutôt un nouveau profil de
    navigateur ou videz le `localStorage`.
