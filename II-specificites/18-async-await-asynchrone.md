# 18\. **Async/Await et programmation asynchrone**

Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction aux futures et à la programmation asynchrone

La programmation asynchrone est une technique puissante qui permet à votre code d'exécuter plusieurs tâches apparemment simultanément, sans bloquer l'exécution du programme. En Rust, cette approche est implémentée via le paradigme `async/await` qui repose sur des "futures" (futurs).

### Qu'est-ce que la programmation asynchrone?

La programmation asynchrone permet d'exécuter des opérations potentiellement longues (comme les requêtes réseau, les opérations de fichiers) sans bloquer le thread principal. Contrairement à la programmation synchrone où chaque opération doit se terminer avant que la suivante puisse commencer, la programmation asynchrone permet de "mettre en pause" une opération pendant qu'elle attend une ressource externe, et de continuer l'exécution d'autres parties du programme.

### Les futures en Rust

En Rust, une future représente un calcul asynchrone qui produira une valeur à un moment donné. Les futures sont des valeurs "paresseuses" - elles ne font rien tant qu'elles ne sont pas "exécutées" (polled).

``` rust
use std::future::Future;
```

Une future en Rust implémente le trait `Future` :

``` rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Heureusement, vous n'aurez généralement pas besoin d'implémenter ce trait manuellement grâce aux mots-clés `async` et `await`.

### Les mots-clés `async` et `await`

#### Le mot-clé `async`

Ce mot-clé permet de transformer une fonction ordinaire en fonction asynchrone qui retourne une future:

``` rust
async fn lire_fichier(chemin: &str) -> Result<String, std::io::Error> {
    std::fs::read_to_string(chemin) // En réalité, ceci est synchrone!
}
```

Une fonction `async` retourne implicitement une future qui, lorsqu'elle sera exécutée, produira la valeur de retour de la fonction.

#### Le mot-clé `await`

Le mot-clé `await` permet "d'attendre" le résultat d'une future:

``` rust
async fn traiter_donnees() -> Result<(), std::io::Error> {
    let contenu = lire_fichier("donnees.txt").await?;
    println!("Contenu: {}", contenu);
    Ok(())
}
```

Lorsque vous utilisez `.await` sur une future, le contrôle est temporairement rendu à l'exécuteur (runtime) jusqu'à ce que la future soit prête à produire une valeur.

### Les exécuteurs (runtimes)

Pour exécuter des futures, Rust a besoin d'un "exécuteur" (runtime) qui gère l'orchestration des tâches asynchrones. Les exécuteurs les plus populaires sont:

1.  **Tokio**: Le plus utilisé et complet
2.  **async-std**: Alternative inspirée par la bibliothèque standard
3.  **smol**: Léger et simple

Voici un exemple d'utilisation de Tokio:

``` rust
use tokio::fs::File;
use tokio::io::{self, AsyncReadExt};

#[tokio::main] // Cette macro configure l'exécuteur Tokio
async fn main() -> io::Result<()> {
    // Ouvrir un fichier de manière asynchrone
    let mut fichier = File::open("exemple.txt").await?;

    // Préparer un buffer pour la lecture
    let mut contenu = String::new();

    // Lire le contenu de manière asynchrone
    fichier.read_to_string(&mut contenu).await?;

    println!("Contenu du fichier: {}", contenu);

    Ok(())
}
```

Pour utiliser Tokio, vous devez l'ajouter comme dépendance dans votre `Cargo.toml`:

``` toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

### Principes de base pour des programmes asynchrones efficaces

1.  **Ne bloquez jamais dans du code asynchrone**: Évitez d'utiliser des opérations bloquantes (comme `std::fs::read_to_string`) dans du code asynchrone.

2.  **Utilisez les versions asynchrones des opérations I/O**: La plupart des exécuteurs fournissent des versions asynchrones des opérations d'I/O standard.

3.  **Exécuter plusieurs futures en parallèle**: Utilisez `join!` ou `select!` pour exécuter plusieurs futures en parallèle.


``` rust
use tokio::time::{sleep, Duration};

async fn tache_1() -> String {
    sleep(Duration::from_millis(100)).await;
    "Résultat de la tâche 1".to_string()
}

async fn tache_2() -> String {
    sleep(Duration::from_millis(50)).await;
    "Résultat de la tâche 2".to_string()
}

#[tokio::main]
async fn main() {
    // Exécuter les deux tâches en parallèle
    let (resultat1, resultat2) = tokio::join!(tache_1(), tache_2());

    println!("Tâche 1: {}", resultat1);
    println!("Tâche 2: {}", resultat2);
}
```

### Exemple concret: Un client HTTP asynchrone

Voici un exemple plus complet utilisant le client HTTP asynchrone `reqwest`:

``` rust
use std::error::Error;
use reqwest::Client;
use tokio::time::Duration;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    // Créer un client HTTP avec configuration
    let client = Client::builder()
        .timeout(Duration::from_secs(10))
        .build()?;

    // Effectuer plusieurs requêtes en parallèle
    let urls = vec![
        "https://www.rust-lang.org",
        "https://crates.io",
        "https://doc.rust-lang.org",
    ];

    // Créer un vecteur de futures pour les requêtes
    let requetes = urls.iter().map(|&url| {
        let client = &client;
        async move {
            println!("Téléchargement de {}", url);
            let debut = std::time::Instant::now();
            let resp = client.get(url).send().await?;
            let statut = resp.status();
            let corps = resp.text().await?;
            let duree = debut.elapsed();
            println!("URL: {}, Statut: {}, Taille: {} octets, Durée: {:?}",
                     url, statut, corps.len(), duree);
            Ok::<_, reqwest::Error>(())
        }
    });

    // Exécuter toutes les requêtes simultanément
    let resultats = futures::future::join_all(requetes).await;

    // Vérifier si toutes les requêtes ont réussi
    for resultat in resultats {
        if let Err(e) = resultat {
            eprintln!("Erreur: {}", e);
        }
    }

    Ok(())
}
```

Pour utiliser ce code, ajoutez ces dépendances à votre `Cargo.toml`:

``` toml
[dependencies]
tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.11", features = ["json"] }
futures = "0.3"
```

### Les Streams: collections asynchrones

Un `Stream` est l'équivalent asynchrone d'un itérateur. Il produit des éléments de manière asynchrone au fil du temps:

``` rust
use futures::stream::{self, StreamExt};
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    // Créer un stream qui émet 5 nombres
    let mut stream = stream::iter(1..=5)
        .map(|i| async move {
            // Simuler un traitement qui prend du temps
            sleep(Duration::from_millis(100 * i)).await;
            i * i
        })
        .buffer_unordered(3); // Traiter 3 éléments en parallèle max

    // Consommer le stream de manière asynchrone
    while let Some(nombre) = stream.next().await {
        println!("Reçu: {}", nombre);
    }

    println!("Stream terminé");
}
```

### Gestion des erreurs asynchrones

La gestion des erreurs dans le code asynchrone fonctionne de manière similaire au code synchrone, mais avec quelques subtilités:

``` rust
use std::io;
use tokio::fs::File;
use tokio::io::AsyncReadExt;

async fn lire_fichier(chemin: &str) -> Result<String, io::Error> {
    let mut fichier = File::open(chemin).await?; // Le ? fonctionne avec await
    let mut contenu = String::new();
    fichier.read_to_string(&mut contenu).await?;
    Ok(contenu)
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    match lire_fichier("fichier_inexistant.txt").await {
        Ok(contenu) => println!("Contenu: {}", contenu),
        Err(e) => println!("Erreur de lecture: {}", e),
    }

    // Utilisation du ? pour la propagation d'erreur
    let contenu = lire_fichier("config.txt").await?;
    println!("Configuration chargée: {}", contenu);

    Ok(())
}
```

### Conclusion

La programmation asynchrone en Rust avec `async/await` offre un modèle puissant pour traiter les opérations concurrentes sans les risques traditionnels de la programmation multithread. Elle permet d'écrire du code qui semble séquentiel mais s'exécute de manière non-bloquante, optimisant ainsi l'utilisation des ressources système.

Les points clés à retenir:

- Les fonctions `async` retournent des futures
- L'opérateur `.await` attend la complétion d'une future
- Un exécuteur (runtime) est nécessaire pour exécuter les futures
- Utilisez les outils appropriés pour la parallélisation (`join!`, `select!`, etc.)
- Privilégiez toujours les versions asynchrones des opérations d'I/O

La maîtrise de la programmation asynchrone en Rust ouvre la porte à des applications hautement performantes capables de gérer des milliers de connexions simultanées avec un minimum de ressources.

⏭️ [Pinning et futures](/II-specificites/19-pinning-futures.md) - Concept de pinning pour les types non déplaçables
