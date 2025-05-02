## 12\. Multi-fichier

Retour à la [Table des matières](/SOMMAIRE.md)

L'organisation du code en plusieurs fichiers est essentielle pour maintenir un projet propre et maintenable à mesure qu'il grandit. Rust offre un système de modules puissant mais qui peut parfois sembler déroutant pour les débutants.

### Principes fondamentaux

En Rust, l'unité d'organisation du code est le **module**. Un module peut être défini dans le même fichier ou dans un fichier séparé.

#### Structure basique d'un projet

Prenons un exemple simple avec la structure suivante :

```
mon_projet/
├── Cargo.toml
└── src/
    ├── main.rs
    ├── modele.rs
    └── utilitaires.rs
```

Dans `main.rs`, nous devons déclarer les modules externes :

``` rust
// src/main.rs
mod modele;        // Déclare l'existence du module modele
mod utilitaires;   // Déclare l'existence du module utilitaires

fn main() {
    // Utilisation d'un élément du module modele
    let utilisateur = modele::Utilisateur::nouveau("Alice");

    // Utilisation d'une fonction du module utilitaires
    utilitaires::afficher_info(&utilisateur);
}
```

Dans les fichiers correspondants, nous définissons le contenu des modules :

``` rust
// src/modele.rs
pub struct Utilisateur {
    pub nom: String,
    connexions: u32,
}

impl Utilisateur {
    pub fn nouveau(nom: &str) -> Self {
        Utilisateur {
            nom: nom.to_string(),
            connexions: 0,
        }
    }

    pub fn incrementer_connexions(&mut self) {
        self.connexions += 1;
    }

    pub fn nombre_connexions(&self) -> u32 {
        self.connexions
    }
}
```

``` rust
// src/utilitaires.rs
use crate::modele::Utilisateur;

pub fn afficher_info(utilisateur: &Utilisateur) {
    println!("Utilisateur: {}", utilisateur.nom);
    println!("Connexions: {}", utilisateur.nombre_connexions());
}
```

### Visibilité et mot-clé `pub`

Par défaut, tout en Rust est privé. Pour rendre un élément accessible depuis l'extérieur de son module, il faut utiliser le mot-clé `pub` :

- `pub struct` pour les structures
- `pub fn` pour les fonctions
- `pub mod` pour les sous-modules
- `pub enum` pour les énumérations
- `pub trait` pour les traits

Pour les structures, chaque champ doit être explicitement marqué comme public avec `pub` s'il doit être accessible depuis l'extérieur.

### Importation avec `use`

Pour éviter de devoir préfixer chaque utilisation avec le nom du module, on peut importer des éléments avec `use` :

``` rust
// Importation spécifique
use crate::modele::Utilisateur;

// Importation multiple
use crate::modele::{Utilisateur, Produit};

// Importation avec alias
use crate::modele::Utilisateur as User;

// Importation de tout le module
use crate::modele::*; // Déconseillé sauf cas particuliers
```

### Organisation hiérarchique

Rust permet une organisation hiérarchique des modules. Par exemple :

```
mon_projet/
├── Cargo.toml
└── src/
    ├── main.rs
    ├── modele/
    │   ├── mod.rs
    │   ├── utilisateur.rs
    │   └── produit.rs
    └── utilitaires/
        ├── mod.rs
        ├── formatage.rs
        └── validation.rs
```

Cette structure est reflétée dans le code :

``` rust
// src/main.rs
mod modele;
mod utilitaires;

use crate::modele::utilisateur::Utilisateur;
use crate::utilitaires::formatage;

fn main() {
    let user = Utilisateur::nouveau("Bob");
    formatage::afficher_nom(&user);
}
```

``` rust
// src/modele/mod.rs
pub mod utilisateur;
pub mod produit;
```

```
// src/modele/utilisateur.rs
pub struct Utilisateur {
    pub nom: String,
}

impl Utilisateur {
    pub fn nouveau(nom: &str) -> Self {
        Utilisateur {
            nom: nom.to_string(),
        }
    }
}
```

```
// src/utilitaires/mod.rs
pub mod formatage;
pub mod validation;
```

```
// src/utilitaires/formatage.rs
use crate::modele::utilisateur::Utilisateur;

pub fn afficher_nom(user: &Utilisateur) {
    println!("Nom: {}", user.nom);
}
```

### Réexportation avec `pub use`

Pour simplifier l'API publique d'un module, on peut réexporter des éléments :

``` rust
// src/modele/mod.rs
mod utilisateur;
mod produit;

// Réexportation pour simplifier l'API
pub use utilisateur::Utilisateur;
pub use produit::Produit;
```

Avec cette réexportation, on peut utiliser :

``` rust
use crate::modele::Utilisateur; // Au lieu de crate::modele::utilisateur::Utilisateur
```

### Chemins relatifs et absolus

Rust offre plusieurs façons de référencer les modules :

- `crate::` - Chemin absolu depuis la racine de la crate
- `super::` - Module parent
- `self::` - Module courant

Exemple :

``` rust
// src/utilitaires/validation.rs
use crate::modele::Utilisateur;         // Chemin absolu
use super::formatage;                   // Module frère (même parent)
use self::validateurs::valider_email;   // Sous-module local

mod validateurs {
    pub fn valider_email(email: &str) -> bool {
        // Implémentation
        email.contains('@')
    }
}

pub fn valider_utilisateur(user: &Utilisateur) -> bool {
    // Utilisation du module frère
    formatage::log_validation(user);

    // Utilisation du sous-module
    if user.email.is_some() {
        return valider_email(user.email.as_ref().unwrap());
    }

    false
}
```

### Modules externes (crates)

Pour utiliser des crates externes, on les déclare dans `Cargo.toml` et on les importe avec `use` :

``` toml
# Cargo.toml
[dependencies]
serde = "1.0"
rand = "0.8"
```

``` rust
// Dans votre code
use serde::{Serialize, Deserialize};
use rand::Rng;
```

### Syntaxe moderne pour les fichiers

Depuis Rust Edition 2018, il existe une alternative à l'utilisation des fichiers `mod.rs` :

```
mon_projet/
├── Cargo.toml
└── src/
    ├── main.rs
    ├── modele.rs           // Définit le module
    ├── modele/             // Contient les sous-modules de modele
    │   ├── utilisateur.rs
    │   └── produit.rs
    └── utilitaires.rs
```

Dans cette structure :

``` rust
// src/modele.rs
pub mod utilisateur;  // Fait référence à src/modele/utilisateur.rs
pub mod produit;      // Fait référence à src/modele/produit.rs

// Réexportations possibles
pub use utilisateur::Utilisateur;
pub use produit::Produit;
```

### Bonnes pratiques

1.  **Évitez les imports globaux** (`use module::*`) qui rendent l'origine des symboles ambiguë
2.  **Utilisez `pub use`** pour créer une API plus intuitive
3.  **Groupez les imports logiquement**, par exemple en séparant crates standards, externes et internes
4.  **Préférez la nouvelle syntaxe** (sans fichiers `mod.rs`) pour plus de clarté
5.  **Donnez aux modules des noms au singulier** : `model` plutôt que `models`

### Organisation recommandée pour un projet de taille moyenne

```
mon_projet/
├── Cargo.toml
└── src/
    ├── main.rs              // Point d'entrée, contient la fonction main()
    ├── lib.rs               // Bibliothèque partagée, si nécessaire
    ├── config.rs            // Configuration
    ├── modele/
    │   ├── mod.rs           // Réexportations
    │   ├── utilisateur.rs
    │   └── session.rs
    ├── services/
    │   ├── mod.rs
    │   ├── authentification.rs
    │   └── stockage.rs
    ├── ui/
    │   ├── mod.rs
    │   └── interface.rs
    └── utils/
        ├── mod.rs
        ├── formatage.rs
        └── journalisation.rs
```

La bonne organisation des fichiers est un élément clé pour maintenir un projet Rust à mesure qu'il évolue. Le système de modules de Rust offre une grande flexibilité pour structurer votre code de manière logique et modulaire, facilitant ainsi la navigation et la maintenance à long terme.

⏭️ [Les macros](/II-specificites/13-macros.md)
