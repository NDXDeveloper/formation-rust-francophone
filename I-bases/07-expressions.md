## 7\. Les expressions

En Rust, la notion d'expressions est fondamentale pour comprendre comment le langage fonctionne. Contrairement à d'autres langages, Rust est principalement basé sur les expressions plutôt que sur les instructions.

### Expressions vs déclarations

La différence fondamentale entre une expression et une déclaration est simple :

- Une **expression** retourne une valeur
- Une **déclaration** n'en retourne pas

Cette distinction est essentielle pour comprendre le comportement du code Rust. Voyons quelques exemples concrets :

``` rust
// Ceci est valide car `if` est une expression qui retourne une valeur
let nombre = if true {
    42
} else {
    24
};
print!("{:?}",nombre)
```

En revanche, le code suivant ne fonctionne pas :

``` rust
// Ceci est invalide car `let` est une déclaration, pas une expression
let resultat = (let variable = 5);
```

L'assignation, en revanche, est une expression particulière :

``` rust
let mut x = 0;
let y = (x = 10); // Valide car (x = 10) est une expression
```

Cependant, **attention** : en Rust, contrairement au C/C++, une assignation retourne la valeur `()` (tuple vide) et non la valeur assignée. Ainsi, `y` aura comme valeur `()` et non `10`.

### Les blocs comme expressions

En Rust, un bloc de code délimité par des accolades est aussi une expression :

``` rust
let valeur = {
    let a = 1;
    let b = 2;
    a + b // Notez l'absence de point-virgule ici!
};
println!("La valeur est: {}", valeur); // Affiche "La valeur est: 3"
```

L'expression complète est évaluée à la valeur de la dernière expression du bloc. Si cette dernière ligne se termine par un point-virgule, le bloc retourne alors `()` (tuple vide).

``` rust
let valeur = {
    let a = 1;
    let b = 2;
    a + b; // Avec un point-virgule ici!
};
// valeur == ()
```

### Le point-virgule et son importance

Le point-virgule en Rust a une signification particulière : il transforme une expression en une déclaration, ce qui change sa valeur de retour à `()`. C'est pourquoi le code suivant ne compile pas :

``` rust
let x: i32 = if true {
    5; // Point-virgule ici
} else {
    10; // Point-virgule ici
};
```

Le compilateur produira une erreur car le type de retour attendu pour `x` est `i32`, mais l'expression `if` retourne `()` à cause des points-virgules.

### Expressions dans les fonctions

Cette caractéristique de Rust permet d'écrire des fonctions très concises :

``` rust
fn est_positif(nombre: i32) -> &'static str {
    if nombre > 0 {
        "positif"
    } else if nombre < 0 {
        "négatif"
    } else {
        "zéro"
    }
}
```

Notez l'absence de mot-clé `return` et de point-virgule à la fin des branches. La fonction retourne directement la valeur de l'expression `if`.

### Expressions avec match

Le pattern matching avec `match` est également une expression :

``` rust
let message = "hello";
let langue = match message {
    "bonjour" => "français",
    "hello" => "anglais",
    "hola" => "espagnol",
    "ciao" => "italien",
    _ => "langue inconnue"
};
println!("Le message est en {}", langue); // Affichera "Le message est en anglais"
```

### Expressions non terminées

Si vous oubliez le point-virgule à la fin d'une expression qui n'est pas la dernière du bloc, cela peut créer des erreurs inattendues :

``` rust
fn main() {
    let x = 5
    println!("x = {}", x); // Erreur: point-virgule manquant après `let x = 5`
}
```

En comprenant bien le système d'expressions en Rust, vous pourrez écrire du code plus élégant et plus concis.

