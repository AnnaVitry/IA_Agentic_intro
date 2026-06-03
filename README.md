# Projet IA Agentique : De l'Inférence Locale à l'Autonomie

Ce projet documente l'évolution d'un système d'Intelligence Artificielle, passant d'un simple moteur de génération de texte (Stateless) à un Agent autonome capable de raisonner, d'utiliser des outils et de consulter des bases de connaissances locales.

## Infrastructure & Environnement

Pour garantir la reproductibilité et la stabilité, le projet repose sur une stack stricte sous Linux (Ubuntu) :

- **`uv` :** Gestionnaire de paquets et d'environnements virtuels (écrit en Rust). Remplace `pip` pour une résolution des dépendances ultra-rapide et sécurisée.

- **Ollama :** Moteur d'inférence local permettant de faire tourner les modèles sans dépendre d'une API Cloud (OpenAI, etc.).

- *Modèle de raisonnement :* `llama3.2:3b`

- *Modèle de vectorisation (Embeddings) :* `nomic-embed-text`

- **Docker :** Utilisé pour conteneuriser les bases de données (ex: PostgreSQL) et isoler l'environnement de développement.

---

## Évolution de l'Architecture (Step-by-Step)

### Étape 1 : Fondations et Concepts LLM

Interaction brute avec l'API d'Ollama pour comprendre les mécanismes fondamentaux :

- **Stateless :** Le modèle n'a aucune mémoire native.

- **Température :** Contrôle de l'entropie (0 = déterministe pour le code/logique, 1 = créatif).

- **Context Window :** La limite de tokens (mémoire à court terme) que le modèle peut traiter simultanément.

### Étape 2 : Abstraction et LCEL avec LangChain

Remplacement des scripts Python basiques par le framework industriel **LangChain**.

- **Objectif :** Rendre le code modulaire et agnostique au modèle choisi.

- **LCEL (LangChain Expression Language) :** Utilisation de l'opérateur Pipe (`|`) pour créer des chaînes déclaratives (`prompt | llm`).

- **Mémoire RAM :** Implémentation d'une *Sliding Window* (fenêtre glissante) pour conserver les *k* derniers messages sans saturer la VRAM.

- **`MessagesPlaceholder` :** Réservoir dynamique dans le prompt pour injecter l'historique proprement.

### Étape 3 : Persistance et Bases Vectorielles (Vector Stores)

Création d'une mémoire "en dur" (à long terme) pour pallier l'amnésie lors du redémarrage du script.

- **Les Embeddings :** Transformation du texte en vecteurs mathématiques.

- **Chroma DB :** Base de données vectorielle locale, idéale pour le prototypage.

- **PGVector :** Extension mathématique sur un véritable serveur PostgreSQL (via Docker), permettant des requêtes hybrides puissantes (SQL + Vecteurs) en production.

### Étape 4 : RAG (Retrieval-Augmented Generation)

Blocage des "hallucinations" en forçant l'IA à lire des sources fiables (fichiers texte, PDF) avant de répondre.

- **L'Ingestion & Chunking :** Découpage intelligent du texte (`RecursiveCharacterTextSplitter`) pour s'adapter à la fenêtre de contexte.

- **L'Approche LangChain :** Code très modulaire et personnalisable, étape par étape (Loader -> Splitter -> Chroma -> Pipe RAG).

- **L'Approche LlamaIndex & FAISS :** Alternative ultra-compacte. LlamaIndex gère automatiquement l'extraction complexe (ex: PDF via `pypdf`), couplé à **FAISS** (méta-bibliothèque de recherche vectorielle en RAM).

### Étape 5 : Agents Autonomes et Outils (LangGraph)

Transformation du LLM d'un simple "parleur" en un "acteur".

- **Boucle ReAct (Reason + Act) :** L'agent réfléchit (*Thought*), choisit un outil, agit (*Action*), observe le résultat (*Observation*), puis répond.

- **Tools :** Création de fonctions Python documentées (Docstrings) que l'IA peut déclencher de manière autonome (ex: calculateur de TVA).

- **LangGraph :** Remplacement de la chaîne linéaire par un Graphe d'État (`StateGraph`). Le système boucle intelligemment entre le cerveau de l'agent (`node_agent`) et ses bras (`ToolNode`) jusqu'à la résolution du problème.