# Labo pratique 1 : configurer les connexions pour votre connecteur Microsoft Graph

## Locataires WWL - Conditions d’utilisation

Si un locataire vous est fourni dans le cadre d’une formation dispensée par un instructeur, notez qu’il est mis à votre disposition pour prendre en charge les labos pratiques de la formation.

Vous ne devez ni partager ni utiliser les locataires en dehors des labos pratiques. Le locataire utilisé dans ce cours est un locataire d’essai. Au terme de la classe, le locataire ne pourra pas faire l’objet d’une prolongation et vous ne pourrez plus l’utiliser ni y accéder.

Vous n’êtes pas autorisé à convertir un locataire en abonnement payant. Les locataires obtenus dans le cadre de ce cours sont la propriété de Microsoft Corporation. Nous nous réservons le droit d’y accéder et d’en reprendre possession à tout moment.

## Résumé

Dans ce labo, vous allez utiliser le centre d’administration Microsoft 365 pour créer une connexion aux fichiers client à l’aide du connecteur du partage de fichiers Microsoft.

Imaginez que vous êtes un responsable du service clientèle chez Contoso, une entreprise de taille moyenne. Votre équipe a rencontré des difficultés avec les temps de réponse et l’efficacité globale lors de la gestion des demandes des clients. Vous décidez de tirer profit du centre d’administration Microsoft 365 pour créer une connexion FileShare qui permet à vos agents du service clientèle d’accéder rapidement aux fichiers client et de les référencer.

## Exercice 1 : configurer les connexions dans le centre d’administration Microsoft 365

### Se connecter à la machine virtuelle

Votre fournisseur d’hébergement de labo fournit un mot de passe pour le compte administrateur MOD, qui est l’administrateur de locataire par défaut. À des fins de sécurité, Microsoft a configuré votre locataire d’évaluation afin que tous les utilisateurs prédéfinis puissent modifier leur mot de passe lors de leur prochaine connexion. Connectez-vous à **LON-CL1** en tant que le compte **Administrateur** local créé par le fournisseur d’hébergement de votre labo avec le mot de passe **Pa55w.rd**.

### Tâche 1 : accorder des autorisations dans le portail Azure

1. Accédez au portail Azure **https://www.portal.azure.com** et connectez-vous à l’aide de vos informations d’identification d’administration. N’enregistrez pas le mot de passe, puis sélectionnez **Oui** pour **Rester connecté ?**.
2. Dans l’écran **Bienvenue dans Microsoft Azure**, sélectionnez **Annuler**.
1. Sélectionnez l’icône de menu déroulant sur le côté gauche de l’écran pour afficher le menu Portail. Sélectionnez **Microsoft Entra ID -> Gérer -> Inscriptions d’applications**.
1. Dans la barre des menus supérieure, sélectionnez **Nouvelle inscription**. La page **Inscrire une application** s’affiche. Dans cette page, indiquez un nom pour l’application ; nommez cette application **Contoso Fileshare**. Laissez l’option des types de comptes pris en charge sur **Comptes dans ce répertoire d’organisation uniquement (Contoso uniquement-locataire unique)**. Ne sélectionnez pas d’**URI de redirection** facultatif.
1. Sélectionnez **Inscrire**. Votre application est créée et un ID d’application lui est attribué. Vous utiliserez ces informations au moment de créer votre agent de connecteur Graph (GCA, Graph Connector Agent) dans les étapes suivantes. Avant de créer le GCA, nous allons toutefois configurer les paramètres nécessaires.

L’étape suivante consiste à accorder des autorisations pour l’agent de connecteur Graph dans le portail Azure :

1. Dans le menu de gauche, sélectionnez **Gérer -> Autorisations d’API**.
1. Sélectionnez **Ajouter une autorisation -> Microsoft Graph -> Autorisations d’application** et autorisez les autorisations aux API suivantes :

    - Directory -> Directory.Read.All
    - ExternalConnection -> ExternalConnection.ReadWrite.OwnedBy
    - ExternalItem -> ExternalItem.ReadWrite.All
      
1. Sélectionnez le bouton **Ajouter des autorisations**.
1. Sélectionnez **Accorder le consentement administrateur pour Contoso** et confirmez en sélectionnant **Oui**.

**Remarque :** ne fermez pas cet onglet de navigateur Edge. Les tâches suivantes nécessitent que vous copiez et collez des informations à partir du portail Azure.

### Tâche 2 : installer le GCA

1. Ouvrez un nouvel onglet de navigateur Microsoft Edge. Accédez à l’URL suivante pour télécharger l’agent du connecteur Graph : **https://www.microsoft.com/en-us/download/details.aspx?id=104045**. Cliquez sur le bouton **Télécharger**. 
1. Ouvrez le fichier **GcaInstaller_3.1.1.0.msi** et suivez les invites de l’Assistant Installation. 
2. Dans la **barre de recherche** située en bas de l’écran, saisissez la **configuration de l’agent du connecteur Graph** et sélectionnez l’application dans le menu lorsqu’elle apparaît.
3. Autorisez l’application à apporter des modifications à l’appareil en sélectionnant **Oui**.
4. Connectez-vous et inscrivez le GCA à l’aide du compte **administrateur MOD**. Une confirmation que l’authentification est terminée s’affiche dans le navigateur Edge. Vous pouvez fermer cette fenêtre.
5. Ouvrez l’application GCA en sélectionnant l’icône en bas de l’écran.
1. Nommez cet agent **ContosoFiles**.
1. Sélectionnez l’onglet Microsoft Edge du portail Azure (il devrait indiquer **Contoso Fileshare - Microsoft Azure**), accédez à l’écran **Vue d’ensemble** et copiez l’**ID d’application (client)**. Collez-le dans l’application d’installation GCA.

Ensuite, vous devez définir une clé secrète client pour cette application dans le portail Azure.

1. Revenez à l’onglet Edge du portail Azure et accédez à **Certificats & secrets**.
1. Sélectionnez **+ Nouveau secret client**. Ajoutez une **description** en saisissant **Fichiers Contoso**. Vous pouvez laisser le champ d’expiration défini sur les 180 jours par défaut.
2. Sélectionnez **Ajouter**.
3. Copiez le champ **Valeur** du secret silencieux.
1. Revenez à l’application de l’agent du connecteur Graph et collez la valeur dans le champ **Mot de passe de l’application (clé secrète client)** de l’application d’installation GCA.
1. Sélectionnez **Inscrire**.
1. Une fois l’inscription terminée, fermez l’application d’installation.

### Tâche 3 : télécharger les fichiers de ressources à partir de Github

Pour configurer le connecteur à l’aide du GCA, vous aurez besoin de fichiers locaux sur votre système. 

1. Ouvrez un nouvel onglet Edge et saisissez **https://github.com/MicrosoftLearning/MS-4014-Build-a-foundation-to-extend-Microsoft-365-Copilot/tree/master/ResourceFiles** dans la barre d’adresse.
2. Sélectionnez le premier fichier dans le dossier **Contoso Chai Tea market trends 2023.xls** pour l’ouvrir.
3. Sélectionnez les **points de suspension (autres actions de fichier)** en haut à droite de l’écran, puis **Télécharger**.
4. Répétez l’étape 3 pour chacun des fichiers restants.
5. Vous pouvez fermer cette fenêtre une fois les téléchargements terminés.
6. Utilisez l’explorateur de fichiers et accédez au répertoire C:\. Créez un dossier nommé **ResourceFiles**.
7. Ouvrez une nouvelle fenêtre d’explorateur de fichiers et accédez au dossier **Téléchargements** pour accéder aux fichiers Contoso que vous avez téléchargés à partir de GitHub. Copiez ces fichiers dans le répertoire **C :\ResourceFiles**.

Vous utiliserez ces fichiers comme contenu source pour la connexion que vous créez dans le centre d’administration Microsoft.

### Tâche 4 : ouvrir le centre d’administration Microsoft

1. Ouvrez un nouvel onglet dans le navigateur Microsoft Edge. Dans votre navigateur Microsoft Edge, accédez à la page **Accueil Microsoft 365** en saisissant l’URL suivante dans la barre d’adresse : **https://portal.office.com**.
1. Si vous êtes invité à vous connecter, saisissez le **nom d’utilisateur** LON-CL1 et le **mot de passe** fournis par votre fournisseur d’hébergement de labo pour votre locataire d’évaluation Microsoft 365. Le nom d’utilisateur doit se présenter sous la forme **<admin@xxxxxZZZZZZ.onmicrosoft.com>**, où xxxxxZZZZZZ est le préfixe de locataire attribué par le fournisseur d’hébergement de votre labo. 
1. La page **Bienvenue dans Microsoft 365** s’affiche dans votre navigateur Microsoft Edge sous l’onglet **Accueil | Microsoft 365**. Cette page est la page d’accueil Microsoft 365 de l’administrateur MOD.
1. Dans la liste des icônes d’application qui s’affiche dans le volet de navigation, sélectionnez **Administrateur** ; cette action ouvre le **centre d’administration Microsoft 365** dans un nouvel onglet de navigateur.
1. Sélectionner **... Affichez tout** pour afficher le menu de navigation complet. Sélectionnez **Paramètres** -> **Recherche et intelligence.**
1. Sous l’onglet **Vue d’ensemble** qui s’affiche, faites défiler vers le bas. Sélectionnez **Ajouter votre première connexion** dans la zone **Connecteurs**.

À partir de là, une liste de sources de données disponibles est répertoriée. Nous configurons une connexion à l’aide du connecteur de partage de fichiers. Ce connecteur permet à la recherche Microsoft 365 Copilot de référencer les fichiers client au sein d’une recherche Copilot et offre des temps de réponse plus rapides et des réponses plus précises pour aider un agent du service clientèle à trouver des réponses.

1. Sélectionnez **Partage de fichiers**, puis **Suivant**.
1. Saisissez le **nom d’affichage**. Le nom d’affichage est un nom unique affiché aux utilisateurs. Saisissons **Fichiers Contoso** dans le champ.
1. Saisissez le **chemin du dossier**. Nous avons des fichiers d’exemple configurés dans le répertoire suivant :

   **C:\ResourceFiles**

Ce chemin est le répertoire dans lequel les fichiers sont stockés.

1. Sélectionnez le GCA que nous avons créé dans les étapes précédentes (ContosoFiles) dans le champ **Agent du connecteur Graph** s’il n’est pas l’option par défaut.
1. Sélectionnez **Windows** comme type d’authentification, puis saisissez le **nom d’utilisateur** LON-CLI et le **mot de passe** pour l’Authentification Windows.
1. Cochez la case pour autoriser Microsoft à créer un index de données tierces dans votre locataire.
1. Sélectionnez **Créer**.  Un écran de réussite s’affiche et votre connexion commence à se synchroniser. Vous pouvez saisir le type de contenu spécifique, les services de votre organisation qui peuvent tirer parti de son utilisation ou des exemples de workflows dans la zone **Description du connecteur**. Par exemple :

    **Ce connecteur comprend des informations contenues dans le serveur de partage de fichiers local. Il inclut des profils client et des questions client qui peuvent bénéficier au support technique pour répondre aux requêtes des clients.**
1. Cliquez sur **Terminé**. Votre connexion s’affiche désormais sous l’onglet **Recherche et intelligence** du centre d’administration Microsoft 365, et peut être référencée dans vos résultats de recherche ou lors de la création d’un agent Microsoft 365 Copilot.
