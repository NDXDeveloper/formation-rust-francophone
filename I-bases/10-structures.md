## 10\. Les structures

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Les structures en Rust permettent de définir des types personnalisés en regroupant des données associées. Elles sont essentielles pour organiser votre code et créer des abstractions.

### Types de structures

Rust propose quatre types de structures différents :

#### 1\. Structures classiques (avec champs nommés)

``` rust
struct Personne {
    nom: String,
    prenom: String,
    age: u32,
    email: Option<String>,
}
```

#### 2\. Structures tuples (sans noms de champs)

``` rust
struct Point3D(f64, f64, f64);
```

#### 3\. Structures unitaires (sans champs)

``` rust
struct Validation;
```

#### 4\. Structures newtype (wrapper autour d'un type)

``` rust
struct Kilometres(f64);
```

### Instanciation et accès aux champs

Voici comment créer et utiliser chaque type de structure :

#### Structure classique

``` rust
let alice = Personne {
    nom: String::from("Dupont"),
    prenom: String::from("Alice"),
    age: 30,
    email: Some(String::from("alice.dupont@exemple.fr")),
};

// Accès aux champs
println!("Nom: {}", alice.nom);
println!("Âge: {}", alice.age);
```

#### Structure tuple

``` rust
let point = Point3D(1.0, 2.0, 3.0);

// Accès par index
println!("x: {}, y: {}, z: {}", point.0, point.1, point.2);
```

#### Structure unitaire

``` rust
let validation = Validation;
// Pas de champs à accéder
```

#### Structure newtype

``` rust
let distance = Kilometres(42.5);
println!("Distance: {} km", distance.0);
```

### Syntaxe de mise à jour (update syntax)

Rust permet de créer une nouvelle instance d'une structure à partir d'une instance existante :

``` rust
let alice = Personne {
    nom: String::from("Dupont"),
    prenom: String::from("Alice"),
    age: 30,
    email: Some(String::from("alice.dupont@exemple.fr")),
};

// Créer un nouvel utilisateur avec certains champs différents
let bob = Personne {
    prenom: String::from("Bob"),
    email: None,
    ..alice // Copie les autres champs depuis alice
};

// bob.nom sera "Dupont" et bob.age sera 30
println!("Informations de Bob:");
println!("Nom: {}", bob.nom);
println!("Prénom: {}", bob.prenom);
println!("Âge: {}", bob.age);
match &bob.email {
    Some(email) => println!("Email: {}", email),
    None => println!("Email: Non renseigné"),
}
```

### Visibilité des champs

Par défaut, les champs d'une structure sont privés en dehors de leur module. Pour les rendre accessibles, utilisez le mot-clé `pub` :

``` rust
pub struct Configuration {
    pub version: String,      // Accessible publiquement
    chemin_donnees: String,   // Privé en dehors du module
}
```

### Méthodes d'instance et méthodes associées

Les méthodes sont définies dans un bloc `impl` :

``` rust
struct Rectangle {
    largeur: f64,
    hauteur: f64,
}

impl Rectangle {
    // Méthode associée (statique) - ne prend pas self
    fn nouveau(largeur: f64, hauteur: f64) -> Rectangle {
        Rectangle { largeur, hauteur }
    }

    // Méthode d'instance - prend &self (référence immuable)
    fn aire(&self) -> f64 {
        self.largeur * self.hauteur
    }

    // Méthode d'instance - prend &mut self (référence mutable)
    fn redimensionner(&mut self, facteur: f64) {
        self.largeur *= facteur;
        self.hauteur *= facteur;
    }

    // Méthode d'instance - prend self (consomme l'instance)
    fn convertir_en_carre(self) -> Rectangle {
        let cote = (self.largeur + self.hauteur) / 2.0;
        Rectangle { largeur: cote, hauteur: cote }
    }
}

// Utilisation
let mut rect = Rectangle::nouveau(5.0, 3.0);
println!("Aire: {}", rect.aire());
rect.redimensionner(2.0);
println!("Nouvelle aire: {}", rect.aire());

let carre = rect.convertir_en_carre();
// rect n'est plus utilisable ici car consommé par convertir_en_carre
```

### Déstructuration

Les structures peuvent être déstructurées pour extraire leurs champs :

``` rust
let rect = Rectangle { largeur: 10.0, hauteur: 5.0 };

let Rectangle { largeur, hauteur } = rect;
println!("Largeur: {}, Hauteur: {}", largeur, hauteur);

// Déstructuration partielle
let Rectangle { largeur, .. } = rect;
println!("Largeur uniquement: {}", largeur);

// Dans un match
match rect {
    Rectangle { largeur: l, hauteur: h } if l == h => println!("C'est un carré !"),
    Rectangle { largeur: l, hauteur: h } => println!("C'est un rectangle: {}x{}", l, h),
}
```

### Destructeur avec le trait Drop

Pour exécuter du code lorsqu'une structure est détruite, implémentez le trait `Drop` :

``` rust
struct Ressource {
    nom: String,
}

impl Ressource {
    fn nouvelle(nom: &str) -> Ressource {
        println!("Création de la ressource: {}", nom);
        Ressource { nom: nom.to_string() }
    }
}

impl Drop for Ressource {
    fn drop(&mut self) {
        println!("Libération de la ressource: {}", self.nom);
    }
}

// Utilisation
{
    let r = Ressource::nouvelle("ma_ressource");
    // Utiliser r...
} // r est détruite ici, drop() est appelée automatiquement
```

``` rust
fn main() {
    struct Ressource {
        nom: String,
        donnees: Vec<u8>,
    }

    impl Ressource {
        fn nouvelle(nom: &str, taille: usize) -> Ressource {
            println!("Création de la ressource: {}", nom);
            Ressource {
                nom: nom.to_string(),
                donnees: vec![0; taille] // Alloue un vecteur de taille spécifiée
            }
        }

        fn utiliser(&self) {
            println!("Utilisation de la ressource '{}' ({} octets)", self.nom, self.donnees.len());
        }

        fn modifier(&mut self, index: usize, valeur: u8) -> Result<(), String> {
            if index < self.donnees.len() {
                self.donnees[index] = valeur;
                Ok(())
            } else {
                Err(format!("Index {} hors limites pour {}", index, self.nom))
            }
        }
    }

    impl Drop for Ressource {
        fn drop(&mut self) {
            println!("Libération de la ressource: {} ({} octets)", self.nom, self.donnees.len());
        }
    }

    // Exemple 1: Utilisation basique dans un bloc
    println!("\n--- Exemple 1: Bloc de base ---");
    {
        let r = Ressource::nouvelle("ma_ressource", 1024);
        r.utiliser();
    } // r est détruite ici, drop() est appelée automatiquement

    // Exemple 2: Plusieurs ressources avec différentes durées de vie
    println!("\n--- Exemple 2: Différentes durées de vie ---");
    {
        let r1 = Ressource::nouvelle("ressource_externe", 2048);
        r1.utiliser();

        {
            let r2 = Ressource::nouvelle("ressource_interne", 512);
            r2.utiliser();
            // r2 est détruite à la fin de ce bloc
        }

        println!("La ressource interne a été libérée, mais r1 existe toujours");
        r1.utiliser();
        // r1 est détruite à la fin de ce bloc
    }

    // Exemple 3: Transfert de propriété (ownership)
    println!("\n--- Exemple 3: Transfert de propriété ---");
    {
        let r3 = Ressource::nouvelle("ressource_transférée", 128);

        // Fonction qui prend ownership de la ressource
        fn consommer_ressource(res: Ressource) {
            println!("Fonction a reçu la ressource '{}'", res.nom);
            // res est détruite à la fin de cette fonction
        }

        consommer_ressource(r3);
        // r3 n'est plus utilisable ici, elle a été déplacée
    }

    // Exemple 4: Utilisation de drop() explicite
    println!("\n--- Exemple 4: Drop explicite ---");
    {
        let r4 = Ressource::nouvelle("ressource_drop_explicite", 256);
        r4.utiliser();

        // Utilisation explicite de drop (généralement à éviter)
        println!("Appel explicite de drop...");
        drop(r4);

        println!("La ressource a déjà été libérée");
        // r4 n'est plus utilisable ici
    }

    // Exemple 5: Ressources dans des structures de données
    println!("\n--- Exemple 5: Ressources dans un vecteur ---");
    {
        let mut ressources = Vec::new();

        for i in 1..=3 {
            let nom = format!("ressource_vec_{}", i);
            ressources.push(Ressource::nouvelle(&nom, i * 100));
        }

        println!("Le vecteur contient {} ressources", ressources.len());

        // Les ressources sont libérées une par une quand le vecteur est détruit
    }

    // Exemple 6: Gestion des erreurs
    println!("\n--- Exemple 6: Gestion des erreurs ---");
    {
        let mut r6 = Ressource::nouvelle("ressource_avec_erreur", 5);

        // Modifie la ressource de manière sécurisée
        match r6.modifier(3, 42) {
            Ok(_) => println!("Modification réussie à l'index 3"),
            Err(e) => println!("Erreur: {}", e),
        }

        // Tentative hors limites
        match r6.modifier(10, 42) {
            Ok(_) => println!("Modification réussie à l'index 10"),
            Err(e) => println!("Erreur: {}", e),
        }

        // r6 sera libérée malgré l'erreur
    }

    println!("\nFin du programme - toutes les ressources ont été libérées");
}
```

### Dérivation de traits

Rust permet de dériver automatiquement certains comportements pour vos structures :

``` rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}

let p1 = Point { x: 1.0, y: 2.0 };
let p2 = p1.clone();

println!("p1: {:?}", p1);     // Affichage grâce à Debug
println!("p1 == p2: {}", p1 == p2);  // Comparaison grâce à PartialEq
```

### Structures transparentes (Tuple structs)

Les structures tuples sont utiles pour créer des types distincts avec la même structure sous-jacente :

``` rust
struct Celsius(f64);
struct Fahrenheit(f64);

impl Celsius {
    fn en_fahrenheit(self) -> Fahrenheit {
        Fahrenheit(self.0 * 9.0 / 5.0 + 32.0)
    }
}

impl Fahrenheit {
    fn en_celsius(self) -> Celsius {
        Celsius((self.0 - 32.0) * 5.0 / 9.0)
    }
}

// Évite les erreurs de conversion accidentelles
let temp = Celsius(25.0);
let temp_f = temp.en_fahrenheit();
println!("25°C = {}°F", temp_f.0);
```

### Récapitulatif

Les structures en Rust sont un outil fondamental pour organiser vos données. Combinées avec les méthodes et les traits, elles permettent de créer des abstractions puissantes tout en garantissant la sécurité et la performance de votre code.

⏭️ [if let / while let](/I-bases/11-if-let-while-let.md)
