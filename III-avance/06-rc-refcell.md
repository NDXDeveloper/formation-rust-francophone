## 6\. Rc et RefCell

Le système de propriété de Rust est l'un de ses atouts principaux, mais il peut parfois sembler restrictif. Les types `Rc` et `RefCell` offrent de la flexibilité supplémentaire lorsque les règles de propriété standard ne conviennent pas à certains scénarios.

### 6.1 RefCell: Mutabilité intérieure

Le type `RefCell` permet d'implémenter le concept de "mutabilité intérieure", c'est-à-dire la possibilité de modifier le contenu d'une valeur même lorsque celle-ci est considérée comme immutable. Contrairement au modèle standard de Rust, les vérifications d'emprunt sont effectuées au moment de l'exécution plutôt qu'à la compilation.

#### Pourquoi avons-nous besoin de RefCell?

Dans certains scénarios, le modèle de propriété standard de Rust est trop restrictif. Par exemple, lorsque nous voulons:

- Modifier un objet à partir de plusieurs références
- Gérer des structures de données complexes avec des références circulaires
- Mettre à jour des données dans une fermeture ou un callback

Examinons un problème concret où `RefCell` devient utile:

``` rust
struct Compteur {
    valeur: u32,
}

impl Compteur {
    fn new() -> Self {
        Compteur { valeur: 0 }
    }

    fn incrementer(&mut self) {
        self.valeur += 1;
    }

    fn valeur(&self) -> u32 {
        self.valeur
    }
}

struct Statistiques {
    compteurs: Vec<Compteur>, // Impossible d'avoir des références mutables ici
}

impl Statistiques {
    fn new() -> Self {
        Statistiques { compteurs: vec![] }
    }

    fn ajouter_compteur(&mut self, compteur: Compteur) {
        self.compteurs.push(compteur);
    }

    fn incrementer_tous(&mut self) {
        for compteur in &mut self.compteurs {
            compteur.incrementer();
        }
    }
}
```

Ce modèle fonctionne bien tant que nous n'avons pas besoin de partager les compteurs. Mais que se passe-t-il si nous voulons que plusieurs structures puissent accéder aux mêmes compteurs?

Voici comment `RefCell` peut nous aider:

```
use std::cell::RefCell;

struct Compteur {
    valeur: u32,
}

impl Compteur {
    fn new() -> Self {
        Compteur { valeur: 0 }
    }

    fn incrementer(&mut self) {
        self.valeur += 1;
    }

    fn valeur(&self) -> u32 {
        self.valeur
    }
}

struct Statistiques {
    compteurs: Vec<RefCell<Compteur>>,
}

impl Statistiques {
    fn new() -> Self {
        Statistiques { compteurs: vec![] }
    }

    fn ajouter_compteur(&mut self, compteur: Compteur) {
        self.compteurs.push(RefCell::new(compteur));
    }

    fn incrementer_tous(&self) { // Notez que cette méthode n'a plus besoin d'être &mut self
        for compteur in &self.compteurs {
            compteur.borrow_mut().incrementer();
        }
    }

    fn afficher_valeurs(&self) {
        for (index, compteur) in self.compteurs.iter().enumerate() {
            println!("Compteur {}: {}", index, compteur.borrow().valeur());
        }
    }
}
```

#### Les méthodes importantes de RefCell

- `borrow()` : Emprunte une référence immutable. Panique si le contenu est déjà emprunté mutablement.
- `borrow_mut()` : Emprunte une référence mutable. Panique si le contenu est déjà emprunté (mutablement ou immutablement).
- `try_borrow()` et `try_borrow_mut()` : Versions qui retournent un `Result` au lieu de paniquer.

#### Attention aux paniques

`RefCell` vérifie les règles d'emprunt à l'exécution. Si ces règles sont violées, votre programme paniquera:

``` rust
use std::cell::RefCell;

fn main() {
    let cellule = RefCell::new(42);

    let reference_mut1 = cellule.borrow_mut();
    let reference_mut2 = cellule.borrow_mut(); // PANIQUE: déjà emprunté mutablement

    // Même problème avec:
    // let reference_mut = cellule.borrow_mut();
    // let reference = cellule.borrow(); // PANIQUE: déjà emprunté mutablement
}
```

Pour éviter cela, utilisez les méthodes `try_borrow` et `try_borrow_mut`:

```
use std::cell::RefCell;

fn main() {
    let cellule = RefCell::new(42);

    let reference_mut1 = cellule.borrow_mut();

    match cellule.try_borrow_mut() {
        Ok(mut val) => println!("Obtenu: {}", val),
        Err(e) => println!("Impossible d'emprunter: {}", e),
    }
}
```

### 6.2 Rc: Compteur de références partagées

`Rc` (Reference Counted) permet de partager la propriété d'une valeur entre plusieurs parties de code. Chaque clone d'un `Rc` incrémente un compteur, et lorsque le dernier clone est supprimé, la valeur est détruite.

``` rust
use std::rc::Rc;

fn main() {
    // Création d'un Rc contenant une String
    let texte = Rc::new(String::from("Bonjour, monde!"));

    println!("Compteur initial: {}", Rc::strong_count(&texte)); // Affiche 1

    {
        let texte2 = Rc::clone(&texte);
        println!("Compteur après clone: {}", Rc::strong_count(&texte)); // Affiche 2

        // Les deux pointent vers la même chaîne
        println!("Adresse texte: {:p}", Rc::as_ptr(&texte));
        println!("Adresse texte2: {:p}", Rc::as_ptr(&texte2));
        println!("texte2: {}", *texte2);
    }

    // texte2 est détruit, le compteur diminue
    println!("Compteur après la fin du bloc: {}", Rc::strong_count(&texte)); // Affiche 1
}
```

#### Limitations importantes

- `Rc` est pour le partage en lecture seule; vous ne pouvez pas obtenir une référence mutable à son contenu directement.
- `Rc` n'est pas thread-safe. Pour un partage entre threads, utilisez `Arc` (Atomic Reference Counting).

### 6.3 Combinaison de Rc et RefCell

La combinaison de `Rc` et `RefCell` est très puissante. Elle permet de partager des données mutables entre plusieurs propriétaires:

``` rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Tâche {
    description: String,
    terminée: bool,
}

impl Tâche {
    fn new(description: &str) -> Self {
        Tâche {
            description: description.to_string(),
            terminée: false,
        }
    }

    fn marquer_terminée(&mut self) {
        self.terminée = true;
    }
}

fn main() {
    // Création d'une tâche partagée
    let tâche_partagée = Rc::new(RefCell::new(Tâche::new("Apprendre Rust")));

    // Deux listes qui partagent la même tâche
    let liste_urgente = vec![Rc::clone(&tâche_partagée)];
    let liste_aujourd_hui = vec![Rc::clone(&tâche_partagée)];

    // Modification de la tâche via la première liste
    println!("État initial de la tâche: {:?}", tâche_partagée.borrow());

    // Marquer comme terminée
    liste_urgente[0].borrow_mut().marquer_terminée();

    // La modification est visible via les deux listes
    println!("État après modification via liste_urgente: {:?}", liste_aujourd_hui[0].borrow());
}
```

### 6.4 Structures circulaires avec Rc et RefCell

`Rc` et `RefCell` permettent de créer des structures de données avec des références circulaires, comme des arbres ou des graphes:

``` rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Nœud {
    valeur: i32,
    enfants: Vec<Rc<RefCell<Nœud>>>,
    parent: Option<Rc<RefCell<Nœud>>>,
}

impl Nœud {
    fn new(valeur: i32) -> Rc<RefCell<Self>> {
        Rc::new(RefCell::new(Nœud {
            valeur,
            enfants: vec![],
            parent: None,
        }))
    }

    fn ajouter_enfant(parent: &Rc<RefCell<Nœud>>, enfant: &Rc<RefCell<Nœud>>) {
        // Ajouter l'enfant au parent
        parent.borrow_mut().enfants.push(Rc::clone(enfant));

        // Définir le parent de l'enfant
        enfant.borrow_mut().parent = Some(Rc::clone(parent));
    }
}

fn main() {
    let racine = Nœud::new(1);
    let enfant1 = Nœud::new(2);
    let enfant2 = Nœud::new(3);

    Nœud::ajouter_enfant(&racine, &enfant1);
    Nœud::ajouter_enfant(&racine, &enfant2);

    // Afficher la structure
    println!("Racine: {}", racine.borrow().valeur);
    println!("Nombre d'enfants: {}", racine.borrow().enfants.len());

    for (i, enfant) in racine.borrow().enfants.iter().enumerate() {
        println!("Enfant {}: {}", i, enfant.borrow().valeur);
        println!("Parent de l'enfant {}: {}", i,
            enfant.borrow().parent.as_ref().unwrap().borrow().valeur);
    }
}
```

#### Attention aux cycles de mémoire

Les références circulaires avec `Rc` peuvent créer des fuites de mémoire car les compteurs ne retournent jamais à zéro. Pour éviter cela:

- Utilisez `Weak` (références faibles) pour briser les cycles
- Concevez votre structure de données pour éviter les cycles
- Envisagez l'utilisation de `Rc::downgrade` pour créer des références faibles

``` rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Nœud {
    valeur: i32,
    enfants: Vec<Rc<RefCell<Nœud>>>,
    parent: Option<Weak<RefCell<Nœud>>>, // Référence faible
}

impl Nœud {
    fn new(valeur: i32) -> Rc<RefCell<Self>> {
        Rc::new(RefCell::new(Nœud {
            valeur,
            enfants: vec![],
            parent: None,
        }))
    }

    fn ajouter_enfant(parent: &Rc<RefCell<Nœud>>, enfant: &Rc<RefCell<Nœud>>) {
        parent.borrow_mut().enfants.push(Rc::clone(enfant));
        enfant.borrow_mut().parent = Some(Rc::downgrade(parent)); // Crée une référence faible
    }
}
```

### 6.5 Quand utiliser Rc et RefCell?

- Utilisez `Rc` quand vous avez besoin de partager la propriété d'une valeur non mutable entre plusieurs parties du code.
- Utilisez `RefCell` quand vous avez besoin de mutabilité intérieure pour une valeur qui n'est pas partagée.
- Combinez `Rc<RefCell<T>>` quand vous avez besoin de partager la propriété d'une valeur mutable.
- Pour un code thread-safe, remplacez `Rc` par `Arc` et `RefCell` par `Mutex` ou `RwLock`.

### 6.6 Alternatives à RefCell

- `Cell<T>` : Une version plus simple de `RefCell` pour les types qui implémentent `Copy`.
- `Mutex<T>` et `RwLock<T>` : Équivalents thread-safe de `RefCell`.

``` rust
use std::cell::Cell;

struct Compteur {
    valeur: Cell<u32>,
}

impl Compteur {
    fn new() -> Self {
        Compteur { valeur: Cell::new(0) }
    }

    fn incrementer(&self) { // Notez que cette méthode prend &self, pas &mut self
        let v = self.valeur.get();
        self.valeur.set(v + 1);
    }

    fn valeur(&self) -> u32 {
        self.valeur.get()
    }
}

fn main() {
    let compteur = Compteur::new();

    println!("Valeur initiale: {}", compteur.valeur());
    compteur.incrementer();
    compteur.incrementer();
    println!("Valeur finale: {}", compteur.valeur());
}
```

En résumé, `Rc` et `RefCell` sont des outils essentiels pour gérer les cas complexes de propriété partagée et de mutabilité en Rust, mais ils doivent être utilisés avec précaution. Comprendre comment et quand les utiliser vous aidera à concevoir des structures de données flexibles tout en préservant la sécurité que Rust garantit.
