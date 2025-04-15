## 11\. if let / while let

Le pattern matching est une fonctionnalité très puissante en Rust. Les expressions `if let` et `while let` offrent une syntaxe plus concise pour certains cas d'utilisation courants.

### if let

L'expression `if let` permet de simplifier le pattern matching quand on ne s'intéresse qu'à un seul motif, évitant ainsi un `match` verbeux.

Prenons un exemple concret avec `Option<T>` :

```
fn obtenir_utilisateur(id: u32) -> Option<String> {
    if id == 42 {
        Some(String::from("Alice"))
    } else if id == 27 {
        Some(String::from("Bob"))
    } else {
        None
    }
}
```

Sans `if let`, nous devrions vérifier le résultat avec un `match` :

```
match obtenir_utilisateur(42) {
    Some(nom) => println!("Utilisateur trouvé : {}", nom),
    None => println!("Aucun utilisateur trouvé"),
}
```

Avec `if let`, le code devient plus compact :

```
if let Some(nom) = obtenir_utilisateur(42) {
    println!("Utilisateur trouvé : {}", nom);
} else {
    println!("Aucun utilisateur trouvé");
}
```

### Pattern matching avancé avec if let

La puissance de `if let` va au-delà des types simples. On peut faire des correspondances de motifs complexes :

```
// Matching sur une valeur spécifique
let nombre = Some(42);
if let Some(42) = nombre {
    println!("Le nombre est exactement 42!");
}

// Déstructuration de structure
struct Point { x: i32, y: i32 }
let point = Point { x: 10, y: 20 };

if let Point { x, y: 20 } = point {
    println!("Point avec y=20 et x={}", x);
}

// Pattern matching sur des enums imbriqués
enum Message {
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

let msg = Message::ChangeColor(255, 0, 0);

if let Message::ChangeColor(r, g, b) = msg {
    println!("Changement de couleur vers RGB({}, {}, {})", r, g, b);
}
```

### while let

De façon similaire, `while let` permet d'exécuter une boucle tant qu'un motif correspond :

```
let mut pile = vec![1, 2, 3, 4, 5];

// Sans while let
loop {
    match pile.pop() {
        Some(valeur) => println!("Valeur dépilée : {}", valeur),
        None => break,
    }
}

// Avec while let
let mut pile = vec![1, 2, 3, 4, 5];
while let Some(valeur) = pile.pop() {
    println!("Valeur dépilée : {}", valeur);
}
```

`while let` est particulièrement utile quand on travaille avec des itérateurs :

```
let mut iterateur = [1, 2, 3].iter().enumerate();

while let Some((index, valeur)) = iterateur.next() {
    println!("Index {} contient {}", index, valeur);
}
```

### @ binding

Le pattern matching de Rust permet également de capturer une valeur tout en vérifiant qu'elle correspond à un pattern spécifique, grâce à l'opérateur `@` :

```
enum Temperature {
    Celsius(i32),
    Fahrenheit(i32),
}

let temp = Temperature::Celsius(25);

if let Temperature::Celsius(c @ 20..=30) = temp {
    println!("Température agréable de {}°C", c);
}

// Version plus lisible et décomposée:
match temp {
    // Si temp est une température en Celsius...
    Temperature::Celsius(valeur) => {
        // ...et que cette valeur est entre 20 et 30 inclus
        if valeur >= 20 && valeur <= 30 {
            println!("Température agréable de {}°C", valeur);
        }
    },
    // Ignore les autres cas (Fahrenheit ou Celsius hors plage)
    _ => {}
}

// Autre version plus lisible et décomposée:
if let Temperature::Celsius(valeur) = temp {
    println!("C'est une température en Celsius: {}°C", valeur);

    // Étape 2: Ajouter une condition séparée pour vérifier la plage
    if valeur >= 20 && valeur <= 30 {
        println!("La température est agréable !");
    }
}
```
