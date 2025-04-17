## 19\. **Les attributs de compilation et optimisation** - Flags de compilation, optimisations spécifiques

L'optimisation des programmes Rust est un aspect crucial lorsqu'on vise des performances maximales. Dans cette section, nous allons explorer les différentes options de compilation, attributs et techniques d'optimisation disponibles pour les développeurs Rust.
## 19.1 Les niveaux d'optimisation de Cargo
Les commandes de base pour la compilation n'ont pas changé :
``` bash
# Compilation sans optimisation (développement)
cargo build

# Compilation avec optimisations (release)
cargo build --release
```
Les niveaux d'optimisation restent les mêmes avec Rust 2024 :

| Niveau | Flag rustc | Description |
| --- | --- | --- |
| Debug (par défaut) | `-C opt-level=0` | Compilation rapide, exécution lente, débogage facile |
| Release | `-C opt-level=3` | Compilation lente, exécution rapide, débogage difficile |
Voici notre exemple de code Fibonacci adapté pour Rust 2024 :
``` rust
fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

fn main() {
    let start = std::time::Instant::now();
    let result = fibonacci(40);
    let duration = start.elapsed();

    println!("Fibonacci(40) = {}", result);
    println!("Calculé en {:?}", duration);
}
```
## 19.2 Niveaux d'optimisation personnalisés
Le paramétrage des niveaux d'optimisation dans `Cargo.toml` reste similaire :
``` toml
[profile.dev]
opt-level = 1  # Optimisation légère en mode développement

[profile.release]
opt-level = 3  # Optimisation agressive en mode release
```
Les niveaux disponibles sont toujours :
- `0` : Aucune optimisation
- `1` : Optimisations de base
- `2` : Optimisations plus agressives
- `3` : Toutes les optimisations possibles
- `s` : Optimise la taille au détriment de la vitesse
- `z` : Optimise davantage la taille, même au détriment de la vitesse

## 19.3 Paramètres de profilage complémentaires
En 2024, le `Cargo.toml` prend en charge des options supplémentaires :
``` toml
[profile.release]
# Optimisations maximales pour la vitesse
opt-level = 3
# Activer les informations de débogage (utile pour le profiling)
debug = true
# Activer la vérification de débordement (overflow)
overflow-checks = true
# Activer Link Time Optimization (LTO)
lto = true
# Utiliser le codegen-unit minimal pour des optimisations plus agressives
codegen-units = 1
# Optimiser pour la CPU hôte
target-cpu = "native"
# Nouveau en 2024: contrôler la compilation incrémentale
incremental = false
```
Voici un exemple pratique mis à jour avec les dernières versions des dépendances :
``` rust
use sha2::{Sha256, Digest};
use std::time::Instant;

/// Fonction qui effectue un travail intensif (calcul de hash répété)
fn intensive_work(iterations: usize) -> String {
    let mut result = String::from("seed data for hashing");

    for _ in 0..iterations {
        let mut hasher = Sha256::new();
        hasher.update(result.as_bytes());
        result = format!("{:x}", hasher.finalize());
    }

    result
}

fn main() {
    // Mesurer les performances
    let iterations = 100_000;
    let start = Instant::now();
    let hash = intensive_work(iterations);
    let duration = start.elapsed();

    println!("Résultat final: {}...", &hash[0..16]);
    println!("Temps d'exécution: {:?} pour {} itérations", duration, iterations);
    println!("Temps moyen par itération: {:?}", duration / iterations as u32);
}
```
## 19.4 Attributs d'optimisation au niveau du code
### `#[inline]` et variantes
L'inlining reste un outil d'optimisation important :
``` rust
// Suggestion pour le compilateur d'incorporer cette fonction
#[inline]
fn add(a: i32, b: i32) -> i32 {
    a + b
}

// Force l'incorporation (sauf si impossible)
#[inline(always)]
fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

// Empêche l'incorporation
#[inline(never)]
fn divide(a: i32, b: i32) -> i32 {
    a / b
}
```
### `#[cold]` et `#[hot]`
Ces attributs améliorent les décisions du compilateur concernant l'optimisation des fonctions :
``` rust
// Fonction rarement appelée (comme la gestion d'erreurs)
#[cold]
fn handle_error(err: &str) {
    eprintln!("Erreur: {}", err);
    std::process::exit(1);
}

// Fonction fréquemment appelée (comme dans une boucle critique)
// Rust 2024 optimise encore mieux les fonctions marquées #[hot]
#[hot]
fn critical_calculation(data: &[f64]) -> f64 {
    data.iter().sum()
}
```
### `#[no_mangle]`
``` rust
#[no_mangle]
pub extern "C" fn rust_function_for_c() -> i32 {
    42
}
```
## 19.5 Optimisations spécifiques avec `target_feature`
En 2024, Rust a élargi la prise en charge des instructions SIMD :
``` rust
// Fonction qui utilise les instructions SIMD AVX2 (si disponibles)
#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "avx2")]
unsafe fn sum_avx2(data: &[f32]) -> f32 {
    use std::arch::x86_64::*;

    let mut sum_vec = _mm256_setzero_ps();
    let chunks = data.chunks_exact(8);

    // Utilisation de chunks_exact pour éviter la vérification de longueur à chaque itération
    for chunk in chunks {
        let chunk_vec = _mm256_loadu_ps(chunk.as_ptr());
        sum_vec = _mm256_add_ps(sum_vec, chunk_vec);
    }

    // Traitement du reste
    let remainder = chunks.remainder();
    if !remainder.is_empty() {
        // Gestion du reste (implémentation simplifiée ici)
        let mut remainder_sum: f32 = 0.0;
        for &value in remainder {
            remainder_sum += value;
        }

        // Ajouter le reste à la somme vectorielle
        let mut result_array: [f32; 8] = [0.0; 8];
        _mm256_storeu_ps(result_array.as_mut_ptr(), sum_vec);
        result_array[0] += remainder_sum;

        return result_array.iter().sum();
    }

    // Extraire la somme des éléments du vecteur
    let mut result_array: [f32; 8] = [0.0; 8];
    _mm256_storeu_ps(result_array.as_mut_ptr(), sum_vec);

    result_array.iter().sum()
}

// Version de secours pour les autres architectures ou si AVX2 n'est pas disponible
#[cfg(not(target_arch = "x86_64"))]
fn sum_avx2(data: &[f32]) -> f32 {
    data.iter().sum()
}
```
## 19.6 Link Time Optimization (LTO)
La configuration LTO dans `Cargo.toml` reste similaire, mais avec des optimisations améliorées en 2024 :
``` toml
[profile.release]
lto = true  # Activer LTO complet
# Ou
lto = "thin"  # Activer LLVM ThinLTO (plus rapide mais moins optimisé)
# Ou (nouveau en 2024)
lto = "fat"   # Optimisation la plus agressive, temps de compilation le plus long
```
## 19.7 Optimisation de la taille binaire
Les techniques pour réduire la taille des binaires sont améliorées en 2024 :
``` toml
[profile.release]
opt-level = "z"  # Optimiser pour la taille
lto = true
codegen-units = 1
panic = "abort"  # Pas de déroulement de pile en cas de panique
strip = true     # Supprimer les symboles de débogage
# Nouveau en 2024: contrôle plus fin de la génération de code
debug-assertions = false
overflow-checks = false
```
## 19.8 Flags de compilation avancés pour rustc
Cargo peut toujours transmettre des flags spécifiques à rustc, avec quelques nouvelles options en 2024 :
``` bash
RUSTFLAGS="-C target-cpu=native -C link-dead-code=no" cargo build --release
```
Flags utiles en 2024 :
- `-C target-cpu=native` : Optimise pour le CPU hôte
- `-C link-dead-code=no` : Élimine plus agressivement le code mort
- `-C debuginfo=2` : Inclut les informations de débogage (niveau 2)
- `-C link-arg=-fuse-ld=lld` : Utilise un éditeur de liens spécifique
- `-C opt-level=3 -C target-feature=+avx2,+fma,+sse4.2` : Activation ciblée d'instructions

Configuration dans `.cargo/config.toml` :
``` toml
[build]
rustflags = ["-C", "target-cpu=native", "-C", "link-dead-code=no"]
```
## 19.9 Optimisations spécifiques aux architectures
Détection de fonctionnalités à l'exécution avec les améliorations 2024 :
``` rust
use std::time::Instant;

// Utilisation du nouveau système de dispatch dynamique (2024)
#[cfg(any(target_arch = "x86_64", target_arch = "x86"))]
#[cfg(feature = "simd_support")]
#[inline]
fn get_fastest_implementation() -> fn(&[f32]) -> f32 {
    #[cfg(target_arch = "x86_64")]
    {
        if std::is_x86_feature_detected!("avx2") {
            return unsafe { sum_avx2 };
        } else if std::is_x86_feature_detected!("sse4.1") {
            return unsafe { sum_sse };
        }
    }

    sum_scalar
}

#[cfg(not(any(target_arch = "x86_64", target_arch = "x86")))]
#[inline]
fn get_fastest_implementation() -> fn(&[f32]) -> f32 {
    sum_scalar
}

// Version scalaire (compatible avec toutes les architectures)
fn sum_scalar(data: &[f32]) -> f32 {
    data.iter().sum()
}

#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "avx2")]
unsafe fn sum_avx2(data: &[f32]) -> f32 {
    // Implémentation AVX2 optimisée pour 2024
    let mut sum_vec = _mm256_setzero_ps();
    let chunks = data.chunks_exact(8);

    for chunk in chunks {
        let chunk_vec = _mm256_loadu_ps(chunk.as_ptr());
        sum_vec = _mm256_add_ps(sum_vec, chunk_vec);
    }

    // Extraire et additionner les éléments
    let mut result_array: [f32; 8] = [0.0; 8];
    _mm256_storeu_ps(result_array.as_mut_ptr(), sum_vec);

    let mut sum = 0.0;
    for val in result_array {
        sum += val;
    }

    // Traiter le reste
    for &val in chunks.remainder() {
        sum += val;
    }

    sum
}

#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "sse4.1")]
unsafe fn sum_sse(data: &[f32]) -> f32 {
    // Implémentation avec SSE4.1
    let mut sum_vec = _mm_setzero_ps();
    let chunks = data.chunks_exact(4);

    for chunk in chunks {
        let chunk_vec = _mm_loadu_ps(chunk.as_ptr());
        sum_vec = _mm_add_ps(sum_vec, chunk_vec);
    }

    // Extraire et additionner les éléments
    let mut result_array: [f32; 4] = [0.0; 4];
    _mm_storeu_ps(result_array.as_mut_ptr(), sum_vec);

    let mut sum = 0.0;
    for val in result_array {
        sum += val;
    }

    // Traiter le reste
    for &val in chunks.remainder() {
        sum += val;
    }

    sum
}

fn main() {
    // Génération de données de test
    let data: Vec<f32> = (0..1_000_000).map(|i| i as f32).collect();

    // Sélection de l'implémentation la plus rapide
    let sum_fn = get_fastest_implementation();

    // Test de performance
    let start = Instant::now();
    let sum = sum_fn(&data);
    let duration = start.elapsed();

    println!("Somme: {}", sum);
    println!("Temps d'exécution: {:?}", duration);
}
```
## 19.10 Déboguer les optimisations avec flags de compilation
Pour comprendre comment le compilateur optimise votre code en 2024 :
``` bash
# Afficher les optimisations LLVM appliquées
RUSTFLAGS="--emit=llvm-ir" cargo build --release

# Générer l'assembleur pour inspecter le code machine
RUSTFLAGS="--emit=asm" cargo build --release

# Nouveau en 2024: Analyser les décisions d'optimisation
RUSTFLAGS="-Z dump-mir=all" cargo build --release
```
## 19.11 Optimisation des tableaux et de la mémoire
En 2024, Rust dispose d'options supplémentaires pour l'optimisation de la mémoire :
``` rust
// Force l'alignement des structures en mémoire pour de meilleures performances
#[repr(align(64))]
struct AlignedData {
    buffer: [f32; 1024],
}

// Spécification du layout des structures
#[repr(C)]
struct ExternStruct {
    a: i32,
    b: i32,
}

// Nouveau en 2024: Contrôle plus fin de l'organisation mémoire
#[repr(packed)]
struct PackedData {
    a: u8,
    b: u32,
    c: u16,
}
```
## 19.12 Nouvelles optimisations spécifiques à Rust 2024
``` rust
// Utilisation de références brutes pour les optimisations de bas niveau
fn optimize_with_raw_refs(data: &mut [u32]) {
    let ptr: &raw [u32] = data;
    // Opérations optimisées sur ptr
}

// Fonctions constantes améliorées en 2024
const fn calculate_fibonacci(n: u32) -> u32 {
    match n {
        0 => 0,
        1 => 1,
        _ => calculate_fibonacci(n - 1) + calculate_fibonacci(n - 2),
    }
}

// Génération de table à la compilation
const FIBONACCI_TABLE: [u32; 20] = {
    let mut table = [0; 20];
    let mut i = 0;
    while i < 20 {
        table[i] = calculate_fibonacci(i as u32);
        i += 1;
    }
    table
};
```
## 19.13 Conclusion
L'optimisation en Rust 2024 Edition offre encore plus d'options qu'auparavant. La démarche à suivre reste similaire :
1. Commencer par écrire un code clair et correct
2. Identifier les goulots d'étranglement à l'aide d'outils de profiling
3. Appliquer des optimisations ciblées aux parties critiques
4. Vérifier les résultats par des benchmarks

Les nouvelles fonctionnalités de Rust 2024 permettent des optimisations encore plus poussées, mais rappelons-nous que le choix d'un meilleur algorithme reste souvent l'optimisation la plus importante.
