## 6\. Les fonctions

Les fonctions sont des blocs de code réutilisables qui permettent d'organiser logiquement votre programme. En Rust, les fonctions jouent un rôle central dans l'organisation du code.

## Déclaration de fonctions

Une fonction en Rust se déclare avec le mot-clé `fn` suivi du nom de la fonction, des paramètres entre parenthèses et du type de retour (optionnel) :

```
fn addition(nombre1: i32, nombre2: i32) -> i32 {
    nombre1 + nombre2
}

fn main() {
    let resultat = addition(5, 3);
    println!("5 + 3 = {}", resultat);
}
```

Points importants :

- Le type de chaque paramètre doit être explicitement déclaré
- Le type de retour est spécifié après une flèche `->`
- Si la fonction ne retourne rien, le type de retour peut être omis ou déclaré comme `-> ()`

## Fonctions sans valeur de retour

Une fonction qui ne retourne rien peut être écrite de plusieurs façons :

``` rust
// Sans type de retour spécifié
fn afficher_message(message: &str) {
    println!("{}", message);
}

// Avec le type de retour vide explicite
fn afficher_alerte(message: &str) -> () {
    println!("ALERTE : {}", message);
}

fn main() {
    // Utilisation basique avec des chaînes littérales
    afficher_message("Bonjour tout le monde");
    afficher_alerte("Données corrompues");

    // Utilisation avec des variables de type &str
    let info: &str = "Le programme démarre";
    afficher_message(info);

    // Utilisation avec une String (conversion implicite de &String en &str)
    let erreur = String::from("Connexion perdue");
    afficher_message(&erreur);
    afficher_alerte(&erreur);

    // Utilisation dans un contexte conditionnel
    let temperature = 38;
    if temperature > 37 {
        afficher_alerte("Température élevée détectée");
    } else {
        afficher_message("Température normale");
    }

    // Utilisation dans une boucle
    for i in 1..=3 {
        afficher_message(&format!("Itération {}", i));
    }

    // Composition de fonctions
    afficher_message_formaté("Utilisateur", "Jean");

    // Utilisation avec des arguments construits dynamiquement
    let nom_fichier = "config.json";
    afficher_alerte(&format!("Impossible d'ouvrir le fichier {}", nom_fichier));

    // Utilisation avec des expressions conditionnelles
    afficher_message(if temperature > 30 { "Il fait chaud" } else { "Il fait frais" });
}

// Fonction auxiliaire qui utilise les autres fonctions
fn afficher_message_formaté(type_message: &str, contenu: &str) {
    let message = format!("[{}] : {}", type_message, contenu);
    afficher_message(&message);
}

```

Le type `()` est appelé "unit type" en Rust, c'est un tuple vide qui représente l'absence de valeur.

## Valeur de retour et expression finale

En Rust, une fonction retourne la valeur de la dernière expression évaluée si celle-ci n'est pas terminée par un point-virgule :

``` rust
fn carre(nombre: i32) -> i32 {
    nombre * nombre  // Pas de point-virgule, c'est la valeur retournée
}

fn cube(nombre: i32) -> i32 {
    // En utilisant le mot-clé return explicite
    return nombre * nombre * nombre;
}

fn main() {
    // 1. Utilisation basique - assigner le résultat à une variable
    let x = 5;
    let resultat_carre = carre(x);
    println!("Le carré de {} est {}", x, resultat_carre);  // Affiche "Le carré de 5 est 25"

    // 2. Utilisation directe dans un println!
    println!("Le cube de {} est {}", x, cube(x));  // Affiche "Le cube de 5 est 125"

    // 3. Composition de fonctions
    let resultat_complexe = cube(carre(2));
    println!("Le cube du carré de 2 est {}", resultat_complexe);  // Affiche "Le cube du carré de 2 est 64"

    // 4. Utilisation dans des expressions mathématiques
    let somme = carre(3) + cube(2);
    println!("3² + 2³ = {}", somme);  // Affiche "3² + 2³ = 17"

    // 5. Utilisation avec des variables mutables
    let mut valeur = 2;
    valeur = carre(valeur);
    println!("Après avoir mis valeur au carré : {}", valeur);  // Affiche "Après avoir mis valeur au carré : 4"

    // 6. Utilisation dans des conditions
    if carre(4) > 15 {
        println!("Le carré de 4 est supérieur à 15");
    }

    // 7. Utilisation dans un vecteur
    let nombres = vec![1, 2, 3, 4];
    let carres: Vec<i32> = nombres.iter().map(|&n| carre(n)).collect();
    println!("Carrés des nombres : {:?}", carres);  // Affiche "Carrés des nombres : [1, 4, 9, 16]"

    // 8. Calcul de statistiques
    let valeurs = vec![2, 3, 4];
    let somme_des_carres: i32 = valeurs.iter().map(|&n| carre(n)).sum();
    println!("Somme des carrés : {}", somme_des_carres);  // Affiche "Somme des carrés : 29"

    // 9. Fonction qui utilise d'autres fonctions
    println!("Somme des carrés et des cubes de 1 à 3 : {}", somme_carres_et_cubes(3));

    // 10. Utilisation avec des expressions conditionnelles
    let nombre = 6;
    let resultat = if nombre % 2 == 0 { carre(nombre) } else { cube(nombre) };
    println!("Résultat conditionnel pour {} : {}", nombre, resultat);  // Affiche "Résultat conditionnel pour 6 : 36"
}

// Fonction qui utilise carre et cube
fn somme_carres_et_cubes(n: i32) -> i32 {
    let mut somme = 0;
    for i in 1..=n {
        somme += carre(i) + cube(i);
    }
    somme
}

```

Vous pouvez utiliser le mot-clé `return` pour retourner une valeur avant la fin de la fonction :

```
fn valeur_absolue(nombre: i32) -> i32 {
    if nombre < 0 {
        return -nombre;
    }
    nombre  // Retourne nombre si positif ou nul
}
```

## Retour de plusieurs valeurs avec les tuples

Les tuples permettent de retourner plusieurs valeurs depuis une fonction :

```
fn statistiques(nombres: &[i32]) -> (i32, i32, f64) {
    let somme: i32 = nombres.iter().sum();
    let max: i32 = *nombres.iter().max().unwrap_or(&0);
    let moyenne: f64 = somme as f64 / nombres.len() as f64;

    (somme, max, moyenne)
}

fn main() {
    let donnees = [5, 8, 2, 9, 3];
    let (somme, maximum, moyenne) = statistiques(&donnees);

    println!("Somme : {}", somme);
    println!("Maximum : {}", maximum);
    println!("Moyenne : {:.2}", moyenne);
}
```

## Fonctions comme paramètres

En Rust, les fonctions peuvent être passées en paramètre à d'autres fonctions :

```
fn appliquer_fonction<F>(x: i32, f: F) -> i32
 where
    F: Fn(i32) -> i32,
{
    f(x)
}

fn main() {
    let double = |x| x * 2;
    let triple = |x| x * 3;

    println!("Double de 5 : {}", appliquer_fonction(5, double));
    println!("Triple de 5 : {}", appliquer_fonction(5, triple));
}
```

## Fonctions anonymes (closures)

Les closures sont des fonctions anonymes que vous pouvez stocker dans des variables :

```
fn main() {
    let addition = |a, b| a + b;
    let multiplication = |a, b| a * b;

    println!("5 + 3 = {}", addition(5, 3));
    println!("5 × 3 = {}", multiplication(5, 3));
}
```

## Visibilité des fonctions

Par défaut, les fonctions sont privées au module dans lequel elles sont définies. Pour les rendre accessibles depuis d'autres modules, utilisez le mot-clé `pub` :

```
pub fn fonction_publique() {
    println!("Cette fonction est accessible depuis d'autres modules");
}

fn fonction_privee() {
    println!("Cette fonction n'est accessible que dans ce module");
}
```

## Fonctions associées et méthodes

Rust permet de définir des fonctions associées à des types (similaires aux méthodes statiques en Java/C++) et des méthodes qui opèrent sur une instance (avec `&self`, `&mut self` ou `self`) :

```
struct Rectangle {
    largeur: u32,
    hauteur: u32,
}

impl Rectangle {
    // Fonction associée (pas de self en paramètre)
    pub fn nouveau(largeur: u32, hauteur: u32) -> Rectangle {
        Rectangle { largeur, hauteur }
    }

    // Méthode (prend self en paramètre)
    pub fn aire(&self) -> u32 {
        self.largeur * self.hauteur
    }

    pub fn redimensionner(&mut self, largeur: u32, hauteur: u32) {
        self.largeur = largeur;
        self.hauteur = hauteur;
    }
}

fn main() {
    // Utilisation d'une fonction associée
    let mut rect = Rectangle::nouveau(10, 5);

    // Utilisation de méthodes
    println!("Aire : {}", rect.aire());
    rect.redimensionner(20, 10);
    println!("Nouvelle aire : {}", rect.aire());
}
```

## Fonctions versus macros

Il est important de distinguer les fonctions des macros en Rust. Les macros sont identifiées par un point d'exclamation `!` à la fin de leur nom :

```
// Ceci est une macro
println!("Hello, world!");

// Ceci est une fonction
fn dire_bonjour() {
    println!("Bonjour !");
}
```

Les macros sont plus puissantes que les fonctions car elles peuvent :

- Accepter un nombre variable d'arguments
- Générer du code à la compilation
- Opérer sur la syntaxe des expressions plutôt que sur les valeurs

Les macros sont un sujet avancé que nous explorerons plus en détail dans un chapitre ultérieur.

## Fonctions génériques

Rust permet de créer des fonctions génériques qui fonctionnent avec différents types :

```
fn afficher<T: std::fmt::Display>(valeur: T) {
    println!("{}", valeur);
}

fn main() {
    afficher(42);        // i32
    afficher(3.14);      // f64
    afficher("Bonjour"); // &str
}
```

La programmation générique sera explorée plus en profondeur dans les chapitres suivants.

Les fonctions en Rust sont très puissantes et constituent la base de toute organisation du code. Comprendre comment les définir et les utiliser efficacement est essentiel pour écrire du code Rust robuste et expressif.
