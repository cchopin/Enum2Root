# Démarrage rapide

1. Ouvrez `index.fr.html` dans un navigateur récent (double-clic).
2. Renseignez les champs de la cible en haut (au minimum l'**IP**).
3. Cochez ce que vous possédez dans la barre **« Ce que je possède »**.
4. Déroulez la méthodologie ; copiez les commandes avec le bouton **copier**
   (votre cible est déjà substituée).

## Flux typique avec un scan

```bash
mkdir -p scans && sudo nmap -p- -sV -sC -T4 -oA scans/10.10.10.10 10.10.10.10
```

Puis cliquez sur **Charger .nmap**, choisissez `scans/10.10.10.10.nmap`, et la page
(ainsi que le graphe) se réduisent aux seuls services réellement ouverts.

## Changer de box

Quand vous passez à une nouvelle cible, cliquez sur **RAZ** : les champs de la cible, le
contexte et le filtre nmap sont vidés. Vos notes et votre progression restent conservées,
cloisonnées par IP. Voir [Statuts & RAZ](statuts-et-raz.md).

## Vie privée

Tout s'exécute localement dans le navigateur. Notes, progression et réglages vivent uniquement
dans le `localStorage` - rien n'est envoyé nulle part. Le seul trafic sortant survient si
**vous** cliquez sur un lien de référence externe.
