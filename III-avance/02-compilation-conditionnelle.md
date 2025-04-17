## 2\. La compilation conditionnelle

La compilation conditionnelle permet d'inclure ou d'exclure certaines parties de votre code selon des conditions spécifiques, comme le système d'exploitation cible, l'architecture, ou des fonctionnalités optionnelles. En Rust, cette fonctionnalité est essentielle pour développer des applications multi-plateformes ou personnalisables.

### Contexte et comparaison avec C

En C, la compilation conditionnelle se fait généralement avec les directives de préprocesseur comme `#ifdef`, `#elif`, et `#endif` :

``` c
#ifdef linux
  #define SYSTEM "linux"
#elif _WIN32
  #define SYSTEM "windows"
#endif

void show_system() {
  printf("%s", SYSTEM);
}
```

En Rust, ce mécanisme est géré de façon plus claire et sûre grâce à l'attribut `#[cfg]` :

``` rust
#[cfg(target_os = "linux")]
const SYSTEM: &str = "linux";

#[cfg(target_os = "windows")]
const SYSTEM: &str = "windows";

fn show_system() {
    println!("{}", SYSTEM);
}

fn main() {
    show_system();
}
```

### Gestion des cas non prévus avec `not` et `any`

Avec le code précédent, si on compile sur un système autre que Windows ou Linux (comme macOS), la compilation échouera car la constante `SYSTEM` ne sera pas définie. Pour résoudre ce problème, nous devons ajouter une définition par défaut :

``` rust
#[cfg(target_os = "linux")]
const SYSTEM: &str = "linux";

#[cfg(target_os = "windows")]
const SYSTEM: &str = "windows";

#[cfg(not(any(target_os = "linux", target_os = "windows")))]
const SYSTEM: &str = "autre système";

fn show_system() {
    println!("Système d'exploitation: {}", SYSTEM);
}

fn main() {
    show_system();
}
```

### Opérateurs de condition pour `cfg`

L'attribut `cfg` peut utiliser trois opérateurs principaux pour combiner des conditions :

- `all` : retourne `true` si toutes les conditions qu'il contient sont vraies (équivalent à AND logique)
- `any` : retourne `true` si au moins une des conditions qu'il contient est vraie (équivalent à OR logique)
- `not` : inverse la condition (équivalent à NOT logique)

Exemples de base :

``` rust
#[cfg(all())]  // Toujours vrai
struct AlwaysIncluded;

#[cfg(any())]  // Toujours faux
struct NeverIncluded;

#[cfg(all(target_os = "linux", target_arch = "x86_64"))]
fn linux_x86_64_only() {
    println!("Cette fonction n'est disponible que sur Linux x86_64");
}

#[cfg(any(target_os = "linux", target_os = "macos"))]
fn unix_like_systems() {
    println!("Cette fonction est disponible sur Linux et macOS");
}
```

### Arguments disponibles pour `cfg`

Voici les arguments les plus couramment utilisés avec `cfg` :

#### Arguments pour les cibles de compilation

- `target_os` : le système d'exploitation cible (`"linux"`, `"windows"`, `"macos"`, `"android"`, etc.)
- `target_arch` : l'architecture du processeur (`"x86_64"`, `"aarch64"`, `"arm"`, `"wasm32"`, etc.)
- `target_family` : la famille de systèmes d'exploitation (`"unix"`, `"windows"`, `"wasm"`)
- `target_env` : l'environnement de compilation (`"gnu"`, `"msvc"`, etc.)
- `target_endian` : l'endianness du processeur (`"big"` ou `"little"`)
- `target_pointer_width` : la taille d'un pointeur en bits (`"16"`, `"32"`, `"64"`, etc.)

#### Exemple d'utilisation des arguments de cible

``` rust
#[cfg(target_arch = "wasm32")]
fn wasm_specific_code() {
    // Code qui ne s'exécute que dans un environnement WebAssembly
}

#[cfg(all(target_pointer_width = "64", target_endian = "little"))]
fn specific_architecture_code() {
    // Code spécifique aux architectures 64 bits little-endian
}
```

#### Arguments pour les fonctionnalités et contextes particuliers

- `feature = "nom_feature"` : vérifie si une fonctionnalité optionnelle est activée
- `test` : vrai lors de la compilation en mode test
- `debug_assertions` : vrai en mode débogage
- `doc` : vrai lors de la génération de documentation
- `doctest` : vrai lors de l'exécution des tests de documentation

### Configuration des features dans Cargo.toml

Les fonctionnalités (features) doivent être déclarées dans votre `Cargo.toml` :

``` toml
[package]
name = "mon_paquet"
version = "0.1.0"

[features]
default = ["fonctionnalite1"]
fonctionnalite1 = []
fonctionnalite2 = []
toutes_fonctionnalites = ["fonctionnalite1", "fonctionnalite2"]
```

Ces fonctionnalités peuvent ensuite être activées lors de la compilation :

``` bash
cargo build --features "fonctionnalite2"
```

Et utilisées dans le code :

``` rust
#[cfg(feature = "fonctionnalite1")]
fn fonction_specifique() {
    println!("Cette fonction n'est disponible qu'avec la fonctionnalité 1");
}
```

### L'attribut `cfg_attr`

L'attribut `cfg_attr` permet d'appliquer conditionnellement d'autres attributs. C'est très utile pour éviter de dupliquer du code.

Par exemple, pour dériver conditionnellement le trait `Debug` uniquement lorsque la fonctionnalité `debug` est activée :

``` rust
// Sans cfg_attr, il faudrait dupliquer la structure :
#[cfg(feature = "debug")]
#[derive(Debug)]
pub struct Utilisateur {
    nom: String,
    age: u32,
}

#[cfg(not(feature = "debug"))]
pub struct Utilisateur {
    nom: String,
    age: u32,
}

// Avec cfg_attr, c'est beaucoup plus simple :
#[cfg_attr(feature = "debug", derive(Debug))]
pub struct Utilisateur {
    nom: String,
    age: u32,
}

fn main() {
    // Test de la feature debug
    let user = Utilisateur {
        nom: String::from("Alice"),
        age: 30,
    };

    // Affichage conditionnel selon la feature
    #[cfg(feature = "debug")]
    println!("Debug: {:?}", user);

    #[cfg(not(feature = "debug"))]
    println!("Utilisateur: {} ({})", user.nom, user.age);
}

```

On peut aussi enchaîner plusieurs attributs conditionnels :

``` rust
#[cfg_attr(feature = "serde", derive(Serialize, Deserialize))]
#[cfg_attr(feature = "debug", derive(Debug))]
pub struct Configuration {
    // champs
}
```

### La macro `cfg!`

La macro `cfg!` évalue une condition de compilation et retourne un booléen. Contrairement à l'attribut `#[cfg]`, cette macro n'exclut pas de code à la compilation, mais permet de prendre des décisions basées sur les conditions de compilation.

``` rust
fn show_system() {
    if cfg!(target_os = "linux") {
        println!("Vous utilisez Linux");
    } else if cfg!(target_os = "windows") {
        println!("Vous utilisez Windows");
    } else if cfg!(target_os = "macos") {
        println!("Vous utilisez macOS");
    } else {
        println!("Vous utilisez un autre système");
    }
}

fn main() {
    show_system();
}
```

Après compilation, le compilateur optimise ce code en éliminant les branches qui ne peuvent jamais être exécutées. Si nous compilons sur Linux, le code sera effectivement transformé en :

```
fn show_system() {
    println!("Vous utilisez Linux");
}
```

### Combinaison avec des modules conditionnels

Une pratique courante est de créer des modules entiers conditionnels pour organiser le code spécifique à une plateforme :

``` rust
#[cfg(target_os = "linux")]
mod platform {
    pub fn initialiser_systeme() {
        // Initialisation spécifique à Linux
    }
}

#[cfg(target_os = "windows")]
mod platform {
    pub fn initialiser_systeme() {
        // Initialisation spécifique à Windows
    }
}

fn main() {
    platform::initialiser_systeme();
}
```

``` rust
#[cfg(target_os = "linux")]
mod platform {
    pub fn initialiser_systeme() {
        println!("Démarrage de l'initialisation du système pour Linux...");
        // Initialisation spécifique à Linux
        println!("Initialisation du système Linux terminée avec succès.");
    }
}

#[cfg(target_os = "windows")]
mod platform {
    pub fn initialiser_systeme() {
        println!("Démarrage de l'initialisation du système pour Windows...");
        // Initialisation spécifique à Windows
        println!("Initialisation du système Windows terminée avec succès.");
    }
}

fn main() {
    println!("Programme démarré. Détection du système d'exploitation...");

    #[cfg(target_os = "linux")]
    println!("Système d'exploitation détecté : Linux");

    #[cfg(target_os = "windows")]
    println!("Système d'exploitation détecté : Windows");

    platform::initialiser_systeme();

    println!("Programme terminé.");
}
```
### Utilisation pour la compatibilité de versions

La compilation conditionnelle est aussi utile pour gérer les différentes versions du langage Rust :

```
#[cfg(rust_1_75_plus)]
fn nouvelle_fonctionnalite() {
    // Version pour Rust 1.75+
    println!("Fonctionnalité pour Rust 1.75 ou plus récent");
}

#[cfg(not(rust_1_75_plus))]
fn nouvelle_fonctionnalite() {
    // Version pour Rust < 1.75
    println!("Fallback pour Rust antérieur à 1.75");
}


fn main() {
    nouvelle_fonctionnalite();

}
```

un autre exemple :
```
[dependencies]
rustversion = "1.0.20"
```

```
#[rustversion::since(1.75)]
fn nouvelle_fonctionnalite() {
    // Utilise une fonctionnalité disponible uniquement dans Rust 1.75+
    println!("Fonctionnalité pour Rust 1.75 ou plus récent");
}

#[rustversion::before(1.75)]
fn nouvelle_fonctionnalite() {
    // Implémentation alternative pour les versions plus anciennes
    println!("Fallback pour Rust antérieur à 1.75");
}

fn main() {
    nouvelle_fonctionnalite();
}
```
Un autre exemple :
```
#[cfg(feature = "rust_1_75")]
fn nouvelle_fonctionnalite() {
    // Utilise une fonctionnalité disponible uniquement dans Rust 1.75+
}

#[cfg(not(feature = "rust_1_75"))]
fn nouvelle_fonctionnalite() {
    // Implémentation alternative pour les versions plus anciennes
}

fn main() {
    nouvelle_fonctionnalite();
}
```
### Conclusion

La compilation conditionnelle en Rust offre une approche puissante et flexible pour adapter votre code à différents environnements, architectures et cas d'utilisation. Grâce à l'attribut `#[cfg]`, l'attribut `cfg_attr` et la macro `cfg!`, vous pouvez écrire un code qui s'adapte automatiquement à son contexte de compilation, tout en maintenant la clarté et la sécurité que Rust garantit.

Cette fonctionnalité est particulièrement précieuse pour les bibliothèques multiplateformes et les applications qui doivent s'adapter à différents environnements d'exécution, tout en conservant une base de code unifiée et facile à maintenir.
