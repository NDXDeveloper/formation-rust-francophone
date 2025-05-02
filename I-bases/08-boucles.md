## 8\. Les boucles

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Les boucles sont essentielles dans tout langage de programmation. Rust offre plusieurs façons d'itérer et de répéter du code avec des fonctionnalités uniques au langage.

### La boucle while

La boucle `while` continue tant que sa condition est vraie :

``` rust
let mut compteur = 0;
while compteur < 5 {
    println!("Compteur: {}", compteur);
    compteur += 1;
}
```

Particularités à noter en Rust :

- Pas de parenthèses autour de la condition
- Les accolades sont obligatoires
- La condition doit évaluer à un `bool`

### La boucle loop

Rust propose une boucle infinie avec le mot-clé `loop`. C'est plus idiomatique que d'écrire `while true` :

``` rust
let mut compteur = 0;
loop {
    println!("Itération {}", compteur);
    compteur += 1;

    if compteur >= 5 {
        break; // Sort de la boucle
    }
}
```

Cette construction est très utilisée dans les cas où l'on souhaite continuer jusqu'à atteindre une condition de sortie particulière, comme dans un jeu vidéo ou lorsqu'on attend une ressource externe.

#### Retourner des valeurs depuis une boucle

Une caractéristique unique à Rust est la possibilité de retourner une valeur depuis une boucle `loop` :

``` rust
let resultat = loop {
    // Faire quelque chose...
    if /* condition */ {
        break "Valeur à retourner"; // Retourne cette valeur
    }
};
```

Par exemple :

``` rust
let mut compteur = 0;
let resultat = loop {
    compteur += 1;

    if compteur == 10 {
        break compteur * 2;
    }
};
println!("Résultat: {}", resultat); // Affiche "Résultat: 20"
```

### La boucle for

La boucle `for` en Rust est très puissante et fonctionne avec tout objet qui implémente le trait `IntoIterator`. Elle est particulièrement utile pour parcourir des collections.

#### Parcourir des plages de nombres

``` rust
// Parcourt les nombres de 0 à 4 inclus
for i in 0..5 {
    println!("i = {}", i);
}

// Pour inclure la borne supérieure (jusqu'à 5 inclus)
for i in 0..=5 {
    println!("i = {}", i);
}
```

#### Parcourir des collections

``` rust
let nombres = vec![10, 20, 30, 40, 50];

// Parcourt les valeurs directement (consomme le vecteur)
for nombre in nombres {
    println!("Nombre: {}", nombre);
}

// Si on veut garder la collection, on utilise .iter()
let couleurs = vec!["rouge", "vert", "bleu"];
for couleur in couleurs.iter() {
    println!("Couleur: {}", couleur);
}

// Pour modifier les éléments, on utilise .iter_mut()
let mut positions = vec![1, 2, 3];
for position in positions.iter_mut() {
    *position *= 2;
}
println!("Positions: {:?}", positions); // Affiche [2, 4, 6]
```

#### Utilisation de enumerate

Si vous avez besoin de l'index pendant l'itération, utilisez `enumerate()` :

``` rust
let fruits = vec!["pomme", "banane", "orange"];
for (index, fruit) in fruits.iter().enumerate() {
    println!("Fruit #{}: {}", index + 1, fruit);
}
```

### Les boucles nommées

Rust permet de donner des noms (ou étiquettes) aux boucles, ce qui est particulièrement utile pour spécifier quelle boucle interrompre ou continuer dans des structures imbriquées :

``` rust
'externe: for x in 1..=5 {
    'interne: for y in 1..=5 {
        println!("x = {}, y = {}", x, y);

        if x == 3 && y == 3 {
            // Interrompt la boucle externe
            break 'externe;
        }

        if y == 2 {
            // Passe à l'itération suivante de la boucle externe
            continue 'externe;
        }
    }
}
```

### Combinaison avec d'autres expressions

Les boucles en Rust peuvent être combinées avec d'autres expressions de manière élégante :

``` rust
let mut total = 0;
let resultat = loop {
    match total {
        n if n >= 10 => break n,
        _ => total += 1,
    }
};
println!("Résultat final: {}", resultat);
```

### Optimisation des boucles

Rust étant axé sur les performances, ses boucles sont généralement optimisées par le compilateur. Par exemple, l'itération sur des plages de nombres avec `for` est souvent aussi efficace qu'une boucle `while` manuelle mais avec une syntaxe plus concise et plus sûre.

### Boucles et closures

On peut combiner les boucles avec des closures pour créer des itérateurs personnalisés :

``` rust
// Créer un itérateur personnalisé avec une closure
let compteur = (0..5).map(|n| n * n);

for valeur in compteur {
    println!("Valeur au carré: {}", valeur);
}
```

En maîtrisant les différents types de boucles en Rust, vous pourrez exprimer clairement et efficacement la répétition et l'itération dans votre code, tout en bénéficiant des garanties de sécurité qu'offre le langage.

⏭️ [Les enums](/I-bases/09-enums.md)
