## 12\. Gestion des erreurs

La gestion des erreurs en Rust est explicite et type-safe, ce qui contribue à la fiabilité du code. Contrairement à d'autres langages qui utilisent des exceptions ou des valeurs nulles, Rust propose des types spécifiques pour représenter les erreurs potentielles.

### Option

`Option<T>` représente une valeur optionnelle : soit une valeur de type `T`, soit rien.

```
enum Option<T> {
    Some(T),
    None,
}
```

Voici un exemple d'utilisation :

```
fn trouver_utilisateur(id: u32) -> Option<String> {
    let utilisateurs = vec![
        (1, "Alice"),
        (2, "Bob"),
        (3, "Charlie"),
    ];

    for (user_id, nom) in utilisateurs {
        if user_id == id {
            return Some(nom.to_string());
        }
    }

    None
}

// Utilisation sécurisée avec pattern matching
match trouver_utilisateur(2) {
    Some(nom) => println!("Utilisateur trouvé : {}", nom),
    None => println!("Utilisateur introuvable"),
}

// Utilisation avec if let
if let Some(nom) = trouver_utilisateur(4) {
    println!("Utilisateur trouvé : {}", nom);
} else {
    println!("Utilisateur introuvable");
}
```

### Result&lt;T, E&gt;

`Result<T, E>` représente soit un succès (`Ok`) contenant une valeur de type `T`, soit une erreur (`Err`) contenant une valeur de type `E`.

```
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Exemple d'utilisation avec la lecture d'un fichier :

```
use std::fs::File;
use std::io::{self, Read};

fn lire_fichier(chemin: &str) -> Result<String, io::Error> {
    let mut fichier = match File::open(chemin) {
        Ok(f) => f,
        Err(e) => return Err(e),
    };

    let mut contenu = String::new();
    match fichier.read_to_string(&mut contenu) {
        Ok(_) => Ok(contenu),
        Err(e) => Err(e),
    }
}

// Utilisation avec pattern matching
match lire_fichier("config.txt") {
    Ok(contenu) => println!("Contenu du fichier : {}", contenu),
    Err(e) => println!("Erreur lors de la lecture : {}", e),
}
```

### Propagation des erreurs avec l'opérateur ?

L'opérateur `?` permet de propager les erreurs de manière concise. Il est équivalent à un match qui retourne l'erreur et déroule l'appel de fonction si `Err`, ou qui continue l'exécution avec la valeur déballée si `Ok`.

Voici comment réécrire la fonction `lire_fichier` avec l'opérateur `?` :

```
use std::fs::File;
use std::io::{self, Read};

fn lire_fichier(chemin: &str) -> Result<String, io::Error> {
    let mut fichier = File::open(chemin)?;
    let mut contenu = String::new();
    fichier.read_to_string(&mut contenu)?;
    Ok(contenu)
}

// Encore plus concis
fn lire_fichier_court(chemin: &str) -> Result<String, io::Error> {
    let mut contenu = String::new();
    File::open(chemin)?.read_to_string(&mut contenu)?;
    Ok(contenu)
}
```

L'opérateur `?` fonctionne avec `Option<T>` de la même manière :

```
fn premier_dernier_caractere(texte: &str) -> Option<(char, char)> {
    let premier = texte.chars().next()?;
    let dernier = texte.chars().last()?;
    Some((premier, dernier))
}
```

### Conversion d'erreurs

Parfois, vous devez convertir un type d'erreur en un autre. L'opérateur `?` peut être utilisé avec la méthode `map_err` pour cela :

```
use std::fs::File;
use std::io;

#[derive(Debug)]
enum AppError {
    FichierIntrouvable(String),
    ErreurLecture(String),
    ErreurDonnees(String),
}

fn lire_configuration() -> Result<String, AppError> {
    let fichier = File::open("config.toml").map_err(|e| {
        AppError::FichierIntrouvable(format!("Impossible d'ouvrir le fichier: {}", e))
    })?;

    // Reste de la fonction...
    Ok("Configuration chargée".to_string())
}
```

### La macro panic!

`panic!` est utilisée pour les erreurs irrécupérables. Elle arrête immédiatement le programme (ou le thread actuel) :

```
fn diviser(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Division par zéro!");
    }
    a / b
}

// Avec un message formaté
fn verifier_age(age: i32) {
    if age < 0 {
        panic!("Âge invalide: {} (doit être positif)", age);
    }
}
```

## unwrap() et expect()
Ces méthodes extraient la valeur ou déclenchent une panique. À utiliser avec précaution :
``` rust
fn exemple_unwrap_et_expect() {
    // Option<T> - unwrap() extrait la valeur si elle existe
    let valeur = Some(42).unwrap();  // Ok: valeur = 42
    println!("Valeur extraite : {}", valeur);

    // ATTENTION : Le code suivant provoque une panique ! Ne l'exécutez pas
    // en production sauf si une panique est le comportement souhaité.
    // let probleme = None.unwrap();  // Provoquerait une panique avec le message:
                                      // "called `Option::unwrap()` on a `None` value"

    // Result<T, E> - expect() permet de personnaliser le message d'erreur
    // Note: Ce code échouera si le fichier n'existe pas
    use std::fs::File;
    match File::open("important.dat") {
        Ok(fichier) => println!("Fichier ouvert avec succès"),
        Err(erreur) => println!("Erreur lors de l'ouverture: {}", erreur),
    }

    // La version avec expect() ci-dessous provoquerait une panique avec un message personnalisé
    // si le fichier n'existe pas:
    // let fichier = File::open("important.dat")
    //     .expect("Le fichier important.dat doit exister!");
}
```
## unwrap_or() et unwrap_or_else()
Ces méthodes fournissent une valeur par défaut en cas d'échec, évitant ainsi les paniques :
``` rust
fn exemple_unwrap_or() {
    // Imaginons une fonction qui peut retourner None
    fn rechercher_utilisateur(id: i32) -> Option<String> {
        match id {
            1 => Some(String::from("Alice")),
            2 => Some(String::from("Bob")),
            _ => None,
        }
    }

    // unwrap_or() - fournit une valeur par défaut statique si None
    let utilisateur1 = rechercher_utilisateur(1).unwrap_or(String::from("Inconnu"));
    println!("Utilisateur 1: {}", utilisateur1);  // "Alice"

    let utilisateur3 = rechercher_utilisateur(3).unwrap_or(String::from("Inconnu"));
    println!("Utilisateur 3: {}", utilisateur3);  // "Inconnu"

    // unwrap_or_else() - calcule une valeur par défaut via une closure si None
    let utilisateur4 = rechercher_utilisateur(4).unwrap_or_else(|| {
        println!("Attention: Utilisateur non trouvé, création d'un utilisateur par défaut");
        String::from("Invité temporaire")
    });
    println!("Utilisateur 4: {}", utilisateur4);  // "Invité temporaire"
}
```
## and_then(), map(), et autres combinateurs
Ces méthodes permettent des transformations fonctionnelles sur `Option` et `Result` :
``` rust
fn exemple_combinateurs() {
    // Exemple avec Option - chaînage d'opérations
    let resultat = Some(5)
        .map(|x| x * 2)                // Some(10)
        .and_then(|x| if x > 5 { Some(x) } else { None })  // Some(10) car 10 > 5
        .map(|x| x.to_string());       // Some("10")

    println!("Résultat: {:?}", resultat);  // Some("10")

    // Exemple avec None - la chaîne s'arrête au premier None
    let resultat_vide = None
        .map(|x: i32| x * 2)           // None
        .and_then(|x| Some(x + 1))     // None (cette opération est ignorée)
        .map(|x| x.to_string());       // None (cette opération est ignorée également)

    println!("Résultat vide: {:?}", resultat_vide);  // None

    // Exemple complet avec Result
    fn traiter_donnees(input: &str) -> Result<i32, String> {
        // Tente de convertir la chaîne en nombre
        input.parse::<i32>()
            // Convertit l'erreur de parsing en une erreur de type String
            .map_err(|e| format!("Erreur de parsing: {}", e))
            // Si la conversion réussit, vérifie si le nombre est positif
            .and_then(|n| {
                if n > 0 {
                    Ok(n * 2)
                } else {
                    Err("Le nombre doit être positif".to_string())
                }
            })
    }

    // Test de la fonction avec différentes entrées
    println!("Traitement de \"42\": {:?}", traiter_donnees("42"));  // Ok(84)
    println!("Traitement de \"-5\": {:?}", traiter_donnees("-5"));  // Err(...)
    println!("Traitement de \"abc\": {:?}", traiter_donnees("abc"));  // Err(...)
}

fn main() {
    exemple_unwrap_et_expect();
    println!("\n-----------\n");
    exemple_unwrap_or();
    println!("\n-----------\n");
    exemple_combinateurs();
}
```
## Points importants à retenir :
1. **unwrap() et expect()** :
    - Utilisez-les uniquement lorsque vous êtes certain que l'opération réussira ou si une panique est acceptable.
    - Pour le code de production, préférez des alternatives plus sûres.

2. **unwrap_or() et unwrap_or_else()** :
    - Alternatives sûres à `unwrap()` qui ne provoqueront jamais de panique.
    - `unwrap_or_else()` est plus efficace quand la valeur par défaut est coûteuse à calculer.

3. **Les combinateurs fonctionnels** :
    - Permettent d'écrire du code concis et expressif.
    - Éliminent la nécessité de nombreux blocs `match` imbriqués.
    - Court-circuitent le traitement dès qu'une erreur ou `None` est rencontré.

Cette approche de gestion des erreurs encourage à traiter explicitement tous les cas possibles, rendant votre code plus robuste et prévisible.

La gestion des erreurs en Rust encourage les bonnes pratiques de programmation en rendant explicites les cas d'erreur, tout en restant flexible pour s'adapter à différents besoins.
