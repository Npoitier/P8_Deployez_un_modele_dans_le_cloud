# P8_Deployez_un_modele_dans_le_cloud
Le projet s'appuie sur un service AWS s3 et a vocation a être déployé sur une machine AWS EC2.
Il a pour but de fournir un exemple d'utilisation de pyspark dans le cloud
## Création de compte Amazon Web Services
Attention! La création d'un compte [aws](https://aws.amazon.com/fr/premiumsupport/knowledge-center/create-and-activate-aws-account/) nécessite de renseigner "un mode de paiement valide" contenant à minima 1$ de solde ! 
## Configuration s3
Accéder à la [AWS Management Console](https://s3.console.aws.amazon.com/s3/get-started?region=eu-west-3&region=eu-west-3)<br>
Créer un bucket S3 avec la configuration suivante :<br>
- bucket name: "nom de bucket" <br>
- AWS region: EU (Paris) eu-west-3 <br>
- Object ownership: ACLs disabled <br>
- Block all public access <br>
- Bucket versioning: Disable <br>
- No tags <br>
- Server-side encryption: Disable <br>
## Ajout d'un utilisateur via IAM
Accéder à la console [IAM](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1&skipRegion=true#/home)
- Cliquer sur "utilisateurs" dans le menu de gauche
- Cliquer sur "Ajouter des utilisateurs" (Bouton dans le nouvel écran)
- Remplir le "Nom d'utilisateur" qui sera l'administrateur AWS
- Choisir "Clé d'accès - Accès par programmation" (cette clé va nouspermettre un accès par programme)
- Cliquer sur "suivant autorisation"
- Choisir "Attacher directement les stratégies existantes"
- Choisir "AdministratorAccess"
- Cliquer sur "suivant balises"
- Cliquer sur "suivant vérification"
- Cliquer sur "Créer un utilisateur"
- Téléchargez.csv et prenez en note "ID de clé d'accès\n" et "Clé d'accès secrète\n" dans un key.txt (mettez le tout dans un dossier AWS de votre ordi)
## Traitement et chargement des données brutes dans s3
Installez un environnement Notebook avec anaconda sur votre ordinateur via [https://sparkbyexamples.com/pyspark/install-pyspark-in-anaconda-jupyter-notebook/](https://sparkbyexamples.com/pyspark/install-pyspark-in-anaconda-jupyter-notebook/) <br>
Créer un favori vers [https://github.com/spark-examples/pyspark-examples](https://github.com/spark-examples/pyspark-examples), ça vous sera utile par la suite pour vos propres développements <br>
- Télécharger les données [https://www.kaggle.com/datasets/moltean/fruits](https://www.kaggle.com/datasets/moltean/fruits)
- télécharger le fichier "Traitement_des_donnees.ipynb" que vous trouverez dans le dossier source et placez le dans le dossier local des notebook
- copier le fichier key.txt que vous avez mis dans le dossier AWS de votre ordinateur dans le dossier local des notebook
- Ouvrez et éxécutez le fichier "Traitement_des_donnees.ipynb" que vous trouverez dans le dossier source (La première partie permet de supprimer les espaces dans votre arborescence, si si c'est utile pour certaines fonctions de spark. La seconde partie à charger l'arborescence sur votre bucket)
## Configuration EC2
## Traitement et sauvegarde des résultats
