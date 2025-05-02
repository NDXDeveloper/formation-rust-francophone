# 15\. Les itérateurs

Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction aux itérateurs

Les itérateurs sont l'un des concepts les plus puissants et fondamentaux en Rust. Ils permettent de parcourir des collections d'éléments de manière efficace, expressive et sans risque d'erreur de mémoire. Contrairement à d'autres langages où les itérateurs peuvent être des concepts secondaires, en Rust, ils font partie intégrante de l'écosystème et sont utilisés partout.

Le trait `Iterator` est au cœur de ce mécanisme, mais son implémentation peut sembler complexe pour les débutants. Ce chapitre vise à démystifier ce concept et à vous montrer comment l'utiliser efficacement dans vos projets.

## Le trait Iterator

Le trait `Iterator` est défini de la façon suivante dans la bibliothèque standard :

``` rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;

    // De nombreuses méthodes par défaut comme map, filter, zip, etc.
    // que nous explorerons plus loin
}
```

La méthode `next()` est la seule que vous devez obligatoirement implémenter. Elle retourne `Some(item)` tant qu'il y a des éléments à parcourir, puis `None` quand l'itération est terminée.

## Types d'itérateurs

En Rust, on peut distinguer principalement deux types d'itérateurs :

### 1\. Les itérateurs sur des collections existantes

Ces itérateurs permettent de parcourir des données stockées dans une structure (comme un vecteur, une liste chaînée, un arbre, etc.).

### 2\. Les générateurs

Ces itérateurs créent leurs propres valeurs à la demande, souvent selon une logique mathématique ou algorithmique.

## Implémenter un itérateur sur une collection

Commençons par un exemple concret : créons un type qui encapsule un vecteur et permettons d'itérer sur ses éléments.

``` rust
// Notre type qui encapsule un Vec
struct Collection<T> {
    data: Vec<T>,
}

impl<T> Collection<T> {
    fn new(data: Vec<T>) -> Self {
        Collection { data }
    }

    // Méthode pour créer un itérateur
    fn iter(&self) -> CollectionIterator<T> {
        CollectionIterator {
            collection: self,
            index: 0,
        }
    }
}

// Notre itérateur qui gardera une référence à la collection
struct CollectionIterator<'a, T> {
    collection: &'a Collection<T>,
    index: usize,
}

// Implémentation du trait Iterator pour notre itérateur
impl<'a, T> Iterator for CollectionIterator<'a, T> {
    type Item = &'a T;

    fn next(&mut self) -> Option<Self::Item> {
        if self.index < self.collection.data.len() {
            let item = &self.collection.data[self.index];
            self.index += 1;
            Some(item)
        } else {
            None
        }
    }
}

fn main() {
    let collection = Collection::new(vec![1, 2, 3, 4, 5]);

    // Utilisation de l'itérateur
    for item in collection.iter() {
        println!("Valeur : {}", item);
    }
}
```

Dans cet exemple :

1.  Nous avons créé une structure `Collection<T>` qui encapsule un `Vec<T>`
2.  Nous avons créé un itérateur spécifique `CollectionIterator<'a, T>` qui garde une référence à la collection
3.  Nous avons implémenté le trait `Iterator` pour notre itérateur

### Variante avec IntoIterator

Pour plus de commodité, Rust permet d'implémenter le trait `IntoIterator` qui permet d'utiliser une structure directement dans une boucle `for` :

``` rust
impl<'a, T> IntoIterator for &'a Collection<T> {
    type Item = &'a T;
    type IntoIter = CollectionIterator<'a, T>;

    fn into_iter(self) -> Self::IntoIter {
        self.iter()
    }
}

// Maintenant on peut faire :
for item in &collection {
    println!("Valeur : {}", item);
}
```

## Implémenter un générateur

Les générateurs sont des itérateurs qui produisent leurs propres valeurs selon un algorithme défini. Voici un exemple amélioré de générateur qui produit la suite de Fibonacci :

``` rust
struct Fibonacci {
    current: u64,
    next: u64,
}

impl Fibonacci {
    fn new() -> Self {
        Fibonacci { current: 0, next: 1 }
    }
}

impl Iterator for Fibonacci {
    type Item = u64;

    fn next(&mut self) -> Option<Self::Item> {
        let current = self.current;

        // Calcul du terme suivant
        self.current = self.next;
        self.next = current + self.next;

        // Les nombres de Fibonacci peuvent devenir très grands,
        // donc vérifions qu'il n'y a pas de dépassement
        if self.current < current {
            return None; // Dépassement détecté
        }

        Some(current)
    }
}

fn main() {
    println!("Les 10 premiers nombres de Fibonacci :");
    for (i, num) in Fibonacci::new().take(10).enumerate() {
        println!("Fibonacci({}) = {}", i, num);
    }

    // Utilisation plus sophistiquée
    let sum: u64 = Fibonacci::new()
        .take_while(|&x| x < 1000)
        .filter(|x| x % 2 == 0)
        .sum();

    println!("\nSomme des nombres de Fibonacci pairs inférieurs à 1000 : {}", sum);
}
```

Dans cet exemple, nous avons créé un générateur qui :

1.  Produit les nombres de la suite de Fibonacci
2.  S'arrête en cas de dépassement de capacité
3.  Est utilisé avec d'autres méthodes comme `take_while`, `filter` et `sum`

## Méthodes avancées sur les itérateurs

L'une des forces des itérateurs en Rust est le grand nombre de méthodes disponibles par défaut. Voici un aperçu des plus utilisées :

``` rust
fn main() {
    let nombres = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // map : transformer chaque élément
    let carres: Vec<i32> = nombres.iter().map(|x| x * x).collect();
    println!("Carrés : {:?}", carres);

    // filter : ne garder que certains éléments
    let pairs: Vec<&i32> = nombres.iter().filter(|&&x| x % 2 == 0).collect();
    println!("Nombres pairs : {:?}", pairs);

    // fold : agréger les valeurs (comme reduce dans d'autres langages)
    let somme = nombres.iter().fold(0, |acc, &x| acc + x);
    println!("Somme : {}", somme);

    // any : vérifier si au moins un élément satisfait une condition
    let contient_impair = nombres.iter().any(|&x| x % 2 != 0);
    println!("Contient au moins un nombre impair : {}", contient_impair);

    // chain : concaténer des itérateurs
    let autres = vec![11, 12, 13];
    let concatenation: Vec<&i32> = nombres.iter().chain(autres.iter()).collect();
    println!("Concaténation : {:?}", concatenation);

    // zip : combiner deux itérateurs
    let lettres = vec!['a', 'b', 'c'];
    let combinaison: Vec<(char, &i32)> = lettres.iter()
        .copied()
        .zip(nombres.iter())
        .take(3)
        .collect();
    println!("Combinaison : {:?}", combinaison);

    // enumerate : obtenir l'indice et la valeur
    for (i, &val) in nombres.iter().enumerate() {
        println!("nombres[{}] = {}", i, val);
    }
}
```

## Les différents types d'itérateurs standards

Rust fournit trois types d'itérateurs principaux pour les collections :

### 1\. `iter()` - Itérateur par référence

``` rust
let v = vec![1, 2, 3];
for x in v.iter() {
    println!("{}", x); // x est de type &i32
}
```

### 2\. `iter_mut()` - Itérateur par référence mutable

``` rust
let mut v = vec![1, 2, 3];
for x in v.iter_mut() {
    *x += 10; // Modifie les valeurs dans le vecteur
}
println!("{:?}", v); // Affiche [11, 12, 13]
```

### 3\. `into_iter()` - Itérateur par valeur (consomme la collection)

``` rust
let v = vec![1, 2, 3];
for x in v.into_iter() {
    println!("{}", x); // x est de type i32
}
// v n'est plus utilisable ici car consommé par into_iter()
```

## Itérateurs paresseux

Une caractéristique importante des itérateurs en Rust est qu'ils sont paresseux (lazy) : ils n'effectuent des calculs que lorsque les valeurs sont demandées. Cela permet d'enchaîner des opérations sans créer d'allocations intermédiaires :

``` rust
fn main() {
    let nombres = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // Aucun calcul n'est effectué ici, juste la définition de l'itérateur
    let iter = nombres.iter()
        .map(|&x| {
            println!("Calcul du carré de {}", x);
            x * x
        })
        .filter(|&x| x % 2 == 0);

    println!("L'itérateur est défini mais aucun calcul n'a encore été effectué");

    // C'est ici que les calculs sont effectués, un élément à la fois
    let resultat: Vec<i32> = iter.collect();
    println!("Résultat : {:?}", resultat);
}
```

## Créer un itérateur personnalisé sans structure supplémentaire

Pour les cas simples, il est possible d'éviter de créer une structure d'itérateur séparée en utilisant les adaptateurs d'itérateurs existants :

``` rust
struct MesNombres {
    limite: u32,
}

impl MesNombres {
    fn new(limite: u32) -> Self {
        MesNombres { limite }
    }

    // L'itérateur est créé directement sans structure intermédiaire
    fn iter(&self) -> impl Iterator<Item = u32> + '_ {
        (1..=self.limite).filter(|&n| n % 3 == 0 || n % 5 == 0)
    }
}

fn main() {
    let nombres = MesNombres::new(20);

    // Affichage des multiples de 3 ou 5 jusqu'à 20
    for n in nombres.iter() {
        println!("{}", n);
    }

    // Somme des multiples de 3 ou 5 jusqu'à 20
    let somme: u32 = nombres.iter().sum();
    println!("Somme : {}", somme);
}
```

## Conclusion

Les itérateurs sont l'une des fonctionnalités les plus puissantes de Rust. Ils offrent :

1.  **Performance** : grâce à la paresse d'évaluation et à l'absence de surcoût d'abstraction
2.  **Sûreté** : en évitant les erreurs d'index et en respectant les règles de propriété et d'emprunt
3.  **Expressivité** : en permettant d'enchaîner des opérations complexes de manière lisible
4.  **Flexibilité** : en s'adaptant à tous types de collections et algorithmes

Maîtriser les itérateurs est essentiel pour écrire du code Rust idiomatique et efficace. En pratique, vous utiliserez souvent des adaptateurs d'itérateurs plutôt que d'implémenter le trait `Iterator` directement, mais comprendre le mécanisme sous-jacent vous aidera à mieux utiliser cet outil puissant.

Pour aller plus loin, explorez le module `std::iter` qui contient de nombreuses fonctions utilitaires pour créer et manipuler des itérateurs sans avoir à implémenter le trait vous-même.

⏭️ [Les traits objets et dynamic dispatch](/II-specificites/16-traits-objets-dynamic-dispatch.md) - Approfondissement sur `dyn Trait`
