# ESP32_CLAUDE_DEV

Projet IoT complet pour **gestion de mesures environnementales** avec **ESP32** et **serveur web**.

## 📋 Vue d'ensemble

Ce projet intègre deux sous-systèmes:

### 1️⃣ **ESP32 Firmware** (Microcontrôleur)
- 📊 **Capteurs**: Température, pH, consommation électrique
- 📡 **Connectivité**: WiFi + MQTT/HTTP vers serveur
- 💾 **Stockage**: Données locales + synchronisation serveur
- ⚡ **Optimisation**: Gestion mémoire stricte, low-power modes

### 2️⃣ **WebServer** (Serveur Synology NAS)
- **Backend**: REST API pour récupérer/stocker mesures
- **Frontend**: Dashboard responsive pour smartphone
- **Base de données**: Historique mesures, alertes, profils utilisateurs
- **Temps réel**: WebSocket pour mises à jour live

---

## 🚀 Démarrage rapide

### Prérequis
- ESP-IDF ou PlatformIO
- Node.js 18+ (WebServer)
- Python 3.9+ (optionnel, backend alternatif)
- Docker (optionnel)

### Installation locale

```bash
# 1. Clone le repo
git clone https://github.com/zangdar05/ESP32_CLAUDE_DEV.git
cd ESP32_CLAUDE_DEV

# 2. Setup ESP32
cd ESP32
# Voir docs/SETUP.md pour détails

# 3. Setup WebServer
cd ../WebServer/backend
npm install
npm run dev
```

Voir **`docs/SETUP.md`** pour guide complet.

---

## 📚 Documentation

| Document | Objectif |
|----------|----------|
| **`docs/CLAUDE_CONTEXT.md`** 🔑 | Comment utiliser Claude Opus pour ce projet |
| **`docs/SKILLS.md`** | 9 techniques avancées pour prompts |
| **`docs/PROMPTS.md`** | 15+ templates de prompts réutilisables |
| **`docs/ARCHITECTURE.md`** | Vue C4 du système complet |
| **`docs/DOMAIN.md`** | Vocabulaire métier partagé |
| **`docs/API.md`** | Spécification APIs complète |
| **`docs/SETUP.md`** | Guide développement local |

---

## 🏗️ Structure du projet

```
ESP32_CLAUDE_DEV/
├── ESP32/                          # Firmware microcontrôleur
│   ├── src/
│   │   ├── main.cpp               # Point d'entrée
│   │   ├── config/                # Configuration globale
│   │   ├── sensors/               # Modules capteurs
│   │   ├── connectivity/          # WiFi, MQTT, HTTP
│   │   └── storage/               # SPIFFS, EEPROM
│   ├── platformio.ini
│   └── README.md
│
├── WebServer/                      # Backend + Frontend
│   ├── backend/                   # API REST + WebSocket
│   │   ├── src/
│   │   │   ├── api/              # Endpoints
│   │   │   ├── services/         # Business logic
│   │   │   ├── database/         # ORM, queries
│   │   │   └── middleware/       # Auth, validation
│   │   └── package.json
│   │
│   └── frontend/                  # React dashboard
│       ├── src/
│       │   ├── pages/
│       │   ├── components/
│       │   └── services/
│       └── package.json
│
├── docs/                          # Documentation (À LIRE)
│   ├── CLAUDE_CONTEXT.md         # 🔑 Configuration Claude
│   ├── SKILLS.md                 # Skills avancées
│   ├── PROMPTS.md                # Templates prompts
│   ├── ARCHITECTURE.md           # C4 diagrams
│   ├── DOMAIN.md                 # Domaine métier
│   ├── API.md                    # Spécification APIs
│   └── SETUP.md                  # Setup développement
│
└── .gitignore
```

---

## 🎯 Première utilisation avec Claude Opus

1. **Lisez** `docs/CLAUDE_CONTEXT.md` (important!)
2. **Comprenez** l'architecture: `docs/ARCHITECTURE.md`
3. **Choisissez** un prompt template dans `docs/PROMPTS.md`
4. **Copiez-colle** dans Claude avec vos paramètres
5. **Claude génère** code cohérent et documenté

**Exemple**: Ajouter capteur PH
```bash
# Ouvrez docs/PROMPTS.md → Section "Prompts ESP32"
# Customisez le template "Prompt 1: Ajouter nouveau capteur"
# Copiez-colle dans Claude Opus
# Récupérez code C++ complète + intégration
```

---

## 🤝 Contribution

- Respectez la structure `docs/` pour toute documentation
- Utilisez les prompts templates de `docs/PROMPTS.md` avec Claude
- Consultez `docs/DOMAIN.md` pour vocabulaire métier
- Validez architecture avec `docs/ARCHITECTURE.md`

---

## 📝 License

À définir (MIT, Apache 2.0, ?)

---

**Prêt à démarrer? → Lire `docs/CLAUDE_CONTEXT.md` 🚀**
