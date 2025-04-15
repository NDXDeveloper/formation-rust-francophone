## 2\. Mise en place des outils

La mise en place d'un environnement de développement Rust efficace est une étape cruciale pour démarrer sereinement. Rust propose un écosystème d'outils robustes et bien intégrés qui vous accompagneront tout au long de votre parcours de développement.

### Installation de Rust

#### Rustup : le gestionnaire d'installation officiel

[Rustup](https://rustup.rs/) est l'outil officiel recommandé pour installer et gérer les versions de Rust. Il fonctionne sur tous les systèmes d'exploitation majeurs (Windows, macOS, Linux).

Pour installer Rust via Rustup :

**Sur Linux et macOS** :

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

**Sur Windows** :

1.  Téléchargez l'installateur depuis [rustup.rs](https://rustup.rs/)
2.  Exécutez-le et suivez les instructions

Cette installation vous fournit :

- `rustc` : le compilateur Rust
- `cargo` : le gestionnaire de paquets et outil de build
- `rustup` : l'outil de gestion des versions de Rust
- La documentation standard

Après l'installation, redémarrez votre terminal ou exécutez :

```
source ~/.cargo/env  # sur Linux/macOS
```

Pour vérifier que tout est correctement installé :

```
rustc --version
cargo --version
```

#### Chaînes de compilation (toolchains)

Rustup permet de gérer plusieurs versions de Rust simultanément :

```
# Installer la version nightly
rustup install nightly

# Définir la version par défaut
rustup default stable

# Utiliser une version spécifique pour un projet
rustup override set nightly
```

### Environnements de développement

Contrairement à d'autres langages, Rust ne privilégie pas un IDE spécifique. Plusieurs options s'offrent à vous :

#### 1\. IDEs dédiés

##### RustRover : l'IDE spécialisé de JetBrains

[RustRover](https://www.jetbrains.com/rust/) est un IDE complet conçu spécifiquement pour Rust par JetBrains. Il offre plusieurs avantages notables :

- **Intégration complète** : Tous les outils Rust (cargo, rustfmt, clippy) sont intégrés nativement [\[1\]](https://www.jetbrains.com/help/rust/2024.2/how-to-move-to-rustrover-from-vs-code.html#migration_from_vscode_to_set_up_work_environment)
- **Disponibilité multi-plateforme** : Windows, macOS, et Linux [\[2\]](https://www.jetbrains.com/help/rust/2024.2/installation-guide.html)
- **Gratuit pour usage non-commercial** : Idéal pour débutants et projets personnels [\[2\]](https://www.jetbrains.com/help/rust/2024.2/installation-guide.html)

**Installation et configuration** :

1.  Téléchargez-le depuis [jetbrains.com/rust](https://www.jetbrains.com/rust/)
2.  Installez-le en suivant l'assistant
3.  Au premier démarrage, l'IDE détectera automatiquement votre installation Rust ou vous guidera pour l'installer

**Fonctionnalités principales** :

- Complétion de code intelligente
- Analyse de code en temps réel
- Debugging intégré avec LLDB
- Refactoring avancé
- Navigation de code intuitive
- Gestion de projet intégrée avec Cargo
- Intégration Git native
- Terminal intégré

**Personnalisation** : RustRover propose une large gamme d'options de personnalisation [\[3\]](https://www.jetbrains.com/help/rust/2024.2/quick-start-guide-rustrover.html#settings) :

```
File → Settings → Editor → Code Style → Rust  # Pour le style de code
File → Settings → Build, Execution, Deployment → Debugger  # Configuration du débogueur
File → Settings → Editor → Live Templates  # Modèles de code personnalisés
```

**Avantages par rapport à d'autres environnements** :

- **Par rapport à VS Code** : Ne nécessite pas d'extensions supplémentaires, tout est intégré nativement [\[1\]](https://www.jetbrains.com/help/rust/2024.2/how-to-move-to-rustrover-from-vs-code.html#migration_from_vscode_to_set_up_work_environment)
- **Par rapport à d'autres IDEs JetBrains** : Fonctionnalités ciblées pour Rust sans surcharge d'outils inutiles pour d'autres langages [\[4\]](https://www.jetbrains.com/help/rust/2024.2/how-to-move-to-rustrover-from-another-jetbrains-ide.html#migration_from_jb_ide_difference)

**Configuration recommandée pour les débutants** :

```
File → Settings → Editor → General → Auto Import → Add unambiguous imports on the fly  # Pour l'auto-import
File → Settings → Tools → Actions on Save → Reformat code  # Format automatique
File → Settings → Tools → Actions on Save → Run code cleanup  # Cleanup automatique
```

**Raccourcis clavier essentiels** :

- `Ctrl` + `Space` : Auto-complétion

- `Alt` + `Enter` : Actions rapides et correctifs

- `Shift` + `F10` : Exécuter le programme

- `F8` : Accéder à l'erreur suivante

- `Ctrl` + `B` : Aller à la définition

- **Visual Studio Code** avec l'extension "rust-analyzer" :


```
code --install-extension rust-lang.rust-analyzer
```

#### 2\. Éditeurs de texte avec extensions Rust

- **VS Code** (le plus populaire dans la communauté Rust)

- **Sublime Text** avec le package "Rust Enhanced" :

    1.  Installez Package Control
    2.  Commande Palette → Package Control: Install Package → "Rust Enhanced"
    3.  Pour une expérience complète, ajoutez aussi "RustFmt" et "LSP"
- **Vim/Neovim** avec les plugins rust.vim et rust-analyzer


```
" Exemple pour vim-plug
Plug 'rust-lang/rust.vim'
Plug 'neovim/nvim-lspconfig'
```

- **Emacs** avec rust-mode et rustic

#### 3\. Environnement en ligne

Pour tester rapidement du code Rust sans installation :

- [Rust Playground](https://play.rust-lang.org/) : idéal pour partager du code ou tester des concepts
- [Replit](https://replit.com/) : environnement de développement complet en ligne

### Cargo : le couteau suisse de Rust

Cargo est beaucoup plus qu'un simple gestionnaire de paquets - c'est l'outil central de l'écosystème Rust.

#### Commandes essentielles

``` bash
# Créer un nouveau projet
cargo new mon_projet
cargo new --lib ma_bibliotheque

# Compiler un projet
cargo build           # en mode debug
cargo build --release # en mode optimisé pour la production

# Exécuter un projet
cargo run             # compile si nécessaire puis exécute

# Tester
cargo test            # exécute tous les tests

# Vérifier la compilation sans produire d'exécutable
cargo check           # plus rapide que cargo build
```

#### Structure d'un projet Cargo

```
mon_projet/
├── Cargo.toml         # Fichier de configuration du projet
├── Cargo.lock         # Verrouillage des versions des dépendances
├── src/               # Code source
│   └── main.rs        # Point d'entrée pour les binaires
│   └── lib.rs         # Point d'entrée pour les bibliothèques
├── tests/             # Tests d'intégration
├── benches/           # Tests de performance
├── examples/          # Exemples d'utilisation
└── target/            # Dossier des builds (généré)
```

#### Cargo.toml : le manifeste du projet

``` toml
[package]
name = "mon_projet"
version = "0.1.0"
edition = "2021"
authors = ["Votre Nom <email@exemple.com>"]
description = "Une courte description du projet"

[dependencies]
# Dépendances externes
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }

[dev-dependencies]
# Dépendances utilisées uniquement pour les tests
criterion = "0.5"

[profile.release]
# Configuration pour les builds optimisés
opt-level = 3
lto = true
```

### Outils complémentaires

Rust fournit un ensemble d'outils additionnels qui améliorent considérablement l'expérience de développement :

#### Formatage et lint

- **Rustfmt** : formatte automatiquement votre code selon les conventions officielles

``` bash
# Installation
rustup component add rustfmt

# Utilisation
cargo fmt
```

- **Clippy** : linter qui suggère des améliorations et détecte des problèmes potentiels

```
# Installation
rustup component add clippy

# Utilisation
cargo clippy
```

#### Documentation

- **Rustdoc** : générateur de documentation (intégré à Cargo)

``` bash
# Générer et ouvrir la documentation de votre projet
cargo doc --open

# Générer la documentation des dépendances aussi
cargo doc --open --document-private-items
```

#### Gestion des versions et dépendances

- **Cargo-edit** : ajoute des commandes pour gérer les dépendances

``` bash
# Installation
cargo install cargo-edit

# Utilisation
cargo add serde --features derive
cargo rm regex
cargo upgrade
```

#### Tests et benchmarks

- **Cargo-tarpaulin** : couverture de code

``` bash
cargo install cargo-tarpaulin
cargo tarpaulin
```

- **Criterion** : framework de benchmark

``` toml
[dev-dependencies]
criterion = "0.5"

[[bench]]
name = "mon_benchmark"
harness = false
```

#### Optimisation et analyse

- **Cargo-bloat** : analyse la taille des artefacts

``` bash
cargo install cargo-bloat
cargo bloat --release
```

- **Cargo-expand** : affiche le code après expansion des macros

``` bash
cargo install cargo-expand
cargo expand
```

### Configuration pour les débutants

Si vous débutez avec Rust, voici une configuration recommandée :

1.  **Installez Rust** via Rustup
2.  **Choisissez un IDE adapté** :
    - RustRover pour une expérience complète sans configuration complexe
    - VS Code avec l'extension rust-analyzer pour une solution légère
3.  **Installez les composants additionnels** :

```
rustup component add rustfmt clippy
```

4.  **Activez les paramètres d'IDE pour un développement efficace** :
    - Formatage automatique à la sauvegarde
    - Vérification du code avec Clippy

### Ressources pour les outils

- [Cargo Book](https://doc.rust-lang.org/cargo/) : documentation complète de Cargo
- [Rustup Book](https://rust-lang.github.io/rustup/) : guide de Rustup
- [Awesome Rust](https://github.com/rust-unofficial/awesome-rust) : liste d'outils et bibliothèques
- [Documentation RustRover](https://www.jetbrains.com/help/rust/) : guide complet de l'IDE

### Conseils pour les différents systèmes d'exploitation

#### Windows

- Installez les Visual Studio Build Tools si vous rencontrez des problèmes de compilation
- Utilisez Git Bash ou Windows Terminal pour une meilleure expérience en ligne de commande
- Le WSL (Windows Subsystem for Linux) offre une excellente alternative pour développer en Rust sous Windows

#### macOS

- Installez les outils de développement XCode en exécutant `xcode-select --install`
- Homebrew peut être utilisé comme alternative pour installer Rust : `brew install rust`

#### Linux

- Installez les packages de développement nécessaires :

``` bash
# Debian/Ubuntu
sudo apt install build-essential pkg-config libssl-dev

# Fedora
sudo dnf install gcc openssl-devel

# Arch Linux
sudo pacman -S base-devel openssl
```

Maintenant que votre environnement de développement est configuré, vous êtes prêt à écrire votre premier programme Rust !
