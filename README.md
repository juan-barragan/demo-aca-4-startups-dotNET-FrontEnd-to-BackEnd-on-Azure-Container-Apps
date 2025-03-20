# Front-end ASP.NET Core + 2 API back-end sur Azure Container Apps

Ce dépôt contient un scénario simple construit pour démontrer comment ASP.NET Core 6.0 peut être utilisé pour créer une application cloud-native hébergée dans Azure Container Apps. Le dépôt se compose des éléments suivants :

* Store - Un projet serveur Blazor représentant le frontend d'une boutique en ligne. L'interface utilisateur de la boutique affiche une liste de tous les produits de la boutique et leur statut d'inventaire associé.
* Products API - Une API simple qui génère des noms de produits fictifs en utilisant le package open-source NuGet [Bogus](https://github.com/bchavez/Bogus).
* Inventory API - Une API simple qui fournit un nombre aléatoire pour un identifiant de produit donné. Les valeurs de chaque paire chaîne/entier sont stockées en cache mémoire pour être cohérentes entre les appels API.
* Dossier Azure - contient les fichiers Azure Bicep utilisés pour créer et configurer toutes les ressources Azure.
* Fichier de workflow GitHub Actions utilisé pour déployer l'application en utilisant CI/CD.

## Ce que vous apprendrez

Cet exercice vous présentera une variété de concepts, avec des liens vers la documentation tout au long du tutoriel.

* [Azure Container Apps](https://docs.microsoft.com/azure/container-apps/overview)
* [GitHub Actions](https://github.com/features/actions)
* [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/)
* [Azure Bicep](https://docs.microsoft.com/azure/azure-resource-manager/bicep/overview?tabs=**bicep**)

## Prérequis

Vous aurez besoin de :

1. Un abonnement Azure.
2. Un compte GitHub, avec accès à GitHub Actions.
3. Soit le [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) installé localement, ou bien un accès a Azure Cloud Shell depuis le portal Azure.

## Diagramme de topologie

L'application est un ensemble 3 conteneurs hébergés dans un environnement Azure Container Apps - l'API `products`, l'API `inventory` et le frontend Blazor Server `store`.

![Application topology](docs/media/topology.png)

Le trafic Internet ne doit pas pouvoir accéder directement aux API back-end car chacun de ces conteneurs est marqué comme "internal ingress only" pendant la phase de déploiement.

Le trafic Internet accédant à l'URL `store.<your app>.<your region>.azurecontainerapps.io` doit être acheminé vers le conteneur `frontend`, qui à son tour effectue des appels interne, au sein de l'environnement Azure Container Apps, vers les API `products` et `inventory`.

## Configuration

À la fin de cette section, vous aurez une application contenant 3 nœuds fonctionnant dans Azure Container Apps. 
Cette application utilise un modèle de deploiement de type `consumption` qui est `serverless`


Ce processus de configuration se compose de deux étapes et devrait vous prendre environ 15 minutes.

1. Utilisez le CLI Azure pour créer `Azure Service Principal`, puis stockez la définition JSON de ce principal dans un secret GitHub afin que le processus CI/CD de GitHub Actions puisse se connecter à votre souscription Azure et déployer le code.
2. Modifiez le fichier de workflow `deploy.yml` et poussez les modifications dans une nouvelle branche `deploy`, ce qui déclenchera GitHub Actions pour créer l'environnement Container Apps, construire des containers pour les projets .NET  et pousser ces conteneurs l'environnement Container Apps.

## Authentification à Azure et configuration du dépôt avec un secret

1. "Forkez" ce dépôt dans votre propre organisation GitHub.
2. Créez un `Azure Service Principal` en utilisant le `Azure CLI`.

```bash
$subscriptionId=$(az account show --query id --output tsv)
az ad sp create-for-rbac --sdk-auth --name WebAndApiSample --role owner --scopes /subscriptions/$subscriptionId
```

3. Copiez le JSON écrit à l'écran dans votre presse-papiers.

```json
{
  "clientId": "",
  "clientSecret": "",
  "subscriptionId": "",
  "tenantId": "",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com/",
  "resourceManagerEndpointUrl": "https://brazilus.management.azure.com",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com",
  "managementEndpointUrl": "https://management.core.windows.net"
}
```

4. Créez un nouveau secret GitHub dans votre fork nommé `AzureSPN`. Collez le JSON retourné par le CLI Azure dans ce nouveau secret.

   ![The AzureSPN secret in GitHub](docs/media/secrets.png)

Créez un deuxième secret GitHub dans votre fork nommé `AZURE_SUBSCRIPTION_ID`. Fournissez l'identifiant de la souscription Azure spécifique que vous souhaitez utiliser comme valeur pour ce secret. 
Une fois terminé vous verrez les 2 secrets

   ![The AzureSPN and subscription id secrets in GitHub](docs/media/secrets2.png)

Remarque : Ne jamais sauvegarder le JSON sur disque, car cela permettrait à quiconque obtenant ce code JSON de créer ou de modifier des ressources dans votre abonnement Azure.!!!

## Déployer le code en utilisant GitHub Actions

Le moyen le plus simple de déployer le code est de faire un commit directement dans la branche deploy. Faites-le en naviguant dans le fichier deploy.yml dans votre navigateur et en cliquant sur le bouton Edit.

![Modifier le fichier de deploiement du workflow.](docs/media/edit-the-deploy-file.png)

Fournissez un nom de groupe de ressources personnalisé pour l'application, puis validez la modification dans une nouvelle branche nommée deploy.

Créer la branche de déploiement.

Une fois que vous aurez cliqué sur le bouton Propose changes, vous serez en mode "créer une pull request". Ne vous inquiétez pas de créer la pull request pour le moment, cliquez simplement sur l'onglet Actions, et vous verrez que le déploiement...

Déploiement démarré.

Lorsque vous cliquez dans le workflow, vous verrez qu'il y a 3 phases que le CI/CD parcourra :

provision - les ressources Azure seront créées pour héberger votre application.
build - les différents projets .NET sont construits dans des conteneurs et publiés dans l'instance Azure Container Registry créée pendant la provision.
deploy - une fois build terminé, les images sont dans ACR, donc les Azure Container Apps sont mises à jour pour héberger les images de conteneurs nouvellement publiées.
Phases de déploiement.

Après quelques minutes, les trois étapes du workflow seront terminées, et chaque boîte dans le diagramme du workflow reflétera le succès. Si quelque chose échoue, vous pouvez cliquer dans les étapes du processus individuel...

Remarque : si vous voyez des échecs ou des problèmes, veuillez soumettre un problème afin que nous puissions mettre à jour l'exemple. De même, si vous avez des idées pour l'améliorer, n'hésitez pas à soumettre une pull request.

Déploiement réussi.

Avec les projets déployés sur Azure, vous pouvez maintenant tester l'application pour vous assurer qu'elle fonctionne.

Essayer l'application dans Azure
Le processus CI/CD deploy crée une série de ressources dans votre abonnement Azure. Celles-ci sont principalement utilisées pour héberger le code du projet, mais il y a également quelques ressources supplémentaires qui aident avec...

Resource	Type de ressource	Objectif
storeai	Application Insights	Cela fournit des informations de télémétrie et de diagnostic pour surveiller les performances de l'application ou pour...
store	Une Azure Container App qui héberge le code du frontend.	L'application store est l'application frontend de la boutique, exécutant un projet Blazor Server qui atteint les API back-end
products	Une Azure Container App qui héberge le code pour une API minimale.	Cette API est une API activée par Swagger UI qui renvoie des noms et des identifiants de produits aux appelants.
inventory	Une Azure Container App qui héberge le code pour une API minimale.	Cette API est une API activée par Swagger UI qui renvoie des quantités pour les identifiants de produits. Un client doit appeler les pr...
storeenv	Un environnement Azure Container Apps	Cet environnement sert de méta-conteneur de mise en réseau pour toutes les instances de toutes les applications de conteneurs...
storeacr	Un Azure Container Registry	C'est le registre de conteneurs dans lequel le processus CI/CD publie mes conteneurs d'application lorsque je...
storelogs	Log Analytics Workspace	C'est là que je peux exécuter des requêtes Kusto personnalisées contre...
Les ressources sont montrées ici dans le portail Azure :

Ressources dans le portail

Cliquez sur l'application de conteneur store pour l'ouvrir dans le portail Azure. Dans l'onglet Overview, vous verrez une URL.

Voir l'URL publique du store.

En cliquant sur cette URL, vous ouvrirez le frontend de l'application dans le navigateur.

La liste des produits, une fois l'application en cours d'exécution.

Vous verrez que la première demande prendra légèrement plus de temps que les demandes suivantes. Lors de la première demande de la page, les API sont appelées côté serveur. Le code utilise IMemoryCache pour...

Code

Vous pouvez trouver le fichier original [ici](https://github.com/stpapaix/demo-aca-4-startups-dotNET-FrontEnd-to-BackEnd-on-Azure-Container-Apps/blob/main/README.md).
stpapaix/demo-aca-4-startups-dotNET-FrontEnd-to-BackEnd-on-Azure-Container-Apps
traduit la ligne suivant en francais
Internet traffic hitting the `store.<your app>.<your region>.azurecontainerapps.io` URL should be proxied to the `frontend` container, which in turn makes outbound calls to both the `products` and `inventory` APIs within the Azure Container Apps Environment.
Le trafic Internet accédant à l'URL store.<votre app>.<votre région>.azurecontainerapps.io doit être acheminé vers le conteneur frontend, qui à son tour effectue des appels sortants vers les API products et inventory au sein de l'environnement Azure Container Apps.




