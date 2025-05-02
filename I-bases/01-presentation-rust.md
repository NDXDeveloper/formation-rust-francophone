## 1\. Présentation de Rust

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Rust est un langage de programmation système moderne, compilé et multi-paradigme, créé pour résoudre des problèmes fondamentaux rencontrés dans d'autres langages. Il représente une fusion intelligente entre programmation impérative (comme C), orientée objet (comme C++), fonctionnelle (comme OCaml), et concurrente (comme Erlang).

### Origines et évolution

Rust a été conçu par Graydon Hoare en 2006 alors qu'il travaillait chez Mozilla. Son objectif initial était de créer un langage permettant de développer des composants sécurisés pour Firefox sans sacrifier les performances. La première version stable (1.0) a été publiée le 15 mai 2015, marquant l'engagement du langage vers la stabilité.

En août 2020, face aux restructurations de Mozilla, la gouvernance du langage a évolué vers la création de la Fondation Rust (8 février 2021), une organisation indépendante et à but non lucratif chargée de soutenir financièrement et structurellement l'écosystème Rust sans en diriger directement le développement technique.

### Philosophie et objectifs clés

Rust repose sur trois piliers fondamentaux :

- **Sécurité** : élimination des erreurs de mémoire (débordements de tampons, dangling pointers, data races) à la compilation
- **Performance** : vitesse d'exécution comparable au C/C++ sans ramasse-miettes (garbage collector)
- **Concurrence** : faciliter la programmation parallèle sans risques de conditions de course

### Adoption industrielle

Depuis 2015, Rust a connu une adoption spectaculaire dans l'industrie :

- **Microsoft** l'utilise dans Windows et Azure
- **Google** l'a introduit dans Android et ses services cloud
- **Amazon** l'a adopté pour AWS
- **Meta** (Facebook) l'utilise pour certains services critiques
- **Mozilla** l'a utilisé pour Servo (moteur de rendu expérimental)
- **Apple** commence à l'intégrer dans ses systèmes

En 2021, Rust est devenu le deuxième langage officiellement supporté pour le développement du noyau Linux (après 30 ans de C exclusif), ce qui confirme sa maturité et sa fiabilité.

### Caractéristiques distinctives

Rust se démarque par plusieurs innovations techniques :

#### Système de propriété (Ownership)

Le concept de "propriété" est au cœur du modèle de mémoire de Rust. Chaque valeur a un unique "propriétaire", et quand le propriétaire sort de portée, la valeur est automatiquement libérée.

``` rust
fn exemple_ownership() {
    let s1 = String::from("bonjour"); // s1 est propriétaire
    let s2 = s1;                      // s1 n'est plus valide
    // println!("{}", s1);            // Erreur de compilation!
    println!("{}", s2);               // OK
}
```

#### Emprunts (Borrowing)

Plutôt que de transférer la propriété, Rust permet d'emprunter des références aux données.

``` rust
fn exemple_borrowing() {
    let s1 = String::from("bonjour");
    let len = calculer_longueur(&s1); // Emprunt immutable
    println!("La longueur de '{}' est {}.", s1, len); // s1 est toujours utilisable
}

fn calculer_longueur(s: &String) -> usize {
    s.len()
}
```

#### Sécurité des threads

Rust garantit la sécurité des threads à la compilation, éliminant une classe entière de bugs.

``` rust
use std::thread;

fn exemple_concurrence() {
    let donnees = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        // 'move' transfère la propriété de 'donnees' au thread
        println!("Le vecteur contient: {:?}", donnees);
    });

    // donnees n'est plus accessible ici
    handle.join().unwrap();
}
```

#### Sécurité sans coût à l'exécution

Contrairement à d'autres langages, les garanties de sécurité de Rust sont principalement appliquées à la compilation, sans pénalité de performance à l'exécution.

#### Écosystème robuste

Rust dispose d'un gestionnaire de paquets et outil de build intégré, Cargo, qui simplifie considérablement le développement :

``` toml
[dependencies]
sha2 = "0.11.0-pre.5"
hex = "0.4.3"

```

``` rust
// Exemple d'utilisation de crates (bibliothèques)
use sha2::Sha256;
use sha2::Digest;
use hex;

fn exemple_hash() {
    let mut hasher = Sha256::new();
    hasher.update(b"Bonjour, monde!");
    let resultat = hasher.finalize();
    println!("SHA-256: {}", hex::encode(resultat));
}
```

### Cas d'utilisation idéaux

Rust excelle particulièrement dans les domaines suivants :

- **Programmation système et embarquée**
- **Applications hautes performances** (serveurs web, bases de données)
- **Outils CLI** (ripgrep, exa, bat)
- **WebAssembly** (applications web performantes)
- **Jeux vidéo** (moteurs de jeu)
- **Systèmes distribués**
- **Blockchain et cryptographie**

### Courbe d'apprentissage

Rust a la réputation d'avoir une courbe d'apprentissage relativement abrupte, principalement en raison de son système de propriété et de son compilateur strict. Cependant, ces difficultés initiales sont compensées par :

- Une communauté exceptionnellement accueillante et bienveillante
- Une documentation de premier ordre
- Des outils d'apprentissage interactifs
- Des messages d'erreur très informatifs et constructifs

Pour suivre ce tutoriel efficacement, il est recommandé d'avoir une expérience préalable avec au moins un autre langage de programmation (C, C++, Java, JavaScript, Python, etc.).

### Ressources essentielles

- **Site officiel** : [rust-lang.org](https://www.rust-lang.org)
- **Documentation** : [doc.rust-lang.org](https://doc.rust-lang.org)
- **Repository GitHub** : [github.com/rust-lang/rust](https://github.com/rust-lang/rust)
- **The Rust Book** : [doc.rust-lang.org/book](https://doc.rust-lang.org/book/) (guide officiel, disponible en français)
- **Rustlings** : [github.com/rust-lang/rustlings](https://github.com/rust-lang/rustlings) (exercices interactifs)
- **Rust by Example** : [doc.rust-lang.org/rust-by-example](https://doc.rust-lang.org/rust-by-example/)
- **Communauté** : [Reddit r/rust](https://www.reddit.com/r/rust/) et [Forums users.rust-lang.org](https://users.rust-lang.org/)

### Cycle de publication

Un aspect distinctif de Rust est son cycle de publication stable et prédictible :

- Une nouvelle version stable toutes les six semaines
- Des éditions majeures tous les trois ans (2015, 2018, 2021, 2024) qui peuvent introduire des changements non rétrocompatibles tout en maintenant la compatibilité du code existant

Cette approche permet à Rust d'évoluer constamment tout en garantissant une stabilité exceptionnelle.

Le moment est maintenant venu d'installer les outils nécessaires et de commencer votre voyage dans l'univers de Rust.

⏭️ [Mise en place des outils](/I-bases/02-outils.md)
