## 3\. Les attributs

Les attributs en Rust sont des métadonnées puissantes qui peuvent être appliquées à différents éléments du code. Ils servent à communiquer des informations supplémentaires au compilateur ou à d'autres outils. Ces métadonnées peuvent modifier le comportement de la compilation, générer des avertissements personnalisés, ou ajouter automatiquement des fonctionnalités à vos types.

### Formes des attributs

Les attributs existent sous deux formes principales : interne et externe.

- **Attributs internes** : Appliqués à l'élément qui les contient, notation: `#![...]`
- **Attributs externes** : Appliqués à l'élément qui les suit, notation: `#[...]`

Voici un exemple illustrant les deux formes :

``` rust
#![allow(dead_code)]              // Attribut interne appliqué au module entier (crate)

#[deprecated(since = "1.0.0", note = "Veuillez utiliser la nouvelle_fonction() à la place")]
fn ancienne_fonction() {          // Attribut externe appliqué à cette fonction
    println!("Cette fonction est obsolète");
}

mod configuration {
    #![warn(missing_docs)]        // Attribut interne appliqué à ce module uniquement

    #[doc = "Renvoie la configuration par défaut du système"]
    pub fn config_par_defaut() -> String {  // Attribut externe sur cette fonction
        "Configuration par défaut".to_string()
    }
}
```

### Catégories d'attributs

Rust propose quatre types d'attributs principaux :

#### 1\. Attributs intégrés au compilateur

Ces attributs sont fournis directement par le compilateur Rust. Voici quelques attributs particulièrement utiles :

##### Gestion des avertissements : `allow`, `warn`, `deny` et `forbid`

Ces attributs contrôlent le niveau des lints (vérifications du compilateur) :

``` rust
#[allow(unused_variables)]
fn traitement_donnees(data: &str) {
    // Le compilateur ne générera pas d'avertissement pour cette variable non utilisée
    let timestamp = chrono::Utc::now();
    println!("Traitement des données: {}", data);
}

#[deny(non_snake_case)]
fn process_data() {
    // Cette fonction provoquera une erreur de compilation si elle contient des variables
    // qui ne respectent pas la convention snake_case
    let valeurImportante = 42;  // Erreur de compilation!
	println!("La valeur importante est : {}", valeur_importante);
}

// Utilisation
fn main() {
    traitement_donnees("exemple");
    process_data();
}
```

version corrigée:
``` rust
#[allow(unused_variables)]
fn traitement_donnees(data: &str) {
    // Le compilateur ne générera pas d'avertissement pour cette variable non utilisée
    let timestamp = chrono::Utc::now();
    println!("Traitement des données: {}", data);
}

#[deny(non_snake_case)]
fn process_data() {
    // Cette fonction provoquera une erreur de compilation si elle contient des variables
    // qui ne respectent pas la convention snake_case
    let valeur_importante = 42;  // Erreur de compilation!
    println!("La valeur importante est : {}", valeur_importante);
}

// Utilisation
fn main() {
    traitement_donnees("exemple");
    process_data();
}
```

##### `must_use` - Rendre obligatoire l'utilisation d'une valeur

Cet attribut est crucial pour les fonctions qui retournent des résultats importants :

``` rust
// Sur un type
#[must_use]
struct ResultatCalcul {
    valeur: f64,
    precision: f64,
}

impl ResultatCalcul {
    fn nouveau(valeur: f64, precision: f64) -> Self {
        Self { valeur, precision }
    }
}

// Sur une fonction
#[must_use = "Ce résultat contient des erreurs potentielles qui doivent être gérées"]
fn diviser(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Division par zéro".to_string())
    } else {
        Ok(a / b)
    }
}

fn main() {
    // Ceci générera un avertissement
    ResultatCalcul::nouveau(3.14, 0.001);

    // Ceci aussi
    diviser(10.0, 2.0);

    // Utilisation correcte:
    let resultat = ResultatCalcul::nouveau(3.14, 0.001);
    println!("Valeur calculée: {}", resultat.valeur);

    // Ou
    if let Ok(quotient) = diviser(10.0, 2.0) {
        println!("Résultat: {}", quotient);
    }
}
```

##### `inline` - Contrôle de l'inlining des fonctions

```
#[inline]
fn fonction_frequemment_appelee(x: i32) -> i32 {
    x * x + 1
}

#[inline(always)]
fn operation_tres_simple(x: i32, y: i32) -> i32 {
    x + y
}

#[inline(never)]
fn fonction_complexe(data: &[i32]) -> i32 {
    data.iter().fold(0, |acc, &x| acc + x * x)
}
```

###### Les bases de l'inlining
L'**inlining** est une technique d'optimisation où le code d'une fonction est directement inséré à l'endroit où elle est appelée, plutôt que de créer un appel de fonction standard. Cela peut améliorer les performances en :
- Éliminant le coût de l'appel de fonction (sauvegarder/restaurer le contexte)
- Permettant d'autres optimisations par le compilateur

###### Les trois variantes de `#[inline]`
####### 1. `#[inline]`
``` rust
#[inline]
fn fonction_frequemment_appelee(x: i32) -> i32 {
    x * x + 1
}
```
Cette variante est une **suggestion** au compilateur que cette fonction pourrait bénéficier d'un inlining. Le compilateur conserve toujours le droit de décider s'il va effectivement réaliser l'inlining ou non, en fonction de divers facteurs comme :
- La taille de la fonction
- La fréquence d'appel
- Les optimisations globales

Utilisée généralement pour des fonctions courtes et fréquemment appelées.
####### 2. `#[inline(always)]`
``` rust
#[inline(always)]
fn operation_tres_simple(x: i32, y: i32) -> i32 {
    x + y
}
```
Cette directive est plus forte et demande au compilateur de **toujours** effectuer l'inlining de la fonction, quelles que soient les autres considérations. À utiliser avec précaution car :
- Elle peut augmenter la taille du code binaire
- Elle peut être contre-productive pour des fonctions complexes
- Le compilateur peut encore l'ignorer dans certains cas extrêmes

Réservée typiquement aux opérations très simples et critiques pour les performances.
####### 3. `#[inline(never)]`
``` rust
#[inline(never)]
fn fonction_complexe(data: &[i32]) -> i32 {
    data.iter().fold(0, |acc, &x| acc + x * x)
}
```
C'est l'opposé de `inline(always)` : elle indique au compilateur de ne **jamais** réaliser l'inlining de cette fonction. Utilisée quand :
- La fonction est grande et complexe
- Elle est rarement appelée
- Pour améliorer la taille du binaire
- Pour des raisons de débogage (plus facile de suivre une fonction non inlinée)

###### Bonnes pratiques
1. **Utilisez rarement ces attributs** - Le compilateur Rust est généralement très bon pour déterminer quand l'inlining est bénéfique.
2. **Profiler avant d'optimiser** - N'ajoutez pas ces attributs sans mesurer leur impact réel sur les performances.
3. **Cas d'usage communs** :
    - `#[inline]` : Fonctions génériques, méthodes d'accès simples, wrappers légers
    - `#[inline(always)]` : Opérations mathématiques très simples, fonctions critiques pour les performances
    - `#[inline(never)]` : Fonctions volumineuses ou rarement appelées

L'inlining est une optimisation de bas niveau que vous n'aurez généralement pas besoin d'utiliser explicitement dans la plupart des programmes Rust, mais qui peut être utile dans les bibliothèques et les systèmes très sensibles aux performances.

##### `doc` - Documentation

``` rust
/// Documentation courte sur une seule ligne
#[doc = "Documentation plus élaborée qui peut s'étendre
 sur plusieurs lignes et inclure des exemples de code."]
fn ma_fonction_documentee() {
    // Code
}
```

#### 2\. Attributs d'outils

Ces attributs sont fournis par des outils externes intégrés à l'écosystème Rust.

##### rustfmt

``` rust
// Empêche le formatteur de modifier cette fonction
#[rustfmt::skip]
fn code_mal_formate() {
    let x    =    42;
    let y=100;
    if x>y{println!("x est plus grand");}else{
        println!("y est plus grand ou égal");
    }
}

// On peut également ignorer des sections précises
fn fonction_formatee() {
    let mut resultat = 0;

    #[rustfmt::skip]
    let tableau = [
        1, 2, 3,
        4,    5,     6,
        7,8,9
    ];

    // Le reste de la fonction sera formaté normalement
}
```

##### clippy

``` rust
#[clippy::cognitive_complexity = "30"]
fn fonction_complexe() {
    // Clippy acceptera une complexité plus élevée pour cette fonction
}
```

#### 3\. Macros attributs

Les macros attributs modifient entièrement le code sur lequel elles sont appliquées. Un exemple simple est `test` :

``` rust
#[test]
fn verifier_addition() {
    assert_eq!(2 + 2, 4);
}

// Un exemple plus avancé avec la bibliothèque web actix
#[get("/utilisateurs/{id}")]
async fn obtenir_utilisateur(id: web::Path<u64>) -> impl Responder {
    format!("Obtention de l'utilisateur avec ID: {}", id)
}
```

Un autre exemple populaire est `cfg_attr` qui permet d'appliquer conditionnellement des attributs :

``` rust
#[cfg_attr(target_os = "linux", path = "implementation_linux.rs")]
#[cfg_attr(target_os = "windows", path = "implementation_windows.rs")]
mod implementation_specifique;
```

#### 4\. Attributs derive

Ces attributs permettent d'implémenter automatiquement des traits pour vos types. Voici quelques exemples courants :

``` rust
// Implémente automatiquement plusieurs traits utiles
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct Utilisateur {
    id: u64,
    nom: String,
    email: String,
    actif: bool,
}

// Serde pour la sérialisation/désérialisation
#[derive(serde::Serialize, serde::Deserialize)]
struct Configuration {
    #[serde(rename = "serverUrl")]
    url_serveur: String,

    #[serde(default = "valeur_par_defaut_port")]
    port: u16,

    #[serde(skip_serializing_if = "Vec::is_empty")]
    extensions: Vec<String>,
}

fn valeur_par_defaut_port() -> u16 {
    8080
}
```

### Attributs pour le contrôle de la visibilité et de la stabilité

Rust offre également des attributs pour contrôler la visibilité et la stabilité des éléments de code :

``` rust
#[non_exhaustive]
pub enum Statut {
    Actif,
    Inactif,
    EnPause,
    // D'autres variantes pourront être ajoutées dans le futur
}

#[deprecated(since = "2.0.0", note = "Utilisez nouvelle_api() à la place")]
pub fn ancienne_api() {
    // ...
}

// Dans les bibliothèques, pour des fonctionnalités expérimentales
#[unstable(feature = "ma_fonctionnalite", issue = "12345")]
pub fn fonction_experimentale() {
    // ...
}
```

### Attributs pour la compilation conditionnelle

Les attributs `cfg` sont essentiels pour adapter le code à différents environnements :

``` rust
#[cfg(target_os = "linux")]
fn initialiser_systeme() {
    println!("Initialisation pour Linux");
}

#[cfg(not(target_os = "linux"))]
fn initialiser_systeme() {
    println!("Initialisation pour d'autres OS");
}

#[cfg(feature = "fonctionnalites_premium")]
pub mod fonctionnalites_premium {
    pub fn activer() {
        println!("Fonctionnalités premium activées");
    }
}

#[cfg(any(target_arch = "x86_64", target_arch = "aarch64"))]
pub fn optimisations_64bits() {
    // Code optimisé pour architectures 64 bits
}
```

### Conclusion

Les attributs en Rust constituent un mécanisme puissant et extensible pour ajouter des métadonnées à votre code. Ils vous permettent de contrôler la compilation, d'ajouter des comportements automatiques à vos types, et de guider les utilisateurs de votre code. Bien utilisés, ils peuvent considérablement améliorer la qualité, la lisibilité et la robustesse de vos programmes Rust.

Pour une documentation exhaustive sur tous les attributs disponibles, consultez la [documentation officielle de Rust](https://doc.rust-lang.org/reference/attributes.html).
