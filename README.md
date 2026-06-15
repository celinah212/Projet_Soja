# ⚡ ClientManager — Projet Celina

> Application web de gestion de clients construite avec **Flask (Python)** et **SQLite**, suivant une architecture en couches **MVC + Repository + Service**.

---

## 📋 Table des matières

1. [Présentation du projet](#-présentation-du-projet)
2. [Fonctionnalités](#-fonctionnalités)
3. [Technologies utilisées](#-technologies-utilisées)
4. [Architecture du projet](#-architecture-du-projet)
5. [Structure des fichiers](#-structure-des-fichiers)
6. [Base de données](#-base-de-données)
7. [Routes et API](#-routes-et-api)
8. [Fonctionnement couche par couche](#-fonctionnement-couche-par-couche)
9. [Frontend — Interfaces et comportements](#-frontend--interfaces-et-comportements)
10. [Style et design (CSS)](#-style-et-design-css)
11. [Installation et lancement](#-installation-et-lancement)
12. [Flux de données complet](#-flux-de-données-complet)

---

## 🧭 Présentation du projet

**ClientManager** (Projet Celina) est une application web full-stack légère permettant de gérer une base de données de clients. L'utilisateur peut **ajouter**, **consulter**, **modifier** et **supprimer** des clients via une interface moderne en thème sombre, sans avoir besoin de recharger la page (interactions asynchrones via `fetch`).

Le projet est développé en **Python 3.14** avec le micro-framework **Flask**, une base de données **SQLite** locale, et un frontend entièrement en **HTML/CSS/JavaScript vanilla** (sans framework JS externe).

---

## ✅ Fonctionnalités

| Fonctionnalité | Description |
|---|---|
| ➕ Ajouter un client | Formulaire de saisie avec validation côté client et serveur |
| 📋 Lister les clients | Tableau en lecture seule de tous les clients enregistrés |
| ✏️ Modifier un client | Modification via une modale avec pré-remplissage des données |
| 🗑️ Supprimer un client | Suppression avec confirmation par modale + renumérotation automatique |
| 🔁 Renumérotation automatique | Après chaque suppression, les numéros sont recalculés et restent continus |
| 🔔 Notifications Toast | Retours visuels animés (succès / erreur) en haut à droite de l'écran |
| 🌙 Thème sombre | Interface Dark Mode avec palette CSS cohérente |
| 📱 Responsive | Mise en page adaptée aux petits écrans (mobile) |

---

## 🛠 Technologies utilisées

| Catégorie | Technologie |
|---|---|
| Langage backend | Python 3.14 |
| Framework web | Flask |
| Base de données | SQLite 3 (fichier local) |
| Moteur de templates | Jinja2 (intégré à Flask) |
| Frontend | HTML5, CSS3, JavaScript ES2020 (vanilla) |
| Communication API | `fetch` API (JSON asynchrone) |
| Versionnement | Git |

---

## 🏗 Architecture du projet

Le projet suit une **architecture en couches** inspirée des patterns **MVC** (Modèle-Vue-Contrôleur) enrichie d'une couche **Repository** et d'une couche **Service** pour une meilleure séparation des responsabilités.

```
┌──────────────────────────────────────────────────────────────┐
│                         NAVIGATEUR                           │
│            (HTML + CSS + JavaScript / Fetch API)             │
└──────────────────────┬──────────────────────────────────────┘
                       │  Requêtes HTTP (GET, POST, PUT, DELETE)
┌──────────────────────▼──────────────────────────────────────┐
│                  CONTROLLER (Flask Blueprint)                │
│            controllers/client_controller.py                  │
│   → Expose les routes HTTP et les endpoints API JSON         │
└──────────────────────┬──────────────────────────────────────┘
                       │  Appels de méthodes
┌──────────────────────▼──────────────────────────────────────┐
│                   SERVICE (Logique métier)                   │
│              services/client_service.py                      │
│   → Validation des données, règles métier                    │
└──────────────────────┬──────────────────────────────────────┘
                       │  Appels de méthodes
┌──────────────────────▼──────────────────────────────────────┐
│                  REPOSITORY (Accès données)                  │
│           repositories/client_repository.py                  │
│   → Requêtes SQL, opérations CRUD, renumérotation           │
└──────────────────────┬──────────────────────────────────────┘
                       │  sqlite3
┌──────────────────────▼──────────────────────────────────────┐
│                  BASE DE DONNÉES SQLite                      │
│              database/clients.db                             │
│   → Table `clients` persistée sur disque                     │
└─────────────────────────────────────────────────────────────┘
```

### Rôle de chaque couche

**`app.py` — Point d'entrée**
Initialise Flask, appelle `init_db()` pour créer la table si elle n'existe pas, enregistre le Blueprint du contrôleur, et démarre le serveur.

**`database/db.py` — Connexion à la base**
Fournit deux fonctions utilitaires : `get_connection()` (ouvre une connexion SQLite avec `row_factory`) et `init_db()` (crée la table `clients` si elle n'existe pas).

**`models/client.py` — Modèle de données**
Classe `Client` représentant une entité client. Fournit `to_dict()` pour la sérialisation JSON et `from_row()` comme constructeur statique depuis une ligne SQLite.

**`repositories/client_repository.py` — Couche d'accès aux données**
Classe `ClientRepository` avec toutes les opérations SQL : lecture, insertion, mise à jour, suppression, vérification d'unicité de l'identifiant, et renumérotation des clients.

**`services/client_service.py` — Logique métier**
Classe `ClientService` qui orchestre les opérations : nettoyage des entrées (`strip()`), validation des champs obligatoires, contrôle de l'unicité de l'identifiant, et délégation au repository.

**`controllers/client_controller.py` — Contrôleur HTTP**
Blueprint Flask `/clients` qui expose les routes de pages (vues HTML) et les endpoints API REST JSON.

---

## 📁 Structure des fichiers

```
Projet Celina/
│
├── app.py                          # Point d'entrée de l'application Flask
│
├── database/
│   ├── db.py                       # Connexion SQLite et initialisation de la table
│   └── clients.db                  # Fichier de base de données SQLite (généré automatiquement)
│
├── models/
│   └── client.py                   # Classe Client (modèle de données)
│
├── repositories/
│   └── client_repository.py        # CRUD SQLite + renumérotation
│
├── services/
│   └── client_service.py           # Logique métier et validation
│
├── controllers/
│   └── client_controller.py        # Routes Flask (Blueprint) + API JSON
│
├── templates/
│   ├── index.html                  # Page d'accueil — tableau de bord
│   ├── ajouter_client.html         # Page d'ajout de client
│   ├── liste_clients.html          # Page de liste (lecture seule)
│   └── gerer_clients.html          # Page de gestion (modification/suppression)
│
└── static/
    ├── css/
    │   └── style.css               # Feuille de style globale (Dark Mode)
    └── js/
        └── main.js                 # Utilitaires JS : API helpers, toast, rendu tableau
```

---

## 🗄 Base de données

### Initialisation

Au démarrage de l'application (`app.py`), la fonction `init_db()` est appelée automatiquement. Elle crée la table `clients` si elle n'existe pas déjà, grâce à `CREATE TABLE IF NOT EXISTS`.

### Schéma de la table `clients`

```sql
CREATE TABLE IF NOT EXISTS clients (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,  -- Clé primaire interne (invisible en UI)
    numero      INTEGER UNIQUE,                     -- Numéro d'ordre affiché à l'utilisateur (recalculé)
    nom         TEXT NOT NULL,                      -- Nom complet du client
    identifiant TEXT NOT NULL UNIQUE,               -- Identifiant unique (ex : JD-001)
    telephone   TEXT NOT NULL,                      -- Numéro de téléphone
    gmail       TEXT NOT NULL                       -- Adresse email Gmail
);
```

### Distinction `id` vs `numero`

| Champ | Rôle | Comportement |
|---|---|---|
| `id` | Clé primaire SQL, utilisée en interne pour les opérations | Jamais réutilisé après suppression (AUTOINCREMENT) |
| `numero` | Numéro d'ordre affiché à l'utilisateur | **Recalculé** après chaque suppression pour rester continu (1, 2, 3…) |

### Connexion SQLite

```python
conn = sqlite3.connect(DB_PATH)
conn.row_factory = sqlite3.Row   # Les lignes sont accessibles par nom de colonne
```

Le chemin de la base (`DB_PATH`) est construit dynamiquement via `os.path.dirname(__file__)`, garantissant que le fichier `.db` est toujours dans le dossier `database/`, peu importe le répertoire de lancement.

---

## 🌐 Routes et API

### Pages (vues HTML)

| Méthode | Route | Template rendu | Description |
|---|---|---|---|
| `GET` | `/` | `index.html` | Tableau de bord principal |
| `GET` | `/clients/ajouter` | `ajouter_client.html` | Formulaire d'ajout |
| `GET` | `/clients/liste` | `liste_clients.html` | Liste en lecture seule |
| `GET` | `/clients/gerer` | `gerer_clients.html` | Gestion (édition/suppression) |

### API REST JSON

| Méthode | Route | Description | Corps (JSON) | Réponse |
|---|---|---|---|---|
| `GET` | `/clients/api/all` | Récupère tous les clients | — | `[{id, numero, nom, identifiant, telephone, gmail}, ...]` |
| `POST` | `/clients/api/add` | Ajoute un client | `{nom, identifiant, telephone, gmail}` | `{success, message}` |
| `PUT` | `/clients/api/update/<id>` | Modifie un client existant | `{nom, identifiant, telephone, gmail}` | `{success, message}` |
| `DELETE` | `/clients/api/delete/<id>` | Supprime un client | — | `{success, message}` |

**Exemple de réponse API (succès) :**
```json
{ "success": true, "message": "Client ajouté avec succès." }
```

**Exemple de réponse API (erreur) :**
```json
{ "success": false, "message": "Cet identifiant est déjà utilisé." }
```

---

## 🔬 Fonctionnement couche par couche

### 1. `database/db.py` — Couche connexion

```python
def get_connection():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row   # Accès aux colonnes par nom
    return conn
```

Chaque opération ouvre sa propre connexion et la ferme après usage (`conn.close()`). C'est un pattern simple adapté à une application mono-utilisateur SQLite.

---

### 2. `models/client.py` — Modèle de données

```python
class Client:
    def __init__(self, id, numero, nom, identifiant, telephone, gmail):
        ...

    def to_dict(self):
        # Sérialise l'objet en dictionnaire → converti en JSON par Flask
        return { "id": ..., "numero": ..., ... }

    @staticmethod
    def from_row(row):
        # Construit un objet Client depuis une ligne SQLite
        return Client(id=row["id"], ...)
```

Le modèle encapsule les données et propose deux conversions : vers JSON (`to_dict`) et depuis une ligne de base de données (`from_row`).

---

### 3. `repositories/client_repository.py` — Couche données

Méthodes principales :

**`get_all()`** — Récupère tous les clients triés par `numero ASC`.

**`get_by_id(client_id)`** — Récupère un client par son `id` interne.

**`add(nom, identifiant, telephone, gmail)`** — Calcule automatiquement le prochain `numero` :
```python
cursor.execute("SELECT COALESCE(MAX(numero), 0) + 1 FROM clients")
next_numero = cursor.fetchone()[0]
```

**`update(client_id, ...)`** — Met à jour tous les champs sauf `id` et `numero`.

**`delete(client_id)`** — Supprime le client, puis **renuméroté tous les clients restants** :
```python
cursor.execute("SELECT id FROM clients ORDER BY id ASC")
for index, row in enumerate(rows, start=1):
    cursor.execute("UPDATE clients SET numero = ? WHERE id = ?", (index, row["id"]))
```

**`identifiant_exists(identifiant, exclude_id=None)`** — Vérifie qu'un identifiant n'est pas déjà pris (avec exclusion du client courant lors d'une modification).

---

### 4. `services/client_service.py` — Logique métier

Le service ajoute une couche de **validation et de règles métier** avant de déléguer au repository :

```python
def add_client(self, nom, identifiant, telephone, gmail):
    # 1. Nettoyage des espaces
    nom = nom.strip()
    ...
    # 2. Vérification des champs obligatoires
    if not nom or not identifiant or ...:
        return {"success": False, "message": "Tous les champs sont obligatoires."}
    # 3. Vérification d'unicité de l'identifiant
    if repository.identifiant_exists(identifiant):
        return {"success": False, "message": "Cet identifiant est déjà utilisé."}
    # 4. Insertion en base
    repository.add(nom, identifiant, telephone, gmail)
    return {"success": True, "message": "Client ajouté avec succès."}
```

Chaque méthode retourne un dictionnaire `{success, message}` directement sérialisable en JSON par le contrôleur.

---

### 5. `controllers/client_controller.py` — Blueprint Flask

Le Blueprint est monté avec le préfixe `/clients` :

```python
client_bp = Blueprint("client", __name__, url_prefix="/clients")
```

Les routes de **pages** retournent un template Jinja2 :
```python
@client_bp.route("/liste")
def liste():
    return render_template("liste_clients.html")
```

Les routes **API** lisent le JSON entrant et retournent du JSON :
```python
@client_bp.route("/api/add", methods=["POST"])
def api_add():
    data = request.get_json()
    result = service.add_client(...)
    return jsonify(result)
```

---

## 🖥 Frontend — Interfaces et comportements

### `static/js/main.js` — Bibliothèque partagée

Fichier chargé sur toutes les pages (sauf `index.html`). Il expose :

**Helpers API (wrappers autour de `fetch`) :**
```javascript
apiGet(url)           // → fetch GET, retourne JSON
apiPost(url, data)    // → fetch POST avec Content-Type: application/json
apiPut(url, data)     // → fetch PUT avec JSON
apiDelete(url)        // → fetch DELETE
```

**`renderClientsTable(clients, tbodyId, readOnly)`** — Génère les lignes HTML du tableau. En mode `readOnly = true` (page liste), n'affiche pas la colonne "Actions". En mode `readOnly = false` (page gestion), ajoute les boutons ✏️ Modifier et 🗑️ Supprimer sur chaque ligne.

**`escHtml(str)`** — Échappe les caractères HTML spéciaux (`&`, `<`, `>`, `"`) pour prévenir les injections XSS lors de l'affichage des données utilisateur.

**`showToast(message, type)`** — Crée dynamiquement une notification flottante en haut à droite. Elle disparaît automatiquement après 3,5 secondes.

---

### Page `index.html` — Tableau de bord

Page d'accueil statique avec trois cartes cliquables menant aux trois fonctionnalités principales. Aucun appel API. Pas de script JS interne.

---

### Page `ajouter_client.html` — Ajout de client

- Formulaire avec 4 champs : Nom, Identifiant, Téléphone, Gmail.
- Au clic sur le bouton, la fonction `submitForm()` :
  1. Lit et valide les champs (côté client : aucun champ vide)
  2. Envoie un `POST /clients/api/add` via `apiPost()`
  3. Affiche un toast selon le résultat
  4. Réinitialise le formulaire en cas de succès

---

### Page `liste_clients.html` — Consultation

- Affiche un spinner de chargement pendant le chargement des données.
- Appelle `GET /clients/api/all` via `apiGet()`.
- Affiche le nombre de clients dans le sous-titre.
- Génère le tableau en lecture seule avec `renderClientsTable(..., true)`.

---

### Page `gerer_clients.html` — Gestion complète

C'est la page la plus complexe :

**Chargement initial :**
```javascript
async function loadClients() {
    const clients = await apiGet("/clients/api/all");
    clientsCache = clients;  // Cache local pour les modales
    renderClientsTable(clients, "tbody-clients", false);
}
```

**Modification (modale Edit) :**
1. Clic sur ✏️ → `openEditModal(id)` récupère les données depuis `clientsCache`
2. Pré-remplit la modale avec les données actuelles du client
3. Soumission → `apiPut()` vers `/clients/api/update/<id>`
4. Rechargement du tableau et fermeture de la modale en cas de succès

**Suppression (modale Delete) :**
1. Clic sur 🗑️ → `confirmDelete(id, nom)` affiche la modale de confirmation avec le nom du client
2. Confirmation → `apiDelete()` vers `/clients/api/delete/<id>`
3. Rechargement du tableau après succès

**Fermeture des modales :**
- Bouton "Annuler" → `closeModal(id)`
- Clic en dehors de la modale → même effet (événement sur `.modal-overlay`)

---

## 🎨 Style et design (CSS)

Le fichier `static/css/style.css` utilise un système de **variables CSS** pour toute la palette :

```css
:root {
  --bg-main:       #0f1117;   /* Fond principal (noir bleuté) */
  --bg-card:       #1a1d27;   /* Fond des cartes/tableaux */
  --bg-card2:      #22263a;   /* Fond alternatif (thead, hover) */
  --border:        #2e3347;   /* Couleur des bordures */
  --accent:        #4f8ef7;   /* Bleu principal (liens, focus, accent) */
  --danger:        #e05252;   /* Rouge (suppression) */
  --success:       #3ecf8e;   /* Vert (succès) */
  --text-primary:  #e8eaf0;   /* Texte principal */
  --text-secondary:#8b90a4;   /* Texte secondaire */
  --text-muted:    #555b72;   /* Texte atténué */
}
```

**Composants CSS clés :**

| Composant | Classe(s) | Description |
|---|---|---|
| Navbar fixe | `.navbar`, `.navbar-brand`, `.navbar-back` | Barre de navigation sticky en haut |
| Cartes d'accueil | `.cards-grid`, `.card-action` | Grille responsive, effet hover (translateY + glow) |
| Formulaire | `.form-card`, `.form-group`, `input:focus` | Carte centrée, champs avec focus bleu |
| Boutons | `.btn`, `.btn-primary`, `.btn-danger`, `.btn-ghost`, `.btn-sm` | Système de boutons avec variantes |
| Tableau | `.table-wrapper`, `thead`, `tbody tr:hover` | Tableau dark avec hover row |
| Badge numéro | `.badge-num` | Pastille bleue pour afficher le `#` |
| Toast | `.toast-container`, `.toast`, `.toast.error`, `.toast.success` | Notifications animées (slideIn) |
| Modales | `.modal-overlay`, `.modal`, `.modal-overlay.open` | Overlay avec animation fadeUp |
| Loader | `.loader::after` | Spinner CSS pur (border-top animé) |
| État vide | `.empty-state` | Affichage centré quand aucune donnée |

**Animations CSS :**
- `@keyframes slideIn` — Entrée des toasts depuis la droite
- `@keyframes fadeUp` — Entrée des modales depuis le bas
- `@keyframes spin` — Rotation du spinner de chargement

**Responsive (breakpoint 640px) :**
```css
@media (max-width: 640px) {
    .container { padding: 1.5rem 1rem; }
    th, td     { font-size: 0.82rem; }
    .form-card { padding: 1.5rem 1rem; }
}
```

---

## 🚀 Installation et lancement

### Prérequis

- Python 3.9 ou supérieur
- `pip`

### Étapes

```bash
# 1. Cloner ou extraire le projet
cd "Projet Celina"

# 2. Créer un environnement virtuel (recommandé)
python -m venv venv
source venv/bin/activate        # macOS / Linux
venv\Scripts\activate           # Windows

# 3. Installer Flask
pip install flask

# 4. Lancer l'application
python app.py
```

L'application démarre sur **http://127.0.0.1:5000** en mode debug.

La base de données `database/clients.db` est **créée automatiquement** au premier lancement si elle n'existe pas.

### Variables d'environnement (optionnel)

Pour désactiver le mode debug en production :
```bash
export FLASK_DEBUG=0
python app.py
```

---

## 🔄 Flux de données complet

### Exemple : Ajout d'un client

```
Utilisateur remplit le formulaire (ajouter_client.html)
        │
        ▼
[JS] submitForm() valide les champs côté client
        │
        ▼
[JS] apiPost("/clients/api/add", {nom, identifiant, telephone, gmail})
        │  HTTP POST JSON
        ▼
[Controller] api_add() reçoit la requête
        │  data = request.get_json()
        ▼
[Service] add_client(nom, identifiant, telephone, gmail)
    ├── strip() des valeurs
    ├── Vérifie champs non vides → {success: false} si manquant
    ├── Vérifie unicité identifiant → {success: false} si doublon
    └── repository.add(...)
        │
        ▼
[Repository] add(nom, identifiant, telephone, gmail)
    ├── Calcule next_numero = MAX(numero) + 1
    └── INSERT INTO clients VALUES (next_numero, nom, ...)
        │
        ▼
[SQLite] Ligne insérée dans clients.db
        │
        ▼
[Service] → {success: true, message: "Client ajouté avec succès."}
        │
        ▼
[Controller] → jsonify({success: true, message: ...})
        │  HTTP 200 JSON
        ▼
[JS] showToast("Client ajouté avec succès.", "success")
     Réinitialisation du formulaire
```

### Exemple : Suppression avec renumérotation

```
[Repository] delete(client_id)
    1. DELETE FROM clients WHERE id = ?
    2. SELECT id FROM clients ORDER BY id ASC
    3. Pour chaque ligne (index commençant à 1) :
       UPDATE clients SET numero = index WHERE id = row_id
       → Les numéros restent continus : 1, 2, 3, ...
```

---

## 📝 Notes techniques

- **Sécurité XSS** : La fonction `escHtml()` dans `main.js` échappe toutes les données utilisateur avant insertion dans le DOM.
- **Identifiant unique** : L'identifiant est soumis à une contrainte `UNIQUE` en base et vérifié côté service avant toute opération.
- **Pas de framework JS** : L'application utilise uniquement l'API `fetch` native et la manipulation du DOM standard, sans dépendance externe côté frontend.
- **Connexions courtes** : Chaque opération SQL ouvre et ferme sa propre connexion — adapté à SQLite en usage mono-utilisateur local.
- **Cache côté client** : La page `gerer_clients.html` stocke le résultat de l'API dans `clientsCache` pour alimenter les modales sans refaire d'appel réseau.
