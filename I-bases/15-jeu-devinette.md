## 15\. Jeu de devinette de mots

Ce chapitre met en pratique les connaissances acquises précédemment en développant un jeu de devinette de mots inspiré par "Wordle" ou "Le Pendu". Voici comment se déroule une partie:

1.  L'ordinateur choisit un mot aléatoire dans une liste prédéfinie
2.  Le joueur doit deviner le mot en proposant des lettres
3.  Vous gagnez si vous trouvez le mot avant d'épuiser vos 6 tentatives

## Présentation du jeu

Exemple d'une partie:

```
=== Jeu de Devinette de Mots ===

Chargement du dictionnaire...
Le mot mystère comporte 5 lettres.
Vous avez 6 tentatives.

Tentative 1/6
Proposez une lettre: a
Mot actuel: _ _ _ _ _
Lettres utilisées: a
Le mot ne contient pas cette lettre!

Tentative 2/6
Proposez une lettre: e
Mot actuel: _ _ _ e _
Lettres utilisées: a, e
Bien joué! La lettre 'e' est dans le mot.

[...]

Tentative 6/6
Proposez une lettre: r
Mot actuel: r u s t e
Lettres utilisées: a, e, u, s, t, r
Félicitations! Vous avez deviné le mot "ruste"!
```

## Préparation du projet

Pour réaliser notre jeu, nous aurons besoin d'une liste de mots. Nous utiliserons la crate `rand` pour sélectionner un mot aléatoirement. Ajoutons ces dépendances à notre fichier `Cargo.toml`:

```  toml
[dependencies]
rand = "0.8.5"
```

## Mettre en place la structure du jeu

Nous allons maintenant implémenter notre jeu étape par étape. Commençons par définir une petite liste de mots à deviner:

``` rust
use rand::seq::SliceRandom;
use std::collections::HashSet;
use std::io::{self, Write};

fn main() {
    println!("=== Jeu de Devinette de Mots ===\n");

    // Notre dictionnaire de mots
    println!("Chargement du dictionnaire...");
    let mots = vec![
        "ruste".to_string(),
        "cargo".to_string(),
        "trait".to_string(),
        "crate".to_string(),
        "macro".to_string(),
        "tuple".to_string(),
        "slice".to_string(),
        "match".to_string(),
        "struct".to_string(),
        "enums".to_string()
    ];

    if jouer(&mots) {
        println!("Félicitations! Vous avez gagné!");
    } else {
        println!("Dommage! Vous avez perdu.");
    }
}
```

## Implémentation de la logique du jeu

Créons maintenant la fonction principale qui gérera le déroulement du jeu:

``` rust
fn jouer(mots: &[String]) -> bool {
    // Choisir un mot aléatoire
    let mut rng = rand::thread_rng();
    let mot_mystere = mots.choose(&mut rng).expect("Le dictionnaire est vide!");

    // Initialisation des variables du jeu
    let max_tentatives = 6;
    let mut tentatives_restantes = max_tentatives;
    let mut lettres_trouvees = HashSet::new();
    let mut lettres_utilisees = HashSet::new();

    println!("Le mot mystère comporte {} lettres.", mot_mystere.len());
    println!("Vous avez {} tentatives.\n", max_tentatives);

    // Boucle principale du jeu
    while tentatives_restantes > 0 {
        println!("Tentative {}/{}", max_tentatives - tentatives_restantes + 1, max_tentatives);

        // Afficher l'état actuel du mot
        afficher_mot_actuel(mot_mystere, &lettres_trouvees);
        afficher_lettres_utilisees(&lettres_utilisees);

        // Récupérer la lettre proposée par le joueur
        let lettre = recuperer_lettre_utilisateur(&lettres_utilisees);

        if lettre.is_none() {
            continue;
        }

        let lettre = lettre.unwrap();
        lettres_utilisees.insert(lettre);

        // Vérifier si la lettre est dans le mot
        if mot_mystere.contains(lettre) {
            lettres_trouvees.insert(lettre);
            println!("Bien joué! La lettre '{}' est dans le mot.", lettre);
        } else {
            tentatives_restantes -= 1;
            println!("Le mot ne contient pas cette lettre! Il vous reste {} tentatives.", tentatives_restantes);
        }

        // Vérifier si le joueur a gagné
        let mot_complet = mot_mystere.chars().all(|c| lettres_trouvees.contains(&c));
        if mot_complet {
            println!("Mot actuel: {}", mot_mystere);
            println!("Félicitations! Vous avez deviné le mot \"{}\"!", mot_mystere);
            return true;
        }

        println!();
    }

    println!("Vous avez épuisé toutes vos tentatives!");
    println!("Le mot était: {}", mot_mystere);
    false
}
```

## Fonctions auxiliaires

Maintenant, implémentons les fonctions auxiliaires nécessaires au bon fonctionnement du jeu:

``` rust
fn afficher_mot_actuel(mot: &str, lettres_trouvees: &HashSet<char>) {
    print!("Mot actuel: ");
    for c in mot.chars() {
        if lettres_trouvees.contains(&c) {
            print!("{} ", c);
        } else {
            print!("_ ");
        }
    }
    println!();
}

fn afficher_lettres_utilisees(lettres_utilisees: &HashSet<char>) {
    print!("Lettres utilisées: ");
    if lettres_utilisees.is_empty() {
        println!("aucune");
    } else {
        let mut lettres: Vec<char> = lettres_utilisees.iter().copied().collect();
        lettres.sort_unstable();
        for (i, &lettre) in lettres.iter().enumerate() {
            if i > 0 {
                print!(", ");
            }
            print!("{}", lettre);
        }
        println!();
    }
}

fn recuperer_lettre_utilisateur(lettres_deja_utilisees: &HashSet<char>) -> Option<char> {
    print!("Proposez une lettre: ");
    io::stdout().flush().unwrap();

    let mut entree = String::new();
    if io::stdin().read_line(&mut entree).is_err() {
        println!("Erreur lors de la lecture de l'entrée!");
        return None;
    }

    let entree = entree.trim().to_lowercase();
    if entree.is_empty() {
        println!("Vous devez entrer une lettre!");
        return None;
    }

    let lettre = entree.chars().next().unwrap();

    if !lettre.is_alphabetic() {
        println!("Veuillez entrer une lettre valide!");
        return None;
    }

    if lettres_deja_utilisees.contains(&lettre) {
        println!("Vous avez déjà proposé cette lettre!");
        return None;
    }

    Some(lettre)
}
```

## Code complet du jeu

Voici le code complet du jeu, bien structuré et commenté:

``` rust
use rand::seq::SliceRandom;
use std::collections::HashSet;
use std::io::{self, Write};

fn main() {
    println!("=== Jeu de Devinette de Mots ===\n");

    println!("Chargement du dictionnaire...");
    let mots = vec![
        "ruste".to_string(),
        "cargo".to_string(),
        "trait".to_string(),
        "crate".to_string(),
        "macro".to_string(),
        "tuple".to_string(),
        "slice".to_string(),
        "match".to_string(),
        "struct".to_string(),
        "enums".to_string()
    ];

    loop {
        if jouer(&mots) {
            println!("Félicitations! Vous avez gagné!");
        } else {
            println!("Dommage! Vous avez perdu.");
        }

        // Demander si le joueur veut rejouer
        if !demander_rejouer() {
            break;
        }
    }

    println!("Merci d'avoir joué!");
}

fn jouer(mots: &[String]) -> bool {
    // Choisir un mot aléatoire
    let mut rng = rand::thread_rng();
    let mot_mystere = mots.choose(&mut rng).expect("Le dictionnaire est vide!");

    // Initialisation des variables du jeu
    let max_tentatives = 6;
    let mut tentatives_restantes = max_tentatives;
    let mut lettres_trouvees = HashSet::new();
    let mut lettres_utilisees = HashSet::new();

    println!("Le mot mystère comporte {} lettres.", mot_mystere.len());
    println!("Vous avez {} tentatives.\n", max_tentatives);

    // Boucle principale du jeu
    while tentatives_restantes > 0 {
        println!("Tentative {}/{}", max_tentatives - tentatives_restantes + 1, max_tentatives);

        // Afficher l'état actuel du mot
        afficher_mot_actuel(mot_mystere, &lettres_trouvees);
        afficher_lettres_utilisees(&lettres_utilisees);

        // Récupérer la lettre proposée par le joueur
        let lettre = recuperer_lettre_utilisateur(&lettres_utilisees);

        if lettre.is_none() {
            continue;
        }

        let lettre = lettre.unwrap();
        lettres_utilisees.insert(lettre);

        // Vérifier si la lettre est dans le mot
        if mot_mystere.contains(lettre) {
            lettres_trouvees.insert(lettre);
            println!("Bien joué! La lettre '{}' est dans le mot.", lettre);
        } else {
            tentatives_restantes -= 1;
            println!("Le mot ne contient pas cette lettre! Il vous reste {} tentatives.", tentatives_restantes);
        }

        // Vérifier si le joueur a gagné
        let mot_complet = mot_mystere.chars().all(|c| lettres_trouvees.contains(&c));
        if mot_complet {
            println!("Mot actuel: {}", mot_mystere);
            println!("Félicitations! Vous avez deviné le mot \"{}\"!", mot_mystere);
            return true;
        }

        println!();
    }

    println!("Vous avez épuisé toutes vos tentatives!");
    println!("Le mot était: {}", mot_mystere);
    false
}

fn afficher_mot_actuel(mot: &str, lettres_trouvees: &HashSet<char>) {
    print!("Mot actuel: ");
    for c in mot.chars() {
        if lettres_trouvees.contains(&c) {
            print!("{} ", c);
        } else {
            print!("_ ");
        }
    }
    println!();
}

fn afficher_lettres_utilisees(lettres_utilisees: &HashSet<char>) {
    print!("Lettres utilisées: ");
    if lettres_utilisees.is_empty() {
        println!("aucune");
    } else {
        let mut lettres: Vec<char> = lettres_utilisees.iter().copied().collect();
        lettres.sort_unstable();
        for (i, &lettre) in lettres.iter().enumerate() {
            if i > 0 {
                print!(", ");
            }
            print!("{}", lettre);
        }
        println!();
    }
}

fn recuperer_lettre_utilisateur(lettres_deja_utilisees: &HashSet<char>) -> Option<char> {
    print!("Proposez une lettre: ");
    io::stdout().flush().unwrap();

    let mut entree = String::new();
    if io::stdin().read_line(&mut entree).is_err() {
        println!("Erreur lors de la lecture de l'entrée!");
        return None;
    }

    let entree = entree.trim().to_lowercase();
    if entree.is_empty() {
        println!("Vous devez entrer une lettre!");
        return None;
    }

    let lettre = entree.chars().next().unwrap();

    if !lettre.is_alphabetic() {
        println!("Veuillez entrer une lettre valide!");
        return None;
    }

    if lettres_deja_utilisees.contains(&lettre) {
        println!("Vous avez déjà proposé cette lettre!");
        return None;
    }

    Some(lettre)
}

fn demander_rejouer() -> bool {
    print!("Voulez-vous rejouer? (o/n): ");
    io::stdout().flush().unwrap();

    let mut entree = String::new();
    if io::stdin().read_line(&mut entree).is_err() {
        return false;
    }

    let entree = entree.trim().to_lowercase();
    entree == "o" || entree == "oui"
}
```

## Améliorations possibles

Ce jeu peut être amélioré de plusieurs façons:

- **Charger un vrai dictionnaire**: Utiliser un fichier externe contenant des milliers de mots pour plus de diversité
- **Ajouter des niveaux de difficulté**: Ajuster le nombre de tentatives ou la complexité des mots selon le niveau choisi
- **Implémenter un système de score**: Attribuer des points en fonction du nombre de tentatives utilisées et du temps de jeu
- **Ajouter des indices**: Permettre au joueur de demander des indices en échange de tentatives
- **Interface graphique**: Créer une version graphique du jeu en utilisant une bibliothèque comme `ggez` ou `piston`
- **Mode multijoueur**: Permettre à deux joueurs de s'affronter, l'un choisissant le mot et l'autre essayant de le deviner

Pour implémenter certaines de ces améliorations, vous pourriez avoir besoin d'autres crates:

- `serde` et `serde_json` pour charger/sauvegarder des fichiers de configuration ou de scores
- `chrono` pour mesurer le temps de jeu
- `crossterm` pour créer une interface console plus élaborée

Ce jeu simple mais complet vous permet de mettre en pratique de nombreux concepts Rust: gestion des entrées/sorties, collections, aléatoire, gestion des erreurs et boucles de jeu.

Si vous avez des questions ou des commentaires sur cette implémentation, n'hésitez pas à ouvrir une issue sur GitHub.
