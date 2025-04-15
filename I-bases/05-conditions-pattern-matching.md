## 5\. Conditions et pattern matching

Les structures de contrôle en Rust permettent de diriger l'exécution du programme selon différentes conditions. Nous allons explorer en détail les conditions ainsi que le pattern matching qui est l'une des fonctionnalités les plus puissantes de Rust.

## Les conditions if / else if / else

En Rust, les conditions suivent une syntaxe similaire aux autres langages, mais avec quelques particularités :

```
let age = 17;
if age >= 18 {
    println!("Vous êtes majeur !");
} else {
    println!("Vous êtes mineur !");
}
```

Points importants à noter :

- Les parenthèses autour de la condition ne sont pas obligatoires (contrairement à C/C++/Java)
- Les accolades `{ }` sont obligatoires, même pour une seule instruction
- Le bloc de condition renvoie une valeur, permettant des affectations conditionnelles

Vous pouvez combiner des conditions avec :

- `&&` : opérateur ET logique
- `||` : opérateur OU logique
- `!` : opérateur de négation

```
let age :i32 = 25;
let etudiant :bool = false;

if age >= 18 && (etudiant || age < 30) {
    println!("Éligible pour le tarif jeune adulte");
}
```

## Assignation conditionnelle

Une particularité de Rust est la possibilité d'utiliser les conditions pour l'assignation de valeurs :

```
let age :i32 = 25;
let statut = if age >= 18 { "majeur" } else { "mineur" };
println!("Statut : {}", statut);
```

Cette forme est plus concise que l'utilisation d'une variable mutable modifiée dans les branches conditionnelles.

## Comparaison avec des booléens

Lorsque vous travaillez avec des booléens, vous pouvez simplifier votre code :

```
let authentifie :bool = true;

// Forme longue
if authentifie == true {
    println!("Accès autorisé");
}

// Forme simplifiée équivalente
if authentifie {
    println!("Accès autorisé");
}

// Forme longue pour la négation
if authentifie == false {
    println!("Accès refusé");
}

// Forme simplifiée équivalente
if !authentifie {
    println!("Accès refusé");
}
```

## Pattern matching avec match

Le pattern matching est une fonctionnalité puissante qui permet de comparer une valeur à différents motifs. L'instruction `match` en Rust est exhaustive, ce qui signifie que tous les cas possibles doivent être traités.

Voici un exemple simple :

```
let langue :&str  = "français";

match langue {
    "français" => println!("Bonjour !"),
    "anglais" => println!("Hello!"),
    "espagnol" => println!("¡Hola!"),
    "italien" => println!("Ciao!"),
    "allemand" => println!("Guten Tag!"),
    _ => println!("Je ne connais pas cette langue..."),
}
```

Le symbole `_` est un joker qui capture tous les cas non spécifiés. Il est obligatoire si tous les cas possibles ne sont pas explicitement traités.

## Branches multiples dans un match

Vous pouvez exécuter plusieurs instructions dans une branche de `match` en utilisant des accolades :

```
let note :i32 = 15;

match note {
    0..=9 => {
        println!("Insuffisant");
        println!("Il faut travailler davantage");
    },
    10..=12 => println!("Passable"),
    13..=15 => println!("Bien"),
    16..=20 => {
        println!("Très bien !");
        println!("Félicitations !");
    },
    _ => println!("Note invalide"),
}
```

## Conditions dans les branches de match

Il est possible d'ajouter des conditions à un pattern avec la syntaxe `if` :

```
let temperature :i32 = 25;

match temperature {
    t if t < 0 => println!("Gel"),
    t if t < 15 => println!("Frais"),
    t if t < 25 => println!("Agréable"),
    t if t < 35 => println!("Chaud"),
    t => println!("Très chaud : {}°C", t),
}
```

## Intervalles de valeurs

Vous pouvez matcher sur des intervalles de valeurs :

```
let score = 75;

match score {
    0..=50 => println!("Échec"),
    51..=70 => println!("Satisfaisant"),
    71..=90 => println!("Bien"),
    91..=100 => println!("Excellent"),
    _ => println!("Score invalide"),
}
```

## Capture de valeurs avec @

Le symbole `@` permet de capturer la valeur qui correspond au pattern :

```
let valeur = 42;

match valeur {
    x @ 0..=9 => println!("{} est un chiffre", x),
    x @ 10..=99 => println!("{} est un nombre à deux chiffres", x),
    x => println!("{} est un grand nombre", x),
}
```

## Patterns alternatifs

Pour matcher plusieurs valeurs possibles, utilisez le symbole `|` (OU) :

```
let jour = "samedi";

match jour {
    "lundi" | "mardi" | "mercredi" | "jeudi" | "vendredi" => println!("Jour ouvrable"),
    "samedi" | "dimanche" => println!("Week-end"),
    _ => println!("Jour invalide"),
}
```

## Déstructuration dans les patterns

Rust permet également de déstructurer des structures complexes dans les patterns :

```
let point = (10, 20);

match point {
    (0, 0) => println!("Origine"),
    (0, y) => println!("Sur l'axe y, à la position {}", y),
    (x, 0) => println!("Sur l'axe x, à la position {}", x),
    (x, y) => println!("Point ({}, {})", x, y),
}
```

Le pattern matching est l'une des fonctionnalités les plus puissantes de Rust, et nous verrons d'autres exemples plus avancés quand nous aborderons les enums et les structures.
