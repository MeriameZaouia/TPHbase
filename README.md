# TP HBase - Compte rendu

Ce projet implémente une application Java utilisant HBase pour créer une table et insérer des données. Le projet utilise Docker Compose pour orchestrer les différents services nécessaires.

## Structure du projet

### Docker Compose

Le fichier `docker-compose.yml` définit les services nécessaires pour exécuter HBase et Zookeeper.

```yaml
version: '2'

services:
  hbase-master:
    image: blueskyareahm/hbase-base:2.1.3
    command: master
    ports:
      - 16000:16000
      - 16010:16010

  hbase-regionserver:
    image: blueskyareahm/hbase-base:2.1.3
    command: regionserver1
    ports:
      - 16030:16030
      - 16201:16201
      - 16301:16301
      
  hbase-regionserver2:
    image: blueskyareahm/hbase-base:2.1.3
    command: regionserver2
    ports:
      - 16031:16030
      - 16202:16201
      - 16302:16301

  zookeeper:
    image: blueskyareahm/hbase-zookeeper:3.4.13
    ports:
      - 2181:2181

  my-app:
    hostname: hbase-regionserver
    build: .
    depends_on:
      - hbase-master
      - hbase-regionserver
      - zookeeper

``` 

### Dockerfile

Le fichier Dockerfile pour construire l'image de l'application Java.

``` bash
dockerfile
Copy code
FROM openjdk:8-jdk-alpine
COPY target/devoir-Hbase-1.0-SNAPSHOT-shaded.jar /app.jar
CMD ["java", "-jar", "/app.jar"]
```


### Code Java

Le code Java pour l'application HBase. Cette application crée une table, insère des données et les récupère.

``` java

package ma.enset.tp_hbase1;

import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import java.io.IOException;

public class App {
    public static final String TABLE_NAME = "users";
    public static final String CF_PERSONAL_DATA = "personal_data";
    public static final String CF_PROFESSIONAL_DATA = "professional_data";

    public static void main(String[] args) {
        org.apache.hadoop.conf.Configuration config = HBaseConfiguration.create();
        config.set("hbase.zookeeper.quorum", "zookeeper");
        config.set("hbase.zookeeper.property.clientPort", "2181");
        config.set("hbase.master", "hbase-master:16000");

        try {
            Connection connection = ConnectionFactory.createConnection(config);
            Admin admin = connection.getAdmin();
            TableName tableName = TableName.valueOf(TABLE_NAME);

            HTableDescriptor tableDescriptor = new HTableDescriptor(tableName);
            tableDescriptor.addFamily(new HColumnDescriptor(CF_PERSONAL_DATA));
            tableDescriptor.addFamily(new HColumnDescriptor(CF_PROFESSIONAL_DATA));

            if (!admin.tableExists(tableName)) {
                admin.createTable(tableDescriptor);
                System.out.println("La table est bien créée.");
            } else {
                System.out.println("La table existe déjà.");
            }

            Table table = connection.getTable(tableName);
            Put put = new Put(Bytes.toBytes("11111"));
            put.addColumn(Bytes.toBytes(CF_PERSONAL_DATA), Bytes.toBytes("name"), Bytes.toBytes("Hasna EL BACHA"));
            put.addColumn(Bytes.toBytes(CF_PERSONAL_DATA), Bytes.toBytes("age"), Bytes.toBytes(50));
            put.addColumn(Bytes.toBytes(CF_PERSONAL_DATA), Bytes.toBytes("diplome"), Bytes.toBytes("Ing"));

            table.put(put);
            System.out.println("La ligne est bien insérée.");

            Get get = new Get(Bytes.toBytes("11111"));
            Result result = table.get(get);
            byte[] age = result.getValue(Bytes.toBytes(CF_PERSONAL_DATA), Bytes.toBytes("age"));

            System.out.println("L'âge de l'utilisateur est : " + Bytes.toString(age));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Exécution du projet

#### Prérequis

Docker
Docker Compose
JDK 8
Maven

#### Étapes
Construisez le projet Java avec Maven :

```bash

mvn clean install
Construisez et démarrez les services Docker :
```

``` bash

docker-compose up --build
```
Vérifiez que les services sont démarrés et fonctionnent correctement.

Exécutez l'application Java dans le conteneur my-app.
