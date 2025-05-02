## 1\. Le formatage des flux

Retour à la [Table des matières](/SOMMAIRE.md)

Le formatage des flux en Rust est un aspect fondamental qui permet de contrôler précisément l'affichage des données. Contrairement à d'autres langages, Rust offre une approche puissante et flexible via ses macros de formatage.

### Les macros `print!` et `println!`

Ces macros sont les plus couramment utilisées pour l'affichage dans la console:

``` rust
println!("Bonjour tout le monde!"); // Affichage simple
println!("{} {} {}", "Bonjour", "monde", "rustacé!"); // Avec placeholders
```

L'un des avantages majeurs du système de formatage de Rust est la possibilité de réorganiser l'ordre d'affichage des arguments sans changer leur ordre dans l'appel de la macro:

``` rust
println!("{1} {0} {2}", "au", "Bienvenue", "langage Rust!");
// Affiche: "Bienvenue au langage Rust!"
```

### Formatage avancé

Le formatage de Rust va bien au-delà du simple positionnement des arguments. Voici quelques possibilités:

**Formatage des nombres décimaux:**

``` rust
let pi: f64 = 3.14159265358979323846;
println!("Pi avec 2 décimales: {:.2}", pi); // Affiche: Pi avec 2 décimales: 3.14
println!("Pi avec 5 décimales: {:.5}", pi); // Affiche: Pi avec 5 décimales: 3.14159
```

**Représentations alternatives des nombres entiers:**

``` rust
let nombre = 42i32;
println!("Binaire: {:b}", nombre);     // Affiche: Binaire: 101010
println!("Octal: {:o}", nombre);       // Affiche: Octal: 52
println!("Hexadécimal: {:x}", nombre); // Affiche: Hexadécimal: 2a
println!("Hexadécimal (majuscules): {:X}", nombre); // Affiche: Hexadécimal (majuscules): 2A
```

**Alignement et remplissage:**

``` rust
let petit = 7;
let grand = 42000;
println!("{:10}", petit);  // Aligne à droite sur 10 caractères
println!("{:<10}", petit); // Aligne à gauche sur 10 caractères
println!("{:^10}", petit); // Centre sur 10 caractères
println!("{:0>5}", petit); // Remplit avec des '0' à gauche sur 5 caractères: 00007
println!("{:_<8}", grand); // Remplit avec des '_' à droite sur 8 caractères: 42000___
```

### Arguments nommés (depuis Rust 1.58)

Depuis la version 1.58, Rust permet d'utiliser des arguments nommés pour améliorer la lisibilité:

``` rust
let nom = "Alice";
let age = 30;
println!("Je m'appelle {nom} et j'ai {age} ans.");
// Équivalent à: println!("Je m'appelle {} et j'ai {} ans.", nom, age);

// Les options de formatage fonctionnent aussi avec les arguments nommés
println!("{age:04}"); // Affiche: 0030
```

### La macro `format!`

La macro `format!` utilise la même syntaxe que `println!` mais au lieu d'afficher sur la sortie standard, elle renvoie une `String`:

``` rust
    let nombre = 42;
    let message = format!("La réponse est {}", nombre);
    println!("{}",message);
    // message contient maintenant la chaîne "La réponse est 42"

    // Formatage complexe
    let utilisateur = "Alice";
    let score = 95.5;
    let formatte = format!("{utilisateur} a obtenu {score:.1} points");
    // formatte contient: "Alice a obtenu 95.5 points"

    println!("{}",formatte);
```

Cette macro est particulièrement utile pour construire des chaînes complexes ou pour préparer un message avant de l'afficher ou de le stocker.

### Écriture dans d'autres flux avec `write!`

Le formatage n'est pas limité à la console. La macro `write!` permet d'écrire dans n'importe quel type implémentant le trait `std::io::Write`:

``` rust
use std::io::Write;
use std::fs::File;

// Écriture dans un fichier
let mut fichier = File::create("sortie.txt").expect("Erreur lors de la création du fichier");
write!(&mut fichier, "Bonjour, {}!", "monde").expect("Erreur d'écriture");

// Écriture dans un buffer de mémoire
let mut buffer = Vec::new();
write!(&mut buffer, "Données: {:?}", [1, 2, 3]).expect("Erreur d'écriture");
println!("Buffer: {:?}", buffer); // Affiche les octets écrits
```

La macro `writeln!` est également disponible pour ajouter automatiquement un saut de ligne après le contenu formaté.

### Debug et Display

Rust distingue deux principales façons d'afficher un type:

- `Debug` (`{:?}`) : représentation orientée développeur
- `Display` (`{}`) : représentation orientée utilisateur

``` rust
// Utilisation de Debug
let nombres = vec![1, 2, 3];
println!("{:?}", nombres); // Affiche: [1, 2, 3]
println!("{:#?}", nombres); // Affiche version "pretty print" avec formatage multi-ligne
```

Le système de formatage de Rust est puissant et fait partie des fondements qui permettent d'écrire du code clair et expressif.

⏭️ [Les traits](/II-specificites/02-traits.md)
