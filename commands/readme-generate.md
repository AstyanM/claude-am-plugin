---
description: Génère un README.md complet depuis le codebase
---

Tu dois générer un README.md complet pour ce projet en te basant sur le codebase réel.

## Étape 1 — Analyse complète du projet

Exécute toutes ces commandes :

```
cat package.json
git remote -v
cat LICENSE 2>/dev/null
cat .env.example 2>/dev/null
cat tsconfig.json 2>/dev/null | head -20
find src -type f -name "*.ts" -o -name "*.tsx" -o -name "*.astro" -o -name "*.vue" -o -name "*.svelte" 2>/dev/null | head -30
ls -R src/ 2>/dev/null | head -40
cat src/pages/index.astro 2>/dev/null || cat src/app/page.tsx 2>/dev/null || cat src/App.tsx 2>/dev/null | head -30
```

## Étape 2 — Plan

Propose un plan de README à l'utilisateur avec les sections que tu recommandes, basé sur :
- Le type de projet détecté (portfolio, lib, app, API, CLI...)
- L'audience probable (recruteurs, devs, utilisateurs finaux)
- Les features réelles trouvées dans le code

Attends la validation avant de rédiger.

## Étape 3 — Rédaction

Rédige le README en suivant strictement la structure canonique et les guidelines du skill readme-editor. Chaque information doit provenir du codebase — ne rien inventer.

## Étape 4 — Vérification

Vérifie la quality checklist. Présente le résultat final.

Si l'utilisateur a précisé des instructions : $ARGUMENTS
