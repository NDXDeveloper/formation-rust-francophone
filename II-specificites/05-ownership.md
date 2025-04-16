## 5\. Propriété (ou ownership)

Le concept de propriété (ownership) est l'une des caractéristiques les plus distinctives de Rust. C'est à la fois ce qui rend le langage si sûr et ce qui peut sembler déroutant pour les nouveaux développeurs.

### Principes fondamentaux

En Rust, chaque valeur a un unique "propriétaire" à un moment donné. Lorsque le propriétaire sort de la portée, la valeur est automatiquement libérée. Cette règle fondamentale permet d'éviter les fuites mémoire et les erreurs d'accès concurrents sans avoir besoin d'un ramasse-miettes (garbage collector).

Observons ce principe avec un exemple simple :

``` rust
fn main() {
    let s1 = String::from("bonjour");   // s1 est le propriétaire
    let s2 = s1;                        // la propriété est transférée à s2

    // println!("{}", s1);              // Erreur : valeur déplacée (moved)
    println!("{}", s2);                 // Ok : s2 est maintenant le propriétaire
}
```

Dans cet exemple, lorsque nous assignons `s1` à `s2`, la propriété est transférée. La variable `s1` n'est plus utilisable, et toute tentative d'y accéder provoquera une erreur de compilation.

### Comprendre le déplacement (move) et la copie (copy)

Pour comprendre pourquoi certains types semblent copiés tandis que d'autres sont déplacés, il est important de distinguer deux catégories de types:

1.  **Types implémentant le trait `Copy`** : Les types primitifs (entiers, flottants, booléens) et les tuples/tableaux qui ne contiennent que des types `Copy`.
2.  **Types ne pouvant pas implémenter `Copy`** : Les types qui gèrent des ressources (comme `String`, `Vec`, etc.).

Exemple avec des types primitifs :

``` rust
fn main() {
    let x = 5;
    let y = x;      // ici x est copié dans y (les deux variables restent valides)

    println!("x = {}, y = {}", x, y);   // Pas d'erreur, x est toujours accessible
}
```

### Emprunt par référence

Pour éviter les transferts de propriété constants, Rust utilise le concept d'emprunt (borrowing) via les références. Il existe deux types de références :

1.  **Références immuables** (`&T`) : Permettent de lire une valeur sans en prendre la propriété
2.  **Références mutables** (`&mut T`) : Permettent de modifier une valeur sans en prendre la propriété

``` rust
fn longueur(s: &String) -> usize {  // s est une référence vers un String
    s.len()
}   // s sort de la portée, mais comme ce n'est qu'une référence,
    // elle n'a pas la propriété, donc rien n'est libéré

fn main() {
    let message = String::from("Salut Rust");

    let taille = longueur(&message);  // nous passons une référence

    println!("La chaîne '{}' a {} caractères", message, taille);
    // message est toujours valide ici
}
```

### Références mutables

Pour modifier une valeur empruntée, nous avons besoin d'une référence mutable :

``` rust
fn ajouter_texte(s: &mut String) {
    s.push_str(" est incroyable!");
}

fn main() {
    let mut message = String::from("Rust");  // la variable doit être mutable

    ajouter_texte(&mut message);  // nous passons une référence mutable

    println!("{}", message);  // Affiche "Rust est incroyable!"
}
```

### Règles d'emprunt

Rust impose des règles strictes sur les références pour garantir la sécurité mémoire :

1.  À tout moment, vous pouvez avoir **soit** :
    - Une seule référence mutable
    - Un nombre quelconque de références immuables
2.  Les références doivent toujours être valides (pas de références pendantes)
3.  Une référence ne peut pas vivre plus longtemps que la donnée qu'elle référence

Exemple illustrant ces règles :

``` rust
fn main() {
    let mut nombre = 42;

    // Création de plusieurs références immuables (autorisé)
    let ref1 = &nombre;
    let ref2 = &nombre;
    println!("{} et {}", ref1, ref2);

    // Création d'une référence mutable (autorisé car les références immuables
    // ne sont plus utilisées après le println! ci-dessus)
    let ref_mut = &mut nombre;
    *ref_mut += 1;
    println!("Après modification: {}", ref_mut);

    // À ce stade, on ne peut pas utiliser ref1 ou ref2 en même temps que ref_mut
    // println!("{} et {} et {}", ref1, ref2, ref_mut); // Erreur!
}
```

### Durée de vie des références

Le compilateur Rust vérifie que toutes les références sont valides à travers ce qu'on appelle le "borrow checker" :

``` rust
fn main() {
    let reference;

    {
        let valeur = 42;
        reference = &valeur;
        // valeur est détruite ici
    }

    // println!("{}", reference);  // Erreur: référence à une valeur qui n'existe plus
}
```

Version corrigée où la référence ne survit pas à la donnée référencée :

``` rust
fn main() {
    let valeur = 42;
    let reference = &valeur;

    println!("{}", reference);  // Ok
}
```

### Le trait Clone pour la duplication explicite

Lorsque nous voulons réellement dupliquer une valeur plutôt que de la déplacer ou de l'emprunter, nous pouvons utiliser le trait `Clone` :

``` rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v2 = v1.clone();  // crée une copie profonde de v1

    println!("v1: {:?}", v1);  // v1 est toujours valide
    println!("v2: {:?}", v2);  // v2 est une copie indépendante
}
```

### Cas pratique

Voici un exemple plus complet montrant comment le système de propriété nous aide à éviter les erreurs courantes :

``` rust
struct Utilisateur {
    nom: String,
    age: u32,
}

fn augmenter_age(utilisateur: &mut Utilisateur) {
    utilisateur.age += 1;
}

fn afficher_info(utilisateur: &Utilisateur) {
    println!("{} a {} ans", utilisateur.nom, utilisateur.age);
}

fn main() {
    let mut alice = Utilisateur {
        nom: String::from("Alice"),
        age: 30,
    };

    afficher_info(&alice);       // emprunte alice de façon immuable
    augmenter_age(&mut alice);   // emprunte alice de façon mutable
    afficher_info(&alice);       // emprunte alice de façon immuable à nouveau

    // alice est toujours disponible ici car nous n'avons utilisé que des références
}
```

Le système de propriété de Rust peut sembler contraignant au début, mais c'est cette rigueur qui permet au compilateur de garantir l'absence de nombreux bugs courants comme les fuites mémoire, les accès concurrents non sécurisés et les erreurs de pointeurs nuls, tout en maintenant des performances excellentes.

Dans le prochain chapitre, nous approfondirons le concept de durée de vie (lifetime) qui est étroitement lié au système de propriété et qui permet au compilateur de garantir que les références sont toujours valides.
