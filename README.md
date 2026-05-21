# patoine-studio-ci

Workflows GitHub Actions partagés pour la pratique Patoine Studio. Ce repo centralise la logique d'automatisation transversale aux repos clients : maintenance mensuelle, déploiement, vérifications de sécurité, et tout outillage qu'on veut maintenir à un seul endroit plutôt que dupliquer chez chaque client.

## Pourquoi ce repo

À mesure que Patoine Studio prend plusieurs clients en gestion mensuelle, dupliquer un workflow GitHub Actions identique dans chaque repo client crée un problème de cohérence. Une typo, une amélioration, un correctif de bug, et c'est cinq pull requests à faire. Ce repo héberge les **reusable workflows** invoqués par chaque repo client via une coquille de 20 à 30 lignes.

Modèle d'architecture : workflow par repo client comme aujourd'hui (préserve l'isolation, garde les snapshots dans le bon repo client, simplifie la fin de mandat), mais la logique d'exécution vit ici à un seul endroit, paramétrée par les variables propres à chaque client.

## Workflows disponibles

### `maintenance-mensuelle.yml`

Cycle de maintenance technique exécuté le premier lundi de chaque mois pour chaque client en gestion mensuelle. Vérifie le formulaire de contact via API Resend, fait un snapshot PageSpeed Insights, vérifie les liens brisés, audite les dépendances, et envoie un rapport au client (plus une alerte interne en cas de problème détecté).

Voir la note Playbook `Checklist de maintenance mensuelle` dans le vault Obsidian pour le contexte commercial et opérationnel complet.

## Comment ajouter un nouveau client

Quand un client bascule en gestion mensuelle, son repo `philippe-raymond`, `h2onet`, `urgence-calfeutrage`, etc. reçoit un workflow d'appel court qui ressemble à :

```yaml
name: Maintenance mensuelle

on:
  schedule:
    - cron: "0 13 * * 1"
  workflow_dispatch:
    inputs:
      mode:
        description: "Mode (test ou prod)"
        required: true
        default: "test"
        type: choice
        options: [test, prod]

permissions:
  contents: write

jobs:
  maintenance:
    uses: elvispat1/patoine-studio-ci/.github/workflows/maintenance-mensuelle.yml@main
    with:
      client_domain: domaine-du-client.ca
      client_label: Nom Complet du Site
      client_email: courriel-leads@client.com
      site_url: https://domaine-du-client.ca
      mode: ${{ inputs.mode || 'prod' }}
    secrets:
      RESEND_API_KEY: ${{ secrets.RESEND_API_KEY }}
      PAGESPEED_API_KEY: ${{ secrets.PAGESPEED_API_KEY }}
```

Les secrets `RESEND_API_KEY` et `PAGESPEED_API_KEY` doivent être configurés au niveau du repo client (Settings > Secrets and variables > Actions). Voir la note Playbook `Checklist de mise en gestion mensuelle` pour la procédure complète d'onboarding technique.

## Versionnement

Les workflows publiés sont consommés par les repos clients via une référence Git. Pour préserver la stabilité, on bascule sur des tags semver dès qu'on aura plus d'un client en gestion mensuelle. En V1 avec un seul client, on consomme `@main` directement.

## Convention de modification

Toute modification structurante d'un reusable workflow se teste d'abord sur une branche dédiée, en faisant pointer le caller du repo client sur cette branche temporairement (`@branche-test`). Validation live en mode test avant merge sur main.
