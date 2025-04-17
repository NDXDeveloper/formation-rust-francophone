## 4\. Variables

Les variables sont un concept fondamental dans tout langage de programmation. Rust introduit plusieurs particularités importantes dans la gestion des variables qui le distinguent des autres langages.

### 4.1 Immuabilité par défaut

La première particularité de Rust est que **toutes les variables sont immuables (constantes) par défaut**. C'est un choix délibéré pour favoriser la programmation sécurisée.

``` rust
fn main() {
    let nombre = 42;
    nombre = 50;  // Erreur de compilation !
    println!("Le nombre est {}", nombre);
}
```

Ce code ne compilera pas car nous essayons de modifier une variable immuable. Pour rendre une variable mutable, nous devons utiliser le mot-clé `mut` :

``` rust
fn main() {
    let mut nombre = 42;
    println!("Le nombre initial est {}", nombre);
    nombre = 50;
    println!("Le nombre est maintenant {}", nombre);
}
```

### 4.2 Types de données

Rust est un langage à typage statique, mais avec inférence de types. Cela signifie que le compilateur détermine automatiquement le type d'une variable à partir de son contexte d'utilisation.

#### Types numériques

``` rust
fn main() {
        // Types explicites
    let entier_signe: i32 = -42;
    let entier_non_signe: u64 = 100;
    let nombre_flottant: f64 = 3.14159;

    // Avec des suffixes de type
    let petit_entier = 42i8;
    let grand_entier = 9999999999u64;
    let pi = 3.14159f32;

    // Avec inférence de type
    let age = 30;  // i32 par défaut
    let prix = 19.99;  // f64 par défaut

    println!("Valeurs: {}, {}, {}, {}, {}, {}, {}, {}", entier_signe, entier_non_signe, nombre_flottant, petit_entier, grand_entier, pi, age, prix);

}
```

Rust propose plusieurs types numériques :

| Type | Description | Plage de valeurs |
| --- | --- | --- |
| i8, i16, i32, i64, i128 | Entiers signés | \-2^(n-1) à 2^(n-1)-1 |
| u8, u16, u32, u64, u128 | Entiers non signés | 0 à 2^n-1 |
| f32, f64 | Nombres à virgule flottante | Dépend de la norme IEEE 754 |
| isize, usize | Dépend de l'architecture (32 ou 64 bits) | Varie selon la plateforme |
| bool | Booléen | true ou false |
| char | Caractère Unicode | Toute valeur scalaire Unicode valide |

#### Opérations sur les variables

Contrairement à d'autres langages comme C/C++, Rust n'a pas d'opérateurs d'incrémentation/décrémentation (`++`, `--`). Il faut utiliser les opérateurs composés :

``` rust
fn main() {
    let mut compteur = 0;

    // Incrémentation
    compteur += 1;
    println!("Compteur: {}", compteur);

    // Décrémentation
    compteur -= 1;
    println!("Compteur: {}", compteur);

    // Autres opérations composées
    let mut valeur = 5;
    println!("valeur: {}", valeur);
    valeur *= 2;  // valeur = valeur * 2
    println!("valeur: {}", valeur);
    valeur /= 5;  // valeur = valeur / 5
    println!("valeur: {}", valeur);
    valeur %= 2;  // valeur = valeur % 2
    println!("valeur: {}", valeur);
}
```

### 4.3 Shadowing (masquage)

Rust permet de redéclarer une variable avec le même nom, ce qui "masque" la déclaration précédente :

``` rust
fn main() {
    let valeur = 5;
    println!("Valeur initiale: {}", valeur);

    let valeur = valeur + 5;  // Nouvelle variable qui masque l'ancienne
    println!("Après addition: {}", valeur);

    let valeur = "Maintenant une chaîne";  // On peut même changer le type !
    println!("{}", valeur);
}
```

Ceci est différent de la mutabilité, car chaque `let` crée une nouvelle variable.

### 4.4 Collections

#### Tableaux (taille fixe)

Les tableaux en Rust ont une taille fixe connue à la compilation :

```rust
fn main() {
    // Tableau avec initialisation
    let nombres = [1, 2, 3, 4, 5];

    // Tableau avec type et taille explicites
    let zeros: [i32; 3] = [0; 3];  // Crée [0, 0, 0]

    // Accès aux éléments (l'indexation commence à 0)
    println!("Premier élément: {}", nombres[0]);
    println!("Dernier élément: {}", nombres[4]);

    // Tableau mutable
    let mut scores = [100, 90, 80, 65];
    scores[0] = 95;

    // Obtenir la longueur
    println!("Taille du tableau nombres : {}", nombres.len());
    println!("Taille du tableau zeros : {}", zeros.len());
    println!("Taille du tableau scores : {}", scores.len());

    // Afficher chaque élément de nombres
    println!("nombres: {:?}", nombres);
    println!("zeros: {:?}", zeros);
    println!("scores: {:?}", scores);


    // Afficher chaque élément de nombres
    print!("nombres: [");
    for (i, &nombre) in nombres.iter().enumerate() {
        if i > 0 { print!(", "); }
        print!("{}", nombre);
    }
    println!("]");

    // Afficher chaque élément de zeros
    print!("zeros: [");
    for (i, &zero) in zeros.iter().enumerate() {
        if i > 0 { print!(", "); }
        print!("{}", zero);
    }
    println!("]");

    // Afficher chaque élément de scores
    print!("scores: [");
    for (i, &score) in scores.iter().enumerate() {
        if i > 0 { print!(", "); }
        print!("{}", score);
    }
    println!("]");
}
```

#### Vecteurs (taille dynamique)

Pour une collection de taille variable, utilisez `Vec<T>` :

```rust
fn main() {
    // Création d'un vecteur vide
    let mut nombres: Vec<i32> = Vec::new();

    // Ajout d'éléments
    nombres.push(1);
    nombres.push(2);
    nombres.push(3);
    println!("nombres : {:?}", nombres);

    // Création avec des valeurs initiales via macro
    let mut couleurs = vec!["rouge", "vert", "bleu"];

    // Modification
    couleurs[0] = "jaune";

    // Suppression du dernier élément
    let dernier = nombres.pop();  // Retourne Some(3)
    println!("dernier : {:?}", dernier);
    match dernier {
        Some(valeur) => println!("Valeur récupérée dans dernier : {}", valeur),
        None => println!("Le vecteur était vide"),
    }

    // Parcours
    for couleur in &couleurs {
        println!("{}", couleur);
    }

    // Longueur
    println!("Nombre de couleurs: {}", couleurs.len());
}
```

### 4.5 Slices

Une slice représente une vue sur une séquence contigüe d'éléments dans une collection, sans en prendre possession :

``` rust
fn main() {
    let nombres = [1, 2, 3, 4, 5];

    // Slice complète du tableau
    let tous = &nombres[..];
    println!("tous: {:?}", tous);

    // Slice partielle (indexation inclusive .. exclusive)
    let milieu = &nombres[1..4];  // [2, 3, 4]
    println!("Milieu: {:?}", milieu);

    // Depuis le début jusqu'à un index
    let debut = &nombres[..3];    // [1, 2, 3]
    println!("debut: {:?}", debut);

    // Depuis un index jusqu'à la fin
    let fin = &nombres[2..];      // [3, 4, 5]
    println!("fin: {:?}", fin);

    println!("Milieu: {:?}", milieu);

    // Slice d'un vecteur
    let vecteur = vec![10, 20, 30, 40, 50];
    println!("vecteur : {:?}", vecteur);
    let partie = &vecteur[1..3];  // [20, 30]
    println!("Partie du vecteur: {:?}", partie);
}
```

### 4.6 Chaînes de caractères

Rust possède deux types principaux pour les chaînes de caractères :

- `String` : chaîne de caractères de taille variable, allouée sur le tas
- `&str` : slice de chaîne (référence à une séquence d'UTF-8)

``` rust
fn main() {
    // Création d'une String
    let mut message = String::from("Bonjour");

    // Ajout à une String
    message.push_str(", monde!");

    // &str littérale (référence à une chaîne statique)
    let salutation: &str = "Salut!";

    // Conversion entre types
    let message_slice: &str = &message;
    let owned_message = salutation.to_string();

    println!("message_slice: {}", message_slice);
    println!("owned_message: {}", owned_message);


    // Concaténation
    let complet = format!("{} {}", salutation, "Comment ça va?");

    println!("{}", message);
    println!("{}", complet);

    // Méthodes utiles
    println!("Longueur: {}", message.len());
    println!("Est vide? {}", message.is_empty());
    println!("Contient 'monde'? {}", message.contains("monde"));
}
```

### 4.7 Constantes et variables statiques

Outre les variables standard, Rust offre deux autres façons de déclarer des valeurs :

``` rust
// Constante (évaluée à la compilation)
const MAX_POINTS: u32 = 100_000;

// Variable statique (durée de vie égale à celle du programme)
static LANGUE: &str = "Français";

fn main() {
    println!("Score maximum: {}", MAX_POINTS);
    println!("Langue: {}", LANGUE);
}
```

Les constantes et variables statiques :

- Doivent avoir leur type explicitement annoté
- Peuvent être déclarées dans n'importe quel scope
- Ne peuvent pas être modifiées (sauf avec `static mut` dans un bloc `unsafe`)

Les constantes sont plus couramment utilisées que les variables statiques en Rust.
