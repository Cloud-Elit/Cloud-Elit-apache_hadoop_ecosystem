# Ecosystème Hadoop

Hadoop a été développé dans le but de répondre aux besoins fondamentaux de la Big Data, à savoir le stockage distribué, le traitement distribué, le support de tout format de données et la mise à l’échelle. En pratique, cela n’est pas suffisant pour mener des grands projets Big Data. D’autres outils autour de la plateforme Hadoop ont été développés pour répondre à d’autres besoins applicatifs et architecturaux. De nombreux outils de l’écosystème Hadoop bénéficient du stockage distribué (HDFS) et de l’exécution distribuée (MapReduce et YARN) de la plateforme Hadoop.

La liste des outils de l’écosystème Hadoop est longue ; certains ont fait leurs preuves et continuent à évoluer et d’autres n’ont pas pu tenir pour longtemps. Dans cette section nous allons présenter brièvement une sélection des outils les plus utilisés aujourd’hui. Avant cela, nous allons présenter quelques architectures Big Data pour comprendre les relations entre ces outils. Le but étant d’avoir une idée claire sur le fonctionnement de l’écosystème et les interactions entre ses composants plutôt que de présenter un tutoriel complet pour chaque outil. De nombreux travaux dans la littérature, comme [Raj, 18], [Singh, 18], [Oussous, 17] et [Landset, 15], étudient l’écosystème Hadoop et tentent de présenter les avantages et les inconvénients de chaque outil ainsi que les différentes combinaisons possibles. 

## Architectures Big Data
Pour mieux expliquer le fonctionnement de l’écosystème Hadoop, nous allons commencer par présenter les architectures les plus appliquées dans les projets Big Data afin de montrer les différentes couches d’une application Big Data ainsi que les tâches dans chaque couche. Cela nous permettra ensuite de mieux comprendre l’utilité de chaque outil et son bon emplacement dans un projet Big Data. Pour cela, nous allons commencer par l’architecture la plus complète qui existe aujourd’hui, à savoir l’architecture "Lambda" qui a été présentée par Nathan Marz et James Warren dans leur livre [Marz, 15]. La Figure 1 illustre l’architecture Lambda.
<p align="center">
  <img src="https://github.com/Cloud-Elit/Cloud-Elit-apache_hadoop_ecosystem/assets/142179779/3aeace89-9060-4459-a9b0-800d7d9fd396" /><br/>
  <i>Figure 1. Architecture Lambda (inspirée de [Marz, 15])</i>
</p>

L’architecture Lambda a pour objectif de répondre aux besoins d’un système distribué robuste et tolérant aux pannes, à la fois contre les défaillances matérielles et les erreurs humaines. Elle est capable de prendre en charge un large éventail de charges de travail et de cas d’utilisation et peut donc être adoptée pour des projets Big Data pour réaliser des traitements de données génériques et évolutifs.

L’architecture Lambda considère différentes sources de données, de tout format, que l’on injecte dans la plateforme Big Data à travers la couche d’ingestion de données. Selon la nature de chaque application, les données seront transmises à la couche de traitement par lots et/ou la couche de traitement rapide. La couche de traitement par lots stocke les données sur le "Data Lake" et leur applique les traitements ensuite ; les résultats seront transmis à la couche de services dans des vues par lots. La couche de service indexe ensuite les vues afin qu’elles puissent être interrogées de manière personnalisée à faible temps de latence. La couche de traitement rapide compense la latence élevée des mises à jour de la couche de service et traite les flux de données en temps réel. Au niveau de la couche de requêtes, il est possible de répondre à toute requête entrante en fusionnant les résultats des vues par lots et des vues générées en temps réel.

Comme il n’est pas toujours indispensable que tous les projets aient une architecture proche de l’architecture Lambda, d’autres alternatives ont fait leurs preuves, en l’occurrence, l’architecture "Kappa". Cette architecture repose sur le principe de fusion de la couche batch (traitement par lots) et la couche temps réel. Cette architecture ne permet donc pas le stockage de manière permanente et est plutôt adaptée pour les projets de traitement des flux de données en temps réel ou quasi-temps réel. L’architecture Kappa présente quatre couches :

- Couche d’ingestion de données.
- Couche de temporisation (buffering) pour la sauvegarde temporaire des données.
- Couche de traitement pour préparer les vues en temps réel.
- Couche d’exploitation.

Concernant les technologies de l’écosystème Hadoop, quelle que soit l’architecture adoptée, pour chaque couche on trouve différents outils qui peuvent être utilisés conjointement ou indépendamment. La Table 1 présente une sélection d’outils pour chaque type de tâche.<br/><br/>
<i>Table 1. Sélection d’outils de l’écosystème Hadoop</i>
|Tâche|Exemples d’outils|
| :-: | :-: |
|Ingestion de données|Flume, Sqoop, Kafka|
|Stockage de données|HBase, Cassandra|
|Analyse de données|Hive, Pig, Spark, Storm|
|Exploration de données|Mahout, Drill|
|Sécurité de données|Ranger, Knox, Sentry|
|Exploitation et développement|Oozie, Zookeeper|   

Dans le reste de cette section, nous allons présenter brièvement une dizaine d’outils les plus populaires dans les projets Big Data.
## Principaux outils de l’écosystème Hadoop
### Apache Flume, l’ingestion massive des données hétérogènes
Flume[^1] est un système distribué conçu pour collecter, agréger et transférer de grandes quantités de données hétérogènes, de diverses sources, vers un magasin de données centralisé tel que HDFS ou HBase. Flume est un outil d’ingestion de données robuste et tolérant aux pannes avec des mécanismes de fiabilité configurables avec de nombreux mécanismes de basculement et de récupération. L’architecture de Flume, illustrée dans la Figure 2, est basée sur des flux de données et permet donc une application analytique de données.
<p align="center">
  <img src="https://github.com/Cloud-Elit/Cloud-Elit-apache_hadoop_ecosystem/assets/142179779/12b97cea-819f-4556-bf59-7f345f13bbdd" /><br/>
  <i>Figure 2. Architecture de Flume</i>
</p>

L’architecture d’Apache Flume est très simple et flexible. Les générateurs de données, via leurs serveurs web, génèrent des quantités massives de données qui sont collectées, sous forme d’évènements, par des agents individuels, appelés "Agents Flume". Un événement Flume est l’unité de données qui doit être transférée de la source à la destination (un message, un clic sur un site web, une connexion, un nouveau fichier, un paiement, etc.). Un agent Flume est un processus qui tourne sur une machine virtuelle Java ; il reçoit les événements des générateurs de données (ou des autres agents Flume) et les stocke dans des magasins centralisés (HDFS ou HBase). Un Agent Flume contient essentiellement trois composants :

- **Source** : c’est le composant qui reçoit les données des générateurs de données. La source transfère les données reçues vers un ou plusieurs canaux sous forme d’événements. 
- **Canal** : c’est le composant qui reçoit les événements de la source et les met en mémoire tampon (buffers) jusqu’à ce que les Sinks les consomment.
- **Sink** : c’est un composant qui consomme les données du canal et les stocke dans la destination (qui peut être un magasin centralisé ou d’autres agents Flume).
### Apache Sqoop, l’ingestion massive des données relationnelles
Sqoop[^2], abréviation de SQL-to-Hadoop, est un système d’import et d’export de données à partir et vers les bases de données relationnelles (Oracle, SQL Server, MySQL, etc.). C’est un canal de transfert de données entre Hadoop et les SGBDRs grâce à des configurations simples. Sqoop permet d’importer et d’exporter toute une base de données ou certaines tables individuelles. Le développeur peut même déterminer quelles colonnes ou quelles lignes seront importées ou exportées. Sqoop utilise la connexion JDBC pour connecter Hadoop à des bases de données relationnelles et peut créer directement des tables dans Apache Hive. Il prend également en charge l’importation incrémentielle si des nouveaux enregistrements ont été ajoutés depuis le dernier import.
### Apache Kafka, l’ingestion des flux de données
Kafka[^3] gère les flux de données générés par des producteurs de données (des sites web, des applications, des capteurs IoT, etc.). Il s’agit d’un système central qui collecte des données volumineuses, sous forme d’évènements, et les rend disponibles en temps réel pour d’autres applications (des consommateurs). La Figure 3 illustre l’architecture Kafka.
<p align="center">
  <img src="https://github.com/Cloud-Elit/Cloud-Elit-apache_hadoop_ecosystem/assets/142179779/64eb2c30-abb0-47ee-8bfb-522cd99546e6" /><br/>
  <i>Figure 3. Architecture de Kafka</i>
</p>

Un Cluster Kafka est composé d’un ensemble de nœuds, appelés des brokers (ou courtiers). Un broker est un composant logiciel composé d’un ou plusieurs topics (ou sujets). Chaque topic est divisé en partitions. La particularité de l’architecture Kafka, c’est qu’une partition peut être placée sur une machine unique ou distribuée sur plusieurs machines pour assurer des écritures ainsi que des lectures en parallèle.

Le fonctionnement de Kafka consiste à ce que les producteurs publient les événements sur des topics et que les consommateurs récupèrent les évènements sur les topics auxquels ils sont abonnés. Un producteur peut publier sur plusieurs topics et un consommateur peut également s’abonner à plusieurs topics. Les données sont répliquées entre différentes partitions sur différents brokers. Cela assure la fiabilité de Kafka et le rend tolérant aux erreurs ; en cas de problème sur un broker, les informations sont récupérées sur un autre. A la différence des autres outils de messagerie traditionnels tel que Tibco RDV, Tibco EMS ou encore RabbitMQ, Kafka présente la capacité de rétention des messages après leur consommation. La rétention est configurable en fonction de la durée et/ou de la quantité de données que l’on souhaite retenir. Kafka présente également l’avantage d’être horizontalement évolutif ; comme il s’agit d’un Cluster distribué, il suffit d’ajouter des nœuds pour monter en puissance.
### Apache HBase, une base de données NoSQL clé-valeur orientée colonnes
HBase[^4] est une base de données NoSQL non relationnelle conçue pour fonctionner avec des grands ensembles de données sur HDFS. HBase utilise le modèle clé-valeur ; chaque clé permet d’identifier une valeur unique dans une table très volumineuse sans schéma fixe (les lignes n’ont pas les mêmes attributs). Une table HBase rassemble des données distribuées et persistées sur HDFS sous forme de fichiers spécifiques, appelés HFiles ; HBase bénéficie donc de la tolérance aux pannes fournie par HDFS. La conception d’une table HBase doit tenir en considération cinq éléments :

- **La clé de la ligne (Row-Key)** : chaque ligne d’une table HBase est identifiée de façon unique par une clé, la row-key. La différence entre l’identifiant d’une table relationnelle et la clé d’une table HBase c’est que la row-key est une colonne interne à la structure de la table HBase (comme par exemple les numéros des lignes d’une feuille Excel).
- **La famille de colonnes (Column Family)** : une famille de colonnes représente les valeurs qui doivent être physiquement stockées dans un même ficher HFile. Le concept de "la famille de colonnes" définit la façon dont les données seront persistées et accédées. Lors de la conception de la table HBase, il faut veiller à regrouper dans une même famille, les colonnes qui s’utilisent souvent ensemble (pour éviter d’écrire et de lire dans différents fichiers HFiles). Pour cette raison, HBase est classé parmi les bases de données NoSQL orientées colonnes.
- **Le qualificateur de colonne (Column Qualifier)** : un qualificateur de colonne, ou tout simplement une colonne, est l’équivalent d’une colonne d’une base de données relationnelle. Chaque colonne possède un label qui la qualifie (d’où la nomination qualificateur de colonne). La particularité des colonnes HBase c’est qu’elles sont dynamiques ; certaines valeurs dans une même colonne peuvent être manquantes et c’est cette propriété qui permet à HBase de stocker des données éparses.
- **La cellule (Cell)** : c’est l’intersection entre une row key, une famille de colonnes et une colonne. Une cellule peut contenir des valeurs simples (texte, nombre, date, ...) ou complexes comme des fichiers textes ou multimédia sous forme d’une séquence de bytes (Byte []).
- **L’horodatage (Timestamp)** : tout comme HDFS, les mises à jour consistent à créer des nouvelles versions ; modifier une cellule dans HBase consiste à créer une nouvelle version de la cellule dans laquelle est stockée la nouvelle valeur. La distinction entre les versions se fait grâce à un horodatage assigné à chaque cellule. Par défaut, HBase garde les trois dernières versions de chaque cellule. 

L’architecture HBase est composée d’un nœud maître, le HMaster, et d’un ensemble de nœuds esclaves, les RegionsServers. Le HMaster gère les métadonnées des tables HBase et orchestre l’exécution des activités des RegionsServers (pour l’affectation des tâches, l’équilibrage de charge et le traitement des données). Les RegionsServers s’occupent des opérations de lecture/écriture des données dans HDFS (les HFiles sont stockés sur HDFS sous forme de blocs). Pour augmenter la taille du Cluster HBase, il suffit d’ajouter des RegionsServers.
### Apache Hive, un entrepôt relationnel pour des données volumineuses
Hive[^5] est un entrepôt de données distribué et tolérant aux pannes qui permet des analyses à grande échelle. Il permet aux utilisateurs de lire, écrire et gérer les données stockées sur HDFS avec des simples requêtes SQL grâce au langage HQL. Hive repose sur Hadoop et est donc conçu pour supporter de très gros volumes de données. Il est basé sur un système qui maintient des "métadonnées" qui décrivent les données stockées dans HDFS. Il utilise une base de données relationnelle appelée "metastore" pour assurer la persistance des métadonnées. Une table dans Hive est donc composée de deux éléments : un ensemble de données stockées sur HDFS et leur schéma stocké dans le metastore. Cela permet aux utilisateurs de manipuler les données comme si elles étaient persistées dans des tables relationnelles. Concrètement, Hive convertit les requêtes HQL en jobs MapReduce ou, à partir de la version 0.13 de Hive, jobs Tez qui est un cadre d’exécution sur Hadoop tout comme MapReduce. Cela permet donc aux analystes et développeurs d’exploiter les données HDFS sans se soucier des aspects de programmation des jobs MapReduce (ou Tez). Les interactions entre Hive et Hadoop, comme illustré dans la Figure 4, s’effectue en trois étapes :

1. **Requête HQL** : l’utilisateur prépare sa requête HQL sur un client Hive qui peut être un Client Shell (Client Beeline), une application JDBC/ODBC ou une interface web telle que Hue. La requête est ensuite envoyée au serveur Hive et sera traitée par le Service Thrift.
1. **Planification du job** : le pilote Hive s’occupe de la compilation, l’optimisation et la planification du job MapReduce (ou Tez). 
1. **Exécution du job** : le job est exécuté sur le cluster Hadoop.
<p align="center">
  <img src="https://github.com/Cloud-Elit/Cloud-Elit-apache_hadoop_ecosystem/assets/142179779/931f27d5-0593-45ad-80a1-0767f0e24d8a" /><br/>
  <i>Figure 4. Architecture de Hive</i>
</p>

### Apache Pig, l’analyse des données non-structurées
Pig[^6] est une plateforme permettant l’écriture des scripts d’analyse de données volumineuses, grâce à son langage Pig Latin, et l’exécution de ces scripts sur un Cluster Hadoop sous forme de jobs MapReduce ou Tez. La différence entre Hive et Pig c’est que le premier permet d’exécuter des requêtes HQL sur des données structurées alors que le deuxième permet d’exécuter des scripts Pig Latin sur des données structurées ou non-structurées. En plus que Pig Latin soit un langage procédural relativement proche à SQL, il est également un langage de flux de données. Cela permet à Pig de disposer d’un ensemble de fonctionnalités pour importer des données sur HDFS ou exporter des données vers des applications tierces. Le cadre Apache Pig englobe différents composants dont les principaux sont illustrés dans la Figure 5.
<p align="center">
  <img src="https://github.com/Cloud-Elit/Cloud-Elit-apache_hadoop_ecosystem/assets/142179779/0764d14f-f57d-423a-9aa9-b7158fcd06ed" /><br/>
  <i>Figure 5. Architecture de Pig</i>
</p>

Les scripts Pig Latin sont gérés par le Parseur qui est un analyseur syntaxique. Il vérifie, en entrée, la syntaxe de chaque script et, en sortie, il renvoie un graphe acyclique dirigé DAG (Directed Acyclic Graph), qui représente les instructions Pig Latin et les opérateurs logiques entre les instructions. Le DAG est ensuite transmis à l’optimiseur logique qui effectue des optimisations logiques. Le compilateur compile ensuite le DAG optimisé en une série de Jobs MapReduce (ou Tez) qui sont enfin soumis à Hadoop pour exécution.
### Apache Spark, le traitement rapide des données massives et des flux de données
Apache Spark[^7] est un système de traitement distribué qui peut être utilisé conjointement ou indépendamment de Hadoop. Le point fort de Spark est la mise en cache en mémoire vive et l’exécution optimisée des requêtes sur des données massives. Cela permet à Spark d’être beaucoup plus rapide que MapReduce qui doit absolument passer par des écritures/lectures sur disques (rappelons que les phases Map et Reduce sont exécutées sur des nœuds différents d’où la nécessité de passer par le réseau ou le disque pour transférer les données). Apache Spark peut être utilisé pour réaliser différentes tâches telles que l’exécution du SQL distribué, l’ingestion de données, l’exécution des algorithmes d’apprentissage automatique, le travail avec des graphiques ou des flux de données, etc. Apache Spark regroupe cinq composants principaux :

1. **Apache Spark Core** : c’est le moteur d’exécution de la plateforme Spark sur lequel reposent tous les autres composants. Il fournit des données de calcul et de référencement en mémoire dans des systèmes de stockage externes.
1. **Spark SQL** : c’est le module qui permet de traiter des données structurées. Les interfaces offertes par Spark SQL fournissent à Apache Spark plus d’informations sur la structure des données et sur le calcul effectué.
1. **Spark Streaming** : ce composant permet à Spark de traiter les flux de données en temps réel. Les données peuvent être chargées de HDFS ou ingérées à travers d’autres sources telles que Kafka ou Flume. Une fois chargées, les données peuvent être traitées à l’aide d’algorithmes complexes et transférées directement vers des systèmes de fichiers ou des bases de données externes.
1. **MLlib** : Apache Spark est équipé d’une bibliothèque riche connue sous le nom de MLlib (pour Machine Learning Library). Cette bibliothèque contient un large éventail d’algorithmes d’apprentissage automatique comme la Classification, la Régression, le Clustering ou encore le Filtrage Collaboratif. Ce module comprend également d’autres outils pour la construction, l’évaluation et le réglage des pipelines de Machine Learning.
1. **GraphX** : Apache Spark est livré avec la bibliothèque GraphX permettant de manipuler et d’effectuer des calculs sur des bases de données graphiques. GraphX ​​unifie le processus ETL (Extract, Transform, Load), l’analyse exploratoire et le calcul itératif des graphes.

Même si le coût de Spark est élevé, car il requiert plusieurs machines pour fonctionner en mémoire, il est considéré comme le moteur de traitement de données le plus efficace, largement utilisé dans les entreprises. De nombreuses caractéristiques le distinguent des autres outils de traitements de données, tels que MapReduce, parmi lesquelles on peut citer :

- **Traitement Rapide** : le point fort d’Apache Spark qui a incité les architectes Big Data à choisir cette technologie est sa vitesse. Les principales unités de données logiques dans Spark sont les RDDs (Resilient Distributed Datasets) qui sont des ensembles distribués d’objets très particuliers. Un RDD peut être divisé en plusieurs partitions logiques qui ont la capacité d’être stockées et traitées sur différentes machines d’un cluster. Le point fort de Spark réside donc dans les RDDs qui permettent de gagner du temps dans les opérations de lecture et d’écriture.
- **Traitement en Temps Réel** : contrairement à MapReduce qui ne traite que les données stockées sur HDFS, Spark est capable de traiter des flux de données en temps réel et est donc capable de produire des résultats instantanés.
- **Bibliothèques Riches** : Apache Spark propose plusieurs bibliothèques pour réaliser des traitements sur des données relationnelles avec SQL, appliquer des algorithmes d’apprentissage automatique, effectuer des analyses complexes sur des graphes, etc.
- **Flexibilité** : même si Spark est lui-même écrit en Scala, les programmes Spark peuvent être écrits avec différents langages : Java, Scala, R ou Python.
### Apache Mahout, l’apprentissage automatique
Mahout[^8] a été présenté initialement comme une bibliothèque pour l’apprentissage automatique offrant diverses implémentations d’algorithmes de Classification, de Clustering et de Recommandation. Au début, Mahout ne fonctionnait qu’avec Hadoop pour exécuter des algorithmes d’apprentissage automatique sur MapReduce pour des données stockées sur HDFS. Aujourd’hui, il est devenu un cadre général permettant un mélange de programmation de flux de données et de calculs algébriques linéaires sur des plateformes de traitement rapides comme Apache Spark. Cela a apporté aux utilisateurs la capacité d’exécuter des prétraitements sur les données et la formation des modèles dans un système de flux de données en temps réel.

Apache Mahout présente de nombreuses caractéristiques telles que :

- **Traitement distribué** : les algorithmes de Mahout sont écrits sur Hadoop ce qui leur permet nativement d’être distribués et applicables sur de gros volumes de données. 
- **Cadre complet** : Mahout offre au développeur un cadre prêt à l’emploi pour effectuer des tâches d’exploration de données sur de gros volumes de données.
- **Bibliothèque pour l’apprentissage automatique** : Mahout offre une bibliothèque qui comprend plusieurs implémentations de Clustering telles que k-means, fuzzy k-means, Canopy, Dirichlet et Mean-Shift. Mahout prend également en charge les implémentations de classification Naïve Bayes distribuée et complémentaire.
### Apache Oozie, la planification des flux de travail
Oozie[^9] est un planificateur (Scheduler, en anglais) de flux de travail (Workflow, en anglais) pour Hadoop. Un workflow est une séquence d’actions organisées dans un graphe acyclique dirigé de dépendance de contrôle. Toutes les actions sont en dépendance contrôlée car l’action suivante ne peut s’exécuter que selon la sortie de l’action en cours. Une action de workflow peut être une action MapReduce, Spark, Hive, Pig, Java, Shell, etc. Il peut y avoir des arbres de décision pour décider comment et dans quelles conditions une tâche doit s’exécuter. Il s’agit donc d’un système qui gère le workflow des travaux dépendants, permettant aux utilisateurs de créer des graphiques acycliques dirigés, qui peuvent être exécutés en parallèle ou de manière séquentielle dans Hadoop. Apache Oozie se compose de deux parties :

- Moteur de workflow qui s’occupe du stockage et de l’exécution des tâches des workflows.
- Moteur de coordinateur qui s’occupe de l’exécution des tâches des workflows en fonction des calendriers prédéfinis et de la disponibilité des données. 
### Apache ZooKeeper, la coordination des services distribués
ZooKeeper[^10] est un service distribué largement utilisé dans les plateformes Hadoop pour gérer la coordination d’un grand nombre d’hôtes. Pour sa simplicité de déploiement, ZooKeeper est aujourd’hui le standard pour la gestion et la coordination des grands Clusters. Il est même utilisé dans Apache HBase pour assurer la coordination, la communication et le partage des états entre le HMaster et les RegionsServers. La coordination et la gestion d’un service dans un environnement distribué est un processus complexe. Grâce à son architecture simple et ses APIs, ZooKeeper résout ce problème et permet aux développeurs de se concentrer sur la logique principale des applications. 

Concrètement, ZooKeeper peut être déployé dans un cluster pour coordonner les nœuds et maintenir des données partagées avec des techniques de synchronisation robustes. Les services fournis par ZooKeeper sont les suivants :

- **Service de nommage** : ce service assure l’identification des nœuds d’un cluster par leur nom.
- **Gestion de la configuration** : ce service transmet à chaque nouveau rejoignant le cluster la configuration nécessaire à jour.
- **Gestion de cluster** : ce service permet de gérer l’ajout ou la suppression des nœuds dans un cluster ainsi que l’état de chaque nœud en temps réel.
- **Élection du chef** : l’élection d’un nœud en tant que "chef" (ou Leader) à des fins de coordination est l’une des principales fonctionnalités sur lesquelles se base ZooKeeper. Il lui permet par exemple d’effectuer une récupération automatique si l’un des nœuds, qu’on appelle les "Followers", tombe en panne.
- **Service de verrouillage et de synchronisation** : ce mécanisme est très utile dans la récupération automatique en cas d’échec d’un nœud.
## Bibliographie
- [Landset, 15] Landset, Sara & Khoshgoftaar, Taghi & Richter, Aaron & Hasanin, Tawfiq. (2015). A survey of open source tools for machine learning with big data in the Hadoop ecosystem. Journal of Big Data. 2. DOI: 10.1186/s40537-015-0032-1. 
- [Marz, 15] Marz, Nathan & Warren, James. (2015). Big Data - Principles and best practices of scalable real-time data systems. Manning Publications Co. ISBN: 9781617290343.
- [Oussous, 17] Oussous, Ahmed & Benjelloun, Fatima-Zahra & Ait Lahcen, Ayoub & Belfkih, Samir. (2017). Big Data Technologies: A Survey. Journal of King Saud University - Computer and Information Sciences. DOI: 10.1016/j.jksuci.2017.06.001.
- [Raj, 18] Raj, Pethuru. (2018). The Hadoop Ecosystem Technologies and Tools. Advances in Computers, Volume 109. DOI: 10.1016/bs.adcom.2017.09.002. 
- [Singh, 18] Singh, Vikash Kumar & Taram, Manish & Agrawal, Vinni & Baghel, Bhartee Singh. (2018). A Literature Review on Hadoop Ecosystem and Various Techniques of Big Data Optimization. Springer Nature Singapore Pte Ltd. Advances in Data and Information Sciences, Lecture Notes in Networks and Systems 38. DOI: 10.1007/978-981-10-8360-0_22.

[^1]: https://flume.apache.org/
[^2]: https://sqoop.apache.org/
[^3]: https://kafka.apache.org/
[^4]: https://hbase.apache.org/
[^5]: https://hive.apache.org/
[^6]: https://pig.apache.org/
[^7]: https://spark.apache.org/
[^8]: https://mahout.apache.org/
[^9]: https://oozie.apache.org/ 
[^10]: https://zookeeper.apache.org/
