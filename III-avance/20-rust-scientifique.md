## 20. **Rust pour le domaine scientifique** - Calcul numérique, interfaçage avec Python, etc.

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Rust se positionne de plus en plus comme une alternative performante et fiable pour le calcul scientifique. Bien que l'écosystème scientifique de Rust soit encore jeune par rapport à Python ou R, il évolue rapidement et offre des avantages considérables en termes de performances et de sécurité. Dans cette section, nous explorerons les outils et techniques actualisés pour utiliser Rust dans le domaine scientifique.
### 20.1 Les bibliothèques numériques en Rust
#### 20.1.1 ndarray - Manipulation de tableaux multidimensionnels
`ndarray` est l'équivalent Rust de NumPy (Python). Il permet de manipuler des tableaux multidimensionnels avec une syntaxe expressive.
``` rust
use ndarray::{arr1, arr2, Array1, Array2};

fn main() {
    // Création d'un vecteur (tableau 1D)
    let vecteur: Array1<f64> = arr1(&[1.0, 2.0, 3.0, 4.0, 5.0]);
    println!("Vecteur: {}", vecteur);

    // Création d'une matrice 2×3 (tableau 2D)
    let matrice: Array2<f64> = arr2(&[[1.0, 2.0, 3.0],
                                     [4.0, 5.0, 6.0]]);
    println!("Matrice: {}", matrice);

    // Opérations vectorielles
    let vecteur_double = &vecteur * 2.0;
    println!("Vecteur × 2: {}", vecteur_double);

    // Opérations matricielles
    let transposee = matrice.t();
    println!("Matrice transposée: {}", transposee);

    // Accès aux éléments
    println!("Élément matrice[1,2]: {}", matrice[[1, 2]]);

    // Tranche (slice) d'un tableau
    let sous_vecteur = vecteur.slice(ndarray::s![1..4]);
    println!("Sous-vecteur: {}", sous_vecteur);
}
```
Pour utiliser `ndarray`, ajoutez cette dépendance à votre `Cargo.toml` :
``` toml
[dependencies]
ndarray = "0.15.6"
```
#### 20.1.2 nalgebra - Algèbre linéaire
`nalgebra` est une bibliothèque d'algèbre linéaire performante et typée qui offre des fonctionnalités avancées pour les matrices, les vecteurs et les transformations.
``` rust
use nalgebra::{Matrix3, Vector3};

fn main() {
    // Création d'une matrice 3×3
    let matrice = Matrix3::new(
        1.0, 2.0, 3.0,
        4.0, 5.0, 6.0,
        7.0, 8.0, 9.0
    );

    // Création d'un vecteur 3D
    let vecteur = Vector3::new(1.0, 2.0, 3.0);

    // Multiplication matrice-vecteur
    let resultat = matrice * vecteur;
    println!("Résultat: {}", resultat);

    // Calcul du déterminant
    println!("Déterminant: {}", matrice.determinant());

    // Inversion de matrice
    match matrice.try_inverse() {
        Some(inverse) => println!("Inverse: {}", inverse),
        None => println!("La matrice n'est pas inversible"),
    }

    // Décomposition en valeurs propres
    if let Some(eigen) = matrice.symmetric_eigen() {
        println!("Valeurs propres: {}", eigen.eigenvalues);
        println!("Vecteurs propres: {}", eigen.eigenvectors);
    }
}
```
Pour utiliser `nalgebra`, ajoutez à votre `Cargo.toml` :
``` toml
[dependencies]
nalgebra = "0.32.3"
```
#### 20.1.3 statrs - Statistiques et distributions
`statrs` fournit des fonctionnalités statistiques comme les distributions de probabilité, les tests statistiques, etc.
``` rust
use statrs::distribution::{Normal, Continuous, Discrete, Binomial};
use statrs::statistics::Statistics;

fn main() {
    // Distribution normale
    let normale = Normal::new(0.0, 1.0).unwrap();

    // PDF (Fonction de densité de probabilité)
    println!("PDF à x=0: {}", normale.pdf(0.0));

    // CDF (Fonction de répartition)
    println!("CDF à x=1.96: {}", normale.cdf(1.96));

    // Intervalle de confiance à 95%
    println!("Quantile à 0.975: {}", normale.inverse_cdf(0.975));

    // Distribution binomiale (10 essais, probabilité 0.5)
    let binomiale = Binomial::new(0.5, 10).unwrap();
    println!("P(X=5) pour Bin(10,0.5): {}", binomiale.pmf(5));

    // Statistiques descriptives
    let donnees = vec![1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0];
    println!("Moyenne: {}", donnees.mean());
    println!("Variance: {}", donnees.variance());
    println!("Écart-type: {}", donnees.std_dev());
    println!("Médiane: {}", donnees.median());
}
```
Pour utiliser `statrs`, ajoutez :
``` toml
[dependencies]
statrs = "0.16.0"
```
### 20.2 Calcul scientifique avec Rust
#### 20.2.1 Création d'un modèle de régression linéaire
Implémentons une régression linéaire simple avec la méthode des moindres carrés :
``` rust
use nalgebra::{DMatrix, DVector};

struct RegressionLineaire {
    coefficients: DVector<f64>,
    intercept: f64,
}

impl RegressionLineaire {
    fn entrainer(&mut self, x: &DMatrix<f64>, y: &DVector<f64>) {
        // Ajouter une colonne de 1 pour l'intercept
        let mut x_augmente = DMatrix::zeros(x.nrows(), x.ncols() + 1);

        // Remplir la première colonne avec des 1 (pour l'intercept)
        for i in 0..x.nrows() {
            x_augmente[(i, 0)] = 1.0;
        }

        // Copier les autres colonnes
        for i in 0..x.nrows() {
            for j in 0..x.ncols() {
                x_augmente[(i, j + 1)] = x[(i, j)];
            }
        }

        // Calculer (X^T * X)^-1 * X^T * y
        let x_t = x_augmente.transpose();
        let x_t_x = &x_t * &x_augmente;

        if let Some(x_t_x_inv) = x_t_x.try_inverse() {
            let x_t_y = &x_t * y;
            let coeffs = &x_t_x_inv * x_t_y;

            self.intercept = coeffs[0];
            self.coefficients = coeffs.rows(1, coeffs.nrows() - 1).into_owned();
        } else {
            println!("Erreur: la matrice X^T*X n'est pas inversible");
        }
    }

    fn predire(&self, x: &DMatrix<f64>) -> DVector<f64> {
        x * &self.coefficients + DVector::repeat(x.nrows(), self.intercept)
    }

    fn nouveau(nb_variables: usize) -> Self {
        RegressionLineaire {
            coefficients: DVector::zeros(nb_variables),
            intercept: 0.0,
        }
    }
}

fn main() {
    // Données d'entraînement (x1, x2, y)
    // Exemple: y = 2*x1 + 3*x2 + 5 + bruit
    let x = DMatrix::from_row_slice(10, 2, &[
        1.0, 2.0,
        2.0, 3.0,
        3.0, 4.0,
        4.0, 5.0,
        5.0, 6.0,
        6.0, 7.0,
        7.0, 8.0,
        8.0, 9.0,
        9.0, 10.0,
        10.0, 11.0,
    ]);

    let y = DVector::from_vec(vec![
        12.1, 17.8, 23.2, 28.7, 34.5,
        39.9, 45.3, 51.0, 56.8, 62.5
    ]);

    // Créer et entraîner le modèle
    let mut modele = RegressionLineaire::nouveau(2);
    modele.entrainer(&x, &y);

    println!("Intercept: {}", modele.intercept);
    println!("Coefficients: {}", modele.coefficients);

    // Données de test
    let x_test = DMatrix::from_row_slice(3, 2, &[
        2.5, 3.5,
        5.5, 6.5,
        8.5, 9.5,
    ]);

    // Prédictions
    let predictions = modele.predire(&x_test);
    println!("Prédictions: {}", predictions);
}
```
#### 20.2.2 Résolution d'équations différentielles
Implémentons une méthode simple pour résoudre une équation différentielle ordinaire (ODE) : la méthode d'Euler.
``` rust
use plotters::prelude::*;
use rayon::prelude::*;

// Type représentant une fonction différentielle dy/dt = f(t, y)
type DiffFn = fn(f64, f64) -> f64;

// Implémentation de la méthode d'Euler pour résoudre une ODE
fn euler(f: DiffFn, y0: f64, t0: f64, t_max: f64, pas: f64) -> Vec<(f64, f64)> {
    let mut resultats = Vec::new();
    let mut t = t0;
    let mut y = y0;

    resultats.push((t, y));

    while t < t_max {
        let dy = f(t, y) * pas;
        y += dy;
        t += pas;
        resultats.push((t, y));
    }

    resultats
}

// Version parallélisée pour les systèmes d'EDO
fn euler_systeme_parallel(
    f: impl Fn(f64, &[f64]) -> Vec<f64> + Sync,
    y0: Vec<f64>,
    t0: f64,
    t_max: f64,
    pas: f64
) -> Vec<(f64, Vec<f64>)> {
    let nb_steps = ((t_max - t0) / pas).ceil() as usize + 1;
    let mut resultats = Vec::with_capacity(nb_steps);

    resultats.push((t0, y0.clone()));

    let mut t = t0;
    let mut y = y0;

    while t < t_max {
        // Calcul parallèle des dérivées
        let derivees = f(t, &y);

        // Mise à jour parallèle des variables d'état
        y = y.par_iter()
             .zip(derivees.par_iter())
             .map(|(yi, dyi)| yi + dyi * pas)
             .collect();

        t += pas;
        resultats.push((t, y.clone()));
    }

    resultats
}

// Exemple d'ODE: dy/dt = y (croissance exponentielle)
fn croissance_exponentielle(t: f64, y: f64) -> f64 {
    y // dy/dt = y => y(t) = y0*e^t
}

// Exemple d'ODE: oscillateur harmonique: d²y/dt² + y = 0 => dy/dt = v, dv/dt = -y
fn oscillateur_harmonique(t: f64, y: f64) -> f64 {
    -y // Pour simplifier, on résout juste dv/dt = -y
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Résoudre l'équation de croissance exponentielle
    let solution_exp = euler(croissance_exponentielle, 1.0, 0.0, 5.0, 0.1);

    // Résoudre l'oscillateur harmonique
    let solution_osc = euler(oscillateur_harmonique, 0.0, 0.0, 20.0, 0.1);

    // Tracer les résultats (avec la crate plotters)
    let root = BitMapBackend::new("euler_solution.png", (800, 600)).into_drawing_area();
    root.fill(&WHITE)?;

    let mut chart = ChartBuilder::on(&root)
        .caption("Solutions d'équations différentielles", ("sans-serif", 30))
        .margin(5)
        .x_label_area_size(30)
        .y_label_area_size(30)
        .build_cartesian_2d(0.0..5.0, 0.0..150.0)?;

    chart.configure_mesh().draw()?;

    // Tracer la solution numérique de l'équation exponentielle
    chart.draw_series(LineSeries::new(
        solution_exp.iter().map(|&(t, y)| (t, y)),
        &RED,
    ))?
    .label("Croissance exponentielle")
    .legend(|(x, y)| PathElement::new(vec![(x, y), (x + 20, y)], &RED));

    // Tracer la solution analytique pour comparaison
    chart.draw_series(LineSeries::new(
        (0..50).map(|i| {
            let t = i as f64 * 0.1;
            (t, (t).exp())
        }),
        &BLUE,
    ))?
    .label("Solution exacte: e^t")
    .legend(|(x, y)| PathElement::new(vec![(x, y), (x + 20, y)], &BLUE));

    chart.configure_series_labels().background_style(&WHITE.mix(0.8)).draw()?;

    println!("Graphique généré : euler_solution.png");

    // Exemple de résolution d'un système d'EDO (pendule simple)
    // d²θ/dt² + g/l sin(θ) = 0
    // On pose y[0] = θ et y[1] = dθ/dt
    let pendule = |_t: f64, y: &[f64]| {
        let g_sur_l = 9.81; // g/l pour l=1m
        vec![
            y[1],                // dθ/dt = ω
            -g_sur_l * y[0].sin() // dω/dt = -g/l * sin(θ)
        ]
    };

    let solution_pendule = euler_systeme_parallel(
        pendule,
        vec![0.5, 0.0], // θ initial = 0.5 rad, vitesse initiale = 0
        0.0,
        10.0,
        0.01
    );

    // Tracer la solution du pendule
    let root2 = BitMapBackend::new("pendule_solution.png", (800, 600)).into_drawing_area();
    root2.fill(&WHITE)?;

    let mut chart2 = ChartBuilder::on(&root2)
        .caption("Mouvement du pendule simple", ("sans-serif", 30))
        .margin(5)
        .x_label_area_size(30)
        .y_label_area_size(30)
        .build_cartesian_2d(0.0..10.0, -0.6..0.6)?;

    chart2.configure_mesh().draw()?;

    // Tracer l'angle θ en fonction du temps
    chart2.draw_series(LineSeries::new(
        solution_pendule.iter().map(|(t, y)| (*t, y[0])),
        &RED,
    ))?
    .label("Angle θ(t)")
    .legend(|(x, y)| PathElement::new(vec![(x, y), (x + 20, y)], &RED));

    // Tracer la vitesse angulaire en fonction du temps
    chart2.draw_series(LineSeries::new(
        solution_pendule.iter().map(|(t, y)| (*t, y[1])),
        &BLUE,
    ))?
    .label("Vitesse angulaire ω(t)")
    .legend(|(x, y)| PathElement::new(vec![(x, y), (x + 20, y)], &BLUE));

    chart2.configure_series_labels().background_style(&WHITE.mix(0.8)).draw()?;

    println!("Graphique du pendule généré : pendule_solution.png");

    Ok(())
}
```
Pour exécuter cet exemple, ajoutez à votre `Cargo.toml` :
``` toml
[dependencies]
plotters = "0.3.5"
rayon = "1.10.0"
```
### 20.3 Interfaçage avec Python
Un des grands avantages de Rust pour le calcul scientifique est sa capacité à s'intégrer avec l'écosystème Python existant.
#### 20.3.1 PyO3 - Créer des extensions Python en Rust
PyO3 est une bibliothèque qui permet de créer des modules Python en Rust, offrant ainsi des performances supérieures pour les parties critiques d'une application scientifique.
Exemple d'une extension Python en Rust:
``` rust
use pyo3::prelude::*;
use pyo3::wrap_pyfunction;
use ndarray::{Array1, Array2};
use numpy::{PyArray1, PyArray2, PyReadonlyArray1, PyReadonlyArray2};

/// Calcul rapide d'une corrélation entre deux séries
#[pyfunction]
fn correlation(
    x: PyReadonlyArray1<f64>,
    y: PyReadonlyArray1<f64>
) -> PyResult<f64> {
    // Convertir les entrées en tableaux ndarray
    let x_array = x.as_array();
    let y_array = y.as_array();

    // Vérifier que les dimensions correspondent
    if x_array.len() != y_array.len() {
        return Err(PyErr::new::<pyo3::exceptions::PyValueError, _>(
            "Les tableaux doivent avoir la même longueur"
        ));
    }

    let n = x_array.len() as f64;

    // Calculer les moyennes
    let mean_x = x_array.sum() / n;
    let mean_y = y_array.sum() / n;

    // Calculer les termes de covariance et de variance
    let mut cov_xy = 0.0;
    let mut var_x = 0.0;
    let mut var_y = 0.0;

    for i in 0..x_array.len() {
        let diff_x = x_array[i] - mean_x;
        let diff_y = y_array[i] - mean_y;

        cov_xy += diff_x * diff_y;
        var_x += diff_x * diff_x;
        var_y += diff_y * diff_y;
    }

    // Calculer le coefficient de corrélation
    if var_x > 0.0 && var_y > 0.0 {
        Ok(cov_xy / (var_x.sqrt() * var_y.sqrt()))
    } else {
        Err(PyErr::new::<pyo3::exceptions::PyValueError, _>(
            "Variance nulle détectée"
        ))
    }
}

/// Multiplication matricielle optimisée avec parallélisation
#[pyfunction]
fn matmul(
    a: PyReadonlyArray2<f64>,
    b: PyReadonlyArray2<f64>,
    py: Python<'_>
) -> PyResult<Py<PyArray2<f64>>> {
    let a_array = a.as_array();
    let b_array = b.as_array();

    // Vérifier compatibilité des dimensions
    if a_array.shape()[1] != b_array.shape()[0] {
        return Err(PyErr::new::<pyo3::exceptions::PyValueError, _>(
            "Dimensions incompatibles pour la multiplication matricielle"
        ));
    }

    let a_rows = a_array.shape()[0];
    let a_cols = a_array.shape()[1];
    let b_cols = b_array.shape()[1];

    // Créer la matrice résultante
    let mut result = Array2::<f64>::zeros((a_rows, b_cols));

    // Multiplication matricielle parallélisée
    rayon::scope(|s| {
        for i in 0..a_rows {
            s.spawn(move |_| {
                for j in 0..b_cols {
                    let mut sum = 0.0;
                    for k in 0..a_cols {
                        sum += a_array[[i, k]] * b_array[[k, j]];
                    }
                    result[[i, j]] = sum;
                }
            });
        }
    });

    // Convertir en tableau Python
    Ok(PyArray2::from_array(py, &result).to_owned())
}

/// Module Rust pour le calcul scientifique
#[pymodule]
fn rapide_calc(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(correlation, m)?)?;
    m.add_function(wrap_pyfunction!(matmul, m)?)?;
    Ok(())
}
```
Pour compiler cette extension, créez un fichier `Cargo.toml` :
``` toml
[package]
name = "rapide_calc"
version = "0.1.0"
edition = "2024"

[lib]
name = "rapide_calc"
crate-type = ["cdylib"]

[dependencies]
pyo3 = { version = "0.20.0", features = ["extension-module"] }
numpy = "0.20.0"
ndarray = "0.15.6"
rayon = "1.10.0"
```
Exemple d'utilisation en Python :
``` python
import numpy as np
import rapide_calc

# Créer des données de test
x = np.random.randn(1000000)
y = 0.8 * x + np.random.randn(1000000) * 0.5

# Calculer la corrélation avec notre module Rust
import time
start = time.time()
corr_rust = rapide_calc.correlation(x, y)
end = time.time()
print(f"Corrélation (Rust): {corr_rust}, temps: {end-start:.6f}s")

# Comparer avec NumPy
start = time.time()
corr_numpy = np.corrcoef(x, y)[0, 1]
end = time.time()
print(f"Corrélation (NumPy): {corr_numpy}, temps: {end-start:.6f}s")

# Test de multiplication matricielle
a = np.random.rand(500, 500)
b = np.random.rand(500, 500)

start = time.time()
c_rust = rapide_calc.matmul(a, b)
end = time.time()
print(f"Multiplication matricielle (Rust): temps: {end-start:.6f}s")

start = time.time()
c_numpy = np.matmul(a, b)
end = time.time()
print(f"Multiplication matricielle (NumPy): temps: {end-start:.6f}s")

# Vérifier l'exactitude
print(f"Différence max: {np.max(np.abs(c_rust - c_numpy))}")
```
#### 20.3.2 Polars - Analyse de données rapide
Polars est une alternative Rust à pandas, offrant des performances exceptionnelles pour l'analyse de données tabulaires. Il dispose d'une API Python, permettant de l'intégrer facilement dans un flux de travail Python existant.
``` rust
use polars::prelude::*;
use std::fs::File;
use rayon::prelude::*;

fn main() -> Result<(), PolarsError> {
    // Charger des données CSV
    let file = File::open("donnees.csv")?;
    let df = CsvReader::new(file)
        .has_header(true)
        .finish()?;

    println!("Structure du DataFrame:");
    println!("{}", df.head(Some(5)));

    // Statistiques descriptives
    println!("Description:");
    println!("{}", df.describe(None));

    // Filtrage des données avec expressions
    let filtre = col("valeur").gt(lit(5));
    let df_filtre = df.lazy().filter(filtre).collect()?;
    println!("Données filtrées (valeur > 5):");
    println!("{}", df_filtre);

    // Agrégation par groupe avec LazyFrame API
    let df_groupe = df.lazy()
        .group_by([col("categorie")])
        .agg([
            col("valeur").mean().alias("moyenne"),
            col("valeur").sum().alias("somme"),
            col("valeur").count().alias("nombre"),
        ])
        .collect()?;

    println!("Agrégation par catégorie:");
    println!("{}", df_groupe);

    // Jointure avec un autre DataFrame
    let autre_file = File::open("categories.csv")?;
    let df_categories = CsvReader::new(autre_file)
        .has_header(true)
        .finish()?;

    let df_joint = df.lazy()
        .join(
            df_categories.lazy(),
            [col("categorie")],
            [col("categorie")],
            JoinType::Left,
        )
        .collect()?;

    println!("Données jointes:");
    println!("{}", df_joint);

    // Opérations parallèles avec Rayon
    let valeurs: Vec<i32> = df.column("valeur")?
        .i32()?
        .into_iter()
        .flatten()
        .collect();

    // Calculer la somme en parallèle
    let somme: i32 = valeurs.par_iter().sum();
    println!("Somme calculée en parallèle: {}", somme);

    // Transformation en parallèle
    let doubles: Vec<i32> = valeurs.par_iter()
        .map(|x| x * 2)
        .collect();

    println!("Premières valeurs doublées: {:?}", &doubles[..5]);

    Ok(())
}
```
Pour utiliser Polars, ajoutez :
``` toml
[dependencies]
polars = { version = "0.37.0", features = ["lazy", "temporal", "random", "strings"] }
rayon = "1.10.0"
```
### 20.4 Visualisation scientifique en Rust
#### 20.4.1 Plotters - Bibliothèque de visualisation

Plotters est une bibliothèque de visualisation qui permet de créer divers types de graphiques.

```rust
use plotters::prelude::*;
use rand::Rng;
use rand_distr::{Distribution, Normal};
use rayon::prelude::*;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Créer des données aléatoires suivant une loi normale
    let normal = Normal::new(0.0, 1.0)?;
    let mut rng = rand::thread_rng();

    // Génération parallèle des données
    let echantillon: Vec<f64> = (0..1000)
        .into_par_iter()
        .map(|_| normal.sample(&mut rand::thread_rng()))
        .collect();

    // Créer un histogramme
    let root = BitMapBackend::new("histogramme.png", (800, 600)).into_drawing_area();
    root.fill(&WHITE)?;

    let mut chart = ChartBuilder::on(&root)
        .caption("Histogramme d'une distribution normale", ("sans-serif", 30))
        .margin(5)
        .x_label_area_size(30)
        .y_label_area_size(30)
        .build_cartesian_2d(-4.0..4.0, 0..100)?;

    chart.configure_mesh()
        .x_desc("Valeur")
        .y_desc("Fréquence")
        .axis_desc_style(("sans-serif", 15))
        .draw()?;

    // Créer des bacs pour l'histogramme
    let mut histogramme = vec![0; 80];

    // Comptage des fréquences
    for &valeur in &echantillon {
        let indice = ((valeur + 4.0) / 0.2) as usize;
        if indice < histogramme.len() {
            histogramme[indice] += 1;
        }
    }

    // Tracer l'histogramme
    chart.draw_series(
        histogramme.iter().enumerate().map(|(i, &compte)| {
            let x0 = -4.0 + i as f64 * 0.2;
            let x1 = x0 + 0.2;
            Rectangle::new([(x0, 0), (x1, compte)], BLUE.filled())
        }),
    )?;

    // Tracer la densité théorique
    chart.draw_series(LineSeries::new(
        (-40..40).map(|i| {
            let x = i as f64 * 0.1;
            (
                x,
                (100.0 * 0.2 / (2.0 * std::f64::consts::PI).sqrt()) * (-x * x / 2.0).exp(),
            )
        }),
        &RED,
    ))?
    .label("Densité théorique")
    .legend(|(x, y)| PathElement::new(vec![(x, y), (x + 20, y)], &RED));

    // Ajouter une légende
    chart.configure_series_labels()
        .background_style(&WHITE.mix(0.8))
        .border_style(&BLACK)
        .draw()?;

    println!("Histogramme créé: histogramme.png");

    // Exemple plus avancé: diagramme de dispersion coloré par densité
    let root2 = BitMapBackend::new("dispersion.png", (800, 600)).into_drawing_area();
    root2.fill(&WHITE)?;

    let mut chart2 = ChartBuilder::on(&root2)
        .caption("Diagramme de dispersion avec densité", ("sans-serif", 30))
        .margin(5)
        .x_label_area_size(30)
        .y_label_area_size(30)
        .build_cartesian_2d(-4.0..4.0, -4.0..4.0)?;

    chart2.configure_mesh().draw()?;

    // Générer des données bivariées
    let points: Vec<(f64, f64)> = (0..2000)
        .into_par_iter()
        .map(|_| {
            let x = normal.sample(&mut rand::thread_rng());
            let y = 0.5 * x + 0.5 * normal.sample(&mut rand::thread_rng());
            (x, y)
        })
        .collect();

    // Calculer la densité locale (nombre de points voisins)
    let densites: Vec<usize> = points
        .par_iter()
        .map(|&(x, y)| {
            points
                .iter()
                .filter(|&&(xi, yi)| {
                    let dx = x - xi;
                    let dy = y - yi;
                    dx * dx + dy * dy < 0.5 * 0.5
                })
                .count()
        })
        .collect();

    // Normaliser les densités
    let max_densite = *densites.iter().max().unwrap_or(&1) as f64;

    // Tracer les points colorés par densité
    for ((&(x, y), &densite)) in points.iter().zip(densites.iter()) {
        let couleur = RGBColor(
            (densite as f64 / max_densite * 255.0) as u8,
            0,
            (255.0 - densite as f64 / max_densite * 255.0) as u8,
        );

        chart2.draw_series(std::iter::once(Circle::new(
            (x, y),
            3,
            couleur.filled(),
        )))?;
    }

    // Ajouter une droite de régression
    let (a, b) = {
        let n = points.len() as f64;
        let sum_x: f64 = points.iter().map(|(x, _)| x).sum();
        let sum_y: f64 = points.iter().map(|(_, y)| y).sum();
        let sum_xy: f64 = points.iter().map(|(x, y)| x * y).sum();
        let sum_xx: f64 = points.iter().map(|(x, _)| x * x).sum();

        let a = (n * sum_xy - sum_x * sum_y) / (n * sum_xx - sum_x * sum_x);
        let b = (sum_y - a * sum_x) / n;

        (a, b)
    };

    chart2.draw_series(LineSeries::new(
        [-4.0, 4.0].iter().map(|&x| (x, a * x + b)),
        RED.stroke_width(2),
    ))?
    .label(format!("y = {:.2}x + {:.2}", a, b))
    .legend(|(x, y)| PathElement::new(vec![(x, y), (x + 20, y)], &RED));

    chart2.configure_series_labels().draw()?;

    println!("Diagramme de dispersion créé: dispersion.png");

    Ok(())
}
```


Pour exécuter cet exemple, ajoutez :

```toml
[dependencies]
plotters = "0.3.5"
rand = "0.8.5"
rand_distr = "0.4.3"
rayon = "1.10.0"
```


#### 20.4.2 Intégration avec des interfaces utilisateur

Voici un exemple plus complet avec une interface graphique pour visualiser des données scientifiques en temps réel :

```rust
use eframe::{egui, epi};
use egui::plot::{Line, Plot, PlotPoints, Points};
use rand::Rng;
use rand_distr::{Distribution, Normal, Exp};
use rayon::prelude::*;

struct AppScientifique {
    donnees: Vec<[f64; 2]>,
    distribution: String,
    moyenne: f64,
    ecart_type: f64,
    taille_echantillon: usize,
    afficher_histogramme: bool,
    histogramme: Vec<usize>,
    bins: Vec<f64>,
}

impl Default for AppScientifique {
    fn default() -> Self {
        Self {
            donnees: Vec::new(),
            distribution: "Normale".to_string(),
            moyenne: 0.0,
            ecart_type: 1.0,
            taille_echantillon: 1000,
            afficher_histogramme: false,
            histogramme: vec![0; 50],
            bins: (0..51).map(|i| -5.0 + 0.2 * i as f64).collect(),
        }
    }
}

impl epi::App for AppScientifique {
    fn name(&self) -> &str {
        "Visualisation Scientifique"
    }

    fn update(&mut self, ctx: &egui::Context, _frame: &epi::Frame) {
        let mut rng = rand::thread_rng();

        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading("Simulation de distribution");

            ui.horizontal(|ui| {
                ui.label("Distribution: ");
                ui.radio_value(&mut self.distribution, "Normale".to_string(), "Normale");
                ui.radio_value(&mut self.distribution, "Exponentielle".to_string(), "Exponentielle");
                ui.radio_value(&mut self.distribution, "Uniforme".to_string(), "Uniforme");
            });

            ui.horizontal(|ui| {
                ui.label("Moyenne: ");
                if ui.add(egui::Slider::new(&mut self.moyenne, -5.0..=5.0).text("μ")).changed() {
                    self.donnees.clear();
                }

                ui.label("Écart-type: ");
                if ui.add(egui::Slider::new(&mut self.ecart_type, 0.1..=5.0).text("σ")).changed() {
                    self.donnees.clear();
                }
            });

            ui.horizontal(|ui| {
                ui.label("Taille de l'échantillon: ");
                if ui.add(egui::Slider::new(&mut self.taille_echantillon, 100..=10000)).changed() {
                    self.donnees.clear();
                }

                ui.checkbox(&mut self.afficher_histogramme, "Afficher histogramme");
            });

            if ui.button("Générer échantillon").clicked() || self.donnees.is_empty() {
                // Générer de nouvelles données en parallèle
                self.donnees = match self.distribution.as_str() {
                    "Normale" => {
                        let normale = Normal::new(self.moyenne, self.ecart_type).unwrap();
                        (0..self.taille_echantillon)
                            .into_par_iter()
                            .map(|i| {
                                let x = i as f64 / (self.taille_echantillon as f64 / 20.0) - 10.0;
                                let y = normale.sample(&mut rand::thread_rng());
                                [x, y]
                            })
                            .collect()
                    },
                    "Exponentielle" => {
                        let exp = Exp::new(1.0 / self.ecart_type).unwrap();
                        (0..self.taille_echantillon)
                            .into_par_iter()
                            .map(|i| {
                                let x = i as f64 / (self.taille_echantillon as f64 / 20.0);
                                let y = exp.sample(&mut rand::thread_rng()) + self.moyenne;
                                [x, y]
                            })
                            .collect()
                    },
                    "Uniforme" => {
                        (0..self.taille_echantillon)
                            .into_par_iter()
                            .map(|i| {
                                let x = i as f64 / (self.taille_echantillon as f64 / 20.0) - 10.0;
                                let y = rng.gen_range(self.moyenne - self.ecart_type * 1.732..=self.moyenne + self.ecart_type * 1.732);
                                [x, y]
                            })
                            .collect()
                    },
                    _ => Vec::new()
                };

                // Calculer l'histogramme
                if self.afficher_histogramme {
                    self.histogramme = vec![0; 50];
                    for &[_, y] in &self.donnees {
                        let bin = ((y + 5.0) / 0.2) as usize;
                        if bin < self.histogramme.len() {
                            self.histogramme[bin] += 1;
                        }
                    }
                }
            }

            // Afficher le graphique
            if !self.donnees.is_empty() {
                if self.afficher_histogramme {
                    // Afficher l'histogramme
                    Plot::new("histogramme")
                        .height(250.0)
                        .show(ui, |plot_ui| {
                            // Convertir l'histogramme en barres
                            for (i, &count) in self.histogramme.iter().enumerate() {
                                let bin_start = self.bins[i];
                                let bin_end = self.bins[i + 1];
                                let bin_width = bin_end - bin_start;

                                let bar_points = vec![
                                    [bin_start, 0.0],
                                    [bin_start, count as f64],
                                    [bin_end, count as f64],
                                    [bin_end, 0.0],
                                ];

                                plot_ui.polygon(egui::plot::Polygon::new(bar_points).color(egui::Color32::from_rgb(100, 150, 200)).fill_alpha(0.5));
                            }

                            // Tracer la densité théorique
                            if self.distribution == "Normale" {
                                let points: PlotPoints = (-50..=50)
                                    .map(|i| {
                                        let x = i as f64 * 0.2;
                                        let pdf = (1.0 / (self.ecart_type * (2.0 * std::f64::consts::PI).sqrt())) *
                                                (-0.5 * ((x - self.moyenne) / self.ecart_type).powi(2)).exp();
                                        [x, pdf * self.taille_echantillon as f64 * 0.2]
                                    })
                                    .collect();

                                plot_ui.line(Line::new(points).color(egui::Color32::RED).width(2.0));
                            }
                        });
                }

                // Afficher le nuage de points
                Plot::new("nuage_points")
                    .height(250.0)
                    .data_aspect(1.0)
                    .show(ui, |plot_ui| {
                        // Convertir les données en points
                        let points = PlotPoints::from(self.donnees.clone());

                        // Afficher les points
                        plot_ui.points(Points::new(points)
                            .color(egui::Color32::from_rgb(50, 100, 150))
                            .radius(2.0));
                    });

                // Afficher les statistiques
                let somme_y: f64 = self.donnees.iter().map(|[_, y]| y).sum();
                let moyenne = somme_y / self.donnees.len() as f64;

                let variance = self.donnees.iter()
                    .map(|[_, y]| (y - moyenne).powi(2))
                    .sum::<f64>() / self.donnees.len() as f64;

                let ecart_type = variance.sqrt();

                ui.heading("Statistiques de l'échantillon");
                ui.label(format!("Moyenne: {:.4}", moyenne));
                ui.label(format!("Écart-type: {:.4}", ecart_type));

                // Tester l'hypothèse de distribution normale avec le test de Shapiro-Wilk (simplifié)
                if self.taille_echantillon <= 5000 {
                    let mut valeurs: Vec<f64> = self.donnees.iter().map(|[_, y]| *y).collect();
                    valeurs.sort_by(|a, b| a.partial_cmp(b).unwrap());

                    // Calculer les statistiques d'ordre
                    let n = valeurs.len();
                    let mean = moyenne;

                    // Calculer W (statistique du test Shapiro-Wilk, simplified)
                    let a_sum: f64 = (0..n/2).map(|i| {
                        let a_i = (i + 1) as f64 / n as f64;
                        a_i * (valeurs[n - i - 1] - valeurs[i])
                    }).sum();

                    let w = a_sum.powi(2) / variance / (n as f64);

                    // Approximation de la p-value (simplement pour illustration)
                    let p_value = if w > 0.9 {
                        // W proche de 1 indique que les données suivent une loi normale
                        (1.0 - w) * 10.0
                    } else {
                        0.001
                    };

                    ui.separator();
                    ui.label("Test de normalité (approximation):");
                    ui.label(format!("Statistique W: {:.4}", w));
                    ui.label(format!("p-value approximative: {:.4}", p_value));

                    if p_value < 0.05 {
                        ui.label("Hypothèse de normalité rejetée (p < 0.05)");
                    } else {
                        ui.label("Données compatibles avec une distribution normale");
                    }
                }
            }
        });
    }
}

fn main() {
    let app = AppScientifique::default();
    let native_options = eframe::NativeOptions {
        initial_window_size: Some(egui::vec2(800.0, 800.0)),
        ..Default::default()
    };
    eframe::run_native(Box::new(app), native_options);
}
```


Pour exécuter cet exemple, ajoutez :

```toml
[dependencies]
eframe = "0.24.0"
rand = "0.8.5"
rand_distr = "0.4.3"
rayon = "1.10.0"
```


### 20.5 Calcul scientifique haute performance

Dans l'édition 2024 de Rust, le calcul scientifique haute performance bénéficie d'améliorations significatives, notamment grâce à l'optimisation des compilateurs et l'exploitation des capacités vectorielles des processeurs modernes.

#### 20.5.1 Vectorisation avec SIMD

L'utilisation des instructions SIMD (Single Instruction, Multiple Data) permet d'accélérer considérablement les calculs numériques :

```rust
use std::simd::{f32x8, SimdFloat};

// Fonction qui additionne deux vecteurs en utilisant SIMD
fn add_vectors_simd(a: &[f32], b: &[f32], result: &mut [f32]) {
    assert_eq!(a.len(), b.len());
    assert_eq!(a.len(), result.len());

    let chunks = a.len() / 8;

    // Traiter 8 éléments à la fois avec SIMD
    for i in 0..chunks {
        let start = i * 8;

        // Charger 8 valeurs de chaque vecteur
        let a_chunk = f32x8::from_slice(&a[start..start + 8]);
        let b_chunk = f32x8::from_slice(&b[start..start + 8]);

        // Additionner les vecteurs
        let sum = a_chunk + b_chunk;

        // Stocker le résultat
        sum.write_to_slice(&mut result[start..start + 8]);
    }

    // Traiter les éléments restants
    for i in (chunks * 8)..a.len() {
        result[i] = a[i] + b[i];
    }
}

fn main() {
    // Vecteurs de test
    let a = vec![1.0; 1_000_000];
    let b = vec![2.0; 1_000_000];
    let mut result = vec![0.0; 1_000_000];

    // Mesurer le temps d'exécution
    let debut = std::time::Instant::now();
    add_vectors_simd(&a, &b, &mut result);
    let duree = debut.elapsed();

    println!("Addition SIMD terminée en {:?}", duree);
    println!("Premiers éléments: {:?}", &result[..5]);

    // Vérification avec addition scalaire
    let debut = std::time::Instant::now();
    let mut result_scalar = vec![0.0; 1_000_000];
    for i in 0..a.len() {
        result_scalar[i] = a[i] + b[i];
    }
    let duree = debut.elapsed();

    println!("Addition scalaire terminée en {:?}", duree);
}
```


Pour utiliser cette fonctionnalité, ajoutez dans votre `Cargo.toml` :

```toml
[dependencies]
std_float = "0.1.0"

[features]
simd = []
```


Et exécutez avec :

```shell script
cargo run --features simd
```


#### 20.5.2 Optimisation pour le calcul matriciel

Voici un exemple d'implémentation optimisée de multiplication matricielle exploitant le parallélisme et la localité des caches :

```rust
use ndarray::{Array2, Axis};
use rayon::prelude::*;

// Multiplication de matrices optimisée
fn matrix_multiply_optimized(a: &Array2<f64>, b: &Array2<f64>) -> Array2<f64> {
    assert_eq!(a.shape()[1], b.shape()[0], "Dimensions incompatibles");

    let a_rows = a.shape()[0];
    let a_cols = a.shape()[1];
    let b_cols = b.shape()[1];

    // Transposer B pour une meilleure localité de cache
    let b_t = b.clone().reversed_axes();

    // Allouer la matrice résultat
    let mut result = Array2::zeros((a_rows, b_cols));

    // Multiplication parallèle par blocs
    const BLOCK_SIZE: usize = 64; // Taille optimale dépend de l'architecture

    // Parcourir les blocs de la matrice résultante
    for i_block in (0..a_rows).step_by(BLOCK_SIZE) {
        let i_end = (i_block + BLOCK_SIZE).min(a_rows);

        // Paralléliser les blocs de lignes
        (i_block..i_end).into_par_iter().for_each(|i| {
            for j_block in (0..b_cols).step_by(BLOCK_SIZE) {
                let j_end = (j_block + BLOCK_SIZE).min(b_cols);

                for k_block in (0..a_cols).step_by(BLOCK_SIZE) {
                    let k_end = (k_block + BLOCK_SIZE).min(a_cols);

                    // Multiplication locale dans le bloc
                    for j in j_block..j_end {
                        let mut sum = 0.0;
                        for k in k_block..k_end {
                            sum += a[[i, k]] * b_t[[j, k]]; // Utiliser B transposée
                        }
                        result[[i, j]] += sum;
                    }
                }
            }
        });
    }

    result
}

fn main() {
    // Créer des matrices de test
    let size = 1024;
    let a = Array2::from_shape_fn((size, size), |_| rand::random::<f64>());
    let b = Array2::from_shape_fn((size, size), |_| rand::random::<f64>());

    // Mesurer le temps de la multiplication optimisée
    let debut = std::time::Instant::now();
    let c_opt = matrix_multiply_optimized(&a, &b);
    let duree_opt = debut.elapsed();
    println!("Multiplication optimisée: {:?}", duree_opt);

    // Méthode standard pour comparaison
    let debut = std::time::Instant::now();
    let c_std = a.dot(&b);
    let duree_std = debut.elapsed();
    println!("Multiplication standard: {:?}", duree_std);

    // Vérifier la différence
    let diff = (&c_opt - &c_std).mapv(|x| x.abs()).sum();
    println!("Différence totale: {}", diff);
}
```


### Conclusion

L'écosystème Rust pour le calcul scientifique continue de s'améliorer, avec l'édition 2024 offrant des performances accrues et une meilleure intégration avec les outils existants. Les bibliothèques comme `ndarray`, `nalgebra`, `polars` et `PyO3` permettent aux développeurs d'exploiter la sécurité et les performances de Rust tout en conservant la richesse de l'écosystème scientifique de Python.

Avec la stabilisation des fonctionnalités avancées comme les génériques associés et les traits spécialisés, Rust devient une option de plus en plus attrayante pour le développement d'applications scientifiques performantes et fiables.

⏭️ [Zero-cost abstractions avancées](/III-avance/21-zero-cost-abstractions.md) - Exemples et techniques pour créer des abstractions sans coût à l'exécution
