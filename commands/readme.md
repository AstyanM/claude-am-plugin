---
description: Analyse et améliore le README.md du projet
---

Tu dois améliorer le README.md de ce projet. Suis ces étapes dans l'ordre :

## Étape 1 — Collecte de données

Exécute ces commandes pour comprendre le projet :

```
cat package.json
git remote -v
cat LICENSE 2>/dev/null || echo "Pas de LICENSE"
ls -la .env.example 2>/dev/null && cat .env.example || echo "Pas de .env.example"
ls src/ 2>/dev/null || ls app/ 2>/dev/null || ls lib/ 2>/dev/null
```

## Étape 2 — Lecture du README actuel

Lis le README.md existant avec l'outil Read. S'il n'existe pas, passe directement à la génération.

## Étape 3 — Diagnostic

Compare le README existant avec la structure canonique du skill readme-editor. Identifie :
- Les sections manquantes
- Les informations obsolètes (versions, dépendances, scripts)
- Les liens cassés ou placeholders
- Les badges qui pourraient être ajoutés

Présente le diagnostic à l'utilisateur avant de modifier quoi que ce soit.

## Étape 4 — Édition

Après validation de l'utilisateur, applique les corrections avec des éditions ciblées (Edit tool). Ne remplace jamais le fichier entier sauf s'il est presque vide.

## Étape 5 — Vérification

Relis le README final et vérifie la quality checklist du skill.

$ARGUMENTS
