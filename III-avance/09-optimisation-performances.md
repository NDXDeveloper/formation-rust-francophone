## 9\. Optimisation des performances - Profiling, benchmarking avec Criterion

### Introduction

L'optimisation des performances est une étape cruciale du développement de logiciels performants en Rust. Cette section explore les techniques pour identifier les goulots d'étranglement dans vos programmes et mesurer précisément les améliorations apportées par vos optimisations.

Rust est conçu pour offrir des performances proches du C/C++ tout en garantissant la sécurité mémoire. Mais comme pour tout langage, il est important de savoir où et comment optimiser.

### Principes d'optimisation

Avant de plonger dans les outils, gardons à l'esprit quelques principes généraux d'optimisation :

1.  **Mesurer avant d'optimiser** : Les intuitions sur la performance sont souvent trompeuses.
2.  **Identifier les points chauds** : Concentrez vos efforts sur les 20% du code qui consomment 80% du temps d'exécution.
3.  **Comparer les alternatives** : Utilisez des benchmarks pour comparer plusieurs approches.
4.  **Maintenir la lisibilité du code** : Une optimisation qui rend le code incompréhensible est rarement justifiée.

### Profiling en Rust

Le profiling consiste à analyser l'exécution de votre programme pour identifier les parties consommant le plus de ressources.

#### Outils de profiling

Plusieurs outils sont disponibles pour le profiling de code Rust :

1.  **perf** (Linux) : Outil d'analyse de performance du noyau Linux
2.  **Instruments** (macOS) : Suite d'outils de profiling d'Apple
3.  **Valgrind** avec Callgrind/KCachegrind : Pour une analyse détaillée de l'utilisation CPU
4.  **flamegraph** : Visualisation hiérarchique du temps CPU
5.  **tracy** : Profiler graphique à faible surcharge
6.  **pprof** : Visualisation de profils de performance

#### Exemple avec perf et flamegraph

Commençons par installer les outils nécessaires :

``` bash
# Sur un système Debian/Ubuntu
sudo apt-get install linux-tools-common linux-tools-generic
cargo install flamegraph
```

Créons un programme Rust avec un point chaud évident :

``` rust
// src/main.rs
fn fonction_lente(n: u64) -> u64 {
    let mut somme = 0;
    for i in 0..n {
        somme += i;
        // Simulation de travail supplémentaire
        for _ in 0..100 {
            somme = somme.wrapping_add(1).wrapping_sub(1);
        }
    }
    somme
}

fn fonction_rapide(n: u64) -> u64 {
    // Formule de Gauss: somme des n premiers entiers = n*(n+1)/2
    n * (n + 1) / 2
}

fn main() {
    let n = 50_000;

    // Mesure du temps pour la méthode lente
    let debut = std::time::Instant::now();
    let resultat1 = fonction_lente(n);
    let duree_lente = debut.elapsed();

    // Mesure du temps pour la méthode rapide
    let debut = std::time::Instant::now();
    let resultat2 = fonction_rapide(n);
    let duree_rapide = debut.elapsed();

    println!("Méthode lente: {} en {:?}", resultat1, duree_lente);
    println!("Méthode rapide: {} en {:?}", resultat2, duree_rapide);
    println!("Rapport de performance: {:.2}x",
        duree_lente.as_secs_f64() / duree_rapide.as_secs_f64());
}
```

Pour profiler ce programme avec flamegraph :

``` bash
cargo flamegraph
```

Cela génère un fichier SVG visualisant la répartition du temps CPU dans notre application. Les sections les plus larges du graphique correspondent aux fonctions consommant le plus de CPU.

#### Analyse de l'allocation mémoire

Pour analyser les allocations mémoire, vous pouvez utiliser DHAT (via Valgrind) ou jemallocator avec le profiling activé :

``` rust
// Cargo.toml
[dependencies]
jemallocator = { version = "0.5", features = ["profiling"] }

// main.rs
#[global_allocator]
static ALLOC: jemallocator::Jemalloc = jemallocator::Jemalloc;

fn main() {
    // Votre code ici

    // Écrire les statistiques d'allocation dans un fichier
    unsafe {
        jemallocator::stats::dump_stats("jeprof.out").unwrap();
    }
}
```

### Benchmarking avec Criterion

Criterion est une crate qui permet de réaliser des benchmarks précis et statistiquement significatifs. Contrairement au benchmarking intégré à Rust (qui est encore instable), Criterion est stable et facile à utiliser.

#### Configuration initiale

Commençons par configurer Criterion dans notre projet :

``` toml
# Cargo.toml
[dev-dependencies]
criterion = "0.5"

[[bench]]
name = "mon_benchmark"
harness = false
```

Ensuite, créons un fichier de benchmark dans `benches/mon_benchmark.rs` :

``` rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

// Fonction à benchmarker - Tri à bulles
fn tri_bulles(mut liste: Vec<i32>) -> Vec<i32> {
    let n = liste.len();
    for i in 0..n {
        for j in 0..(n - i - 1) {
            if liste[j] > liste[j + 1] {
                liste.swap(j, j + 1);
            }
        }
    }
    liste
}

// Fonction à comparer - Tri rapide
fn tri_rapide(mut liste: Vec<i32>) -> Vec<i32> {
    if liste.len() <= 1 {
        return liste;
    }

    let pivot = liste.remove(0);
    let mut plus_petits = Vec::new();
    let mut plus_grands = Vec::new();

    for x in liste {
        if x <= pivot {
            plus_petits.push(x);
        } else {
            plus_grands.push(x);
        }
    }

    let mut resultat = tri_rapide(plus_petits);
    resultat.push(pivot);
    resultat.extend(tri_rapide(plus_grands));
    resultat
}

fn benchmark_tris(c: &mut Criterion) {
    let mut groupe = c.benchmark_group("Algorithmes de tri");

    // Génération de données aléatoires
    let donnees: Vec<i32> = (0..1000).map(|_| rand::random::<i32>()).collect();

    // Benchmark du tri à bulles
    groupe.bench_function("Tri à bulles", |b| {
        b.iter(|| tri_bulles(black_box(donnees.clone())))
    });

    // Benchmark du tri rapide
    groupe.bench_function("Tri rapide", |b| {
        b.iter(|| tri_rapide(black_box(donnees.clone())))
    });

    groupe.finish();
}

criterion_group!(benches, benchmark_tris);
criterion_main!(benches);
```

Ajoutons la dépendance rand pour générer des nombres aléatoires :

``` toml
[dev-dependencies]
criterion = "0.5"
rand = "0.8"
```

Pour exécuter les benchmarks :

``` bash
cargo bench
```

#### Interprétation des résultats

Criterion génère un rapport HTML détaillé dans `target/criterion/` avec :

- Temps d'exécution moyens
- Écarts-types et intervalles de confiance
- Graphiques de comparaison
- Détection automatique de régression de performance

#### Paramètres avancés de Criterion

Criterion offre de nombreuses options pour personnaliser vos benchmarks :

``` rust
fn benchmark_parametrique(c: &mut Criterion) {
    let mut groupe = c.benchmark_group("Taille variable");

    // Tester avec différentes tailles d'entrée
    for taille in [10, 100, 1000, 10000].iter() {
        groupe.bench_with_input(
            format!("Tri à bulles {}", taille),
            taille,
            |b, &taille| {
                // Génération des données avec la taille spécifiée
                let donnees: Vec<i32> = (0..taille).map(|_| rand::random::<i32>()).collect();
                b.iter(|| tri_bulles(black_box(donnees.clone())))
            }
        );
    }

    groupe.finish();
}
```

### Micro-optimisations en Rust

Voyons quelques techniques d'optimisation spécifiques à Rust :

#### 1\. Choix des types appropriés

``` rust
fn benchmark_types(c: &mut Criterion) {
    let mut groupe = c.benchmark_group("Types");

    // Benchmark avec Vec<i32>
    groupe.bench_function("Vec<i32>", |b| {
        b.iter(|| {
            let mut v = Vec::new();
            for i in 0..1000 {
                v.push(i);
            }
            black_box(v)
        })
    });

    // Benchmark avec Vec<i32> préalloué
    groupe.bench_function("Vec<i32> préalloué", |b| {
        b.iter(|| {
            let mut v = Vec::with_capacity(1000);
            for i in 0..1000 {
                v.push(i);
            }
            black_box(v)
        })
    });

    // Benchmark avec [i32; 1000]
    groupe.bench_function("Array [i32; 1000]", |b| {
        b.iter(|| {
            let mut arr = [0i32; 1000];
            for i in 0..1000 {
                arr[i] = i;
            }
            black_box(arr)
        })
    });

    groupe.finish();
}
```

#### 2\. Optimisation des itérateurs

``` rust
fn somme_avec_for(v: &[i32]) -> i32 {
    let mut somme = 0;
    for x in v {
        somme += x;
    }
    somme
}

fn somme_avec_iter(v: &[i32]) -> i32 {
    v.iter().sum()
}

fn somme_avec_iter_fold(v: &[i32]) -> i32 {
    v.iter().fold(0, |acc, x| acc + x)
}

fn benchmark_iterations(c: &mut Criterion) {
    let donnees: Vec<i32> = (0..10000).collect();

    let mut groupe = c.benchmark_group("Techniques d'itération");

    groupe.bench_function("For loop", |b| {
        b.iter(|| somme_avec_for(black_box(&donnees)))
    });

    groupe.bench_function("Iterator sum", |b| {
        b.iter(|| somme_avec_iter(black_box(&donnees)))
    });

    groupe.bench_function("Iterator fold", |b| {
        b.iter(|| somme_avec_iter_fold(black_box(&donnees)))
    });

    groupe.finish();
}
```

#### 3\. Caches et réutilisation des calculs

Prenons l'exemple d'un algorithme de calcul du nombre de Fibonacci :

``` rust
// Version naïve - calcul récursif
fn fibonacci_naif(n: u64) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci_naif(n - 1) + fibonacci_naif(n - 2),
    }
}

// Version optimisée - mémoïsation avec table de hachage
use std::collections::HashMap;

fn fibonacci_memo(n: u64, cache: &mut HashMap<u64, u64>) -> u64 {
    if let Some(&result) = cache.get(&n) {
        return result;
    }

    let result = match n {
        0 => 0,
        1 => 1,
        _ => fibonacci_memo(n - 1, cache) + fibonacci_memo(n - 2, cache),
    };

    cache.insert(n, result);
    result
}

// Version itérative - approche ascendante
fn fibonacci_iteratif(n: u64) -> u64 {
    if n <= 1 {
        return n;
    }

    let mut a = 0;
    let mut b = 1;

    for _ in 2..=n {
        let temp = a + b;
        a = b;
        b = temp;
    }

    b
}

fn benchmark_fibonacci(c: &mut Criterion) {
    let mut groupe = c.benchmark_group("Fibonacci");

    // On limite à un petit n pour la version naïve qui est exponentielle
    groupe.bench_function("Naïf (n=20)", |b| {
        b.iter(|| fibonacci_naif(black_box(20)))
    });

    groupe.bench_function("Mémoïsation (n=50)", |b| {
        b.iter(|| {
            let mut cache = HashMap::new();
            fibonacci_memo(black_box(50), &mut cache)
        })
    });

    groupe.bench_function("Itératif (n=50)", |b| {
        b.iter(|| fibonacci_iteratif(black_box(50)))
    });

    groupe.finish();
}
```

### Optimisation des fonctions hachage

Illustrons un cas pratique avec l'utilisation de différentes fonctions de hachage, en utilisant les crates mentionnées dans vos dépendances :

``` rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use md5::Md5;
use sha1::Sha1;
use sha2::{Sha256, Sha512, Digest};

fn hash_md5(donnees: &[u8]) -> [u8; 16] {
    let mut hasher = Md5::new();
    hasher.update(donnees);
    hasher.finalize().into()
}

fn hash_sha1(donnees: &[u8]) -> [u8; 20] {
    let mut hasher = Sha1::new();
    hasher.update(donnees);
    hasher.finalize().into()
}

fn hash_sha256(donnees: &[u8]) -> [u8; 32] {
    let mut hasher = Sha256::new();
    hasher.update(donnees);
    hasher.finalize().into()
}

fn hash_sha512(donnees: &[u8]) -> [u8; 64] {
    let mut hasher = Sha512::new();
    hasher.update(donnees);
    hasher.finalize().into()
}

fn hash_crc32(donnees: &[u8]) -> u32 {
    crc::Crc::<u32>::new(&crc::CRC_32_ISO_HDLC).checksum(donnees)
}

fn benchmark_hash(c: &mut Criterion) {
    // Générer une chaîne de 1KB pour le test
    let test_data = vec![0u8; 1024];

    let mut groupe = c.benchmark_group("Fonctions de hachage");

    groupe.bench_function("MD5", |b| {
        b.iter(|| hash_md5(black_box(&test_data)))
    });

    groupe.bench_function("SHA-1", |b| {
        b.iter(|| hash_sha1(black_box(&test_data)))
    });

    groupe.bench_function("SHA-256", |b| {
        b.iter(|| hash_sha256(black_box(&test_data)))
    });

    groupe.bench_function("SHA-512", |b| {
        b.iter(|| hash_sha512(black_box(&test_data)))
    });

    groupe.bench_function("CRC32", |b| {
        b.iter(|| hash_crc32(black_box(&test_data)))
    });

    groupe.finish();
}

criterion_group!(benches, benchmark_hash);
criterion_main!(benches);
```

### Optimisation des allocations mémoire

Une source commune de problèmes de performance en Rust est l'allocation mémoire excessive :

``` rust
fn benchmark_allocations(c: &mut Criterion) {
    let mut groupe = c.benchmark_group("Allocations");

    // Concaténation de chaînes avec allocation à chaque itération
    groupe.bench_function("String += (inefficace)", |b| {
        b.iter(|| {
            let mut resultat = String::new();
            for i in 0..1000 {
                resultat += &i.to_string();
            }
            black_box(resultat)
        })
    });

    // Utilisation de String::with_capacity pour préallouer
    groupe.bench_function("String::with_capacity", |b| {
        b.iter(|| {
            let mut resultat = String::with_capacity(4000); // Estimation approximative
            for i in 0..1000 {
                resultat += &i.to_string();
            }
            black_box(resultat)
        })
    });

    // Utilisation de String::push_str pour éviter les allocations temporaires
    groupe.bench_function("push_str", |b| {
        b.iter(|| {
            let mut resultat = String::with_capacity(4000);
            for i in 0..1000 {
                resultat.push_str(&i.to_string());
            }
            black_box(resultat)
        })
    });

    // Utilisation d'un StringBuilder (format!)
    groupe.bench_function("format! macro", |b| {
        b.iter(|| {
            let mut morceaux = Vec::with_capacity(1000);
            for i in 0..1000 {
                morceaux.push(i.to_string());
            }
            let resultat = format!("{}", morceaux.join(""));
            black_box(resultat)
        })
    });

    groupe.finish();
}
```

### Optimisation multithread avec crossbeam-channel

La crate `crossbeam-channel` mentionnée dans vos dépendances est excellente pour paralléliser les calculs. Voici un exemple de benchmark :

``` rust
use crossbeam_channel::{bounded, unbounded};
use std::thread;

fn traitement_sequentiel(donnees: &[i32]) -> Vec<i32> {
    donnees.iter().map(|x| x * x).collect()
}

fn traitement_parallele(donnees: &[i32], nb_threads: usize) -> Vec<i32> {
    let (sender, receiver) = unbounded();
    let (result_sender, result_receiver) = bounded(donnees.len());

    // Diviser les données en morceaux
    let taille_chunk = (donnees.len() + nb_threads - 1) / nb_threads;

    // Lancer les threads de travail
    for _ in 0..nb_threads {
        let sender_clone = sender.clone();
        let result_sender_clone = result_sender.clone();

        thread::spawn(move || {
            while let Ok((index, valeur)) = sender_clone.recv() {
                let resultat = valeur * valeur;
                result_sender_clone.send((index, resultat)).unwrap();
            }
        });
    }

    // Envoyer les données aux threads
    for (i, &valeur) in donnees.iter().enumerate() {
        sender.send((i, valeur)).unwrap();
    }

    // Fermer le canal pour indiquer qu'il n'y a plus de données
    drop(sender);
    drop(result_sender);

    // Récupérer les résultats
    let mut resultats = vec![0; donnees.len()];
    while let Ok((index, resultat)) = result_receiver.recv() {
        resultats[index] = resultat;
    }

    resultats
}

fn benchmark_parallelisme(c: &mut Criterion) {
    // Générer un grand tableau de données
    let donnees: Vec<i32> = (0..100_000).collect();

    let mut groupe = c.benchmark_group("Parallélisme");

    groupe.bench_function("Séquentiel", |b| {
        b.iter(|| traitement_sequentiel(black_box(&donnees)))
    });

    let nb_threads = num_cpus::get();
    groupe.bench_function(format!("Parallèle ({} threads)", nb_threads), |b| {
        b.iter(|| traitement_parallele(black_box(&donnees), nb_threads))
    });

    groupe.finish();
}
```

### Conseils pour l'optimisation

1.  **Compiler en mode release** : Les optimisations du compilateur font souvent une énorme différence.

``` bash
cargo run --release
cargo bench  # Les benchmarks sont déjà en mode release
```

2.  **Utiliser les optimisations à la compilation** : Dans `Cargo.toml` :

``` toml
[profile.release]
opt-level = 3
lto = "fat"  # Link-Time Optimization
codegen-units = 1
panic = "abort"  # Simplifier le code de gestion des panics
```

3.  **Analyser les allocations** : Souvent, réduire les allocations améliore grandement les performances.

4.  **Paralléliser intelligemment** : Utilisez `rayon` ou `crossbeam` pour les tâches parallélisables.

5.  **Mesurer, optimiser, mesurer à nouveau** : Ne supposez jamais qu'une modification améliore les performances sans le vérifier.


### Conclusion

L'optimisation des performances en Rust nécessite une compréhension des outils de profiling et de benchmarking, ainsi que des spécificités du langage. Criterion offre un moyen robuste de mesurer les performances et de comparer différentes approches.

N'oubliez pas que Rust est déjà optimisé par défaut grâce à son modèle d'ownership, son absence de garbage collector et ses abstractions à coût zéro. Souvent, l'optimisation la plus efficace consiste simplement à utiliser les bonnes abstractions que le langage et son écosystème offrent déjà.

Comme le dit Donald Knuth : "L'optimisation prématurée est la racine de tous les maux". Concentrez-vous d'abord sur l'exactitude et la clarté du code, puis optimisez les parties critiques identifiées par le profiling.

&nbsp;
