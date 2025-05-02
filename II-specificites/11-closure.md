## 11\. Closure

Retour à la [Table des matières](/SOMMAIRE.md)

Les closures représentent un concept fondamental en Rust et sont particulièrement puissantes pour qui cherche à écrire du code concis et expressif. Si vous avez déjà utilisé des langages fonctionnels, ce concept vous sera familier.

### Qu'est-ce qu'une closure ?

Une closure est une fonction anonyme qui peut capturer des variables de son environnement. On peut la considérer comme une fonction qui "emporte" avec elle le contexte dans lequel elle a été créée.

Voyons un exemple simple :

``` rust
let multiplication = |nombre: i32, multiplicateur: i32| nombre * multiplicateur;
println!("{}", multiplication(2, 3)); // Affiche "6"
```

Cette syntaxe utilise les barres verticales `|...|` pour définir les paramètres, suivies du corps de la fonction.

L'aspect vraiment intéressant des closures est leur capacité à capturer leur environnement :

``` rust
let facteur = 5;
let multiplication = |nombre: i32| nombre * facteur;
println!("{}", multiplication(3)); // Affiche "15"
```

Ici, la closure `multiplication` capture la variable `facteur` de son environnement et l'utilise dans son corps, bien que cette variable ne soit pas passée en paramètre.

### Types de closures

En Rust, il existe trois traits principaux pour les closures, chacun avec des caractéristiques différentes :

#### 1\. `Fn` - Closures qui empruntent immutablement

``` rust
fn appliquer<F>(valeur: i32, f: F) -> i32
    where F: Fn(i32) -> i32
{
    f(valeur)
}

let double = |x| x * 2;
let resultat = appliquer(5, double); // resultat = 10
```

#### 2\. `FnMut` - Closures qui empruntent mutablement

``` rust
fn accumuler<F>(mut valeurs: Vec<i32>, mut f: F) -> Vec<i32>
    where F: FnMut(&mut Vec<i32>, i32)
{
    for i in 0..5 {
        f(&mut valeurs, i);
    }
    valeurs
}

let mut somme = 0;
let resultat = accumuler(vec![], |vec, val| {
    somme += val;
    vec.push(somme);
});
println!("{:?}", resultat); // Affiche [0, 1, 3, 6, 10]
```

#### 3\. `FnOnce` - Closures qui consomment leurs variables capturées

``` rust
fn consommer<F, T>(f: F) -> i32
    where F: FnOnce() -> T,
          T: Into<i32>
{
    f().into()
}

let valeur = String::from("42");
let resultat = consommer(move || {
    // Cette closure prend possession de `valeur`
    valeur.parse::<i32>().unwrap()
});
// `valeur` n'est plus utilisable ici
println!("{}", resultat); // Affiche "42"
```

Le mot-clé `move` force la closure à prendre possession des variables qu'elle capture plutôt que de les emprunter.

### Hiérarchie des traits de closures

Il est important de comprendre la hiérarchie entre ces traits :

- Toute closure qui implémente `Fn` implémente aussi `FnMut` et `FnOnce`
- Toute closure qui implémente `FnMut` implémente aussi `FnOnce`
- `FnOnce` est le trait le plus général

Cette hiérarchie est logique : si une closure peut être appelée plusieurs fois sans modifier son environnement (`Fn`), elle peut certainement être appelée en modifiant son environnement (`FnMut`) ou en consommant ses ressources (`FnOnce`).

### Fonctions comme closures

Une caractéristique intéressante est que les fonctions ordinaires implémentent aussi ces traits :

``` rust
fn double(x: i32) -> i32 {
    x * 2
}

fn appliquer<F>(valeur: i32, f: F) -> i32
    where F: Fn(i32) -> i32
{
    f(valeur)
}

let resultat = appliquer(5, double); // Fonctionne parfaitement !
```

### Inférence de type

Rust peut généralement inférer les types dans les closures :

``` rust
    // Les types sont inférés automatiquement grâce à l'utilisation
    let addition = |a, b| a + b;

    println!("2 + 3 = {}", addition(2, 3));  // Le contexte d'utilisation aide à l'inférence

    // On peut aussi les spécifier explicitement
    let soustraction = |a: i32, b: i32| -> i32 { a - b };

    println!("5 - 2 = {}", soustraction(5, 2));
```

### Cas d'utilisation concrets

Les closures sont particulièrement utiles pour :

1.  **Les callbacks** :

``` rust
struct BoutonUI {
    texte: String,
    action: Option<Box<dyn Fn()>>,
}

impl BoutonUI {
    fn new(texte: &str) -> Self {
        BoutonUI {
            texte: texte.to_string(),
            action: None,
        }
    }

    fn on_click<F: Fn() + 'static>(&mut self, callback: F) {
        self.action = Some(Box::new(callback));
    }

    fn cliquer(&self) {
        if let Some(action) = &self.action {
            action();
        }
    }
}

// Utilisation
let mut bouton = BoutonUI::new("OK");
let compteur = std::cell::RefCell::new(0);
bouton.on_click(move || {
    *compteur.borrow_mut() += 1;
    println!("Bouton cliqué! Total: {}", *compteur.borrow());
});

bouton.cliquer(); // Affiche "Bouton cliqué! Total: 1"
bouton.cliquer(); // Affiche "Bouton cliqué! Total: 2"
```

2.  **Les itérateurs** :

``` rust
let nombres = vec![1, 2, 3, 4, 5];
let somme: i32 = nombres.iter().map(|x| x * 2).sum();
println!("Somme des doubles: {}", somme); // Affiche "Somme des doubles: 30"
```

3.  **Le traitement conditionnel** :

``` rust
enum Validation {
    Réussite(String),
    Échec(String),
}

fn valider<F>(donnée: &str, validation: F) -> Validation
    where F: Fn(&str) -> bool
{
    if validation(donnée) {
        Validation::Réussite(format!("{} est valide", donnée))
    } else {
        Validation::Échec(format!("{} est invalide", donnée))
    }
}

let resultat = valider("12345", |s| s.len() > 3 && s.chars().all(|c| c.is_digit(10)));
match resultat {
    Validation::Réussite(msg) => println!("{}", msg),
    Validation::Échec(msg) => println!("{}", msg),
}
```

Les closures constituent un outil essentiel dans votre arsenal Rust. Leur capacité à capturer l'environnement et à encapsuler des comportements les rend indispensables pour écrire du code élégant et modulaire.

## 12\. Multi-fichier

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

⏭️ [Multi-fichier](/II-specificites/12-multi-fichier.md)
