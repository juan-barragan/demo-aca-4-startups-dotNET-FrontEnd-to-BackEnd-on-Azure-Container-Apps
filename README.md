# Front-end ASP.NET Core + 2 API back-end sur Azure Container Apps

Ce dépôt contient un scénario simple construit pour démontrer comment ASP.NET Core 6.0 peut être utilisé pour créer une application cloud-native hébergée dans Azure Container Apps. Le dépôt se compose des éléments suivants :

* Store - Un projet serveur Blazor représentant le frontend d'une boutique en ligne. L'interface utilisateur de la boutique affiche une liste de tous les produits de la boutique et leur statut d'inventaire associé.
* Products API - Une API simple qui génère des noms de produits fictifs en utilisant le package open-source NuGet [Bogus](https://github.com/bchavez/Bogus).
* Inventory API - Une API simple qui fournit un nombre aléatoire pour un identifiant de produit donné. Les valeurs de chaque paire chaîne/entier sont stockées en cache mémoire pour être cohérentes entre les appels API.
* Dossier Azure - contient les fichiers Azure Bicep utilisés pour créer et configurer toutes les ressources Azure.
* Fichier de workflow GitHub Actions utilisé pour déployer l'application en utilisant CI/CD.

## Ce que vous apprendrez

Cet exercice vous présentera une variété de concepts, avec des liens vers la documentation de support tout au long du tutoriel.

* [Azure Container Apps](https://docs.microsoft.com/azure/container-apps/overview)
* [GitHub Actions](https://github.com/features/actions)
* [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/)
* [Azure Bicep](https://docs.microsoft.com/azure/azure-resource-manager/bicep/overview?tabs=**bicep**)

## Prérequis

Vous aurez besoin d'un abonnement Azure et d'un très petit ensemble d'outils et de compétences pour commencer :

1. Un abonnement Azure. Inscrivez-vous [gratuitement](https://azure.microsoft.com/free/).
2. Un compte GitHub, avec accès à GitHub Actions.
3. Soit le [CLI Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) installé localement, soit un accès à [GitHub Codespaces](https://github.com/features/codespaces), ce qui vous permettrait de...

## Diagramme de topologie

L'application résultante est un ensemble de conteneurs hébergés dans un environnement Azure Container - l'API `products`, l'API `inventory` et le frontend Blazor Server `store`.

![Application topology](docs/media/topology.png)

Le trafic Internet ne doit pas pouvoir accéder directement aux API back-end car chacun de ces conteneurs est marqué comme "internal ingress only" pendant la phase de déploiement. Le trafic Internet...

## Configuration

À la fin de cette section, vous aurez une application à 3 nœuds fonctionnant dans Azure. Ce processus de configuration se compose de deux étapes et devrait vous prendre environ 15 minutes.

1. Utilisez le CLI Azure pour créer un principal de service Azure, puis stockez la sortie JSON de ce principal dans un secret GitHub afin que le processus CI/CD de GitHub Actions puisse se connecter à votre abonnement Azure et...
2. Modifiez le fichier de workflow `deploy.yml` et poussez les modifications dans une nouvelle branche `deploy`, déclenchant GitHub Actions pour construire les projets .NET dans des conteneurs et pousser ces conteneurs dans un nouveau...

## Authentification à Azure et configuration du dépôt avec un secret

1. Forkez ce dépôt dans votre propre organisation GitHub.
2. Créez un principal de service Azure en utilisant le CLI Azure.

```bash
$subscriptionId=$(az account show --query id --output tsv)
az ad sp create-for-rbac --sdk-auth --name WebAndApiSample --role owner --scopes /subscriptions/$subscriptionId
