# 14\. Box

Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction aux pointeurs intelligents

En Rust, le type `Box<T>` est l'un des pointeurs intelligents (smart pointers) les plus simples et fondamentaux. Contrairement à d'autres langages où l'allocation mémoire est implicite, Rust vous donne un contrôle précis sur l'emplacement de stockage de vos données.

## Qu'est-ce que Box ?

`Box<T>` est essentiellement un pointeur vers des données stockées sur le tas (heap) plutôt que sur la pile (stack). Il sert de propriétaire unique des données qu'il pointe, et lorsqu'un `Box` sort de portée, les données qu'il contient sont automatiquement désallouées grâce au système de propriété de Rust.

## Pourquoi utiliser Box ?

Voici les cas d'utilisation les plus courants de `Box` :

1.  **Stocker des données volumineuses sur le tas** plutôt que sur la pile
2.  **Créer des structures de données récursives** comme des arbres ou des listes chaînées
3.  **Manipuler des types dont la taille n'est pas connue à la compilation**
4.  **Utiliser le trait object polymorphisme** avec `Box<dyn Trait>`
5.  **Interfaçage avec du code C** (FFI) où une adresse mémoire stable est nécessaire

## Comprendre la mémoire : pile vs tas

Pour mieux comprendre l'utilité de `Box`, clarifions la distinction entre pile et tas :

- **La pile (stack)** : région de mémoire à allocation rapide et de taille fixe, gérée automatiquement (LIFO)
- **Le tas (heap)** : région de mémoire plus grande, à allocation dynamique, permettant des objets de taille variable

En Rust, les variables locales sont normalement allouées sur la pile. Cependant, lorsque vous utilisez `Box`, vous demandez explicitement au compilateur d'allouer les données sur le tas.

## Exemples d'utilisation de Box

### 1\. Stocker des données volumineuses

``` rust
fn main() {
    // Allocation sur la pile (potentiellement problématique si très grand)
    let data_on_stack = [0u8; 1000000]; // Tableau d'un million d'octets

    // Allocation sur le tas via Box (recommandé pour les grandes structures)
    let data_on_heap = Box::new([0u8; 1000000]);

    println!("Taille sur la pile: {}", std::mem::size_of_val(&data_on_stack));
    println!("Taille sur la pile avec Box: {}", std::mem::size_of_val(&data_on_heap));
    // data_on_heap n'occupe que 8 octets sur la pile (pointeur), les données sont sur le tas
}
```

### 2\. Structures récursives

Sans `Box`, les types récursifs posent problème car le compilateur ne peut pas déterminer leur taille. Voici un exemple d'arbre binaire :

``` rust
#[derive(Debug)]
enum BinaryTree<T> {
    Node {
        value: T,
        left: Box<BinaryTree<T>>,
        right: Box<BinaryTree<T>>,
    },
    Empty,
}

impl<T> BinaryTree<T> {
    fn new_leaf(value: T) -> Self {
        BinaryTree::Node {
            value,
            left: Box::new(BinaryTree::Empty),
            right: Box::new(BinaryTree::Empty),
        }
    }

    fn new_node(value: T, left: BinaryTree<T>, right: BinaryTree<T>) -> Self {
        BinaryTree::Node {
            value,
            left: Box::new(left),
            right: Box::new(right),
        }
    }
}

fn main() {
    // Création d'un petit arbre:
    //      10
    //     /  \
    //    5    15
    //   / \
    //  3   7
    let tree = BinaryTree::new_node(
        10,
        BinaryTree::new_node(
            5,
            BinaryTree::new_leaf(3),
            BinaryTree::new_leaf(7),
        ),
        BinaryTree::new_leaf(15),
    );

    println!("{:?}", tree);
}
```

Sans les `Box`, cette structure serait impossible à compiler car sa taille serait infinie.

### 3\. Implémentation d'une liste chaînée

Voici une implémentation améliorée d'une liste chaînée utilisant `Box` :

``` rust
#[derive(Debug)]
struct LinkedList<T> {
    value: T,
    next: Option<Box<LinkedList<T>>>,
}

impl<T> LinkedList<T> {
    // Crée un nouveau nœud terminal
    pub fn new(value: T) -> Self {
        LinkedList {
            value,
            next: None,
        }
    }

    // Ajoute un élément à la fin de la liste
    pub fn append(&mut self, value: T) {
        match self.next.as_mut() {
            Some(next) => next.append(value),
            None => self.next = Some(Box::new(LinkedList::new(value))),
        }
    }

    // Ajoute un élément au début et retourne la nouvelle tête
    pub fn prepend(self, value: T) -> Self {
        LinkedList {
            value,
            next: Some(Box::new(self)),
        }
    }

    // Retourne le nombre d'éléments dans la liste
    pub fn len(&self) -> usize {
        match &self.next {
            Some(next) => 1 + next.len(),
            None => 1,
        }
    }
}

impl<T: std::fmt::Display> LinkedList<T> {
    // Affiche tous les éléments de la liste
    pub fn display(&self) {
        print!("{}", self.value);

        let mut current = &self.next;
        while let Some(next) = current {
            print!(" → {}", next.value);
            current = &next.next;
        }
        println!();
    }
}

fn main() {
    // Créons une liste avec différentes méthodes
    let mut list = LinkedList::new(1);
    list.append(2);
    list.append(3);

    // Affichage et longueur
    list.display();
    println!("Longueur: {}", list.len());

    // Ajout en tête et création d'une nouvelle liste
    let new_list = list.prepend(0);
    new_list.display();
    println!("Nouvelle longueur: {}", new_list.len());
}
```

### 4\. Polymorphisme avec Box

Un autre cas d'utilisation important non mentionné dans le tutoriel original :

``` rust
trait Animal {
    fn make_sound(&self) -> String;
}

struct Dog {
    name: String,
}

impl Animal for Dog {
    fn make_sound(&self) -> String {
        format!("{} dit: Woof!", self.name)
    }
}

struct Cat {
    name: String,
}

impl Animal for Cat {
    fn make_sound(&self) -> String {
        format!("{} dit: Meow!", self.name)
    }
}

fn main() {
    // Collection d'animaux de différents types concrets
    let animals: Vec<Box<dyn Animal>> = vec![
        Box::new(Dog { name: String::from("Rex") }),
        Box::new(Cat { name: String::from("Félix") }),
    ];

    // Utilisation polymorphique
    for animal in animals {
        println!("{}", animal.make_sound());
    }
}
```

## Performance et considérations

Il est important de noter que l'utilisation de `Box` implique :

1.  Une allocation dynamique sur le tas
2.  Une indirection (déréférencement du pointeur)
3.  Un coût supplémentaire pour le système de mémoire

Pour cette raison, il est généralement préférable d'utiliser des structures de données standards comme `Vec` pour les collections, sauf si vous avez besoin des propriétés spécifiques de `Box`.

## Déréférencement

Grâce au trait `Deref`, vous pouvez accéder aux méthodes et propriétés du contenu de `Box` directement :

``` rust
fn main() {
    let x = Box::new(42);

    // Pas besoin d'écrire (*x) + 1
    let y = *x + 1;

    println!("y = {}", y); // 43
}
```

## Conclusion

`Box<T>` est un outil fondamental dans la boîte à outils Rust pour gérer l'allocation mémoire sur le tas. Bien qu'il soit souvent moins visible que d'autres fonctionnalités du langage, sa compréhension est essentielle pour créer des structures de données complexes et pour optimiser l'utilisation de la mémoire dans vos applications Rust.

⏭️ [Les itérateurs](/II-specificites/15-iterateurs.md)
