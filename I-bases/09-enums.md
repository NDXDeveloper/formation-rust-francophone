## 9\. Les enums

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Les énumérations (ou `enum`) en Rust sont beaucoup plus puissantes que celles trouvées dans d'autres langages comme C ou C++. Elles ne se limitent pas à de simples valeurs numériques mais peuvent contenir des données complexes.

### Définition d'une enum

Une enum en Rust peut avoir plusieurs variantes, chacune pouvant être de différentes formes :

``` rust
enum Message {
    Texte(String),                     // Variante tuple contenant une String
    Point { x: f64, y: f64 },          // Variante struct avec champs nommés
    ChangeCouleur(u8, u8, u8),         // Variante tuple avec plusieurs valeurs
    Quitter,                           // Variante unitaire (sans données)
}
```

### Enums avec valeurs explicites

Il est possible de définir des valeurs explicites pour les variantes, similaire aux enums en C :

``` rust
enum Statut {
    Actif = 1,
    Inactif = 0,
    Suspendu = 2,
}
```

Cependant, contrairement au C, Rust ne fait pas d'incrémentation automatique des valeurs. Si vous spécifiez une valeur pour une variante, vous devez le faire pour toutes les autres si vous souhaitez des valeurs consécutives.

### Pattern matching avec les enums

Les enums sont particulièrement puissantes lorsqu'elles sont utilisées avec le pattern matching :

``` rust
fn traiter_message(msg: Message) {
    match msg {
        Message::Texte(contenu) => println!("Message texte reçu : {}", contenu),
        Message::Point { x, y } => println!("Point aux coordonnées : ({}, {})", x, y),
        Message::ChangeCouleur(r, g, b) => println!("Changement de couleur RGB : {}, {}, {}", r, g, b),
        Message::Quitter => println!("Demande de fermeture reçue"),
    }
}

fn main() {
    // Exemple 1: Création et traitement de différentes variantes de Message

    // Variante Texte contenant une chaîne de caractères
    let msg1 = Message::Texte(String::from("Bonjour tout le monde"));
    traiter_message(msg1);

    // Variante Point avec champs nommés
    let msg2 = Message::Point { x: 10.5, y: 20.3 };
    traiter_message(msg2);

    // Variante ChangeCouleur avec trois valeurs numériques
    let msg3 = Message::ChangeCouleur(255, 0, 128);
    traiter_message(msg3);

    // Variante unitaire Quitter
    let msg4 = Message::Quitter;
    traiter_message(msg4);

    // Exemple 2: Utilisation dans une fonction qui retourne un Message
    let message = creer_message_aleatoire();
    traiter_message(message);

    // Exemple 3: Utilisation avec if let pour vérifier une variante spécifique
    let message_texte = Message::Texte(String::from("Un message important"));
    if let Message::Texte(contenu) = message_texte {
        println!("Contenu du message: {}", contenu);
    } else {
        println!("Ce n'est pas un message texte");
    }

    // Exemple 4: Utilisation dans un vecteur
    let messages = vec![
        Message::Texte(String::from("Premier message")),
        Message::Point { x: 1.0, y: 2.0 },
        Message::ChangeCouleur(100, 150, 200),
        Message::Quitter
    ];

    for msg in messages {
        traiter_message(msg);
    }
}

// Fonction qui génère un message aléatoire
fn creer_message_aleatoire() -> Message {
    // Simulons un choix aléatoire (ici fixe pour l'exemple)
    Message::ChangeCouleur(128, 128, 128)
}

```

### Enums standards de la bibliothèque Rust

Rust propose plusieurs enums standards très utiles :

#### Option

Utilisée pour représenter une valeur optionnelle : soit présente (`Some(T)`), soit absente (`None`) :

``` rust
fn diviser(numerateur: f64, denominateur: f64) -> Option<f64> {
    if denominateur == 0.0 {
        None
    } else {
        Some(numerateur / denominateur)
    }
}

// Utilisation
match diviser(10.0, 2.0) {
    Some(resultat) => println!("Résultat : {}", resultat),
    None => println!("Division par zéro impossible"),
}
```

#### Result&lt;T, E&gt;

Utilisée pour représenter une opération qui peut réussir (`Ok(T)`) ou échouer (`Err(E)`) :

``` rust
use std::fs::File;

fn ouvrir_fichier(chemin: &str) {
    match File::open(chemin) {
        Ok(fichier) => println!("Fichier ouvert avec succès : {:?}", fichier),
        Err(erreur) => println!("Erreur lors de l'ouverture : {}", erreur),
    }
}
```

### Méthodes sur les enums

Comme pour les structures, il est possible d'implémenter des méthodes sur les enums :

``` rust
enum Adresse {
    IPv4(u8, u8, u8, u8),
    IPv6(String),
}

impl Adresse {
    fn est_loopback(&self) -> bool {
        match self {
            Adresse::IPv4(127, 0, 0, 1) => true,
            Adresse::IPv6(s) if s == "::1" => true,
            _ => false,
        }
    }

    fn afficher(&self) {
        match self {
            Adresse::IPv4(a, b, c, d) => println!("{}.{}.{}.{}", a, b, c, d),
            Adresse::IPv6(s) => println!("{}", s),
        }
    }
}

// Utilisation
let localhost_v4 = Adresse::IPv4(127, 0, 0, 1);
println!("Est un loopback ? {}", localhost_v4.est_loopback()); // true
localhost_v4.afficher(); // 127.0.0.1


let localhost_v6 = Adresse::IPv6(String::from("::1")
);
println!("Est un loopback ? {}", localhost_v6.est_loopback()); // true
localhost_v6.afficher(); // ::1
```

### Enums avec généricité

Les enums peuvent également utiliser des types génériques :

``` rust
enum Resultat<T, E> {
    Succes(T),
    Erreur(E),
}

// Exemple d'utilisation
let r1: Resultat<i32, &str> = Resultat::Succes(42);
let r2: Resultat<i32, &str> = Resultat::Erreur("quelque chose s'est mal passé");
```


``` rust
// Définition de l'énumération générique Resultat
enum Resultat<T, E> {
    Succes(T),
    Erreur(E),
}

// Implémentation de méthodes pour Resultat
impl<T, E> Resultat<T, E> {
    // Méthode pour vérifier si c'est un succès
    fn est_succes(&self) -> bool {
        match self {
            Resultat::Succes(_) => true,
            Resultat::Erreur(_) => false,
        }
    }

    // Méthode qui renvoie la valeur ou une valeur par défaut
    fn unwrap_ou(self, defaut: T) -> T {
        match self {
            Resultat::Succes(val) => val,
            Resultat::Erreur(_) => defaut,
        }
    }

    // Méthode qui transforme la valeur si c'est un succès
    fn map<U, F>(self, f: F) -> Resultat<U, E>
    where
        F: FnOnce(T) -> U,
    {
        match self {
            Resultat::Succes(val) => Resultat::Succes(f(val)),
            Resultat::Erreur(e) => Resultat::Erreur(e),
        }
    }
}

// Fonction qui peut échouer et retourne un Resultat
fn diviser(x: i32, y: i32) -> Resultat<i32, String> {
    if y == 0 {
        Resultat::Erreur(String::from("Division par zéro"))
    } else {
        Resultat::Succes(x / y)
    }
}

// Fonction qui extrait le contenu d'un Resultat
fn extraire_resultat<T, E>(res: Resultat<T, E>) -> String
where
    T: std::fmt::Display,
    E: std::fmt::Display,
{
    match res {
        Resultat::Succes(val) => format!("Succès: {}", val),
        Resultat::Erreur(err) => format!("Erreur: {}", err),
    }
}

fn main() {
    // Création et utilisation basique
    let r1: Resultat<i32, &str> = Resultat::Succes(42);
    let r2: Resultat<i32, &str> = Resultat::Erreur("quelque chose s'est mal passé");

    println!("r1 est un succès? {}", r1.est_succes()); // true
    println!("r2 est un succès? {}", r2.est_succes()); // false

    // Utilisation de unwrap_ou
    let valeur1 = r1.unwrap_ou(0);
    let valeur2 = r2.unwrap_ou(0);

    println!("Valeur de r1 ou défaut: {}", valeur1); // 42
    println!("Valeur de r2 ou défaut: {}", valeur2); // 0

    // Exemple avec une fonction qui peut échouer
    let resultat1 = diviser(10, 2);
    let resultat2 = diviser(10, 0);

    println!("{}", extraire_resultat(resultat1)); // Succès: 5
    println!("{}", extraire_resultat(resultat2)); // Erreur: Division par zéro

    // Chaînage d'opérations avec map
    let r3: Resultat<i32, &str> = Resultat::Succes(10);
    let r4 = r3.map(|x| x * 2);

    match r4 {
        Resultat::Succes(val) => println!("Nouvelle valeur après map: {}", val), // 20
        Resultat::Erreur(e) => println!("Erreur: {}", e),
    }

    // Exemple avec un calcul plus complexe
    fn calcul_complexe(x: i32, y: i32) -> Resultat<i32, String> {
        // Première opération pouvant échouer
        let res1 = diviser(x, y);

        // Traitement du résultat intermédiaire
        match res1 {
            Resultat::Succes(val) => {
                // Seconde opération pouvant échouer
                if val > 10 {
                    Resultat::Erreur(String::from("Résultat trop grand"))
                } else {
                    Resultat::Succes(val * val)
                }
            },
            Resultat::Erreur(e) => Resultat::Erreur(e),
        }
    }

    // Test du calcul complexe
    let calc1 = calcul_complexe(10, 2);
    let calc2 = calcul_complexe(100, 2);
    let calc3 = calcul_complexe(10, 0);

    println!("{}", extraire_resultat(calc1)); // Succès: 25
    println!("{}", extraire_resultat(calc2)); // Erreur: Résultat trop grand
    println!("{}", extraire_resultat(calc3)); // Erreur: Division par zéro
}
```

Cet exemple montre :
1. **Définition** de l'énumération générique `Resultat<T, E>`
2. **Implémentation de méthodes utiles** comme `est_succes()`, `unwrap_ou()` et `map()`
3. **Utilisation dans des fonctions** qui peuvent échouer (`diviser`)
4. **Gestion des erreurs** avec `match`
5. **Chaînage d'opérations** avec la méthode `map`
6. **Calcul complexe** combinant plusieurs opérations pouvant échouer


Les enums sont un outil fondamental en Rust qui, combinées avec le pattern matching, permettent d'écrire du code expressif, sûr et facile à maintenir.

⏭️ [Les structures](/I-bases/10-structures.md)
