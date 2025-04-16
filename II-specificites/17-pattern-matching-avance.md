# 17\. Pattern matching avancé

Le pattern matching est l'une des fonctionnalités les plus puissantes de Rust. Nous avons déjà vu ses bases dans la section 5, mais il est temps d'explorer ses capacités avancées. Ces techniques vous permettront d'écrire du code plus expressif et concis tout en conservant la sécurité et les vérifications d'exhaustivité que Rust garantit.

## 17.1. Rappel des concepts de base

Avant d'aborder les fonctionnalités avancées, rappelons rapidement les bases du pattern matching en Rust :

``` rust
fn exemple_basique(valeur: i32) {
    match valeur {
        0 => println!("Zéro"),
        1 => println!("Un"),
        2 => println!("Deux"),
        _ => println!("Autre nombre"),
    }
}
```

## 17.2. Pattern matching sur les structures

### 17.2.1. Déstructuration des structures

Nous pouvons extraire et nommer les champs d'une structure :

``` rust
struct Point {
    x: i32,
    y: i32,
}

fn analyser_point(point: Point) {
    match point {
        Point { x: 0, y: 0 } => println!("Point à l'origine"),
        Point { x, y: 0 } => println!("Point sur l'axe X: x = {}", x),
        Point { x: 0, y } => println!("Point sur l'axe Y: y = {}", y),
        Point { x, y } => println!("Point quelconque: ({}, {})", x, y),
    }
}
```

### 17.2.2. Utiliser la syntaxe abrégée

Si le nom des variables correspond au nom des champs, on peut utiliser une syntaxe plus courte :

``` rust
fn analyser_coordonnées(point: Point) {
    let Point { x, y } = point;
    println!("Coordonnées: x = {}, y = {}", x, y);
}
```

### 17.2.3. Ignorer certains champs avec `..`

Pour les structures avec de nombreux champs, on peut ignorer ceux qui ne nous intéressent pas :

``` rust
struct Personne {
    nom: String,
    age: u32,
    adresse: String,
    telephone: String,
    email: String,
}

fn verifier_age(personne: &Personne) {
    match personne {
        Personne { age: 18..=65, .. } => println!("Âge actif"),
        Personne { nom, age, .. } => println!("{} a {} ans (hors âge actif)", nom, age),
    }
}
```

## 17.3. Pattern matching sur les enums complexes

### 17.3.1. Enums avec données

Le pattern matching est particulièrement utile avec les enums qui contiennent des données :

``` rust
enum Message {
    Texte(String),
    ChangePosition { x: i32, y: i32 },
    ChangeColor(u8, u8, u8),
    Quitter,
}

fn traiter_message(msg: Message) {
    match msg {
        Message::Texte(contenu) => println!("Message reçu: {}", contenu),
        Message::ChangePosition { x, y } => println!("Nouvelle position: ({}, {})", x, y),
        Message::ChangeColor(r, g, b) => println!("Nouvelle couleur: RGB({}, {}, {})", r, g, b),
        Message::Quitter => println!("Fermeture du programme"),
    }
}
```

### 17.3.2. Pattern matching imbriqué

On peut combiner les patterns pour gérer des structures de données imbriquées :

``` rust
enum Contenu {
    Texte(String),
    Nombre(i32),
    Liste(Vec<Contenu>),
}

fn analyser_contenu(contenu: &Contenu) {
    match contenu {
        Contenu::Texte(t) if t.len() > 10 => println!("Texte long: {}...", &t[0..10]),
        Contenu::Texte(t) => println!("Texte court: {}", t),
        Contenu::Nombre(n) if *n < 0 => println!("Nombre négatif: {}", n),
        Contenu::Nombre(n) => println!("Nombre positif ou nul: {}", n),
        Contenu::Liste(vec) if vec.is_empty() => println!("Liste vide"),
        Contenu::Liste(vec) => {
            println!("Liste avec {} éléments:", vec.len());
            for (i, item) in vec.iter().enumerate() {
                print!("  {}. ", i + 1);
                analyser_contenu(item);
            }
        }
    }
}
```

## 17.4. Les guards dans le pattern matching

Les **guards** (gardes) sont des expressions conditionnelles qui peuvent être ajoutées après un pattern avec le mot-clé `if`. Le pattern est considéré comme correspondant uniquement si la condition est vraie.

``` rust
fn categoriser_nombre(x: i32) {
    match x {
        n if n < 0 => println!("{} est négatif", n),
        n if n % 2 == 0 => println!("{} est positif et pair", n),
        n if n % 2 != 0 => println!("{} est positif et impair", n),
        // Ce cas ne sera jamais atteint, mais le compilateur ne peut pas le détecter
        _ => unreachable!(),
    }
}
```

### 17.4.1. Combiner les guards avec des ranges

``` rust
fn evaluer_note(note: u8) {
    match note {
        n if n >= 90 => println!("Excellent"),
        n @ 80..=89 => println!("Très bien: {}", n),
        n @ 70..=79 => println!("Bien: {}", n),
        n @ 60..=69 => println!("Assez bien: {}", n),
        n @ 50..=59 => println!("Passable: {}", n),
        n => println!("Insuffisant: {}", n),
    }
}
```

Notez l'utilisation de l'opérateur `@` qui permet de capturer la valeur tout en utilisant un pattern de range.

## 17.5. L'opérateur de binding `@`

L'opérateur `@` (parfois appelé "pattern binding") vous permet de capturer une valeur tout en la testant contre un pattern.

``` rust
enum Identifiant {
    Numéro(u32),
    Chaîne(String),
}

fn identifier(id: Identifiant) {
    match id {
        Identifiant::Numéro(n @ 1000..=9999) => {
            println!("Identifiant à 4 chiffres trouvé: {}", n)
        }
        Identifiant::Numéro(n) => {
            println!("Autre identifiant numérique: {}", n)
        }
        Identifiant::Chaîne(s @ _) if s.len() > 5 => {
            println!("Identifiant long: {}", s)
        }
        Identifiant::Chaîne(s) => {
            println!("Identifiant court: {}", s)
        }
    }
}
```

## 17.6. Patterns avec des références et des mutabilités

### 17.6.1. Matching sur les références

``` rust
fn analyser_valeur(valeur: &Option<String>) {
    match valeur {
        &Some(ref s) if s.starts_with("A") => println!("Chaîne commençant par A: {}", s),
        &Some(ref s) => println!("Chaîne quelconque: {}", s),
        &None => println!("Aucune valeur"),
    }
}
```

Cette syntaxe, bien que toujours fonctionnelle dans les versions antérieures de Rust, n'est plus recommandée dans les versions récentes grâce aux améliorations d'ergonomie du langage.
Grâce à la fonctionnalité 'match ergonomics' introduite dans Rust, lorsque vous faites un match sur une référence (comme `&mut Option<String>`), le compilateur déduit automatiquement que les variables liées dans les motifs doivent être des références du même type. Ainsi, il n'est plus nécessaire d'utiliser explicitement les qualificateurs `ref` ou `ref mut` dans ce contexte.
Dans cet exemple, `s` est automatiquement traité comme une référence mutable à la `String` contenue dans l'`Option`, ce qui vous permet d'appeler des méthodes mutables comme `push_str` directement sur `s`.
``` rust
fn transformer_si_necessaire(opt: &mut Option<String>) {
    match opt {
        Some(s) if s.len() < 10 => {
            s.push_str(" (complété)");
            println!("Chaîne complétée: {}", s);
        }
        Some(s) => println!("Chaîne assez longue: {}", s),
        None => println!("Aucune chaîne à transformer"),
    }
}
```

## 17.7. Patterns multiples avec `|`

L'opérateur `|` permet de tester plusieurs patterns dans une même branche :

``` rust
fn classifier_caractere(c: char) {
    match c {
        'a' | 'e' | 'i' | 'o' | 'u' => println!("Voyelle"),
        '0'..='9' => println!("Chiffre"),
        'A'..='Z' => println!("Majuscule"),
        'a'..='z' => println!("Minuscule"),
        _ => println!("Autre caractère"),
    }
}
```

## 17.8. Patterns dans différents contextes

Le pattern matching en Rust n'est pas limité à l'expression `match`. On peut l'utiliser dans plusieurs contextes :

### 17.8.1. Dans les instructions `let`

``` rust
let Point { x, y } = Point { x: 10, y: 20 };
println!("x = {}, y = {}", x, y);
```

### 17.8.2. Dans les paramètres de fonction

``` rust
fn afficher_coordonnees((x, y): (i32, i32)) {
    println!("Coordonnées: ({}, {})", x, y);
}
```

### 17.8.3. Dans les boucles `for`

``` rust
struct Point {
    x: i32,
    y: i32,
}
fn main() {
    let points = vec![
        Point { x: 0, y: 0 },
        Point { x: 1, y: 2 },
        Point { x: 10, y: -10 },
    ];

    for Point { x, y } in points {
        println!("Point à ({}, {})", x, y);
    }
}
```

## 17.9. Pattern matching exhaustif

Rust vérifie l'exhaustivité du pattern matching, ce qui signifie que tous les cas possibles doivent être traités :

``` rust
enum Direction {
    Nord,
    Sud,
    Est,
    Ouest,
}

fn decrire_direction(direction: Direction) -> &'static str {
    match direction {
        Direction::Nord => "vers le haut",
        Direction::Sud => "vers le bas",
        Direction::Est => "vers la droite",
        // Si on oublie Direction::Ouest, le compilateur générera une erreur
    }
}
```

### 17.9.1. Gestion de l'exhaustivité avec `_`

Le pattern `_` est un joker qui correspond à toute valeur :

``` rust
fn decrire_direction_complete(direction: Direction) -> &'static str {
    match direction {
        Direction::Nord => "vers le haut",
        Direction::Sud => "vers le bas",
        _ => "latéralement", // Couvre Est et Ouest
    }
}
```

## 17.10. Les patterns irréfutables vs réfutables

Un pattern est **irréfutable** s'il correspond toujours à la valeur fournie (comme dans `let x = 5;`). Un pattern est **réfutable** s'il peut ne pas correspondre à la valeur fournie (comme dans `if let Some(x) = option_value`).

``` rust
fn main() {
    // Pattern irréfutable - correspond toujours
    let (x, y) = (1, 2);

    // Créer une valeur option pour l'exemple
    let option = Some(42);

    // Pattern réfutable - peut échouer
    let Some(valeur) = option;  // Erreur de compilation!

    // Correct : utiliser if let pour un pattern réfutable
    if let Some(valeur) = option {
        println!("Valeur trouvée: {}", valeur);
    }
}
```

## 17.11. Pattern matching sur les slices

On peut faire du pattern matching sur les slices et les tableaux :

``` rust
fn analyser_slice(slice: &[i32]) {
    match slice {
        [] => println!("Slice vide"),
        [seul] => println!("Slice avec un seul élément: {}", seul),
        [premier, second] => println!("Slice avec deux éléments: {} et {}", premier, second),
        [premier, .., dernier] => println!("Slice avec au moins deux éléments, premier: {}, dernier: {}", premier, dernier),
    }
}
```

## 17.12. Patterns et Option/Result

Le pattern matching est particulièrement utile avec les types `Option` et `Result` :

``` rust
fn traiter_resultat(resultat: Result<String, std::io::Error>) {
    match resultat {
        Ok(contenu) if contenu.is_empty() => println!("Contenu vide"),
        Ok(contenu) => println!("Opération réussie: {}", contenu),
        Err(e) if e.kind() == std::io::ErrorKind::NotFound => {
            println!("Fichier non trouvé")
        }
        Err(e) => println!("Erreur: {}", e),
    }
}
```

## 17.13. Exemple complet : analyseur d'expression

Voici un exemple plus complet utilisant le pattern matching avancé pour analyser et évaluer des expressions arithmétiques simples :

``` rust
enum Expression {
    Valeur(i32),
    Addition(Box<Expression>, Box<Expression>),
    Soustraction(Box<Expression>, Box<Expression>),
    Multiplication(Box<Expression>, Box<Expression>),
    Division(Box<Expression>, Box<Expression>),
}

fn evaluer(expr: &Expression) -> Result<i32, String> {
    match expr {
        Expression::Valeur(val) => Ok(*val),

        Expression::Addition(gauche, droite) => {
            let g = evaluer(gauche)?;
            let d = evaluer(droite)?;
            Ok(g + d)
        }

        Expression::Soustraction(gauche, droite) => {
            let g = evaluer(gauche)?;
            let d = evaluer(droite)?;
            Ok(g - d)
        }

        Expression::Multiplication(gauche, droite) => {
            let g = evaluer(gauche)?;
            let d = evaluer(droite)?;
            Ok(g * d)
        }

        Expression::Division(gauche, droite) => {
            let g = evaluer(gauche)?;
            let d = evaluer(droite)?;

            match d {
                0 => Err("Division par zéro".to_string()),
                _ => Ok(g / d),
            }
        }
    }
}

fn main() {
    // Exemple: (5 * 2) + (10 / 2)
    let expression = Expression::Addition(
        Box::new(Expression::Multiplication(
            Box::new(Expression::Valeur(5)),
            Box::new(Expression::Valeur(2)),
        )),
        Box::new(Expression::Division(
            Box::new(Expression::Valeur(10)),
            Box::new(Expression::Valeur(2)),
        )),
    );

    match evaluer(&expression) {
        Ok(resultat) => println!("Résultat: {}", resultat),
        Err(e) => println!("Erreur: {}", e),
    }
}
```

## 17.14. Conclusion

Le pattern matching avancé est l'une des fonctionnalités les plus puissantes de Rust. Il vous permet d'écrire du code expressif, concis et sûr. Combiner les patterns avec des guards, l'opérateur de binding `@`, et la déstructuration de structures complexes vous donne un contrôle précis sur la logique de votre programme.

Ces techniques sont particulièrement utiles pour travailler avec des structures de données complexes, des types algébriques (enums avec données), et pour implémenter des algorithmes qui nécessitent une analyse précise des données.

L'exhaustivité vérifiée par le compilateur garantit que vous n'oubliez aucun cas possible, ce qui contribue à la robustesse de votre code. Maîtriser le pattern matching avancé vous permettra d'exploiter pleinement la puissance du système de types de Rust.
