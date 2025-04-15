 
## 13\. Cargo

## Introduction à Cargo

Cargo est le gestionnaire de paquets et système de build officiel de Rust. Il simplifie considérablement la gestion des projets en automatisant de nombreuses tâches comme la compilation, la gestion des dépendances, les tests et la distribution de code.

## Création d'un projet

Pour créer un nouveau projet Rust avec Cargo :

```
# Créer un projet binaire
cargo new mon_projet

# Créer une bibliothèque
cargo new ma_bibliotheque --lib
```

Cette commande génère une structure de projet standard :

```
mon_projet/
├── Cargo.toml  # Fichier de configuration du projet
├── .gitignore  # Configuration git par défaut
└── src/
    └── main.rs # Point d'entrée du programme (ou lib.rs pour une bibliothèque)
```

## Le fichier Cargo.toml

Le fichier `Cargo.toml` est le cœur de la configuration d'un projet Rust, écrit au format [TOML](https://toml.io/). Il contient :

```
[package]
name = "mon_projet"         # Nom du projet
version = "0.1.0"           # Version selon SemVer
edition = "2021"            # Édition de Rust (2015, 2018, 2021)
authors = ["Votre Nom <votre.email@exemple.com>"]  # Optionnel
description = "Une description de mon projet"      # Optionnel
license = "MIT"                                    # Optionnel
repository = "https://github.com/vous/mon_projet"  # Optionnel

[dependencies]
# Liste des dépendances
sha2 = "0.10.8"
crossbeam-channel = "0.5.15"
```

## Commandes principales de Cargo

Voici les commandes Cargo les plus utilisées :

```
cargo build            # Compile le projet
cargo build --release  # Compile en mode optimisé pour la production
cargo run              # Compile et exécute le projet
cargo test             # Exécute les tests
cargo check            # Vérifie la compilation sans produire d'exécutable
cargo update           # Met à jour les dépendances
cargo doc              # Génère la documentation
cargo doc --open       # Génère et ouvre la documentation dans le navigateur
cargo clean            # Supprime les fichiers générés
cargo publish          # Publie la bibliothèque sur crates.io
```

## Profils de compilation

Cargo propose différents profils de compilation pour adapter les optimisations :

```
[profile.dev]          # Compilation rapide pour le développement
opt-level = 0          # Niveau d'optimisation (0-3)
debug = true           # Inclut les informations de débogage

[profile.release]      # Optimisations pour la production
opt-level = 3          # Optimisations maximales
debug = false          # Pas d'informations de débogage
lto = true             # Link-time optimization
codegen-units = 1      # Optimise davantage mais compile plus lentement
```

## Dépendances conditionnelles et fonctionnalités

Vous pouvez activer des fonctionnalités spécifiques ou définir des dépendances conditionnelles :

```
[dependencies]
sha2 = { version = "0.10.8", optional = true }
md5 = { version = "0.7.0", features = ["std"] }

[target.'cfg(windows)'.dependencies]
winapi = "0.3.9"

[features]
default = ["sha256"]  # Fonctionnalités activées par défaut
sha256 = ["sha2"]     # La fonctionnalité sha256 active la dépendance sha2
all = ["sha256", "md5"] # Combinaison de fonctionnalités
```

Pour utiliser des fonctionnalités spécifiques :

```
cargo build --features "sha256 md5"
```

## Espaces de travail (Workspaces)

Les espaces de travail permettent de gérer plusieurs paquets connexes dans un seul projet :

```
# Dans le fichier Cargo.toml racine
[workspace]
members = [
    "core",
    "utils",
    "cli",
]
```

Cette structure facilite le développement de projets modulaires tout en partageant le répertoire `target/` pour les builds.

## Gestion de version des dépendances

Cargo utilise la notation [SemVer](https://semver.org/) pour spécifier les versions :

| Spécification | Signification |
| --- | --- |
| `"1.2.3"` | Exactement la version 1.2.3 |
| `"^1.2.3"` | ≥1.2.3, <2.0.0 (compatible avec 1.x.y) |
| `"~1.2.3"` | ≥1.2.3, <1.3.0 (patchs uniquement) |
| `"*"` | Toute version (déconseillé) |
| `">=1.2.0, <1.4.0"` | Plage explicite de versions |

Par exemple :

```
[dependencies]
sha2 = "0.10"         # Accepte 0.10.x
crossbeam = "~0.5.15" # Accepte 0.5.15 à 0.5.z
serde = ">=1.0.0, <2.0.0" # Entre 1.0.0 et 2.0.0 exclus
```

## Fichier Cargo.lock

Le fichier `Cargo.lock` enregistre les versions exactes des dépendances utilisées pour assurer une compilation reproductible. Pour les applications, il devrait être inclus dans le contrôle de version (git), tandis que pour les bibliothèques, il devrait être ignoré.

## Cargo script et outils supplémentaires

### cargo-edit

Le plugin `cargo-edit` facilite la gestion des dépendances en ligne de commande :

```
# Installation
cargo install cargo-edit

# Utilisation
cargo add serde --features derive
cargo rm unused-dependency
cargo upgrade
```

### cargo-watch

Utile pour le développement, il relance les commandes quand les fichiers changent :

```
cargo install cargo-watch
cargo watch -x test -x run
```

### cargo-expand

Pour déboguer les macros, vous pouvez voir le code généré :

```
cargo install cargo-expand
cargo expand
```

## Publication sur crates.io

Pour publier votre crate sur le registre officiel, assurez-vous d'avoir des métadonnées complètes :

```
[package]
name = "ma_bibliotheque"
version = "0.1.0"
description = "Une description claire de ma bibliothèque"
authors = ["Votre Nom <email@exemple.com>"]
license = "MIT OR Apache-2.0"
repository = "https://github.com/vous/ma_bibliotheque"
documentation = "https://docs.rs/ma_bibliotheque"
readme = "README.md"
keywords = ["mot-clef1", "mot-clef2"]
categories = ["development-tools"]
```

Ensuite, connectez-vous et publiez :

```
cargo login <votre-token-api>
cargo publish
```

**Important** : Une fois une version publiée, elle ne peut pas être modifiée ou supprimée. Vérifiez soigneusement avant de publier.
