# P8_Deployez_un_modele_dans_le_cloud
Le projet s'appuie sur un service AWS s3 et a vocation a être déployé sur une machine AWS EC2.
Il a pour but de fournir un exemple d'utilisation de pyspark dans le cloud
## 1.Création de compte Amazon Web Services
Attention! La création d'un compte [aws](https://aws.amazon.com/fr/premiumsupport/knowledge-center/create-and-activate-aws-account/) nécessite de renseigner "un mode de paiement valide" contenant à minima 1$ de solde ! 
## 2.Configuration s3
Accéder à la [AWS Management Console](https://s3.console.aws.amazon.com/s3/get-started?region=eu-west-3&region=eu-west-3)<br>
Créer un bucket S3 avec la configuration suivante :<br>
- bucket name: "nom de bucket" <br>
- AWS region: EU (Paris) eu-west-3 <br>
- Object ownership: ACLs disabled <br>
- Block all public access <br>
- Bucket versioning: Disable <br>
- No tags <br>
- Server-side encryption: Disable <br>
## 3.Ajout d'un utilisateur via IAM
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
- Téléchargez.csv et prenez en note "ID de clé d'accès", "Entrée" et "Clé d'accès secrète" dans un key.txt (mettez le tout dans un dossier AWS de votre ordi)
## 4.Traitement et chargement des données brutes dans s3
Installez un environnement Notebook avec anaconda sur votre ordinateur via [https://sparkbyexamples.com/pyspark/install-pyspark-in-anaconda-jupyter-notebook/](https://sparkbyexamples.com/pyspark/install-pyspark-in-anaconda-jupyter-notebook/) <br>
Créer un favori vers [https://github.com/spark-examples/pyspark-examples](https://github.com/spark-examples/pyspark-examples), ça vous sera utile par la suite pour vos propres développements <br>
- Télécharger les données [https://www.kaggle.com/datasets/moltean/fruits](https://www.kaggle.com/datasets/moltean/fruits)
- télécharger le fichier "Traitement_des_donnees.ipynb" que vous trouverez dans le dossier source et placez le dans le dossier local des notebook
- copier le fichier key.txt que vous avez mis dans le dossier AWS de votre ordinateur dans le dossier local des notebook
- Ouvrez, paramétrez et éxécutez le fichier "Traitement_des_donnees.ipynb" que vous trouverez dans le dossier source (La première partie permet de supprimer les espaces dans votre arborescence, si si c'est utile pour certaines fonctions de spark. La seconde partie à charger l'arborescence sur votre bucket)
## 5.Configuration EC2 [source https://github.com/nsaintgeours/sparkyfruit](https://github.com/nsaintgeours/sparkyfruit)
- Accèder au [service AWS EC2](https://eu-west-3.console.aws.amazon.com/ec2)
- Dans le menu de gauche, sélectionner **Instances / Instances**
- Cliquer sur le bouton orange en haut à droite : **Launch instances / Launch instances**
- **Démarrer une nouvelle instance EC2** avec les caractéristiques suivantes : 
	- name: "nom de bucket par exemple"
	- Amazon Machine Image (AMI): laisser le choix par défaut (*Amazon Linux 2 AMI (HVP) - Kernel 5.10 SSD volume type*), 64 bits
	- Instance type : **t2.medium**
	- Key pair (login) : cliquer sur **Generate a new key pair** et se laisser guider
	- Network settings : 
		- Firewall (security groups) : sélectionner "Create security group" avec la configuration par défaut suivante :
			- Allow SSH traffic from : Anywhere
			- Allow HTTPs traffic from the internet: No
			- Allow HTTP traffic from the internet: No
	- Configure storage : **30 GiB gp2 (root volume)**
	- Lancer l'instance (ça peut prendre une ou deux minutes)
- Une fois l'instance EC2 démarrée, on va noter dans un coin son **adresse IP publique**, qui nous permettra d'accèder au serveur virtuel depuis internet
	- Menu de gauche : sélectionner **Instances / Instances**
	- Dans la liste des instances, sélectionner l'instance que l'on vient de démarrer (case à cocher à gauche du nom de l'instance)
	- Les propriétés de l'instance EC2 apparaissent dans la moitié basse de la fenêtre
	- Dans l'onglet *Details*, noter la valeur de la **Public IPv4 address** (par exemple : `35.181.154.147`)
	- :warning: l'adresse publique de notre instance EC2 change à chaque arrêt redémarrage de l'instance ! :warning:
- Il nous faut ensuite **autoriser le flux réseau entrant sur le port 8888** pour pouvoir utiliser les notebooks Jupyter sur notre instance EC2. Pour ce faire, on suit les étapes suivantes : 
	- Menu de gauche, aller dans **Network & Security / Security Groups**
	- Sélectionner dans la liste le groupe de sécurité qui vient d'être créé, il s'appelle normalement `launch-wizard-1` (visible sur le récap instance EC2)
	- Les propriétés du groupe de sécurité apparaissent dans la moitié basse de la fenêtre : sélectionner l'onglet **Inbound rules**
	- Cliquer sur le **bouton "Edit inbound rules"**
	- Cliquer sur le bouton **Add rule** et ajouter une règle d'accès avec les caractéristiques suivantes : 
		- Type : *Custom TCP*
		- Protocol : *TCP*
		- Port range: *8888*
		- Source: *Anywhere ipv4*
	- Cliquer sur le bouton **Save rules**
	- C'est bon pour ce point !
- On peut maintenant **se connecter à l'instance EC2** : 
	- Menu de gauche : sélectionner **Instances / Instances**
	- Dans la liste des instances, sélectionner l'instance que l'on vient de démarrer (case à cocher à gauche du nom de l'instance)
	- Cliquer sur le **bouton "Connect"**
	- on arrive sur une nouvelle page *Connect to instance* : laisser les options par défaut et cliquer en bas sur le **bouton orange "Connect"**
	- une nouvelle page s'ouvre dans le navigateur web, avec **une console Linux** : ça y est, on est sur notre serveur virtuel !
## 6.Utiliser un notebook Jupyter sur le serveur virtuel EC2
- Depuis la **console Linux** ouverte au point 5, on suite les étapes suivantes pour installer Jupyter et ouvrir un notebook :
	- Python3 est déjà installé par défaut, on peut vérifier la version avec :	
	```
	[ec2-user@ip-172-31-33-35 ~]$ python3 --version
	Python 3.7.10
	```
	
	- installer Jupyter (à l'aide de `pip3`, le gestionnaire de packages Python): 
	
	```
	[ec2-user@ip-172-31-33-35 ~]$ pip3 install jupyter
	```
	
	- pour sécuriser l'accès aux notebooks Jupyter sur notre serveur virtuel, on configure un mot de passe (il ne s'affiche pas) :
	
	```
	[ec2-user@ip-172-31-33-35 ~]$ jupyter notebook password
	```
	
	- on lance Jupyter Notebook (avec des options utiles pour que ça fonctionne sur l'instance EC2) : 

	```
	[ec2-user@ip-172-31-33-35 ~]$ jupyter notebook --ip 0.0.0.0 --no-browser --allow-root
	```

- **Dans le navigateur web, accèder à l'URL suivante : `xx.xx.xx.xxx:8888`**, en remplaçant `xx.xx.xx.xxx` par l'adresse IP publique de l'instance EC2	
- Le serveur Jupyter Notebook s'ouvre, avec sécurisation de l'accès par le mot de passe que l'on a défini précédemment	
- Créer **un nouveau notebook**, l'ouvrir, et tester si tout fonctionne !

## 7.Accèder aux images stockées sur S3 depuis le notebook Jupyter

Pour que l'on puisse accéder aux données stockées sur S3 depuis notre notebook Jupyter, il faut que le serveur virtuel EC2 ait le droit d'accès à S3, en lecture. 
On suit les étapes suivantes pour lui donner ce droit d'accès : 

- Dans un navigateur web, accèder au [**service AWS IAM**](https://us-east-1.console.aws.amazon.com/iamv2)
	- Dans le menu de gauche, cliquer sur **Roles**
	- Cliquer sur le bouton **Create Roles** et créer un nouveau rôle avec les options suivantes :
		- Trusted Entity Type : laisser par défaut la valeur *"AWS Service"*
		- Use case : sélectionner *"EC2"*
	- Permission policies : rechercher **AmazonS3FullAccess**
	- Role name: ce que l'on veut, par exemple `s3_access`
	- Cliquer sur le bouton "Create Role"

- Accèder au [service AWS EC2](https://eu-west-3.console.aws.amazon.com/ec2)
	- Dans le menu de gauche, aller dans **Instances / Instances**
	- Sélectionner notre instance EC2 dans la liste des instances (case à cocher à gauche)
	- En haut, aller dans menu **Actions / Security / Modify IAM role**
	- Sélectionner le rôle *s3_access** que l'on vient de créer, puis cliquer sur le bouton **Update IAM Role**

- Pour vérifier que notre instance EC2 a maintenant accès à l'espace de stockage S3, on tape la commande suivante dans la **console Linux de notre instance EC2** : 

```
[ec2-user@ip-172-31-33-35 ~]$ aws s3 ls s3://sparkyfruit
```
--> on doit obtenir la liste des dossiers présents dans le bucket s3.

On va maintenant écrire du code dans le notebook Jupyter pour charger l'une des images présentes sur S3. 
Pour cela, on a besoin de deux packages Python : `boto3`, qui permet d'interagir avec S3, et `pillow`, qui permet de manipuler des images. 
On commence donc par installer ces deux packages sur notre instance EC2. 

- dans la **console Linux** de notre instance EC2 (on peut en lancer une nouvelle sans fermer la précédente depuis le **bouton orange "Connect"** de l'instance), taper :

	```
	[ec2-user@ip-172-31-33-35 ~]$ pip3 install boto3
	[ec2-user@ip-172-31-33-35 ~]$ pip3 install pillow
	```
		
- comme déjà expliqué au point 6, relancer le serveur Jupyter Notebook et ouvrir un nouveau notebook

- dans le notebook Jupyter, taper le bloc de code suivant permet de charger et d'afficher une des images stockées sur votre bucket S3 : 

	```
	import boto3
	import PIL.Image

	s3_bucket_name = "sparkyfruit"
	image_key = "apple_6/r1_216.jpg"

	s3 = boto3.client('s3')
	image = s3.get_object(Bucket=s3_bucket_name, Key=image_key)
	PIL.Image.open(image['Body'])
	```

Et voilà !


## 8.Installer PySpark et configurer son accès à S3

### 8.1. Installer PySpark

- Spark a besoin de **Java** pour tourner, on commence donc par installer **Java OpenJDK 11**, via la console Linux de notre instance EC2 :

```
[ec2-user@ip-172-31-33-35 ~]sudo amazon-linux-extras install java-openjdk11
```

- on installe ensuite **PySpark** à l'aide du gestion de package `pip3`. On choisit dans ce tutoriel d'installer la version `3.2.2` de PySpark :

```
[ec2-user@ip-172-31-33-35 ~]$ pip3 install pyspark==3.2.2
```


### 8.2. Configurer PySpark pour son accès à S3

Il nous faut configurer PySpark pour qu'il puisse accéder aux données stockées sur S3.
:warning: Cette partie est un peu pénible. :warning:

- Identifier le dossier dans lequel le **package `pyspark` a été installé**, et s'y déplacer : 

```
[ec2-user@ip-172-31-33-35 ~]$ cd /home/ec2-user/.local/lib/python3.7/site-packages/pyspark
```

> **_NOTE:_**  Si ce n'est pas le bon chemin de dossier, on peut rechercher le dossier avec la commande suivante : `find . -name pyspark`

- Se déplacer dans le dossier `/jars` qui contient tous les modules Java utiles à Pyspark

```
[ec2-user@ip-172-31-33-35 ~]$ cd jars
```

- Dans ce dossier, on commence par **vérifier la version du module `hadoop-client`** qui a été installée. Pour cela : 

```
[ec2-user@ip-172-31-33-35 ~]$ ls | grep hadoop-client
hadoop-client-api-3.3.2.jar
hadoop-client-runtime-3.3.2.jar
```

Sur mon instance, la version du module `hadoop-client` est la **`3.3.2`** (même numéro de version que `pyspark`, mais ça n'est pas forcément le cas !).

- toujours dans ce dossier, **on ajoute le module Java `hadoop-aws`** que l'on télécharge depuis internet, avec le numéro de version identifié précedement (ici `3.3.2`):

```
[ec2-user@ip-172-31-33-35 ~]$ wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.3.2/hadoop-aws-3.3.2.jar
```

- toujours dans ce dossier, **on ajoute le module Java `aws-java-sdk-bundle`** que l'on télécharge depuis internet, avec le numéro de version `1.12.246`:

```
[ec2-user@ip-172-31-33-35 ~]$ wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.12.246/aws-java-sdk-bundle-1.12.246.jar
```

> **_NOTE:_** Pour connaître la version de `aws-jdk-sdk-bundle` à installer, il faut regarder les dépendances de la librairie `hadoop-aws`. Ces
dépenances sont listés sur le site suivant : [https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-aws](https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-aws). Il suffit de sélectionner la bonne version d'`hadoop-aws` (ici `3.3.2`), puis de naviguer dans la page jusqu'à la section **Compiled dependencies**, où l'on trouve la librairie `com.amazonaws » aws-java-sdk-bundle` avec son numéro de version associé.

Et voilà pour cette partie ardue, c'est terminé !


### 8.3. Utiliser PySpark


- Depuis la **console Linux de notre instance EC2**, relancer le serveur Jupyter Notebook et ouvrir un notebook (voir point 6)
```
jupyter notebook --ip 0.0.0.0 --no-browser --allow-root
```
- Dans le notebook Jupyter, le bloc de code suivant permet de lancer une petite démo de PySpark : 

```
from pyspark import SparkContext
from pyspark.sql import SparkSession

sc = SparkContext()
sc.setLogLevel("ERROR")

spark = SparkSession(sparkContext=sc)
nums = spark.sparkContext.parallelize([1,2,3,4])
print(nums.map(lambda x: x*x).collect())
```

Si votre installation de PySpark fonctionne, vous obtenez :

```
[1, 4, 9, 16]
```
## 9.Traitement et sauvegarde des résultats
Ouvrez un nouveau Notebook <br>
copiez le contenu de toutes les cellules du fichier "P8_Deployez_un_modele_dans_le_cloud.ipynb" <br>
Depuis un console linux (cf point 5 le **bouton orange "Connect"** de l'instance EC2), tapez
```
pip3 install findspark
pip3 install pandas
pip3 install numpy
```
Configurez les variables
Ecrire la fonction de création du fichier key.txt depuis le notebook après les import
```
fichier = open("keys.txt", "w")
fichier.write("ID de clé d'accès\n")
fichier.write("Entrée" et "Clé d'accès secrète\n")
fichier.close()
```
Pensez à supprimer la cellule pour ne pas donner vos accés si vous mettez votre projet sur git !!!!

Rappel pour lancer jupiter depuis la console
```
jupyter notebook --ip 0.0.0.0 --no-browser --allow-root
```

