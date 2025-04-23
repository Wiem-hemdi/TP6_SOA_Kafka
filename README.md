# Kafka-MongoDB Integration Project 

![Kafka+MongoDB+Node.js](https://miro.medium.com/v2/resize:fit:1400/1*Q5XjGNn2Hn8F9z6eoXp-og.png)

## 📌 Prérequis (Windows)

- Java 8+ (pour Kafka)
- Node.js 16+
- Kafka 3.9.0
- **MongoDB Compass** (inclut le Shell) [Télécharger ici](https://www.mongodb.com/try/download/compass)
- PowerShell en mode administrateur

---

## 🛠️ Installation

### 1. Installer MongoDB (Solution Windows)
```powershell
# Après installation de MongoDB Compass, ajouter au PATH
[Environment]::SetEnvironmentVariable("Path", "$env:Path;C:\Program Files\MongoDB\Server\<version>\bin", "User")
### 2. Démarrer MongoDB (Sans service)
# Créer le dossier de données
mkdir C:\mongodb-data

# Démarrer MongoDB manuellement (garder ce terminal ouvert)
mongod --dbpath="C:\mongodb-data"

### 3. Configurer Kafka
# Démarrer Zookeeper (terminal 1)
bin\windows\zookeeper-server-start.bat config\zookeeper.properties

# Démarrer Kafka (terminal 2)
bin\windows\kafka-server-start.bat config\server.properties

# Créer un topic
bin\windows\kafka-topics.bat --create --topic test-topic --bootstrap-server localhost:9092

🚀 Utilisation
1. Installer les dépendances Node.js
npm init -y
npm install kafkajs mongoose express

2. Fichiers principaux
producer.js
const { Kafka } = require('kafkajs');

const kafka = new Kafka({
  clientId: 'my-app',
  brokers: ['localhost:9092']
});

const producer = kafka.producer();

const run = async () => {
  await producer.connect();
  setInterval(async () => {
    try {
      await producer.send({
        topic: 'test-topic',
        messages: [{ value: 'Hello KafkaJS user!' }],
      });
      console.log('Message produit avec succès');
    } catch (err) {
      console.error("Erreur lors de la production de message", err);
    }
  }, 1000);
};

run().catch(console.error);
consumer.js
const { Kafka } = require('kafkajs');
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/kafkamessages', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

const MessageSchema = new mongoose.Schema({
  value: String,
  timestamp: { type: Date, default: Date.now },
});
const Message = mongoose.model('Message', MessageSchema);

const kafka = new Kafka({
  clientId: 'my-app',
  brokers: ['localhost:9092']
});

const consumer = kafka.consumer({ groupId: 'test-group' });

const run = async () => {
  await consumer.connect();
  await consumer.subscribe({ topic: 'test-topic', fromBeginning: true });

  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      const msg = new Message({ value: message.value.toString() });
      await msg.save();
      console.log({ value: message.value.toString() });
    },
  });
};

run().catch(console.error);

api.js
const express = require('express');
const mongoose = require('mongoose');

const app = express();
const PORT = 3000;

mongoose.connect('mongodb://localhost:27017/kafkamessages', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

const MessageSchema = new mongoose.Schema({
  value: String,
  timestamp: { type: Date, default: Date.now },
});
const Message = mongoose.model('Message', MessageSchema);

app.get('/messages', async (req, res) => {
  const messages = await Message.find().sort({ timestamp: -1 });
  res.json(messages);
});

app.listen(PORT, () => {
  console.log(`🚀 Serveur API REST sur http://localhost:${PORT}/messages`);
});
3. Lancer les composants (dans des terminaux séparés)

Terminal	Commande	Description
3	node producer.js	Producteur de messages Kafka
4	node consumer.js	Consommateur + MongoDB
5	node api.js	API REST sur port 3000

🔍 Vérification
1. Vérifier MongoDB
# Dans un nouveau terminal
mongo --eval "db.version()"

# Accéder à la base kafkamessages
mongo kafkamessages
> db.messages.find().pretty()

2. Tester l'API
curl http://localhost:3000/messages
# Ou dans le navigateur: http://localhost:3000/messages

🐛 Dépannage (Windows)
Erreur "Accès refusé"
# Solution 1: Exécuter en admin
Start-Process powershell -Verb RunAs -ArgumentList "net start MongoDB"

# Solution 2: Démarrer manuellement
mongod --dbpath="C:\mongodb-data"

Commandes MongoDB introuvables
Vérifiez le chemin d'installation par défaut : C:\Program Files\MongoDB\Server\<version>\bin

Ajoutez-le au PATH :
[Environment]::SetEnvironmentVariable("Path", "$env:Path;C:\Program Files\MongoDB\Server\7.0\bin", "User")

📦 Structure des fichiers
project/
├── producer.js       # Producteur Kafka
├── consumer.js       # Consommateur → MongoDB
├── api.js            # API Express
└── package.json

🔄 Flux de données
Kafka Producer → Kafka Topic → Kafka Consumer → MongoDB → API REST → Client

📚 Documentation
KafkaJS

Mongoose

MongoDB Shell

Express.js



