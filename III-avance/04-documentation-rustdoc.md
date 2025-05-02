## 4\. Documentation et rustdoc

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Rust offre un puissant générateur de documentation intégré, appelé `rustdoc`. Cet outil est celui qui génère toute la documentation officielle de la bibliothèque standard Rust (disponible sur [doc.rust-lang.org](https://doc.rust-lang.org/std/)). La bonne nouvelle est que cet outil est très simple à utiliser et produit une documentation professionnelle.

### Génération de la documentation

Avant d'apprendre à rédiger de la documentation, voyons comment la générer et à quoi elle ressemble.

#### Avec Cargo

Si vous utilisez Cargo (ce qui est recommandé), générer la documentation est très simple :

``` bash
cargo doc
```

La documentation générée se trouvera dans le dossier `target/doc/nom_de_votre_crate/`. Pour la consulter, ouvrez le fichier `index.html` avec votre navigateur, ou utilisez directement la commande :

``` bash
cargo doc --open
```

Cette commande génère la documentation et l'ouvre automatiquement dans votre navigateur par défaut.

Si vous voulez également inclure la documentation des dépendances de votre projet :

``` bash
cargo doc --document-private-items --open
```

L'option `--document-private-items` permet d'inclure les éléments privés (non publics) dans la documentation générée, ce qui est utile pendant le développement.

#### Sans Cargo

Si vous préférez ne pas utiliser Cargo, vous pouvez directement utiliser `rustdoc` :

``` bash
rustdoc src/lib.rs
```

La documentation sera générée dans le dossier `doc/` du répertoire courant.

`rustdoc` peut également traiter des fichiers markdown (.md) :

``` bash
rustdoc README.md
```

Cela créera un fichier `doc/README.html` qui peut être intégré à votre documentation.

### Rédiger de la documentation

#### Commentaires de documentation

En Rust, la documentation est écrite à l'aide de commentaires spéciaux qui commencent par `///` (pour documenter l'élément qui suit) ou `//!` (pour documenter l'élément qui les contient).

Voici un exemple simple :

``` rust
/// Calcule la somme de deux nombres.
///
/// # Exemples
///
/// ```
/// let somme = mon_crate::addition(5, 3);
/// assert_eq!(somme, 8);
/// ```
pub fn addition(a: i32, b: i32) -> i32 {
    a + b
}

/// Structure représentant un point en 2D.
pub struct Point {
    /// Coordonnée horizontale du point
    pub x: f64,
    /// Coordonnée verticale du point
    pub y: f64,
}

impl Point {
    /// Crée un nouveau point aux coordonnées spécifiées.
    ///
    /// # Arguments
    ///
    /// * `x` - La coordonnée horizontale
    /// * `y` - La coordonnée verticale
    ///
    /// # Retourne
    ///
    /// Une nouvelle instance de `Point`
    pub fn new(x: f64, y: f64) -> Self {
        Point { x, y }
    }

    /// Calcule la distance entre ce point et l'origine (0, 0).
    pub fn distance_origine(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

Les commentaires `///` sont du sucre syntaxique pour l'attribut `#[doc = "..."]`. On peut donc écrire la documentation de manière équivalente :

``` rust
#[doc = "Calcule la somme de deux nombres."]
#[doc = ""]
#[doc = "# Exemples"]
#[doc = ""]
#[doc = "```"]
#[doc = "let somme = mon_crate::addition(5, 3);"]
#[doc = "assert_eq!(somme, 8);"]
#[doc = "```"]
pub fn addition(a: i32, b: i32) -> i32 {
    a + b
}
```

#### Documentation du module ou de la crate

Pour documenter un module ou une crate entière (au lieu d'un élément spécifique), utilisez `//!` :

``` rust
//! # Ma Bibliothèque de Cryptographie
//!
 //! Cette crate fournit des fonctions pour calculer différents types de hachages.
//! Les algorithmes suivants sont implémentés :
//!
 //! - MD5
//! - SHA-1
//! - SHA-256
//! - CRC32

/// Calcule le hachage MD5 d'une chaîne de caractères.
pub fn md5_hash(input: &str) -> String {
    use md5::{Md5, Digest};
    let mut hasher = Md5::new();
    hasher.update(input.as_bytes());
    format!("{:x}", hasher.finalize())
}

/// Calcule le hachage SHA-256 d'une chaîne de caractères.
pub fn sha256_hash(input: &str) -> String {
    use sha2::{Sha256, Digest};
    let mut hasher = Sha256::new();
    hasher.update(input.as_bytes());
    format!("{:x}", hasher.finalize())
}
```

### Markdown et formatage

La documentation Rust utilise le format CommonMark (une spécification de Markdown). Vous pouvez donc utiliser toutes les fonctionnalités de Markdown pour améliorer vos documents :

``` rust
/// # Titre principal
///
/// ## Sous-titre
///
/// Texte normal avec du *texte en italique* et du **texte en gras**.
///
/// ### Listes
///
/// - Item 1
/// - Item 2
///   - Sous-item A
///   - Sous-item B
///
/// ### Code
///
/// Voici du code en ligne : `println!("Hello, world!")`.
///
/// Et un bloc de code :
///
/// ```rust
/// fn exemple() {
///     println!("Ceci est un exemple de code");
/// }
/// ```
///
/// ### Tableaux
///
/// | Colonne 1 | Colonne 2 |
/// |-----------|-----------|
/// | Cellule 1 | Cellule 2 |
/// | Cellule 3 | Cellule 4 |
///
/// ### Liens
///
/// [Documentation de Rust](https://doc.rust-lang.org)
pub fn fonction_bien_documentee() {
    // Implémentation...
}
```

### Sections standards de documentation

Il existe des conventions pour organiser la documentation en sections. Les sections courantes sont :

``` rust
/// # Panics
/// Décrit les situations où la fonction panique.
///
/// # Errors
/// Décrit les erreurs qui peuvent être retournées et dans quelles conditions.
///
/// # Safety
/// Si la fonction est marquée comme `unsafe`, expliquez pourquoi et comment l'utiliser en toute sécurité.
///
/// # Examples
/// Des exemples d'utilisation, qui seront également testés par rustdoc.
///
/// ```
/// let resultat = ma_crate::ma_fonction(42);
/// assert_eq!(resultat, 84);
/// ```
///
/// # Arguments
/// Documentation des paramètres.
///
/// # Returns
/// Documentation de la valeur de retour.
///
/// # See also
/// Liens vers d'autres éléments connexes.
pub fn ma_fonction(valeur: i32) -> Result<i32, String> {
    if valeur < 0 {
        Err("La valeur ne peut pas être négative".to_string())
    } else {
        Ok(valeur * 2)
    }
}
```

### Liens intra-doc

Une fonctionnalité puissante de `rustdoc` est la capacité à créer automatiquement des liens vers d'autres éléments de votre crate ou de la bibliothèque standard. Pour cela, utilisez la syntaxe `[nom_de_l_élément]` :

``` rust
/// Cette fonction calcule le hachage CRC32 d'une chaîne.
///
 /// Elle utilise la structure [`Hasher`] pour effectuer le calcul.
///
 /// # Exemples
///
 /// ```
/// let hash = mon_crate::crc32_hash("Hello world");
/// println!("Le hachage CRC32 est : {}", hash);
/// ```
///
 /// Voir aussi [`sha1_hash`] pour une alternative plus sécurisée.
pub fn crc32_hash(input: &str) -> u32 {
    use crc::{Crc, CRC_32_ISO_HDLC};

    const CRC32: Crc<u32> = Crc::<u32>::new(&CRC_32_ISO_HDLC);
    CRC32.checksum(input.as_bytes())
}

/// Calcule le hachage SHA-1 d'une chaîne de caractères.
///
 /// Cette fonction utilise l'algorithme [`sha1::Sha1`] de la crate externe.
pub fn sha1_hash(input: &str) -> String {
    use sha1::{Sha1, Digest};
    let mut hasher = Sha1::new();
    hasher.update(input.as_bytes());
    format!("{:x}", hasher.finalize())
}

/// Interface abstraite pour les objets réalisant des opérations de hachage.
pub trait Hasher {
    /// Calcule un hachage à partir d'une chaîne de caractères.
    fn hash(&self, input: &str) -> String;
}
```

Les liens intra-doc fonctionnent également avec les chemins complets et peuvent pointer vers des méthodes ou des constantes :

``` rust
/// Voir [`std::collections::HashMap`] pour une implémentation standard.
///
 /// La méthode [`HashTable::insert`] permet d'ajouter une entrée.
pub struct HashTable<K, V> {
    // Implémentation
}

impl<K, V> HashTable<K, V> {
    /// Insère une paire clé-valeur dans la table.
    ///
/// Cela fonctionne de manière similaire à [`std::collections::HashMap::insert`].
    pub fn insert(&mut self, key: K, value: V) {
        // Implémentation
    }
}
```

### Masquer des éléments dans la documentation

Si vous souhaitez qu'un élément public soit accessible dans le code mais n'apparaisse pas dans la documentation, utilisez l'attribut `#[doc(hidden)]` :

``` rust
/// Cette structure est documentée normalement.
pub struct DocumentedStruct;

#[doc(hidden)]
/// Cette fonction ne sera pas visible dans la documentation générée,
/// même si elle est publique et pourra être utilisée par d'autres crates.
pub fn internal_function() {
    // Implémentation...
}
```

### Alias de recherche

Pour aider les utilisateurs à trouver vos éléments via la fonction de recherche, vous pouvez définir des alias de recherche :

``` rust
/// Structure représentant une erreur réseau.
#[doc(alias = "network error")]
#[doc(alias = "connection error")]
#[doc(alias = "io error")]
pub struct NetworkError {
    // Champs...
}
```

Avec ces alias, quand un utilisateur recherche "network error", "connection error" ou "io error" dans la documentation, `NetworkError` apparaîtra dans les résultats même si ces termes n'apparaissent pas dans le nom de la structure.

### Tests dans la documentation

Un avantage majeur de la documentation Rust est que les exemples de code sont automatiquement testés lors de l'exécution de `cargo test`. Cela garantit que votre documentation reste à jour avec votre code.

``` rust
/// Convertit une température de Celsius en Fahrenheit.
///
/// # Exemples
///
/// ```
/// let celsius = 25.0;
/// let fahrenheit = mon_crate::celsius_to_fahrenheit(celsius);
/// assert_eq!(fahrenheit, 77.0);
/// ```
pub fn celsius_to_fahrenheit(celsius: f64) -> f64 {
    celsius * 9.0 / 5.0 + 32.0
}
```

Si vous ne voulez pas que votre exemple soit testé, utilisez la syntaxe suivante :

``` rust
/// ```no_run
/// // Ce code ne sera pas exécuté pendant les tests
/// let result = ma_crate::fonction_qui_fait_quelque_chose_de_lourd();
/// ```
///
/// ```ignore
/// // Ce code ne sera même pas compilé
/// ceci n'est pas du code Rust valide;
/// ```
pub fn fonction_documentee() {
    // Implémentation...
}
```

### Personnalisation de l'apparence

Pour personnaliser l'apparence de votre documentation, vous pouvez définir plusieurs attributs au niveau de la crate (dans `lib.rs` ou `main.rs`) :

``` rust
//! # Ma Bibliothèque de Hachage
//!
//! Une collection d'algorithmes de hachage cryptographiques.

#![doc(html_logo_url = "https://example.com/logo.png")]
#![doc(html_favicon_url = "https://example.com/favicon.ico")]
#![doc(html_root_url = "https://docs.rs/ma_crate/0.1.0")]
#![doc(issue_tracker_base_url = "https://github.com/username/ma_crate/issues/")]
```

Ces attributs permettent de :

- Définir un logo personnalisé qui apparaîtra dans la barre latérale
- Définir une favicon personnalisée pour les onglets du navigateur
- Spécifier l'URL racine de la documentation (utile pour les liens)
- Définir l'URL de base du système de suivi des problèmes

### Documentation du fichier README

Vous pouvez inclure le contenu de votre fichier README.md comme page d'accueil de votre documentation en ajoutant ceci à votre `Cargo.toml` :

``` toml
[package]
name = "ma_crate"
version = "0.1.0"
authors = ["Votre Nom <votre.email@exemple.com>"]
edition = "2021"
description = "Une description de ma crate"
repository = "https://github.com/username/ma_crate"
readme = "README.md"
```

### Exercices pratiques

Pour bien maîtriser `rustdoc`, essayez ces exercices :

1.  Documentez une structure avec plusieurs champs et méthodes
2.  Incluez des exemples de code dans votre documentation
3.  Créez des liens intra-doc vers d'autres éléments
4.  Générez la documentation avec `cargo doc --open` et vérifiez le résultat
5.  Ajoutez une page de documentation personnalisée en markdown

La documentation est un aspect essentiel du développement en Rust. Une bonne documentation aide non seulement les autres développeurs, mais aussi vous-même lorsque vous revenez sur votre code après un certain temps. Prendre le temps de bien documenter votre code est un investissement qui sera toujours rentable à long terme.

⏭️ [Ajouter des tests](/III-avance/05-tests.md)
