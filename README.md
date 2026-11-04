# 🛒 Boutique Diayma

Application web de boutique en ligne en **ASP.NET Core MVC (C#)**, réalisée dans le cadre du **TD2 – C# et Technologies .NET, Visual Studio : Débogage** (ESP/UCAD, année 2026/2027, enseignant E. H. Ousmane Diallo).

| | |
|---|---|
| **Étudiante** | Mame Fama Fall, Master 2 GLSI (Soir) |
| **Dépôt GitHub** | [BoutiqueDiayma-FAMA](https://github.com/mamefama-rgb/BoutiqueDiayma-FAMA) |
| **Exécutable Windows** | [Lien Google Drive](https://drive.google.com/drive/folders/1BoK07kj5DxzqjBdmL9tXR36prQFo6TO4?usp=sharing) |

## Sommaire

1. [Récupération et exécution du code](#1-récupération-et-exécution-du-code)
2. [Projets de la solution](#2-projets-de-la-solution)
3. [Version du SDK .NET](#3-version-du-sdk-net)
4. [Installation du SDK](#4-installation-du-sdk)
5. [Dépôt GitHub](#5-dépôt-github)
6. [Bugs trouvés](#6-bugs-trouvés)
7. [Points d'arrêt](#7-points-darrêt)
8. [Parcours du code avant l'affichage des produits](#8-parcours-du-code-avant-laffichage-des-produits)
9. [Déploiement en exécutable Windows](#9-déploiement-en-exécutable-windows)
10. [Lien de l'exécutable](#10-lien-de-lexécutable)
11. [Partie optionnelle](#11-partie-optionnelle)

---

## 1. Récupération et exécution du code

Le code est récupéré avec `git clone` depuis le dépôt fourni, puis ouvert dans l'éditeur (`Diayma.sln`). Les avertissements affichés au premier chargement sont fermés : ils signalent que le framework cible est en fin de support.

```
git clone https://github.com/devehod/BoutiqueDiayma2025.git
```

## 2. Projets de la solution

> **Réponse :** la solution `Diayma.sln` contient **un seul projet**, `Diayma` (fichier `P2FixAnAppDotNetCode/Diayma.csproj`), une application ASP.NET Core MVC.

## 3. Version du SDK .NET

> **Réponse :** le projet cible **`netcoreapp2.0`** (.NET Core 2.0).

- Paquets principaux : `Microsoft.AspNetCore.All 2.0.6` et `Microsoft.Extensions.Localization 2.1.1`.
- Le SDK nécessaire est donc de la famille **2.x** ; le dernier disponible est le **2.1.818**.

## 4. Installation du SDK

> ** Version installée :** 2.1.818

## 5. Dépôt GitHub

Un dépôt personnel a été créé sur GitHub et le code y a été poussé :

```
git remote set-url origin https://github.com/mamefama-rgb/BoutiqueDiayma-FAMA.git
git branch -M main
git push -u origin main
```

## 6. Bugs trouvés

### 🐞 Bug 1 : le total du panier ignore les quantités

- **Symptôme** : avec 3 exemplaires d'un produit à 5 000, le total affiché est 5 000 au lieu de 15 000.
- **Cause** : dans `Models/Cart.cs`, `GetTotalValue()` additionnait `x.Product.Price` sans multiplier par `x.Quantity`.
- **Correction** :

```csharp
return GetCartLineList().Sum(x => x.Product.Price * x.Quantity);
```

`GetAverageValue()` a été corrigée de la même manière (total divisé par le nombre d'articles).

### 🐞 Bug 2 : mauvaise devise affichée

- **Symptôme** : les prix, en FCFA, s'affichaient avec « £ » en anglais et « € » en français.
- **Cause** : l'appel `ToString("c")` utilise le symbole monétaire de la culture courante.
- **Correction** : `ToString("c")` a été remplacé par `ToString("N0")` suivi du texte « FCFA » dans les vues `Product/Index`, `Cart/Index` et `Components/CartSummary/Default`.

### Autres anomalies repérées (non corrigées)

- Aucun contrôle du stock : on peut commander plus d'exemplaires qu'il n'y en a en stock, et le stock devient négatif.
- Le panier (`Cart`) est enregistré en singleton, donc partagé entre tous les visiteurs.

## 7. Points d'arrêt

| Fichier | Ligne | Code |
|---|---|---|
| `Components/CartSummaryViewComponent.cs` | 12 | `_cart = cart as Cart;` |
| `Controllers/ProductController.cs` | 15 | `_productService = productService;` |
| `Controllers/OrderController.cs` | 17 | `_cart = pCart;` |
| `Controllers/CartController.cs` | 15 | `_cart = pCart;` |
| `Startup.cs` | 20 | `Configuration = configuration;` |

Au chargement de la page d'accueil, seuls trois de ces points d'arrêt sont atteints : `Startup` (ligne 20), `ProductController` (ligne 15) et `CartSummaryViewComponent` (ligne 12). Les points d'arrêt de `CartController` et `OrderController` ne s'arrêtent que lorsqu'on clique sur « Ajouter au panier » ou « Commander ».


[Point d'arrêt dans Startup](docs/breakpoint-startup.png)


## 8. Parcours du code avant l'affichage des produits

### Parcours détaillé

| # | Namespace | Classe | Méthode | Mode | Justification |
|---|---|---|---|---|---|
| 1 | `P2FixAnAppDotNetCode` | `Program` | `Main` | Pas à pas principal (F10) | Point d'entrée. Le code est celui du modèle de projet, il n'y a rien à explorer. |
| 2 | `P2FixAnAppDotNetCode` | `Program` | `BuildWebHost` | Pas à pas principal (F10) | Construit l'hôte web avec `WebHost.CreateDefaultBuilder(args).UseStartup<Startup>().Build()`. Le code du framework n'est pas intéressant. |
| 3 | `P2FixAnAppDotNetCode` | `Startup` | `Startup(IConfiguration)` (constructeur) | Pas à pas détaillé (F11) | Premier arrêt (point d'arrêt ligne 20). La configuration est reçue par injection de dépendances. |
| 4 | `P2FixAnAppDotNetCode` | `Startup` | `ConfigureServices` | Pas à pas détaillé (F11) | Enregistre la localisation, `Cart` et `LanguageService` (singletons), `ProductService`, `ProductRepository`, `OrderService`, `OrderRepository` (transients), la session, MVC et les cultures `en`, `fr`. |
| 5 | `P2FixAnAppDotNetCode` | `Startup` | `Configure` | Pas à pas principal (F10) | Construit le pipeline HTTP : fichiers statiques, culture de la requête, session, puis la route par défaut `{controller=Product}/{action=Index}/{id?}`. |
| 6 | `P2FixAnAppDotNetCode.Models.Repositories` | `ProductRepository` | constructeur puis `GenerateProductData` | Pas à pas détaillé (F11), puis sortant (Shift+F11) | Créé par le conteneur d'injection avant le contrôleur. Génère les 9 produits de la boutique. On sort de `GenerateProductData` une fois la liste constatée. |
| 7 | `P2FixAnAppDotNetCode.Models.Repositories` | `OrderRepository` | constructeur | Pas à pas sortant (Shift+F11) | Également requis par `ProductService`. Il crée seulement une liste vide de commandes. |
| 8 | `P2FixAnAppDotNetCode.Models.Services` | `ProductService` | constructeur | Pas à pas principal (F10) | Reçoit les deux dépôts par injection. |
| 9 | `P2FixAnAppDotNetCode.Controllers` | `ProductController` | constructeur | Pas à pas détaillé (F11) | Deuxième point d'arrêt atteint (ligne 15). `ProductService` et `LanguageService` sont injectés. |
| 10 | `P2FixAnAppDotNetCode.Controllers` | `ProductController` | `Index` | Pas à pas détaillé (F11) | Action appelée par la route par défaut. Elle demande la liste des produits puis retourne la vue. |
| 11 | `P2FixAnAppDotNetCode.Models.Services` | `ProductService` | `GetAllProducts` | Pas à pas détaillé (F11) | Appelle le dépôt et convertit le tableau en `List<Product>`. |
| 12 | `P2FixAnAppDotNetCode.Models.Repositories` | `ProductRepository` | `GetAllProducts` | Pas à pas détaillé (F11) | Filtre les produits dont le stock est supérieur à 0 et les trie par nom. |
| 13 | `P2FixAnAppDotNetCode.Models` | `Cart` | constructeur | Pas à pas sortant (Shift+F11) | Singleton créé à la première demande, lors de l'affichage du panier dans la mise en page. |
| 14 | `P2FixAnAppDotNetCode.Components` | `CartSummaryViewComponent` | constructeur | Pas à pas détaillé (F11) | Troisième point d'arrêt (ligne 12). Le panier est réceptionné par injection. |
| 15 | `P2FixAnAppDotNetCode.Components` | `CartSummaryViewComponent` | `Invoke` | Pas à pas détaillé (F11) | Retourne la vue `CartSummary/Default.cshtml`. Le panier est vide, donc seul le bouton panier s'affiche. |
| 16 | `P2FixAnAppDotNetCode.Components` | `LanguageSelectorViewComponent` | `Invoke` | Pas à pas principal (F10) | Affiche le sélecteur de langue dans le pied de page. |

Après l'étape 16, la page `Views/Product/Index.cshtml`, avec la mise en page `Views/Shared/_Layout.cshtml`, est envoyée au navigateur et la liste des produits s'affiche.

### Choix du mode de pas à pas

- **Pas à pas détaillé (F11)** : utilisé pour entrer dans le code de l'application, là où l'on veut comprendre ce qui s'exécute (`Startup`, contrôleur, services).
- **Pas à pas principal (F10)** : utilisé pour passer par-dessus une méthode sans y entrer, quand son contenu est connu ou provient du framework (`Program`, `Configure`).
- **Pas à pas sortant (Shift+F11)** : utilisé pour quitter une méthode une fois l'information utile obtenue, ou quand on se retrouve dans du code du framework.

### Remarque

Le panier (`Cart`) est enregistré en singleton. Il est donc partagé entre tous les visiteurs de la boutique.

## 9. Déploiement en exécutable Windows

Publication autonome pour Windows 64 bits :

```
dotnet publish -c Release -r win-x64 --self-contained true -o publish-FAMA
```

Le dossier `publish-FAMA` contient `Diayma.exe`, qui lance le serveur web sur `http://localhost:5000`.

[Publication de l'exécutable](docs/publish-executable.png)

Le profil `Properties/PublishProfiles/FolderProfile.pubxml` (`win-x64`, autonome) peut aussi être utilisé depuis l'éditeur, après avoir changé le dossier de destination.

## 10. Lien de l'exécutable

Le dossier publié a été compressé et déposé ici : [Télécharger l'exécutable (Google Drive)](https://drive.google.com/drive/folders/1BoK07kj5DxzqjBdmL9tXR36prQFo6TO4?usp=drive_link)

Après décompression, lancer `Diayma.exe` depuis le dossier décompressé.

## 11. Partie optionnelle

### a) Langue d'affichage : le wolof

- `Startup.cs` : ajout de la culture `wo` à la liste des cultures prises en charge, en conservant `fr` et `fr-FR`.
- `Models/Services/LanguageService.cs` : nouveau cas `"Wolof"` qui retourne la culture `wo`.
- `Views/Shared/Components/LanguageSelector/Default.cshtml` : nouvelle entrée « Wolof » dans le sélecteur.
- Dossier `Resources` : un fichier `*.wo.resx` pour chacun des 9 fichiers `*.fr.resx`, avec les textes traduits.

### b) Commits significatifs

1. Correction du total du panier et de l'affichage des prix en FCFA
2. Ajout de la langue wolof
3. README : parcours de débogage de l'étape 8
