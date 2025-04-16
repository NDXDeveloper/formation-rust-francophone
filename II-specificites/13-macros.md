# 13\. Les macros

Les macros sont l'un des éléments les plus puissants de Rust, permettant une métaprogrammation qui étend considérablement les capacités du langage. Contrairement aux fonctions, les macros manipulent le code avant la compilation, ce qui leur confère une flexibilité unique.

## 13.1 Définition et syntaxe de base

Une macro en Rust se reconnaît par l'utilisation du symbole `!` après son nom. Elle peut être appelée de trois façons différentes :

``` rust
// Définir la macro avant de l'utiliser
macro_rules! la_macro {
    () => {
        println!("Macro appelée avec des parenthèses");
    };
    [] => {
        println!("Macro appelée avec des crochets");
    };
    {} => {
        println!("Macro appelée avec des accolades");
    };
}

fn main() {
    la_macro!();  // avec des parenthèses
    la_macro![];  // avec des crochets
    la_macro!{};  // avec des accolades
}

```

Les macros déclaratives (ou "macro par l'exemple") sont définies avec `macro_rules!`, suivies d'un ensemble de règles de transformation basées sur le pattern-matching.

Voici un exemple simple :

``` rust
macro_rules! dire_bonjour {
    () => {
        println!("Bonjour !");
    }
}

fn main() {
    dire_bonjour!(); // Affiche: Bonjour !
}
```

## 13.2 Les arguments et tokens

Les macros reçoivent ce qu'on appelle des "flux de tokens" plutôt que de simples arguments. Ces tokens sont capturés et peuvent être manipulés selon des règles définies.

``` rust
macro_rules! saluer {
    ($personne:expr) => {
        println!("Bonjour, {} !", $personne);
    };
}

fn main() {
    saluer!("Marie");      // Affiche: Bonjour, Marie !
    saluer!("le monde");   // Affiche: Bonjour, le monde !
}
```

Dans cet exemple, `$personne` est une metavariable et `expr` est un spécificateur de fragment indiquant que la macro attend une expression.

## 13.3 Pattern-matching et branches multiples

Une macro peut avoir plusieurs branches, chacune correspondant à un modèle d'entrée différent :

``` rust
macro_rules! saluer {
    // Cas avec un seul argument
    ($personne:expr) => {
        println!("Bonjour, {} !", $personne);
    };

    // Cas avec deux arguments
    ($personne:expr, $langue:expr) => {
        match $langue {
            "français" => println!("Bonjour, {} !", $personne),
            "anglais" => println!("Hello, {} !", $personne),
            "espagnol" => println!("¡Hola, {} !", $personne),
            _ => println!("Salutations, {} !", $personne),
        }
    };
}

fn main() {
    saluer!("Marie");                // Affiche: Bonjour, Marie !
    saluer!("Pierre", "français");   // Affiche: Bonjour, Pierre !
    saluer!("John", "anglais");      // Affiche: Hello, John !
    saluer!("Carlos", "espagnol");   // Affiche: ¡Hola, Carlos !
}
```

## 13.4 Spécificateurs de fragments

Rust propose plusieurs spécificateurs de fragments permettant de définir le type d'éléments que peut capturer une macro. Voici la liste complète avec des exemples :

| Spécificateur | Description | Exemple |
| --- | --- | --- |
| `ident` | Un identifiant | `nom_variable`, `type_donnee` |
| `path` | Un chemin qualifié | `std::vec::Vec`, `self::fonction` |
| `expr` | Une expression | `2 + 2`, `fonction(x)` |
| `ty` | Un type | `i32`, `Vec<String>` |
| `pat_param` | Un motif | `Some(x)`, `(a, b)` |
| `pat` | Un motif (plus large) | `_`, `1..=5` |
| `stmt` | Une instruction | `let x = 3;` |
| `block` | Un bloc de code | `{ println!("hello"); }` |
| `item` | Un élément | `fn foo() {}`, `struct Bar;` |
| `meta` | Un attribut | `#[derive(Debug)]` |
| `tt` | Un arbre de tokens | Tout ce qui est entre `[]`, `()`, ou `{}` |
| `lifetime` | Une durée de vie | `'a`, `'static` |
| `vis` | Un qualificateur de visibilité | `pub`, `pub(crate)` |
| `literal` | Une expression littérale | `"texte"`, `42`, `'a'` |

Exemple utilisant différents spécificateurs :

``` rust
macro_rules! declarer_fonction {
    ($vis:vis $nom:ident($param:ident: $type:ty) -> $retour:ty $corps:block) => {
        $vis fn $nom($param: $type) -> $retour $corps
    };
}

declarer_fonction! {
    pub add_one(x: i32) -> i32 {
        x + 1
    }
}

fn main() {
    println!("1 + 1 = {}", add_one(1)); // Affiche: 1 + 1 = 2
}
```

## 13.5 Répétition

L'une des fonctionnalités les plus puissantes des macros est la capacité à traiter un nombre variable d'arguments grâce à la répétition :

``` rust
macro_rules! creer_vecteur {
    // Cas vide
    () => {
        Vec::new()
    };

    // Un ou plusieurs éléments séparés par des virgules
    ($($element:expr),+ $(,)?) => {{
        let mut v = Vec::new();
        $(
            v.push($element);
        )+
        v
    }};
}

fn main() {
    let v1: Vec<i32> = creer_vecteur!();           // Vec vide
    let v2 = creer_vecteur![1, 2, 3];              // Vec avec 3 éléments
    let v3 = creer_vecteur!(String::from("hello"),
                           "world".to_string(),);  // Virgule finale optionnelle

    println!("{:?}", v1); // Affiche: []
    println!("{:?}", v2); // Affiche: [1, 2, 3]
    println!("{:?}", v3); // Affiche: ["hello", "world"]
}
```

Explications des opérateurs de répétition :

- $,\* : répétition 0 ou plusieurs fois, éléments séparés par des virgules
- $,+ : répétition 1 ou plusieurs fois, éléments séparés par des virgules
- $;\* : répétition avec des points-virgules comme séparateurs
- `$(,)?` : virgule optionnelle à la fin

## 13.6 Récursivité dans les macros

Les macros peuvent être récursives, ce qui est utile pour traiter des structures complexes :

``` rust
macro_rules! afficher_valeurs {
    // Cas de base: une seule valeur
    ($derniere:expr) => {
        println!("Valeur: {}", $derniere);
    };

    // Cas récursif: première valeur suivie d'autres
    ($premiere:expr, $($reste:expr),+) => {
        println!("Valeur: {}", $premiere);
        afficher_valeurs!($($reste),+);
    };
}

fn main() {
    afficher_valeurs!(1, 2, 3, "quatre", 5.0);
    // Affiche:
    // Valeur: 1
    // Valeur: 2
    // Valeur: 3
    // Valeur: quatre
    // Valeur: 5
}
```

## 13.7 Génération de code

Les macros excellant dans la génération de code répétitif. Voici un exemple plus avancé qui génère des structures et leurs implémentations :

``` rust
macro_rules! creer_structure_avec_getters {
    (
        $(
            struct $nom:ident {
                $(
                    $champ:ident: $type:ty
                ),+
            }
        )+
    ) => {
        $(
            // Définition de la structure
            pub struct $nom {
                $(
                    $champ: $type
                ),+
            }

            // Implémentation des getters
            impl $nom {
                $(
                    pub fn $champ(&self) -> &$type {
                        &self.$champ
                    }
                )+
            }
        )+
    };
}

creer_structure_avec_getters! {
    struct Utilisateur {
        id: u64,
        nom: String,
        email: String
    }

    struct Produit {
        id: u32,
        nom: String,
        prix: f64
    }
}

fn main() {
    let user = Utilisateur {
        id: 42,
        nom: "Alice".to_string(),
        email: "alice@example.com".to_string()
    };

    println!("Utilisateur: {} ({})", user.nom(), user.email());
    // Affiche: Utilisateur: Alice (alice@example.com)
}
```

## 13.8 Portée et exportation des macros

### Utilisation dans un fichier

Pour utiliser des macros définies dans un fichier séparé, ajoutez en haut du fichier contenant les macros :

```
#![macro_use]
```

### Exportation de macros

Pour rendre une macro disponible en dehors de votre crate, ajoutez :

```
#[macro_export]
macro_rules! ma_macro_exportee {
    // ...
}
```

### Importation de macros

Les macros peuvent être importées comme les autres éléments avec `use` :

```
// Importation d'une macro depuis une dépendance
use nom_crate::ma_macro;

// Dans les anciennes versions de Rust, on utilisait:
#[macro_use]
extern crate nom_crate;
```

exemple :
``` rust
// dans un fichier macros.rs

// Cette macro permet de générer des structures avec leurs getters et constructeurs automatiquement
// Elle rend le code plus concis en évitant la répétition pour chaque structure
#[macro_export]  // Rend la macro accessible depuis n'importe où dans le crate
macro_rules! creer_structure_avec_getters {
    // Début du pattern matching pour la macro
    (
        // Cette partie capture une ou plusieurs structures
        $(
            // Capture le nom de la structure
            struct $nom:ident {
                // Capture un ou plusieurs champs avec leur type
                $(
                    $champ:ident: $type:ty
                ),+  // Le + indique un ou plusieurs éléments
            }
        )+  // Le + permet de définir plusieurs structures à la fois
    ) => {
        // Code généré pour chaque structure capturée
        $(
            // Génère une structure publique avec les champs privés
            pub struct $nom {
                $(
                    $champ: $type  // Champs privés par défaut (encapsulation)
                ),+
            }

            // Implémentation des méthodes pour la structure
            impl $nom {
                // Constructeur public pour créer des instances
                pub fn new($($champ: $type),+) -> Self {
                    Self {
                        $($champ),+  // Initialise tous les champs à partir des paramètres
                    }
                }

                // Génère un getter pour chaque champ
                $(
                    // Chaque getter retourne une référence immutable au champ
                    pub fn $champ(&self) -> &$type {
                        &self.$champ  // Retourne une référence pour éviter le transfert de propriété
                    }
                )+
            }
        )+
    };
}

// Utilisation de la macro pour créer deux structures différentes
creer_structure_avec_getters! {
    // Première structure: Utilisateur
    struct Utilisateur {
        id: u64,        // Identifiant unique
        nom: String,    // Nom de l'utilisateur
        email: String   // Adresse email
    }

    // Deuxième structure: Produit
    struct Produit {
        id: u32,        // Identifiant du produit
        nom: String,    // Nom du produit
        prix: f64       // Prix du produit
    }
}
```
``` rust
//dans main.rs

// Import du module contenant nos macros et structures générées
mod macros;
// Import spécifique de la structure Utilisateur depuis le module macros
use crate::macros::Utilisateur;

fn main() {
    // Création d'une instance de la structure Utilisateur
    // Utilisation du constructeur new() qui permet d'accéder aux champs privés
    // sans exposer directement leur implémentation (encapsulation)
    let user = Utilisateur::new(
        42,                             // id: u64 - Identifiant unique de l'utilisateur
        "Alice".to_string(),            // nom: String - Conversion de la chaîne littérale en String
        "alice@example.com".to_string() // email: String - Adresse email de l'utilisateur
    );

    // Affichage des informations de l'utilisateur en utilisant les getters
    // Les getters retournent des références (&) aux champs privés
    println!("Utilisateur: {} ({})", user.nom(), user.email());
    // L'affichage produit: "Utilisateur: Alice (alice@example.com)"
}

```

## 13.9 Règles d'hygiène

Les macros en Rust sont "hygiéniques", ce qui signifie que les variables déclarées à l'intérieur d'une macro n'interfèrent pas avec celles du code environnant :

``` rust
macro_rules! creer_variable {
    ($nom:ident, $valeur:expr) => {
        let $nom = $valeur;
    };
}

fn main() {
    let x = 1;

    {
        creer_variable!(x, 2); // Crée une nouvelle variable x dans ce scope
        println!("x dans le bloc intérieur: {}", x); // Affiche: 2
    }

    println!("x dans le bloc principal: {}", x); // Affiche: 1
}
```

## 13.10 Macros utiles intégrées

Rust fournit plusieurs macros standard très utiles :

| Macro | Description | Exemple |
| --- | --- | --- |
| `panic!` | Arrête le programme avec un message | `panic!("Erreur fatale")` |
| `assert!` | Vérifie une condition ou panique | `assert!(x > 0)` |
| `assert_eq!` | Vérifie l'égalité ou panique | `assert_eq!(2 + 2, 4)` |
| `dbg!` | Affiche une expression et sa valeur | `dbg!(x + y)` |
| `vec!` | Crée un vecteur | `vec![1, 2, 3]` |
| `compile_error!` | Génère une erreur de compilation | `compile_error!("Message")` |
| `format!` | Formate du texte en String | `format!("x={}", 42)` |
| `unreachable!` | Code qui ne devrait jamais être atteint | `unreachable!()` |
| `unimplemented!` | Marque du code non implémenté | `unimplemented!()` |
| `todo!` | Marque du code à faire | `todo!("Implémenter cette fonction")` |
| `column!` | Renvoie la colonne courante dans le code | `column!()` |
| `line!` | Renvoie le numéro de ligne courant | `line!()` |
| `file!` | Renvoie le nom du fichier courant | `file!()` |

## 13.11 Cas d'utilisation concret

Voici un exemple pratique utilisant des macros pour simplifier la création d'un composant d'interface utilisateur :
``` bash
cargo add uuid --features v4
```
```
[dependencies]
...
uuid = { version = "1.6.1", features = ["v4"] }
```
``` rust
macro_rules! create_component {
    ($name:ident, $($prop:ident: $type:ty),* $(,)?) => {
        pub struct $name {
            $(pub $prop: $type,)*
            pub id: String,
            pub visible: bool,
        }

        impl $name {
            pub fn new($($prop: $type,)*) -> Self {
                Self {
                    $($prop,)*
                    id: uuid::Uuid::new_v4().to_string(),
                    visible: true,
                }
            }

            pub fn toggle_visibility(&mut self) {
                self.visible = !self.visible;
            }
        }
    };
}

// Utilisation de la macro
create_component!(Button, text: String, onClick: fn());
create_component!(TextInput, value: String, placeholder: String);

fn handle_click() {
    println!("Button clicked!");
}

fn main() {
    let mut btn = Button::new(
        "Click me".to_string(),
        handle_click
    );

    let input = TextInput::new(
        "".to_string(),
        "Enter your name".to_string()
    );

    println!("Button ID: {}, visible: {}", btn.id, btn.visible);
    btn.toggle_visibility();
    println!("Button now visible: {}", btn.visible);
}
```

## 13.12 Limitations et bonnes pratiques

### Limitations

- Débogage complexe : les erreurs dans les macros peuvent être difficiles à tracer
- Syntaxe particulière : moins intuitive que le code Rust standard
- Potentiel abus : trop de macros peuvent rendre le code moins lisible

### Bonnes pratiques

1.  **Documentation** : documentez vos macros de manière exhaustive
2.  **Simplicité** : gardez vos macros aussi simples que possible
3.  **Organisation** : isolez vos macros dans des modules ou fichiers séparés
4.  **Tests** : testez bien vos macros avec différents cas d'entrée
5.  **Nommage** : utilisez des noms explicites qui indiquent qu'il s'agit d'une macro

```
// Mauvais
macro_rules! m {
    // ...
}

// Bon
macro_rules! create_error_type {
    // ...
}
```

## 13.13 Alternative aux macros

Dans certains cas, la généricité et les traits peuvent offrir une alternative plus typée et plus facile à déboguer que les macros :

``` rust
// Avec des macros
macro_rules! min {
    ($x:expr, $y:expr) => {
        if $x < $y { $x } else { $y }
    };
}

// Avec des génériques
fn min<T: PartialOrd>(x: T, y: T) -> T where T: Copy {
    if x < y { x } else { y }
}
```

Les macros restent cependant incontournables pour certains cas d'utilisation comme la métaprogrammation avancée ou la génération de code.

* * *

Les macros constituent un outil extrêmement puissant dans l'arsenal du programmeur Rust. Bien que leur syntaxe puisse paraître intimidante au premier abord, elles offrent des possibilités de métaprogrammation qui dépassent largement celles offertes par de nombreux autres langages. Elles permettent notamment de réduire considérablement la duplication de code et d'étendre efficacement le langage pour répondre à des besoins spécifiques.
