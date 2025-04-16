## 4\. Généricité

La généricité est l'un des concepts fondamentaux de Rust qui permet d'écrire du code flexible et réutilisable sans sacrifier la sécurité ni les performances. Elle permet de manipuler différents types de données avec le même code, tout en gardant la vérification des types à la compilation.

### Les bases de la généricité

Au lieu d'écrire des fonctions ou des structures spécifiques pour chaque type, la généricité nous permet de créer des abstractions qui fonctionnent avec plusieurs types différents.

Prenons un exemple simple : une fonction qui trouve la plus grande valeur entre deux éléments.

``` rust
fn plus_grand_i32(a: i32, b: i32) -> i32 {
    if a > b {
        a
    } else {
        b
    }
}

fn plus_grand_f64(a: f64, b: f64) -> f64 {
    if a > b {
        a
    } else {
        b
    }
}
```

Ces deux fonctions sont presque identiques, seuls les types changent. Avec la généricité, nous pouvons écrire une seule fonction qui fonctionne pour les deux types :

``` rust
fn plus_grand<T: PartialOrd>(a: T, b: T) -> T {
    if a > b {
        a
    } else {
        b
    }
}

fn main() {
    let entier = plus_grand(42, 24);
    let flottant = plus_grand(3.14, 2.71);

    println!("Le plus grand entier est : {}", entier);
    println!("Le plus grand flottant est : {}", flottant);
}
```

Dans cet exemple :

- `<T: PartialOrd>` est un paramètre de type générique
- `T: PartialOrd` indique que le type `T` doit implémenter le trait `PartialOrd`, qui permet la comparaison avec les opérateurs `>`, `<`, etc.

### Structures génériques

Les structures peuvent également être génériques, permettant de stocker différents types de données.

``` rust
struct Paire<T> {
    premier: T,
    second: T,
}

impl<T> Paire<T> {
    fn nouveau(premier: T, second: T) -> Self {
        Paire {
            premier,
            second,
        }
    }

    fn premier(&self) -> &T {
        &self.premier
    }

    fn second(&self) -> &T {
        &self.second
    }
}

fn main() {
    let paire_entiers = Paire::nouveau(10, 20);
    let paire_chaines = Paire::nouveau(String::from("bonjour"), String::from("monde"));

    println!("Paire d'entiers : {} et {}", paire_entiers.premier(), paire_entiers.second());
    println!("Paire de chaînes : {} et {}", paire_chaines.premier(), paire_chaines.second());
}
```

Une structure peut également utiliser plusieurs paramètres de type :

``` rust
struct PaireMixte<T, U> {
    premier: T,
    second: U,
}

impl<T, U> PaireMixte<T, U> {
    fn nouveau(premier: T, second: U) -> Self {
        PaireMixte {
            premier,
            second,
        }
    }

    // Cette méthode intervertit le premier et le second élément
    // et retourne une nouvelle paire avec des types inversés
    fn intervertir(self) -> PaireMixte<U, T> {
        PaireMixte {
            premier: self.second,
            second: self.premier,
        }
    }
}
```

### Implémentations conditionnelles

Rust permet de créer des implémentations conditionnelles pour les types génériques, en fonction des traits qu'ils implémentent :

``` rust
use std::fmt::Display;

struct Paire<T> {
    premier: T,
    second: T,
}

impl<T: Display> Paire<T> {
    // Cette méthode n'existe que si T implémente Display
    fn afficher(&self) {
        println!("Paire : ({}, {})", self.premier, self.second);
    }
}

fn main() {
    // Exemple d'utilisation
    let paire = Paire {
        premier: 1,
        second: 2,
    };
    paire.afficher();

}
```

Cette implémentation n'est disponible que si `T` implémente le trait `Display`.

### Généricité et traits : un duo puissant

Les traits constituent la véritable force de la généricité en Rust. Ils permettent de définir un comportement commun que plusieurs types peuvent partager.

Créons un exemple plus élaboré avec une hiérarchie d'animaux :

``` rust
trait Animal {
    fn nom(&self) -> &str;
    fn espece(&self) -> &str;
    fn faire_bruit(&self);

    // Méthode par défaut qui utilise les autres méthodes du trait
    fn se_presenter(&self) {
        println!("Je m'appelle {}, je suis un {} et je fais : ", self.nom(), self.espece());
        self.faire_bruit();
    }
}

struct Chat {
    nom_animal: String,
}

struct Chien {
    nom_animal: String,
    race: String,
}

struct Oiseau {
    nom_animal: String,
    peut_voler: bool,
}

impl Animal for Chat {
    fn nom(&self) -> &str {
        &self.nom_animal
    }

    fn espece(&self) -> &str {
        "chat"
    }

    fn faire_bruit(&self) {
        println!("Miaou !");
    }

    // On surcharge la méthode par défaut pour les chats
    fn se_presenter(&self) {
        println!("Je suis {} le chat, humain, je t'ignore royalement.", self.nom());
    }
}

impl Animal for Chien {
    fn nom(&self) -> &str {
        &self.nom_animal
    }

    fn espece(&self) -> &str {
        "chien"
    }

    fn faire_bruit(&self) {
        println!("Wouf wouf !");
    }
}

impl Animal for Oiseau {
    fn nom(&self) -> &str {
        &self.nom_animal
    }

    fn espece(&self) -> &str {
        "oiseau"
    }

    fn faire_bruit(&self) {
        println!("Cui cui !");
    }
}

// Fonction générique qui prend n'importe quel Animal
fn presenter_animal<T: Animal>(animal: &T) {
    animal.se_presenter();
}

fn main() {
    let felix = Chat { nom_animal: String::from("Félix") };
    let rex = Chien {
        nom_animal: String::from("Rex"),
        race: String::from("Labrador")
    };
    let tweet = Oiseau {
        nom_animal: String::from("Tweet"),
        peut_voler: true
    };

    // Utilisation de la fonction générique avec différents types
    presenter_animal(&felix);
    presenter_animal(&rex);
    presenter_animal(&tweet);
}
```

### Types mixtes et traits multiples

Un des aspects puissants de la généricité en Rust est la possibilité de combiner plusieurs contraintes de traits.

``` rust
use std::fmt::Debug;

// Cette fonction requiert que T implémente à la fois Animal et Debug
fn inspecter_animal<T: Animal + Debug>(animal: &T) {
    println!("Inspection de l'animal:");
    println!("Représentation de débogage: {:?}", animal);
    animal.se_presenter();
}
```

Pour que cette fonction fonctionne, il faudrait ajouter `#[derive(Debug)]` aux structures `Chat`, `Chien` et `Oiseau`.

### La clause `where`

Lorsque les contraintes génériques deviennent complexes, la syntaxe peut rapidement devenir difficile à lire. C'est là qu'intervient la clause `where`, qui permet de définir les contraintes de manière plus lisible :

``` rust
use std::fmt::Display;

// Sans clause where (difficile à lire avec plusieurs paramètres et contraintes)
fn analyser_animaux<T: Animal + Debug + Clone, U: Animal + Display, V: Copy + Debug>(
    animal1: &T,
    animal2: &U,
    extra: V
) -> bool {
    // Code...
    true
}

// Avec clause where (beaucoup plus lisible)
fn analyser_animaux<T, U, V>(animal1: &T, animal2: &U, extra: V) -> bool
where
    T: Animal + Debug + Clone,
    U: Animal + Display,
    V: Copy + Debug,
{
    // Code...
    true
}
```

### Monomorphisation et performances

Un des avantages majeurs de la généricité en Rust est qu'elle n'a pas d'impact sur les performances à l'exécution. Cela est dû à la *monomorphisation*, un processus où le compilateur génère du code spécifique pour chaque instanciation concrète d'un type générique utilisé dans le programme.

``` rust
fn plus_grand<T: PartialOrd>(a: T, b: T) -> T {
    if a > b { a } else { b }
}

fn main() {
    let entier = plus_grand(10, 5);
    let flottant = plus_grand(3.14, 2.71);
}
```

À la compilation, Rust génère essentiellement deux fonctions distinctes, comme si nous avions écrit :

``` rust
fn plus_grand_i32(a: i32, b: i32) -> i32 {
    if a > b { a } else { b }
}

fn plus_grand_f64(a: f64, b: f64) -> f64 {
    if a > b { a } else { b }
}

fn main() {
    let entier = plus_grand_i32(10, 5);
    let flottant = plus_grand_f64(3.14, 2.71);
}
```

Cette approche élimine le coût d'exécution habituellement associé au polymorphisme dans d'autres langages, tout en préservant la flexibilité du code.

### Généricité et types associés

Les traits en Rust peuvent définir des *types associés*, qui sont des placeholders pour des types qui seront spécifiés lors de l'implémentation du trait :

``` rust
trait Conteneur {
    type Element;  // Type associé

    fn ajouter(&mut self, element: Self::Element);
    fn obtenir(&self, index: usize) -> Option<&Self::Element>;
    fn taille(&self) -> usize;
}

struct Vecteur<T> {
    elements: Vec<T>,
}

impl<T> Conteneur for Vecteur<T> {
    type Element = T;  // On spécifie le type associé

    fn ajouter(&mut self, element: T) {
        self.elements.push(element);
    }

    fn obtenir(&self, index: usize) -> Option<&T> {
        self.elements.get(index)
    }

    fn taille(&self) -> usize {
        self.elements.len()
    }
}
```

Les types associés permettent souvent d'écrire du code plus clair que l'utilisation de paramètres de type supplémentaires.

### Contraintes par défaut et spécialisation

Rust permet de spécifier des contraintes par défaut pour les paramètres de type génériques :

``` rust
trait AvecValeurParDefaut {
    type Sortie: Default;

    fn valeur_ou_defaut(&self, valeur: Option<Self::Sortie>) -> Self::Sortie {
        valeur.unwrap_or_else(|| Self::Sortie::default())
    }
}
```

### Conclusion

La généricité en Rust offre un excellent équilibre entre flexibilité, sécurité des types et performance. Elle permet :

1.  D'écrire du code qui fonctionne avec différents types
2.  De vérifier les contraintes à la compilation
3.  De n'avoir aucun impact sur les performances à l'exécution grâce à la monomorphisation
4.  De combiner la puissance des traits avec la flexibilité de la généricité

Cette approche est au cœur de nombreuses bibliothèques de l'écosystème Rust, et comprendre la généricité est essentiel pour écrire du code Rust idiomatique et performant.

Pour aller plus loin, explorez les traits et types génériques de la bibliothèque standard comme `Iterator`, `IntoIterator` et `FromIterator`, qui illustrent parfaitement comment la généricité et les traits peuvent être combinés pour créer des abstractions puissantes et performantes.
