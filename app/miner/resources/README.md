# Utilisation

## Exécution

Pour utiliser **xcraft-miner**, vous pouvez l'exécuter directement via npx ou bunx sans installation préalable :

```bash
npx xcraft-miner@latest
```

ou

```bash
bunx xcraft-miner@latest
```

L'application utilise l'intelligence artificielle pour générer automatiquement de la documentation technique à partir du code source de vos projets. Elle supporte les projets Xcraft, .NET et C++, et peut également effectuer des traductions de documents.

## Paramètres

L'application accepte plusieurs paramètres en ligne de commande :

### `-t, --type`

Spécifie le type de projet à analyser ou d'opération à effectuer.

- **Valeurs supportées** : `xcraft`, `dotnet`, `cxx`, `translate`
- **Par défaut** : `xcraft`

### `-p, --provider`

Fournisseur d'IA à utiliser.

- **Valeurs supportées** : `open-ai`, `ollama`
- **Par défaut** : `open-ai`
- **Peut être surchargé** par la configuration du module

### `-m, --model`

Modèle d'IA à utiliser.

- **Par défaut** : `anthropic/claude-sonnet-4`
- **Peut être surchargé** par la configuration du module

### `-H, --host`

URL de l'hôte du fournisseur d'IA.

- **Par défaut** : `https://openrouter.ai/api/v1`
- **Peut être surchargé** par la configuration du module

### `-k, --authKey`

Clé d'authentification pour le fournisseur d'IA.

- **Requis** : Oui (sauf si configuré dans le fichier de configuration)

### `-T, --temperature`

Contrôle la créativité de l'IA (valeur entre 0 et 1).

- **Par défaut** : `0.2`
- **Peut être surchargé** par la configuration du module

### `-s, --seed`

Graine pour la génération déterministe.

- **Par défaut** : `21121871`
- **Peut être surchargé** par la configuration du module

### `-i, --input`

Chemin vers le module ou dossier source à analyser, ou fichier à traduire.

- **Format** : Chemin absolu ou relatif
- **Usage** : Pour la documentation, spécifie le module à analyser. Pour la traduction, spécifie le fichier source à traduire.

### `-o, --output`

Nom du fichier de sortie pour la documentation générée.

- **Format** : Nom de fichier (ex: `README.md`, `API.md`)
- **Usage** : Pour la documentation, définit le type de document à générer. Pour la traduction, définit le fichier de sortie.

## Configurations

### Configuration par module

Pour configurer la génération de documentation pour un module spécifique, vous devez créer une structure de dossiers dans le répertoire `doc/autogen/` de votre module :

#### Structure des dossiers

```
votre-module/
├── doc/
│   └── autogen/
│       ├── prompts/
│       │   ├── README.md
│       │   └── autre-doc.md
│       ├── instructions/
│       │   ├── README.md
│       │   └── autre-doc.md
│       └── filters/
│           ├── README.js
│           └── autre-doc.js
└── .mignore
```

**Note importante** : Le type de projet (`xcraft`, `dotnet`, `cxx`, `translate`) ne fait pas partie du chemin des dossiers dans le module cible. Les fichiers sont organisés directement par nom de document.

#### Types de fichiers de configuration

**Prompts (`prompts/`)**

Contiennent les instructions principales pour l'IA concernant le type de documentation à générer. Le système recherche automatiquement les fichiers dans l'ordre suivant :

1. `doc/autogen/prompts/[nom-document].md` dans le module cible
2. Ressources de l'application : `prompts/[type]/[nom-document].md`
3. Module goblin-miner : `prompts/[type]/[nom-document].md`
4. Fichier de base du goblin-miner : `prompts/[type]/base.md`

**Instructions (`instructions/`)**

Contiennent des instructions spécifiques supplémentaires pour affiner la génération. Même logique de recherche que pour les prompts.

**Filtres (`filters/`)**

Modules JavaScript qui définissent quels fichiers inclure ou exclure de l'analyse. Même logique de recherche que pour les prompts, mais avec l'extension `.js`. Les filtres doivent exporter une fonction qui retourne `true` pour exclure un fichier.

#### Fichier `.mignore`

Permet d'exclure des fichiers ou dossiers de l'analyse. Supporte également des sections conditionnelles avec la syntaxe `[nom-fichier.md]` pour appliquer des exclusions spécifiques selon le document généré.

Exemple de `.mignore` :

```
# Exclusions globales
node_modules/
test/

# Exclusions spécifiques pour README.md
[README.md]
examples/
internal/
foobar.js

# Exclusions pour API.md
[API.md]
private/
```

### Filtres par type de projet

L'application applique automatiquement des filtres selon le type de projet :

**Type `xcraft`** : Exclut automatiquement

- Les fichiers non-JavaScript (sauf `package.json` et les fichiers dans `bin/`)
- Les fichiers dans `node_modules`
- Les fichiers commençant par `eslint.`

**Type `dotnet`** : Inclut uniquement

- Les fichiers `.cs`
- Les fichiers `.csproj`
- Les fichiers `.sln`

**Type `cxx`** : Inclut uniquement

- Les fichiers `Makefile`
- Les fichiers `CMakeLists.txt`
- Les fichiers `.c`, `.cxx`, `.cpp`
- Les fichiers `.h`, `.hxx`
- Les fichiers `.cxproj`, `.sln`

**Type `translate`** : Traite le fichier source spécifié sans filtrage particulier

## Exemples

### Génération basique pour un projet Xcraft

```bash
npx xcraft-miner@latest -k "votre-cle-api"
```

### Génération pour un module spécifique

```bash
npx xcraft-miner@latest -t xcraft -k "votre-cle-api" -i "./lib/mon-module" -o "README.md"
```

### Génération pour un projet .NET

```bash
npx xcraft-miner@latest -t dotnet -k "votre-cle-api" -i "/chemin/vers/projet" -o "README.md"
```

### Génération pour un projet C++

```bash
npx xcraft-miner@latest -t cxx -k "votre-cle-api" -i "/chemin/vers/projet" -o "README.md"
```

### Traduction d'un document

```bash
npx xcraft-miner@latest -t translate -k "votre-cle-api" -i "/chemin/vers/document.md" -o "document.en.md"
```

### Utilisation avec un fournisseur personnalisé

```bash
npx xcraft-miner@latest -p "open-ai" -m "anthropic/claude-3-sonnet" -H "https://api.anthropic.com" -k "votre-cle-api"
```

### Génération avec Ollama (local)

```bash
npx xcraft-miner@latest -p "ollama" -m "llama3.2" -H "http://localhost:11434/v1" -k "dummy"
```

### Utilisation avec paramètres d'inférence personnalisés

```bash
npx xcraft-miner@latest -k "votre-cle-api" -T 0.7 -s 42 -i "./lib/mon-module"
```

### Génération de documentation API

```bash
npx xcraft-miner@latest -k "sk-..." -i "/home/user/projets/mon-module" -o "API.md"
```

L'application analyse automatiquement les fichiers source du module spécifié, applique les filtres configurés selon le type de projet, et génère une documentation complète en utilisant l'intelligence artificielle selon les prompts et instructions définis. Pour la traduction, elle traite le fichier source et génère une version traduite avec le suffixe de langue approprié.