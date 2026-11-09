# Network Intrusion Detection System (IDS)

**Développé par : Yosri Hamdouni**  
GitHub : [yosri-trade](https://github.com/yosri-trade)

Un système de détection d'intrusion réseau (IDS) développé en Python, capable d'identifier des activités suspectes grâce à une détection basée sur des signatures et à l'analyse statistique des anomalies.

## 📋 Table des matières

- [Vue d'ensemble](#-vue-densemble)
- [Architecture](#️-architecture)
- [Installation](#-installation)
- [Utilisation](#-utilisation)
- [Configuration](#️-configuration)
- [Tests](#-tests)
- [Docker](#-docker)
- [Sécurité](#-sécurité)
- [Machine Learning](#-machine-learning-optionnel)
- [Dépannage](#-dépannage)
- [Licence](#-licence)
- [Contribution](#-contribution)
- [Références](#-références)

## 🎯 Vue d'ensemble

Ce projet implémente un IDS (Intrusion Detection System) avec les fonctionnalités suivantes :

- **Capture de paquets :** capture en temps réel (Live) et lecture de fichiers PCAP (Replay).
- **Détection par signatures :** détection des scans de ports, SYN Flood, Ping Sweep et ports suspects.
- **Détection d'anomalies :** analyse statistique du trafic et utilisation optionnelle du Machine Learning avec Isolation Forest.
- **Journalisation structurée :** génération de logs JSON avec rotation automatique.
- **Alertes en temps réel :** affichage des alertes avec Rich et Colorama.
- **Tests automatisés :** suite de tests basée sur pytest.

## 🏗️ Architecture

```text
              ┌──────────────────────┐
              │    Packet Sniffer    │
              │    (Live / Replay)   │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │    Packet Parser     │
              │    (Normalization)   │
              └──────────┬───────────┘
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
      ┌────────────┐ ┌──────────┐ ┌──────────┐
      │ Signature  │ │ Anomaly  │ │  Logger  │
      │  Engine    │ │  Engine  │ │   JSON   │
      └──────┬─────┘ └────┬─────┘ └────┬─────┘
             │            │            │
             └────────────┼────────────┘
                          │
                          ▼
              ┌──────────────────────┐
              │    Alert Manager     │
              │      (Display)       │
              └──────────────────────┘
```

### Modules principaux

| Module | Description |
|---|---|
| `core/sniffer.py` | Capture des paquets réseau |
| `core/packet_parser.py` | Analyse et normalisation des paquets |
| `core/signature_engine.py` | Détection basée sur des signatures |
| `core/anomaly_engine.py` | Détection statistique et Machine Learning |
| `core/logger.py` | Journalisation structurée au format JSON |
| `core/alert.py` | Gestion et affichage des alertes |
| `cli/interface.py` | Interface en ligne de commande |
| `main.py` | Point d'entrée principal |

## 📦 Installation

### Prérequis

- Python 3.10 ou version supérieure.
- pip.
- Sur Linux : privilèges administrateur pour la capture réseau en temps réel.
- Docker et Docker Compose (facultatifs).

### Installation avec pip

```bash
# Accéder au répertoire du projet
cd IDS_Project

# Créer un environnement virtuel
python -m venv venv

# Activer l'environnement virtuel
source venv/bin/activate

# Sur Windows :
# venv\Scripts\activate

# Installer les dépendances
pip install -r requirements.txt
```

### Installation avec Docker

```bash
docker-compose up --build
```

## 🚀 Utilisation

### 1. Générer un fichier PCAP d'exemple

```bash
python tools/pcap_generator.py \
    --output data/pcap_samples/sample_traffic.pcap
```

### 2. Démarrer l'IDS en mode Replay

```bash
python main.py --start \
    --mode replay \
    --pcap data/pcap_samples/sample_traffic.pcap
```

### 3. Démarrer l'IDS en mode Live

Cette fonctionnalité nécessite les privilèges appropriés et une interface réseau autorisée.

```bash
sudo python main.py --start --mode live --interface eth0
```

### 4. Afficher les statistiques

```bash
python main.py --stats
```

### 5. Afficher les alertes récentes

```bash
python main.py --show-alerts --tail 10
```

### 6. Afficher l'aide

```bash
python main.py --help
```

### Commandes disponibles

| Option | Description |
|---|---|
| `--start` | Démarrer le système IDS |
| `--mode {live,replay}` | Choisir le mode de capture |
| `--interface IFACE` | Spécifier l'interface réseau |
| `--pcap PATH` | Spécifier le fichier PCAP |
| `--stats` | Afficher les statistiques |
| `--show-alerts` | Afficher les alertes |
| `--tail N` | Limiter le nombre d'alertes affichées |
| `--config PATH` | Charger un fichier de configuration personnalisé |
| `--dry-run` | Vérifier la configuration sans démarrer l'IDS |
| `--version` | Afficher la version |

## ⚙️ Configuration

Le fichier `config.yaml` permet de personnaliser le comportement du système.

```yaml
capture:
  mode: replay
  interface: eth0
  pcap_path: data/pcap_samples/sample_traffic.pcap
  bpf_filter: ""
  throttle_packets_per_sec: 0

signatures:
  portscan:
    enabled: true
    ports_threshold: 20
    window_seconds: 10

  syn_flood:
    enabled: true
    syn_threshold: 200
    window_seconds: 10

  ping_sweep:
    enabled: true
    hosts_threshold: 50
    window_seconds: 30

  suspicious_ports:
    enabled: true
    ports: [21, 22, 23, 3389, 4444, 6667]

anomaly:
  packet_rate_threshold: 1000
  payload_size_threshold_bytes: 10000
  use_ml: false
  model_path: data/datasets/isof_model.joblib
  baseline_path: data/datasets/baseline_train.csv

logging:
  traffic_log: data/logs/traffic.log
  alerts_log: data/logs/alerts.log
  max_bytes: 10485760
  backup_count: 7
  level: INFO
```

## 🧪 Tests

### Exécuter tous les tests

```bash
pytest -v
```

### Exécuter les tests avec couverture

```bash
pytest --cov=core --cov=cli --cov-report=html
```

### Exécuter des tests spécifiques

```bash
pytest tests/test_parser.py -v
pytest tests/test_signature_engine.py -v
pytest tests/test_anomaly_engine.py -v
pytest tests/test_sniffer.py -v
```

## 🐳 Docker

### Construire l'image Docker

```bash
make docker-build
```

Ou directement :

