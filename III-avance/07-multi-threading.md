# 7\. Le Multi-threading

## Introduction au multi-threading en Rust

Le multi-threading est un concept fondamental en programmation moderne, permettant d'exécuter plusieurs segments de code en parallèle. Rust offre une approche unique grâce à son système de propriété, garantissant la sécurité des données partagées entre threads.

Commençons par un exemple simple :

``` rust
use std::thread;

fn main() {
    // On lance le thread
    let handle = thread::spawn(|| {
        println!("Salutations depuis un thread !");
        // On peut effectuer des opérations longues ici
        for i in 1..5 {
            println!("Compteur dans le thread: {}", i);
            // Simulons un traitement
            std::thread::sleep(std::time::Duration::from_millis(500));
        }
        // Valeur de retour du thread
        "Opération terminée avec succès"
    });

    println!("Le thread principal continue son exécution pendant que le thread secondaire travaille");

    // On attend que le thread termine son travail avant de quitter
    let result = handle.join().unwrap();
    println!("Message du thread: {}", result);
}
```

La fonction `thread::spawn` exécute le code de la closure dans un nouveau thread et renvoie un `JoinHandle`. Nous appelons ensuite la méthode `join()` pour attendre la fin de l'exécution du thread et récupérer sa valeur de retour.

## Partage de données entre threads

L'avantage de Rust réside dans sa gestion sécurisée du partage de données. Essayons de partager des variables entre threads :

``` rust
use std::thread;

fn main() {
    let data = vec![1u32, 2, 3];

    // Tentative de capture directe
    let handle = thread::spawn(|| {
        // Ceci ne compilera pas
        println!("Les données dans le thread: {:?}", data);
    });

    handle.join().unwrap();
}
```

Ce code génère une erreur de compilation car Rust ne peut pas garantir que la variable `data` restera valide pendant toute la durée d'exécution du thread. Nous pouvons utiliser le mot-clé `move` pour transférer la propriété :

``` rust
use std::thread;

fn main() {
    let data = vec![1u32, 2, 3];

    // On transfère la propriété avec 'move'
    let handle = thread::spawn(move || {
        println!("Les données dans le thread: {:?}", data);
    });

    // Après move, data n'est plus accessible ici
    // println!("{:?}", data); // Erreur de compilation

    handle.join().unwrap();
}
```

Mais cela pose un autre problème : que faire si nous voulons modifier les données dans plusieurs threads ? Essayons :

``` rust
use std::thread;

fn main() {
    let mut data = vec![1u32, 2, 3];
    let mut handles = Vec::new();

    for i in 0..3 {
        // On va tenter de modifier data dans chaque thread
        handles.push(thread::spawn(move || {
            data[i] += 1; // Erreur: plusieurs références mutables
        }));
    }

    for handle in handles {
        handle.join().expect("Échec de join");
    }
}
```

Ce code ne compilera pas car il viole les règles fondamentales du système de propriété de Rust : nous essayons d'avoir plusieurs références mutables à `data` simultanément.

## Solutions pour le partage sécurisé

Pour résoudre ce problème, Rust propose plusieurs mécanismes de synchronisation.

### Mutex (Mutual Exclusion)

Le `Mutex` permet d'utiliser une même donnée depuis plusieurs endroits en garantissant qu'un seul thread y accède à la fois :

``` rust
use std::thread;
use std::sync::Mutex;

fn main() {
    // On crée notre mutex contenant les données
    let data = Mutex::new(vec![1u32, 2, 3]);
    let mut handles = Vec::new();

    for i in 0..3 {
        // Ceci ne fonctionnera pas directement
        let data_clone = data.clone(); // Erreur : Mutex n'implémente pas Clone

        handles.push(thread::spawn(move || {
            // On verrouille le mutex pour accéder aux données
            let mut locked_data = data_clone.lock().unwrap();
            locked_data[i] += 1;
        }));
    }

    for handle in handles {
        handle.join().expect("Échec de join");
    }
}
```

Ce code ne compile pas car `Mutex` seul ne peut pas être partagé entre threads. C'est là qu'intervient `Arc`.

### Arc (Atomic Reference Counting)

`Arc` est similaire à `Rc`, mais thread-safe grâce à l'utilisation de compteurs atomiques et l'implémentation du trait `Sync` :

``` rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // On combine Arc et Mutex pour un partage sécurisé
    let data = Arc::new(Mutex::new(vec![1u32, 2, 3]));
    let mut handles = Vec::new();

    for i in 0..3 {
        // On clone Arc (pas les données sous-jacentes)
        let data_clone = Arc::clone(&data);

        handles.push(thread::spawn(move || {
            // On verrouille pour modification
            let mut locked_data = data_clone.lock().unwrap();
            locked_data[i] += 1;
            println!("Thread {} a modifié la valeur à {}", i, locked_data[i]);
        }));
    }

    for handle in handles {
        handle.join().expect("Échec de join");
    }

    // Vérification du résultat final
    let result = data.lock().unwrap();
    println!("État final des données: {:?}", *result);
}
```

Dans cet exemple, `Arc` permet de partager le `Mutex` entre threads, et le `Mutex` assure que seul un thread à la fois peut modifier les données.

## Les Channels

Une autre approche pour la communication entre threads est l'utilisation de channels. Plutôt que de partager l'état, on partage la communication :

```rust
use std::thread;
use std::sync::mpsc; // multiple producer, single consumer

fn main() {
    // Création du channel
    let (tx, rx) = mpsc::channel();

    // Lancer des calculs dans des threads séparés
    for i in 1..5 {
        let tx_clone = tx.clone();
        thread::spawn(move || {
            // Simulation d'un calcul
            let result = i * i;
            println!("Thread {} a calculé {}", i, result);

            // Envoi du résultat au thread principal
            tx_clone.send(result).expect("Échec de l'envoi");
        });
    }

    // Libérer le tx original (important!)
    drop(tx);

    // Réception et traitement des résultats dans le thread principal
    let mut sum = 0;
    while let Ok(result) = rx.recv() {
        println!("Reçu: {}", result);
        sum += result;
    }

    println!("Somme de tous les résultats: {}", sum);
}
```

Les channels sont particulièrement utiles pour implémenter des patterns de type "worker" où des tâches sont distribuées à plusieurs threads qui renvoient leurs résultats au thread principal.

### Channels et timeout

Un problème courant est d'éviter que le thread principal attende indéfiniment. Vous pouvez utiliser `recv_timeout` :

``` rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        thread::sleep(Duration::from_secs(2));
        tx.send("Réponse tardive").unwrap();
    });

    match rx.recv_timeout(Duration::from_secs(1)) {
        Ok(msg) => println!("Reçu: {}", msg),
        Err(mpsc::RecvTimeoutError::Timeout) => println!("Timeout dépassé, pas de réponse dans le délai imparti"),
        Err(mpsc::RecvTimeoutError::Disconnected) => println!("L'émetteur a été déconnecté"),
    }
}
```

### Transmission synchrone avec `sync_channel`

Il existe également une variante synchrone des channels qui limite la taille de la file d'attente :

``` rust
use std::thread;
use std::sync::mpsc;

fn main() {
    // Création d'un channel synchrone avec une capacité de 2 messages
    let (tx, rx) = mpsc::sync_channel(2);

    thread::spawn(move || {
        for i in 1..6 {
            println!("Envoi de {}", i);
            // Bloquera après avoir envoyé 2 messages jusqu'à ce que le récepteur consomme
            tx.send(i).unwrap();
            println!("Message {} envoyé", i);
        }
    });

    thread::sleep(std::time::Duration::from_secs(1));

    for _ in 1..6 {
        let received = rx.recv().unwrap();
        println!("Reçu: {}", received);
        thread::sleep(std::time::Duration::from_millis(500));
    }
}
```

## Gestion des erreurs avec les threads

### Empoisonnement de Mutex

Un aspect important à connaître est l'empoisonnement des Mutex. Si un thread panic! alors qu'il détient le verrou d'un Mutex, celui-ci est marqué comme "empoisonné" :

``` rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let data = Arc::new(Mutex::new(0));
    let data_clone = Arc::clone(&data);

    // Un thread qui va paniquer
    let handle = thread::spawn(move || {
        let mut locked_data = data_clone.lock().unwrap();
        *locked_data += 1;

        // Simulons une erreur
        panic!("Oups, quelque chose s'est mal passé!");
    });

    // Le thread principal attend, mais le thread secondaire a paniqué
    let _ = handle.join();

    // Tentative d'accès au mutex empoisonné
    match data.lock() {
        Ok(mut guard) => {
            *guard += 1;
            println!("Valeur actuelle: {}", *guard);
        },
        Err(poisoned) => {
            // On peut récupérer les données malgré l'empoisonnement
            let mut guard = poisoned.into_inner();
            *guard += 1;
            println!("Récupéré après empoisonnement: {}", *guard);
        }
    }
}
```

### Capturer les panics avec `join()`

Vous pouvez également utiliser le résultat de `join()` pour détecter si un thread a paniqué :

``` rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        if rand::random::<bool>() {
            panic!("Simulation d'une erreur");
        } else {
            "Tout s'est bien passé"
        }
    });

    match handle.join() {
        Ok(result) => println!("Thread terminé avec succès: {}", result),
        Err(e) => println!("Thread paniqué: {:?}", e),
    }
}
```

## Les types atomiques

Les types atomiques permettent d'effectuer des opérations simples sur des valeurs partagées sans utiliser de verrous comme les Mutex :

``` rust
use std::sync::atomic::{AtomicU32, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicU32::new(0));
    let mut handles = Vec::new();

    // Créer 10 threads qui vont chacun incrémenter le compteur 1000 fois
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                // Incrémenter de manière atomique
                counter.fetch_add(1, Ordering::Relaxed);
            }
        });
        handles.push(handle);
    }

    // Attendre que tous les threads terminent
    for handle in handles {
        handle.join().unwrap();
    }

    println!("Compteur final: {}", counter.load(Ordering::Relaxed));
}
```

Les opérations atomiques sont plus efficaces que les Mutex pour des opérations simples comme les compteurs.

### Ordonnancement de la mémoire

Le paramètre `Ordering` détermine les garanties de synchronisation de mémoire :

- `Relaxed` : aucune garantie synchronisation de mémoire supplémentaire, uniquement l'atomicité de l'opération
- `Acquire` : pour les opérations de lecture (empêche la réorganisation des lectures/écritures suivantes)
- `Release` : pour les opérations d'écriture (empêche la réorganisation des lectures/écritures précédentes)
- `AcqRel` : combine Acquire et Release (pour les opérations d'échange)
- `SeqCst` : l'ordonnancement le plus strict, garantit un ordre total sur toutes les opérations

``` rust
use std::sync::atomic::{AtomicBool, Ordering};
use std::sync::Arc;
use std::thread;
use std::sync::atomic::AtomicU32;

fn main() {
    let flag = Arc::new(AtomicBool::new(false));
    let data = Arc::new(AtomicU32::new(0));

    // Thread producteur
    let flag_clone = Arc::clone(&flag);
    let data_clone = Arc::clone(&data);
    let producer = thread::spawn(move || {
        // Effectuer des calculs
        data_clone.store(42, Ordering::Relaxed);
        // Signaler que les données sont prêtes (avec synchronisation)
        flag_clone.store(true, Ordering::Release);
    });

    // Thread consommateur
    let consumer = thread::spawn(move || {
        // Attendre que le flag soit activé
        while !flag.load(Ordering::Acquire) {
            thread::yield_now(); // Céder le CPU
        }

        // À ce stade, on est assuré de voir la valeur 42 dans data
        let value = data.load(Ordering::Relaxed);
        assert_eq!(value, 42);
    });

    producer.join().unwrap();
    consumer.join().unwrap();
}
```

## Gestion de threads avancée avec `crossbeam`

La bibliothèque `crossbeam` offre des outils plus sophistiqués pour la concurrence :

``` rust
use crossbeam_channel as channel;
use std::thread;
use std::time::Duration;

fn main() {
    // Canal sans capacité limite
    let (s, r) = channel::unbounded();

    // Envoyer des messages depuis plusieurs threads producteurs
    for i in 0..5 {
        let s = s.clone();
        thread::spawn(move || {
            s.send(format!("Message du producteur {}", i)).unwrap();
        });
    }

    // Créer un deadlane après lequel on arrête d'attendre
    let timeout = Duration::from_secs(1);

    // Recevoir les messages pendant un certain temps
    while let Ok(msg) = channel::select! {
        recv(r) -> msg => {
            // Convertir le Result<String, RecvError> en Result<String, RecvTimeoutError>
            msg.map_err(|_| channel::RecvTimeoutError::Disconnected)
        },
        default(timeout) => Err(channel::RecvTimeoutError::Timeout),
    } {
        println!("Reçu: {}", msg);
    }

    println!("Temps écoulé ou tous les messages ont été reçus");
}
```

### Scoped threads avec `crossbeam`

Un problème courant en Rust est de référencer des variables locales dans des threads. `crossbeam` propose des threads à portée limitée qui sont automatiquement joints à la fin de leur portée :

``` rust
use crossbeam::thread;

fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    let mut results = vec![0; 5];

    // Tous les threads créés ici sont joints à la fin du bloc
    thread::scope(|s| {
        for (i, &num) in numbers.iter().enumerate() {
            // On peut emprunter des références aux variables locales
            s.spawn(move |_| {
                // Calcul coûteux
                let result = num * num;
                // On peut écrire directement dans results
                results[i] = result;
            });
        }
        // Tous les threads sont automatiquement joints ici
    }).unwrap();

    println!("Résultats: {:?}", results);
}
```

## Pools de threads

Pour les applications nécessitant beaucoup de tâches parallèles, il est souvent plus efficace de réutiliser les threads plutôt que d'en créer de nouveaux à chaque fois. Voici un exemple simple utilisant `threadpool` :

``` rust
use threadpool::ThreadPool;
use std::sync::mpsc::channel;

fn main() {
    let n_workers = 4;
    let n_jobs = 8;
    let pool = ThreadPool::new(n_workers);

    let (tx, rx) = channel();

    // Soumettre des tâches au pool
    for i in 0..n_jobs {
        let tx = tx.clone();
        pool.execute(move || {
            // Travail simulé
            let result = i * i;
            tx.send(result).expect("Échec de l'envoi du résultat");
        });
    }

    drop(tx);

    // Collecter les résultats
    let mut results = Vec::new();
    while let Ok(result) = rx.recv() {
        results.push(result);
    }

    // Attendre que toutes les tâches soient terminées
    pool.join();

    // Trier pour un affichage prévisible
    results.sort();
    println!("Résultats: {:?}", results);
}
```

## Rayon : parallélisme de données

La bibliothèque `rayon` simplifie considérablement le parallélisme en Rust en proposant des itérateurs parallèles :

``` rust
use rayon::prelude::*;

fn main() {
    let nombres = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // Calcul séquentiel
    let somme_seq: u64 = nombres.iter()
                                .map(|&x| {
                                    // Simuler un calcul coûteux
                                    let y = x * x;
                                    y as u64
                                })
                                .sum();

    // Version parallèle avec rayon
    let somme_par: u64 = nombres.par_iter()
                                .map(|&x| {
                                    // Le même calcul, mais exécuté en parallèle
                                    let y = x * x;
                                    y as u64
                                })
                                .sum();

    println!("Somme séquentielle: {}", somme_seq);
    println!("Somme parallèle: {}", somme_par);

    // Division-fusion parallèle avec rayon
    let resultat = nombres.par_iter()
                         .filter(|&&x| x % 2 == 0)  // Garder les nombres pairs
                         .map(|&x| x * x)           // Élever au carré
                         .reduce(|| 0, |a, b| a + b); // Réduire en somme

    println!("Somme des carrés des nombres pairs: {}", resultat);
}
```

## Conclusion

Le multi-threading en Rust combine sécurité des types et performance grâce à plusieurs mécanismes :

1.  Le système de propriété empêche les conditions de course au moment de la compilation
2.  Les `Mutex` et `Arc` permettent le partage sécurisé de données mutables
3.  Les channels facilitent la communication entre threads
4.  Les types atomiques offrent des opérations légères sans verrouillage
5.  Des bibliothèques comme `crossbeam` et `rayon` simplifient les patterns complexes

En suivant ces principes, vous pouvez développer des applications concurrentes robustes et performantes sans les bugs difficiles à reproduire qui affligent habituellement la programmation parallèle.
