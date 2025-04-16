## 7\. Déréférencement

Le déréférencement est un concept fondamental en Rust qui permet d'accéder à la valeur pointée par une référence. Comprendre ce mécanisme est essentiel pour maîtriser la manipulation des références et des pointeurs en Rust.

### Principe de base

Le déréférencement utilise l'opérateur `*` pour accéder à la valeur contenue à l'adresse mémoire pointée par une référence. Voyons un exemple simple :

``` rust
fn une_fonction(x: &mut i32) {
    *x = 2; // Déréférencement pour modifier la valeur pointée
}

fn main() {
    let mut x = 0;
    println!("avant : {}", x);
    une_fonction(&mut x);
    println!("après : {}", x); // Affiche "après : 2"
}
```

Dans cet exemple, nous modifions la valeur de `x` à travers une référence mutable passée à la fonction. L'opérateur `*` nous permet d'accéder et de modifier la valeur pointée par cette référence.

### Le trait `Deref` et `DerefMut`

Contrairement au C/C++ où le déréférencement est une opération de bas niveau sur les pointeurs, Rust implémente cette fonctionnalité à travers les traits `Deref` et `DerefMut`.

- `Deref` : Permet le déréférencement en lecture
- `DerefMut` : Permet le déréférencement en écriture

Une référence immutable (`&T`) implémente `Deref`, tandis qu'une référence mutable (`&mut T`) implémente à la fois `Deref` et `DerefMut`. C'est ce qui rend possible l'exemple précédent.

### Implémentation personnalisée

On peut implémenter le trait `Deref` pour nos propres types :

``` rust
use std::ops::Deref;

struct MonEntier {
    valeur: i32
}

impl Deref for MonEntier {
    // Type cible du déréférencement
    type Target = i32;

    fn deref(&self) -> &Self::Target {
        &self.valeur
    }
}

fn main() {
    let mon_entier = MonEntier { valeur: 42 };

    // Utilisation du déréférencement
    assert_eq!(42, *mon_entier);

    // On peut aussi l'utiliser implicitement
    let longueur = mon_entier.abs(); // Appelle la méthode abs() de i32
}
```

Cet exemple montre comment déréférencer notre type personnalisé `MonEntier` vers `i32`, permettant ainsi d'utiliser les méthodes de `i32` directement sur notre type.

### Déréférencement implicite (Deref coercion)

Rust dispose d'une fonctionnalité puissante appelée "déréférencement implicite" ou "Deref coercion". Elle permet au compilateur d'appliquer automatiquement des déréférencements successifs pour adapter le type d'une référence au type attendu.

``` rust
use std::ops::Deref;

struct MonTexte {
    contenu: String
}

impl Deref for MonTexte {
    type Target = String;

    fn deref(&self) -> &String {
        &self.contenu
    }
}

fn afficher(s: &str) {
    println!("{}", s);
}

fn main() {
    let texte = MonTexte { contenu: String::from("Bonjour Rust") };

    // Déréférencement en cascade:
    // &MonTexte -> &String -> &str
    afficher(&texte);
}
```

Dans cet exemple, le compilateur transforme automatiquement `&MonTexte` en `&str` par une série de déréférencements :

1.  `&MonTexte` → `&String` (via notre implémentation de `Deref`)
2.  `&String` → `&str` (via l'implémentation de `Deref` pour `String`)

### Déréférencement en chaîne

Le déréférencement en chaîne est particulièrement puissant en Rust :

``` rust
struct UneStruct;
impl UneStruct {
    fn methode(&self) {
        println!("Méthode de UneStruct appelée");
    }
}

fn main() {
    let instance = UneStruct;

    // Tous ces appels sont équivalents
    instance.methode();
    (&instance).methode();
    (&&instance).methode();
    (&&&&&&&&instance).methode(); // Fonctionne également !
}
```

Le compilateur applique autant de déréférencements que nécessaire pour trouver la méthode appropriée.
