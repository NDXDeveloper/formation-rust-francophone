## 5\. Ajouter des tests

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Les tests sont une partie essentielle du développement logiciel, et Rust offre un support natif pour les tests unitaires, d'intégration et de documentation. Dans ce chapitre, nous allons explorer les différentes façons de tester votre code Rust et les outils fournis par l'écosystème.

### L'attribut `#[test]`

En Rust, la façon la plus simple de définir un test est d'annoter une fonction avec l'attribut `#[test]`. Lorsque vous exécutez `cargo test`, le framework de test de Rust recherche toutes les fonctions marquées avec cet attribut et les exécute.

Voici un exemple de base :

``` rust
fn somme(a: i32, b: i32) -> i32 {
    a + b
}

#[test]
fn test_somme() {
    assert_eq!(somme(2, 3), 5);
    assert_eq!(somme(-1, 1), 0);
    assert_eq!(somme(0, 0), 0);
}
```

Pour exécuter les tests, vous pouvez utiliser :

``` bash
cargo test
```

Si vous n'utilisez pas Cargo :

``` bash
rustc --test mon_fichier.rs
./mon_fichier
```

### Assertions de test

Rust fournit plusieurs macros pour effectuer des assertions dans vos tests :

``` rust
#[test]
fn test_assertions() {
    // Vérifie l'égalité
    assert_eq!(2 + 2, 4);

    // Vérifie l'inégalité
    assert_ne!(5, 10);

    // Assertion simple (le test réussit si l'expression est true)
    assert!(true);

    // Assertion avec message personnalisé
    assert!(10 > 5, "10 devrait être supérieur à 5");

    // Assertions d'égalité avec message personnalisé
    assert_eq!(4, 2 + 2, "Vérification de l'addition");
    assert_ne!(5, 10, "Ces valeurs ne devraient pas être égales");
}
```

### Organisation des tests

#### Tests unitaires dans le même fichier

Une pratique courante consiste à regrouper les tests unitaires dans un module dédié au sein du même fichier que le code testé :

``` rust
// Dans un fichier nommé calcul.rs
pub fn carré(nombre: i32) -> i32 {
    nombre * nombre
}

pub fn est_pair(nombre: i32) -> bool {
    nombre % 2 == 0
}

#[cfg(test)]
mod tests {
    use super::*; // Importe toutes les fonctions du module parent

    #[test]
    fn test_carré() {
        assert_eq!(carré(2), 4);
        assert_eq!(carré(3), 9);
        assert_eq!(carré(-4), 16);
    }

    #[test]
    fn test_est_pair() {
        assert!(est_pair(2));
        assert!(est_pair(0));
        assert!(!est_pair(3));
        assert!(!est_pair(-1));
    }
}
```

L'attribut `#[cfg(test)]` est important : il indique que ce module ne doit être compilé que lorsque l'on exécute les tests. Lors d'une compilation normale avec `cargo build`, ce module est ignoré, ce qui permet d'éviter d'inclure du code de test dans votre application finale.

#### Tests d'intégration dans un dossier séparé

Les tests unitaires vérifient que des fonctions individuelles fonctionnent correctement, mais les tests d'intégration vérifient que plusieurs parties de votre bibliothèque fonctionnent bien ensemble. En Rust, ces tests sont placés dans un dossier `tests` à la racine de votre projet.

Voici la structure typique d'un projet Rust avec des tests d'intégration :

```
mon_projet/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   └── ...
└── tests/
    ├── intégration_basique.rs
    ├── fonctionnalités_avancées.rs
    └── ...
```

Exemple de test d'intégration (`tests/intégration_basique.rs`) :

``` rust
// On importe la bibliothèque à tester
use ma_bibliothèque;

#[test]
fn test_intégration_calculs() {
    // Test de plusieurs fonctionnalités ensemble
    let résultat = ma_bibliothèque::calcul::carré(ma_bibliothèque::calcul::somme(3, 4));
    assert_eq!(résultat, 49); // (3 + 4)² = 49
}

#[test]
fn test_intégration_validation() {
    // Test de l'interaction entre plusieurs modules
    let entrée = "42";
    let nombre = ma_bibliothèque::conversion::str_to_i32(entrée).unwrap();
    assert!(ma_bibliothèque::calcul::est_pair(nombre));
}
```

### Gérer les paniques avec `#[should_panic]`

Dans certains cas, vous voulez vérifier qu'une fonction panique (lance une erreur) dans certaines conditions. Pour cela, utilisez l'attribut `#[should_panic]`.

``` rust
pub fn diviser(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Division par zéro!");
    }
    a / b
}

#[test]
#[should_panic(expected = "Division par zéro!")]
fn test_division_par_zero() {
    diviser(10, 0);
}
```

L'attribut `#[should_panic]` indique que la fonction de test doit paniquer pour réussir. L'argument `expected` est optionnel mais très utile : il vérifie que le message de panique contient la chaîne spécifiée, ce qui permet de s'assurer que la fonction panique pour la bonne raison.

Si vous exécutez `cargo test` avec ce code, le test `test_division_par_zero` réussira car la fonction `diviser` panique bien avec le message attendu.

### Tests renvoyant un `Result`

Une alternative à `assert!` et `#[should_panic]` consiste à écrire des tests qui renvoient un `Result<(), E>`. Cela permet d'utiliser l'opérateur `?` pour gérer les erreurs de manière plus élégante :

``` rust
pub fn inverser(nombre: f64) -> Result<f64, String> {
    if nombre == 0.0 {
        return Err("Impossible d'inverser zéro".to_string());
    }
    Ok(1.0 / nombre)
}

#[test]
fn test_inverser() -> Result<(), String> {
    let résultat = inverser(2.0)?;
    assert_eq!(résultat, 0.5);

    let erreur = inverser(0.0);
    assert!(erreur.is_err());

    Ok(())
}
```

L'avantage de cette approche est qu'elle permet d'écrire des tests plus expressifs et de mieux contrôler le flux d'exécution, en particulier lorsque vous testez des fonctions qui renvoient des `Result`.

### Organisation avancée des tests d'intégration

Pour les projets plus importants, vous pourriez vouloir organiser vos tests d'intégration en modules et sous-dossiers :

```
tests/
├── fonctionnalités/
│   ├── mod.rs
│   ├── basiques.rs
│   └── avancées.rs
├── performances/
│   ├── mod.rs
│   └── benchmarks.rs
└── intégration_principale.rs
```

Dans ce cas, vous devez déclarer les modules dans des fichiers `mod.rs` ou directement dans le fichier de test principal :

Dans `tests/fonctionnalités/mod.rs` :

``` rust
// Déclare les sous-modules
mod basiques;
mod avancées;
```

Dans `tests/intégration_principale.rs` :

``` rust
// Importe les modules de test
mod fonctionnalités;
mod performances;

// Vous pouvez aussi avoir des tests directement ici
#[test]
fn test_global() {
    // Test de haut niveau qui vérifie l'ensemble du système
}
```

### Ignorer des tests

Si vous avez un test qui est temporairement cassé ou qui prend trop de temps, vous pouvez le marquer avec `#[ignore]` :

``` rust
#[test]
#[ignore = "Ce test est trop lent pour être exécuté à chaque fois"]
fn test_performance_intensive() {
    // Test qui prend beaucoup de temps...
}
```

Pour exécuter uniquement les tests ignorés :

``` bash
cargo test -- --ignored
```

Pour exécuter tous les tests, y compris les tests ignorés :

``` bash
cargo test -- --include-ignored
```

### Filtrer les tests

Vous pouvez exécuter un sous-ensemble de tests en spécifiant une partie de leur nom :

``` bash
# Exécute tous les tests dont le nom contient "somme"
cargo test somme
```

### Les tests dans la documentation

Comme nous l'avons vu dans le chapitre précédent, les exemples de code dans la documentation sont également exécutés comme des tests lorsque vous lancez `cargo test`. C'est un excellent moyen de s'assurer que votre documentation reste à jour et que les exemples fonctionnent réellement.

``` rust
/// Calcule le carré d'un nombre.
///
/// # Exemples
///
/// ```
/// let résultat = ma_bibliothèque::carré(4);
/// assert_eq!(résultat, 16);
/// ```
pub fn carré(n: i32) -> i32 {
    n * n
}
```

#### Options pour les tests de documentation

Rust offre plusieurs options pour contrôler comment les exemples de code dans la documentation sont testés :

1.  **Test standard (par défaut)** :

``` rust
/// ```
/// let x = 2 + 2;
/// assert_eq!(x, 4);
/// ```
```

Ce code est compilé et exécuté lors des tests.

2.  **Spécifier le langage** :

``` rust
/// ```rust
/// let x = 2 + 2;
/// assert_eq!(x, 4);
/// ```
```

Identique au précédent. Si vous spécifiez un autre langage que Rust, le code ne sera pas testé.

3.  **Ignorer le test** :

``` rust
/// ```ignore
/// let x = fonctionnalité_non_implémentée();
/// ```
```

Le code ne sera pas compilé ni exécuté, mais il sera visible dans la documentation.

4.  **Compiler sans exécuter** :

``` rust
/// ```no_run
/// use std::fs::File;
/// let f = File::open("fichier.txt").unwrap();
/// ```
```

Le code sera compilé pour vérifier sa validité, mais il ne sera pas exécuté (utile pour les opérations d'E/S).

5.  **Vérifier que la compilation échoue** :

``` rust
/// ```compile_fail
/// let x: i32 = "pas un nombre"; // Ceci devrait échouer à la compilation
/// ```
```

Le test réussit si le code ne compile pas.

6.  **Permettre une exécution qui peut échouer** :

``` rust
/// ```allow_fail
/// assert_eq!(2 + 2, 5); // Cette assertion échouera, mais le test global réussira
/// ```
```

Le code doit compiler, mais l'exécution peut échouer sans faire échouer le test.

7.  **Combiner des options** :

``` rust
/// ```no_run,compile_fail
/// let x: i32 = "ceci ne compilera pas";
/// // Mais même si ça compilait, on ne l'exécuterait pas
/// ```
```

8.  **Tester avec le harness de test** :

``` rust
/// ```test_harness
/// #[test]
/// fn mon_test() {
///     assert!(2 + 2 == 4);
/// }
/// ```
```

Compile le code comme si l'option `--test` était passée au compilateur.

#### Masquer du code dans la documentation

Il est souvent utile de masquer certaines parties du code dans la documentation tout en les incluant dans les tests :

``` rust
/// Fonction qui retourne un résultat.
///
/// # Exemples
///
/// ```
/// # use std::fs::File;
/// # fn fonction_qui_peut_échouer() -> Result<(), std::io::Error> {
/// let fichier = File::open("config.txt")?;
/// # Ok(())
/// # }
/// ```
```

Dans cet exemple, les lignes commençant par `#` seront invisibles dans la documentation générée, mais seront incluses lors des tests. Cela permet de montrer du code concis aux utilisateurs tout en s'assurant que les tests sont complets et valides.

### Tests de performance (benchmarks)

Pour les tests de performance, l'attribut `#[bench]` était disponible dans Rust nightly, mais le projet a évolué et aujourd'hui, il est recommandé d'utiliser des crates externes comme `criterion` pour les benchmarks.

Voici un exemple avec `criterion` (ajoutez-le d'abord à votre `Cargo.toml`) :

``` rust
// benches/mes_benchmarks.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use ma_bibliothèque::fibonacci;

fn bench_fibonacci(c: &mut Criterion) {
    c.bench_function("fibonacci 20", |b| b.iter(|| fibonacci(black_box(20))));
}

criterion_group!(benches, bench_fibonacci);
criterion_main!(benches);
```

Dans votre `Cargo.toml` :

``` toml
[dev-dependencies]
criterion = "0.4"

[[bench]]
name = "mes_benchmarks"
harness = false
```

Puis exécutez-le avec :

``` bash
cargo bench
```

### Tester la couverture du code

Pour mesurer la couverture de vos tests (quelles parties du code sont exécutées pendant les tests), vous pouvez utiliser des outils comme `grcov` ou `tarpaulin`.

Exemple avec `tarpaulin` :

```
# Installation
cargo install cargo-tarpaulin

# Exécution
cargo tarpaulin
```

### Bonnes pratiques pour les tests

1.  **Écrivez des tests unitaires pour chaque fonction publique** - Assurez-vous que chaque comportement attendu est vérifié.

2.  **Utilisez des tests d'intégration pour les interactions entre composants** - Les tests unitaires ne suffisent pas toujours pour détecter les problèmes d'interaction.

3.  **Maintenez la documentation à jour avec des exemples testés** - Les exemples dans la documentation sont souvent le premier point de contact des utilisateurs avec votre code.

4.  **Utilisez des fixtures pour les tests qui partagent une configuration** - Créez des fonctions d'aide pour initialiser des données de test communes.

5.  **Testez les cas limites et les erreurs** - Ne testez pas seulement le "happy path", mais aussi les cas d'erreur et les valeurs extrêmes.

6.  **Ajoutez des tests de non-régression** - Lorsque vous corrigez un bug, ajoutez un test qui aurait détecté ce bug.

7.  **Gardez les tests rapides** - Les tests lents sont moins susceptibles d'être exécutés régulièrement.


### Exemple complet

Voici un exemple plus complet qui combine différentes techniques de test :

``` rust
// src/lib.rs
pub mod calcul {
    /// Calcule la factorielle d'un nombre.
    ///
    /// # Exemples
    ///
    /// ```
    /// assert_eq!(ma_bibliothèque::calcul::factorielle(0), 1);
    /// assert_eq!(ma_bibliothèque::calcul::factorielle(5), 120);
    /// ```
    ///
    /// # Panics
    ///
    /// Panique si le nombre est supérieur à 20 pour éviter un débordement.
    pub fn factorielle(n: u64) -> u64 {
        if n > 20 {
            panic!("Factorielle trop grande pour être calculée");
        }

        match n {
            0 | 1 => 1,
            _ => n * factorielle(n - 1)
        }
    }

    /// Calcule la suite de Fibonacci pour un indice donné.
    pub fn fibonacci(n: u64) -> u64 {
        match n {
            0 => 0,
            1 => 1,
            _ => fibonacci(n - 1) + fibonacci(n - 2)
        }
    }

    #[cfg(test)]
    mod tests {
        use super::*;

        #[test]
        fn test_factorielle_basique() {
            assert_eq!(factorielle(0), 1);
            assert_eq!(factorielle(1), 1);
            assert_eq!(factorielle(5), 120);
        }

        #[test]
        #[should_panic(expected = "Factorielle trop grande")]
        fn test_factorielle_panique() {
            factorielle(30);
        }

        #[test]
        fn test_fibonacci() {
            assert_eq!(fibonacci(0), 0);
            assert_eq!(fibonacci(1), 1);
            assert_eq!(fibonacci(10), 55);
        }

        #[test]
        #[ignore = "Test de performance à exécuter séparément"]
        fn test_fibonacci_grand_nombre() {
            // Ce test est ignoré par défaut car il est lent
            assert_eq!(fibonacci(40), 102334155);
        }
    }
}
```

Et un test d'intégration :

``` rust
// tests/intégration.rs
use ma_bibliothèque::calcul;

#[test]
fn test_combinaison_factorielle_fibonacci() {
    // Calcule la factorielle du 10ème nombre de Fibonacci
    let fib10 = calcul::fibonacci(10);
    assert_eq!(fib10, 55);

    let fact_fib = calcul::factorielle(fib10);
    // C'est un très grand nombre, mais juste pour l'exemple
    assert!(fact_fib > 0);
}
```

Les tests sont un élément essentiel du développement en Rust. Ils vous permettent de vérifier que votre code fonctionne comme prévu et continuera à fonctionner même lorsque vous apporterez des modifications. L'écosystème Rust offre de nombreux outils pour faciliter l'écriture et l'exécution de tests, ce qui contribue à la réputation de Rust en matière de fiabilité et de robustesse.

⏭️ [Rc et RefCell](/III-avance/06-rc-refcell.md)
