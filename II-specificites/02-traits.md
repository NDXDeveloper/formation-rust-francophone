## 2\. Les traits

Retour à la [Table des matières](/SOMMAIRE.md)

Les traits sont l'un des concepts les plus puissants de Rust. Ils définissent un comportement partagé entre différents types, un peu comme les interfaces dans d'autres langages, mais avec des fonctionnalités supplémentaires.

### Définition et utilisation de base

Un trait est une collection de méthodes définissant un comportement commun:

``` rust
trait Descriptible {
    fn decrire(&self) -> String;
}
```

Cette définition indique que tout type implémentant `Descriptible` doit fournir une méthode `decrire` qui renvoie une chaîne de caractères.

### Implémentation d'un trait

Voici comment implémenter un trait pour différentes structures:

``` rust
// Définition d'un trait
trait Animal {
    fn espece(&self) -> &str;
    fn nom(&self) -> &str;
    fn faire_bruit(&self) -> String;
}

// Structure Chat
struct Chat {
    nom: String,
}

impl Animal for Chat {
    fn espece(&self) -> &str {
        "Chat"
    }

    fn nom(&self) -> &str {
        &self.nom
    }

    fn faire_bruit(&self) -> String {
        format!("{} fait miaou!", self.nom)
    }
}

// Structure Chien
struct Chien {
    nom: String,
    race: String,
}

impl Animal for Chien {
    fn espece(&self) -> &str {
        "Chien"
    }

    fn nom(&self) -> &str {
        &self.nom
    }

    fn faire_bruit(&self) -> String {
        format!("{} fait wouf!", self.nom)
    }
}

// Utilisation
let felix = Chat { nom: String::from("Félix") };
let rex = Chien { nom: String::from("Rex"), race: String::from("Berger allemand") };

println!("{}", felix.faire_bruit()); // Affiche: Félix fait miaou!
println!("{}", rex.faire_bruit());   // Affiche: Rex fait wouf!
```

### Méthodes par défaut

L'une des forces des traits en Rust est la possibilité de fournir des implémentations par défaut pour certaines méthodes:

``` rust
trait Animal {
    fn espece(&self) -> &str;
    fn nom(&self) -> &str;

    fn faire_bruit(&self) -> String;

    fn presentation(&self) -> String {
        format!("Je m'appelle {} et je suis un {}", self.nom(), self.espece())
    }

    fn description_complete(&self) -> String {
        format!("{}. {}", self.presentation(), self.faire_bruit())
    }
}
```

Avec cette définition, les types qui implémentent `Animal` n'ont besoin de définir que `espece`, `nom` et `faire_bruit`. Les méthodes `presentation` et `description_complete` ont déjà une implémentation par défaut.

### Traits comme paramètres

Les traits permettent de créer des fonctions génériques qui acceptent n'importe quel type implémentant un certain comportement:

``` rust
fn afficher_animal(animal: &impl Animal) {
    println!("Information: {}", animal.presentation());
    println!("Bruit: {}", animal.faire_bruit());
}

// Utilisation
let felix = Chat { nom: String::from("Félix") };
afficher_animal(&felix);
```

``` rust
// Définition d'un trait
trait Animal {
    fn espece(&self) -> &str;
    fn nom(&self) -> &str;

    fn faire_bruit(&self) -> String;

    fn presentation(&self) -> String {
        format!("Je m'appelle {} et je suis un {}", self.nom(), self.espece())
    }

    fn description_complete(&self) -> String {
        format!("{}. {}", self.presentation(), self.faire_bruit())
    }
}
// Structure Chat
struct Chat {
    nom: String,
}
impl Animal for Chat {
    fn espece(&self) -> &str {
        "Chat"
    }

    fn nom(&self) -> &str {
        &self.nom
    }

    fn faire_bruit(&self) -> String {
        format!("{} fait miaou!", self.nom)
    }
}
fn afficher_animal(animal: &impl Animal) {
    println!("Information: {}", animal.presentation());
    println!("Bruit: {}", animal.faire_bruit());
}

fn main() {
    // Utilisation
    let felix = Chat { nom: String::from("Félix") };
    afficher_animal(&felix);
}

```
Une autre syntaxe équivalente utilise la notation de généricité bornée:

``` rust
fn afficher_animal<T: Animal>(animal: &T) {
    println!("Information: {}", animal.presentation());
    println!("Bruit: {}", animal.faire_bruit());
}
```

Cette approche est particulièrement utile quand plusieurs paramètres partagent la même contrainte de trait.

### Combinaison de traits

Vous pouvez exiger qu'un type implémente plusieurs traits:

``` rust
use std::fmt::{Debug, Display};

fn afficher_et_debugger<T: Display + Debug>(valeur: T) {
    println!("Display: {}", valeur);
    println!("Debug: {:?}", valeur);
}
```

La syntaxe `where` peut rendre cela plus lisible pour des contraintes complexes:

``` rust
fn traiter<T>(valeur: T)
where
    T: Animal + Clone + Debug,
{
    // Implémentation...
}
```

### Supertraits (traits hérités)

Un trait peut exiger qu'un type implémente déjà un autre trait:

``` rust
trait Identifiable {
    fn id(&self) -> u64;
}

// Le trait Enregistrable exige que le type implémente aussi Identifiable
trait Enregistrable: Identifiable {
    fn enregistrer(&self) -> bool;

    fn verifier_enregistrement(&self) -> bool {
        // Utilise l'id du trait Identifiable
        println!("Vérification de l'enregistrement pour l'ID: {}", self.id());
        true
    }
}
```

Pour implémenter `Enregistrable` sur un type, il faudra obligatoirement implémenter aussi `Identifiable`.

### Traits dérivables

Rust permet d'implémenter automatiquement certains traits communs grâce à l'attribut `#[derive]`:

``` rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}

// Ces traits sont automatiquement implémentés
let p1 = Point { x: 1.0, y: 2.0 };
let p2 = p1.clone();  // Clone est disponible
println!("{:?}", p1); // Debug est disponible
assert!(p1 == p2);    // PartialEq est disponible
```

Les traits dérivables les plus courants incluent:

- `Debug`: pour l'affichage de débogage
- `Clone`, `Copy`: pour la duplication des valeurs
- `PartialEq`, `Eq`: pour les comparaisons d'égalité
- `PartialOrd`, `Ord`: pour les comparaisons d'ordre
- `Hash`: pour le hachage
- `Default`: pour les valeurs par défaut

### Traits associés (associated items)

Les traits peuvent contenir plus que des méthodes:

``` rust
trait Conteneur {
    // Type associé
    type Element;

    // Constante associée
    const CAPACITE_MAX: usize;

    // Méthodes
    fn ajouter(&mut self, element: Self::Element);
    fn est_vide(&self) -> bool;
}

struct Pile<T> {
    elements: Vec<T>,
}

impl<T> Conteneur for Pile<T> {
    type Element = T;
    const CAPACITE_MAX: usize = 100;

    fn ajouter(&mut self, element: T) {
        self.elements.push(element);
    }

    fn est_vide(&self) -> bool {
        self.elements.is_empty()
    }
}
```

### Exemple complet: utilisation pratique des traits

Voici un exemple plus complexe combinant plusieurs concepts:

``` rust
use std::fmt::{Display, Formatter, Result};

// Trait pour calculer des statistiques
trait Statistiques {
    fn moyenne(&self) -> f64;
    fn ecart_type(&self) -> f64;

    // Méthode par défaut
    fn resume(&self) -> String {
        format!("Moyenne: {:.2}, Écart-type: {:.2}", self.moyenne(), self.ecart_type())
    }
}

// Structure représentant un ensemble de données
struct DonneesNumeriques {
    valeurs: Vec<f64>,
}

impl DonneesNumeriques {
    fn new(valeurs: Vec<f64>) -> Self {
        Self { valeurs }
    }
}

// Implémentation du trait Statistiques
impl Statistiques for DonneesNumeriques {
    fn moyenne(&self) -> f64 {
        if self.valeurs.is_empty() {
            return 0.0;
        }
        let somme: f64 = self.valeurs.iter().sum();
        somme / self.valeurs.len() as f64
    }

    fn ecart_type(&self) -> f64 {
        if self.valeurs.len() <= 1 {
            return 0.0;
        }

        let moyenne = self.moyenne();
        let variance: f64 = self.valeurs.iter()
            .map(|x| (x - moyenne).powi(2))
            .sum::<f64>() / (self.valeurs.len() as f64);

        variance.sqrt()
    }
}

// Implémentation de Display pour un affichage personnalisé
impl Display for DonneesNumeriques {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        write!(f, "Données [{}]: {}", self.valeurs.len(), self.resume())
    }
}

// Fonction générique acceptant n'importe quel type implémentant Statistiques
fn analyser<T: Statistiques + Display>(donnees: &T) {
    println!("Analyse statistique:");
    println!("  {}", donnees);
    println!("  Résumé: {}", donnees.resume());
}

// Utilisation
fn main() {
    let donnees = DonneesNumeriques::new(vec![12.5, 14.3, 8.7, 16.2, 10.9]);
    analyser(&donnees);
}
```

Les traits sont l'un des mécanismes fondamentaux de Rust qui permettent de créer des abstractions puissantes sans sacrifier les performances. Ils sont au cœur de nombreuses fonctionnalités du langage, de la généricité à la surcharge d'opérateurs, en passant par la programmation orientée trait qui définit le style idiomatique de Rust.

⏭️ [Les attributs](/II-specificites/03-attributs.md)
