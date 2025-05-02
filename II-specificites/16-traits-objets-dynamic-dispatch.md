# 16\. Les traits objets et dynamic dispatch

Retour à la [Table des matières](/SOMMAIRE.md)

Dans les sections précédentes, nous avons exploré les traits et la généricité en Rust. Nous allons maintenant approfondir un concept important : les traits objets et le dynamic dispatch. Ces mécanismes nous permettent d'implémenter un polymorphisme plus souple, au prix d'une légère perte de performance.

## 16.1. Rappel sur les traits et la généricité

Jusqu'à présent, nous avons utilisé les traits principalement avec la généricité, comme ceci :

``` rust
fn dessiner<T: Dessin>(element: &T) {
    element.dessiner();
}
```

Cette approche utilise ce qu'on appelle le **static dispatch** (envoi statique) : le compilateur génère du code spécifique pour chaque type concret qui implémente `Dessin`. C'est efficace en termes de performance, mais cela peut augmenter la taille du code compilé.

## 16.2. Les traits objets : introduction à `dyn Trait`

Un **trait objet** est une façon d'utiliser un trait de manière polymorphique sans généricité. En Rust, on les représente avec le mot-clé `dyn` suivi du nom du trait :

``` rust
fn dessiner(element: &dyn Dessin) {
    element.dessiner();
}
```

Cette syntaxe indique à Rust que nous souhaitons utiliser le **dynamic dispatch** (envoi dynamique). La décision concernant quelle implémentation de la méthode appeler se fait à l'exécution plutôt qu'à la compilation.

## 16.3. Comment fonctionnent les traits objets en interne

Un trait objet `dyn Trait` est en réalité composé de deux pointeurs :

1.  Un pointeur vers les données concrètes
2.  Un pointeur vers une **vtable** (table de fonctions virtuelles) qui contient les adresses des méthodes implémentées pour ce type spécifique

Cette structure est similaire aux objets dans les langages orientés objet comme Java ou C++.

## 16.4. Exemple concret de traits objets

Imaginons que nous développons un système de formes géométriques :

``` rust
trait Forme {
    fn aire(&self) -> f64;
    fn perimetre(&self) -> f64;
    fn description(&self) -> String;
}

struct Cercle {
    rayon: f64,
}

impl Forme for Cercle {
    fn aire(&self) -> f64 {
        std::f64::consts::PI * self.rayon * self.rayon
    }

    fn perimetre(&self) -> f64 {
        2.0 * std::f64::consts::PI * self.rayon
    }

    fn description(&self) -> String {
        format!("Cercle de rayon {}", self.rayon)
    }
}

struct Rectangle {
    largeur: f64,
    hauteur: f64,
}

impl Forme for Rectangle {
    fn aire(&self) -> f64 {
        self.largeur * self.hauteur
    }

    fn perimetre(&self) -> f64 {
        2.0 * (self.largeur + self.hauteur)
    }

    fn description(&self) -> String {
        format!("Rectangle de dimensions {} x {}", self.largeur, self.hauteur)
    }
}
```

Maintenant, nous pouvons créer un vecteur de formes diverses :

``` rust
fn main() {
    let formes: Vec<Box<dyn Forme>> = vec![
        Box::new(Cercle { rayon: 5.0 }),
        Box::new(Rectangle { largeur: 4.0, hauteur: 3.0 }),
    ];

    for forme in &formes {
        println!("Description: {}", forme.description());
        println!("Aire: {}", forme.aire());
        println!("Périmètre: {}", forme.perimetre());
        println!("---");
    }
}
```

## 16.5. Restrictions des traits objets

Tous les traits ne peuvent pas être utilisés comme traits objets. Pour qu'un trait puisse être utilisé avec `dyn Trait`, il doit être **object-safe**. Un trait est object-safe si :

1.  Toutes ses méthodes retournent des types dont la taille est connue à la compilation (pas de `Self` en position de retour)
2.  Aucune de ses méthodes n'a de paramètres de type générique
3.  Toutes ses méthodes sont object-safe

Par exemple, ce trait n'est pas object-safe :

``` rust
trait NonObjectSafe {
    fn clone(&self) -> Self;  // Retourne Self, non object-safe
    fn methode_generique<T>(&self, valeur: T);  // Paramètre générique, non object-safe
}
```

## 16.6. Utilisation avec des références

Nous n'avons pas besoin d'utiliser `Box` systématiquement. On peut aussi utiliser les traits objets avec des références :

``` rust
fn afficher_info(forme: &dyn Forme) {
    println!("Description: {}", forme.description());
    println!("Aire: {}", forme.aire());
}

fn main() {
    let cercle = Cercle { rayon: 5.0 };
    let rectangle = Rectangle { largeur: 4.0, hauteur: 3.0 };

    afficher_info(&cercle);
    afficher_info(&rectangle);
}
```

## 16.7. Static dispatch vs Dynamic dispatch

Comparons les deux approches :

``` rust
// Static dispatch - générique
fn afficher_info_statique<T: Forme>(forme: &T) {
    println!("Description: {}", forme.description());
    println!("Aire: {}", forme.aire());
}

// Dynamic dispatch - trait objet
fn afficher_info_dynamique(forme: &dyn Forme) {
    println!("Description: {}", forme.description());
    println!("Aire: {}", forme.aire());
}
```

**Avantages du static dispatch :**

- Performance optimale (pas de coût d'indirection)
- Possibilité de monomorphisation (optimisations spécifiques au type)

**Avantages du dynamic dispatch :**

- Code compilé plus compact (une seule version de la fonction)
- Flexibilité pour manipuler des types différents dans une même collection
- Plus proche du modèle de programmation orientée objet traditionnel

## 16.8. Utilisation des traits objets dans les API

Les traits objets sont particulièrement utiles pour les API publiques :

``` rust
pub struct CanvasApp {
    elements: Vec<Box<dyn Dessinable>>,
}

impl CanvasApp {
    pub fn new() -> Self {
        CanvasApp { elements: Vec::new() }
    }

    pub fn ajouter_element(&mut self, element: Box<dyn Dessinable>) {
        self.elements.push(element);
    }

    pub fn dessiner_tout(&self) {
        for element in &self.elements {
            element.dessiner();
        }
    }
}
```

Cette approche permet aux utilisateurs d'étendre votre application avec leurs propres types.

## 16.9. Traits objets et closures

Les closures en Rust implémentent les traits `Fn`, `FnMut` ou `FnOnce`. Nous pouvons utiliser ces traits comme traits objets :

``` rust
fn execute_action(action: Box<dyn Fn(i32) -> i32>, valeur: i32) -> i32 {
    action(valeur)
}

fn main() {
    let doubler = Box::new(|x| x * 2);
    let tripler = Box::new(|x| x * 3);

    println!("10 doublé: {}", execute_action(doubler, 10));  // 20
    println!("10 triplé: {}", execute_action(tripler, 10));  // 30
}
```

## 16.10. Combinaison avec d'autres fonctionnalités de Rust

Les traits objets se combinent bien avec d'autres fonctionnalités de Rust :

### Avec `Any` pour faire du downcasting

``` rust
use std::any::Any;

trait FormeAvancee: Any {
    fn aire(&self) -> f64;
    fn as_any(&self) -> &dyn Any;
}

impl FormeAvancee for Cercle {
    fn aire(&self) -> f64 {
        std::f64::consts::PI * self.rayon * self.rayon
    }

    fn as_any(&self) -> &dyn Any {
        self
    }
}

fn main() {
    let forme: Box<dyn FormeAvancee> = Box::new(Cercle { rayon: 5.0 });

    // Downcasting
    if let Some(cercle) = forme.as_any().downcast_ref::<Cercle>() {
        println!("C'est un cercle de rayon {}", cercle.rayon);
    }
}
```

### Avec des Smart Pointers

``` rust
use std::rc::Rc;

trait Forme {
    fn aire(&self) -> f64;
    fn perimetre(&self) -> f64;
    fn description(&self) -> String;
}

struct Cercle {
    rayon: f64,
}

impl Forme for Cercle {
    fn aire(&self) -> f64 {
        std::f64::consts::PI * self.rayon * self.rayon
    }

    fn perimetre(&self) -> f64 {
        2.0 * std::f64::consts::PI * self.rayon
    }

    fn description(&self) -> String {
        format!("Cercle de rayon {}", self.rayon)
    }
}

fn main() {
    let forme: Rc<dyn Forme> = Rc::new(Cercle { rayon: 5.0 });
    let forme_partagee = Rc::clone(&forme);

    println!("Description: {}", forme.description());
    println!("Description (partagée): {}", forme_partagee.description());
}
```

## 16.11. Conclusion

Les traits objets et le dynamic dispatch offrent une flexibilité précieuse en Rust, permettant d'implémenter un polymorphisme semblable à celui des langages orientés objet. Bien qu'ils introduisent un léger surcoût à l'exécution, ils peuvent conduire à un code plus expressif et plus maintenable dans certaines situations.

En pratique, Rust encourage à utiliser le static dispatch quand c'est possible pour des raisons de performance, mais les traits objets restent un outil puissant dans notre boîte à outils pour les cas où la flexibilité est prioritaire.

Le choix entre static et dynamic dispatch dépend donc de vos besoins spécifiques en termes de performance, de flexibilité et de design d'API.

⏭️ [Pattern matching avancé](/II-specificites/17-pattern-matching-avance.md) - Patterns plus complexes, guards, etc.
