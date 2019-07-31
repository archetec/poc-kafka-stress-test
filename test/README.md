# Introduction

Cette image est utilisée pour tester une configuration Mirror Maker. Elle permet de pré-charger 2 clusters Kafka avec une série de topics qui sont répliqués et de rouler un "stress test".  Il est possible de configurer le nombre de topics et le nombre de partitions pour chaque topic.

# Configuration

Les variables d'environnement suivantes doivent être configurées (en plus des variables nécessaire pour l'image kafka-mirrormaker):

* TEST_TOPIC_NAME_PREFIX : Préfixe pour le noms des topics.  Le script va automatiquement ajouter un chiffre (partant de 0) comme suffixe. Ex.: ia-test-
* TEST_NB_TOPICS : Nombre de topics à créer.  Les topics sont automatiquement crées dans les 2 clusters.
* TEST_NB_PARTITIONS : Nombre de partitions pour chaque topic.
* TEST_SOURCE_ZOOKEEPER : Adresse du Zookeeper du cluster source.
* TEST_TARGET_ZOOKEEPER : Adresse du Zookeeper du cluster cible.
* TEST_NUM_RECORDS : Nombre de messages à générer dans le topic 1 (préfix suivi de 1)
* TEST_RECORD_SIZE : Dimension en byte du message à générer
* TEST_THROUGHPUT : Nombre de messages à la seconde
* TEST_NB_DATA_GENERATOR_THREADS : Nombre de threads qui seront lancés pour générer de messages.  Chaque thread est indépendant et va générer des messages dans un topic différent, jusqu'à la limite de TEST_NUM_RECORDS.  Donc 5 threads à 100 msg/s va donner une charge de 500 msg/s repartie sur 5 topics.

Les variables d'environnement suivantes sont optionnelles:

* KAFKA_TOOLS_LOG4J_LOGLEVEL : Niveau de détails des messages dans le journal d'exécution (par défaut WARN). Ex.: DEBUG

# Utilisation

(Cette partie suppose que vous utilisez le docker-compose dans /test/kafka-mirrormaker-local et que vous n'avez pas modifié les valeurs par défaut. Sinon, vous devrez faire les ajustements nécessaires.)

L'image dérive de l'image kafka-mirrormaker (build dans ce repo).  Le script de démarrage (run) est le même mais a été modifié pour créer les topics dans les 2 clusters avant de démarrer Mirror Maker, pour éviter les rebalancement inutiles pendant l'initialisation du test.

Une fois Mirror Maker démarré, le script produit aussi des données aléatoires selon les paramètres utilisés.  TEST_NB_DATA_GENERATOR_THREADS permet de spécifier le nombre de générateurs indépendants.  Le premier va publier dans ia-test-1, le second dans ia-test-2 et ainsi de suite.  Il est donc possible de tester la mise à l'échelle de Mirror Maker avec différents paramètres de charge.

L'objectif de ce test est de 

1) Mesurer le temps de rebalancement du consumer de Mirror Maker lors de l'ajout d'un topic
2) Mesurer la performance de Mirror Mirror sous charge

Le temps de rebalancement est influencé par le nombre de topics et de partitions à distribuer aux consommateurs.  Le lag est tant qu'à lui influencé par la quantité de messages à répliquer.  Pendant un rebalancement Mirror Maker cesse de répliquer les messages, il y a donc un retard qui va devoir être repris. Les tests devraient permettre de mesurer le comportement de Mirror Maker pendant un rebalancement et après (temps pour reprendre le retard accumulé).

Pour faire le test:

* Lancer le docker-compose
* Se connecter au log du container mirrormaker:
```bash
docker logs -f mirrormaker
```
* Surveiller la création des topics (cela peut prendre un certain temps, environ 2 secondes par topic) et le démarrage de Mirror Maker
* Une fois Mirror Maker démarré, créer un nouveau topic sur le broker1 avec un nom qui répond au critère du whitelist.  Cela va force un rebalancement du consumer (Mirror Maker peut prendre un certain temps à détecter le nouveau topic).  Pour créer le topic, utiliser Kafka Tool ou encore via la ligne de commande, se connecter sur la console du broker1 :
```bash
docker -it broker1 bash
```
Puis exécuter :
```bash
kafka-topics --create --topic ia-test-new --partitions 1 --replication-factor 1 --zookeeper zookeeper1:2181
```
* Le journal de Mirror Maker va afficher le début du rebalancement ainsi que la fin avec le temps en ms pour chaque consommateur
* Vous pouvez surveiller le lag du consommateur de Mirror Maker dans Kafka Tool ou encore via une commande bash (vous devez être connecté à un des containers du cluster)
```bash
while(true); do kafka-consumer-groups --bootstrap-server broker1:29092 --describe --offsets --group mirrormaker | grep 'ia-test-1 '; done
```
* Documenter les résultats.

