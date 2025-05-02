## 22\. **Les atomics et la mémoire ordonnée** - `std::sync::atomic` et modèles de mémoire

Retour à la [Table des matières](/SOMMAIRE.md)

### Introduction aux atomics

Les types atomiques permettent de manipuler des valeurs partagées entre plusieurs threads sans avoir recours à des mécanismes de verrouillage comme les `Mutex`. Ils garantissent que les opérations sont effectuées de manière atomique, c'est-à-dire qu'elles ne peuvent pas être interrompues à mi-chemin.

La bibliothèque standard Rust fournit plusieurs types atomiques dans le module `std::sync::atomic` :

- `AtomicBool` : version atomique de `bool`
- `AtomicI8`, `AtomicI16`, `AtomicI32`, `AtomicI64`, `AtomicIsize` : versions atomiques des entiers signés
- `AtomicU8`, `AtomicU16`, `AtomicU32`, `AtomicU64`, `AtomicUsize` : versions atomiques des entiers non signés
- `AtomicPtr<T>` : version atomique d'un pointeur brut

### Utilisation basique des atomics

Voici un exemple simple d'utilisation d'un `AtomicUsize` :

``` rust
use std::sync::atomic::{AtomicUsize, Ordering};

fn main() {
    // Création d'une variable atomique avec une valeur initiale de 0
    let compteur = AtomicUsize::new(0);

    // Lecture de la valeur
    let valeur = compteur.load(Ordering::Relaxed);
    println!("Valeur initiale : {}", valeur);

    // Incrémentation
    compteur.fetch_add(1, Ordering::Relaxed);
    println!("Après incrémentation : {}", compteur.load(Ordering::Relaxed));

    // Écriture d'une nouvelle valeur
    compteur.store(42, Ordering::Relaxed);
    println!("Après store : {}", compteur.load(Ordering::Relaxed));

    // Comparaison et échange
    let ancien = compteur.compare_exchange(42, 100, Ordering::Relaxed, Ordering::Relaxed);
    println!("Résultat de compare_exchange : {:?}", ancien);
    println!("Valeur finale : {}", compteur.load(Ordering::Relaxed));
}
```

### Ordres de mémoire (Memory Ordering)

Vous avez probablement remarqué le paramètre `Ordering` dans les méthodes utilisées ci-dessus. Il s'agit d'une caractéristique fondamentale des opérations atomiques en Rust.

L'ordre de mémoire détermine comment les opérations atomiques sont synchronisées avec d'autres opérations de mémoire. Rust propose cinq niveaux d'ordre, du moins restrictif au plus restrictif :

1.  `Ordering::Relaxed` : Garantit uniquement l'atomicité de l'opération, sans synchronisation supplémentaire.
2.  `Ordering::Release` : Tout accès mémoire effectué avant cette opération sera visible aux autres threads qui effectuent une opération `Acquire` sur la même variable.
3.  `Ordering::Acquire` : Tout accès mémoire effectué après cette opération verra les effets des opérations `Release` effectuées par d'autres threads sur la même variable.
4.  `Ordering::AcqRel` : Combine les effets de `Acquire` et `Release`.
5.  `Ordering::SeqCst` (Sequential Consistency) : Le plus strict, garantit un ordre total des opérations atomiques pour tous les threads.

### Exemple avec différents ordres de mémoire

Voyons un exemple plus concret où les ordres de mémoire ont un impact réel :

``` rust
use std::sync::atomic::{AtomicBool, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    // Données partagées entre les threads
    let drapeau = Arc::new(AtomicBool::new(false));
    let donnees = Arc::new([0; 1000]);

    // Clone pour le thread de production
    let drapeau_prod = Arc::clone(&drapeau);
    let donnees_prod = Arc::clone(&donnees);

    // Thread qui prépare des données puis lève un drapeau
    let producteur = thread::spawn(move || {
        // Simulation de préparation de données
        // (En réalité, nous modifierions donnees_prod si c'était un type mutable)
        thread::sleep(std::time::Duration::from_millis(100));

        println!("Producteur : données prêtes");

        // Notification que les données sont prêtes via le drapeau
        // Release garantit que toutes les écritures précédentes sont visibles
        // après cette opération
        drapeau_prod.store(true, Ordering::Release);
    });

    // Thread qui attend le drapeau pour lire les données
    let consommateur = thread::spawn(move || {
        // Attente active du drapeau
        while !drapeau.load(Ordering::Acquire) {
            thread::yield_now(); // Céder le CPU à d'autres threads
        }

        // Avec Acquire, nous sommes assurés que toutes les modifications faites
        // avant le store(true, Release) par le producteur sont visibles ici
        println!("Consommateur : drapeau détecté, traitement des données");

        // Utilisation des données
        let somme: usize = donnees.iter().sum();
        println!("Consommateur : somme des données = {}", somme);
    });

    producteur.join().unwrap();
    consommateur.join().unwrap();
}
```

Dans cet exemple, l'utilisation de `Ordering::Release` pour le drapeau dans le thread producteur et `Ordering::Acquire` dans le thread consommateur crée une barrière de synchronisation qui garantit que toutes les écritures effectuées par le producteur avant de lever le drapeau sont visibles par le consommateur après avoir détecté le drapeau.

### Compteur atomique partagé entre threads

Voici un exemple plus pratique d'utilisation des atomiques pour compter dans plusieurs threads :

``` rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let compteur = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];

    // Création de 10 threads qui incrémentent chacun le compteur 1000 fois
    for id in 0..10 {
        let compteur_clone = Arc::clone(&compteur);

        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                // fetch_add retourne l'ancienne valeur et ajoute 1
                compteur_clone.fetch_add(1, Ordering::Relaxed);
            }
            println!("Thread {} terminé", id);
        });

        handles.push(handle);
    }

    // Attente de la fin de tous les threads
    for handle in handles {
        handle.join().unwrap();
    }

    println!("Valeur finale du compteur : {}", compteur.load(Ordering::Relaxed));
    // La valeur attendue est 10 * 1000 = 10000
}
```

### Cas d'usage : Implémentation d'un compteur de référence simplifié

Les atomiques sont à la base de nombreuses structures de données concurrentes. Voici une implémentation simplifiée d'un compteur de référence atomique (similaire à `Arc`) :

``` rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::ops::Deref;
use std::ptr::NonNull;

struct RefCounter<T> {
    // Pointeur vers nos données et le compteur
    ptr: NonNull<RefCounterInner<T>>,
}

struct RefCounterInner<T> {
    compte: AtomicUsize,
    valeur: T,
}

impl<T> RefCounter<T> {
    fn new(valeur: T) -> Self {
        // Allocation sur le tas
        let inner = Box::new(RefCounterInner {
            compte: AtomicUsize::new(1),
            valeur,
        });

        // Conversion en pointeur non nul
        let ptr = NonNull::new(Box::into_raw(inner)).unwrap();

        RefCounter { ptr }
    }
}

impl<T> Deref for RefCounter<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        // Safety: ptr est valide tant que le compteur > 0
        unsafe { &self.ptr.as_ref().valeur }
    }
}

impl<T> Clone for RefCounter<T> {
    fn clone(&self) -> Self {
        // Incrémentation du compteur de façon atomique
        let inner = unsafe { self.ptr.as_ref() };
        inner.compte.fetch_add(1, Ordering::Relaxed);

        RefCounter { ptr: self.ptr }
    }
}

impl<T> Drop for RefCounter<T> {
    fn drop(&mut self) {
        let inner = unsafe { self.ptr.as_ref() };

        // Décrémentation du compteur et vérification s'il atteint zéro
        // En utilisant Acquire-Release pour assurer la visibilité des modifications
        if inner.compte.fetch_sub(1, Ordering::Release) == 1 {
            // Barrière de synchronisation pour s'assurer que tous les threads ont fini
            // d'utiliser la donnée avant de la désallouer
            atomic_fence(Ordering::Acquire);

            // Safety: Nous sommes le dernier propriétaire, il est sûr de libérer la mémoire
            unsafe {
                let _ = Box::from_raw(self.ptr.as_ptr());
            }
        }
    }
}

// Fonction helper pour la barrière atomique
fn atomic_fence(order: Ordering) {
    use std::sync::atomic::fence;
    fence(order);
}

fn main() {
    // Exemple d'utilisation
    let donnees = RefCounter::new(vec![1, 2, 3, 4, 5]);

    // Créer des clones
    let clone1 = donnees.clone();
    let clone2 = donnees.clone();

    // Utilisation via Deref
    println!("Longueur vecteur: {}", donnees.len());
    println!("Premier élément de clone1: {}", clone1[0]);
    println!("Dernier élément de clone2: {}", clone2[4]);

    // Lorsque toutes les références sont supprimées, la mémoire est libérée automatiquement
}
```

### Fence atomique et ordonnancement global

Parfois, on souhaite imposer des contraintes d'ordonnancement sans effectuer d'opération sur une variable atomique spécifique. Rust fournit la fonction `fence` pour cela :

``` rust
use std::sync::atomic::{fence, Ordering};
use std::thread;
use std::sync::{Arc, Mutex};

fn main() {
    let donnees = Arc::new(Mutex::new(vec![0; 10]));
    let donnees_clone = Arc::clone(&donnees);

    let thread_a = thread::spawn(move || {
        // Modifier les données
        {
            let mut d = donnees_clone.lock().unwrap();
            d[0] = 42;
            d[1] = 43;
        }

        // Barrière qui garantit que toutes les écritures précédentes sont visibles
        // aux opérations qui suivent les fence(Acquire) des autres threads
        fence(Ordering::Release);
    });

    let thread_b = thread::spawn(move || {
        // Barrière qui garantit que toutes les écritures précédant les fence(Release)
        // des autres threads sont visibles après
        fence(Ordering::Acquire);

        // Lire les données
        let d = donnees.lock().unwrap();
        println!("Données lues : {:?}", &d[..2]);
    });

    thread_a.join().unwrap();
    thread_b.join().unwrap();
}
```

### Points importants à retenir

1.  **Quand utiliser les atomiques :**

    - Pour des compteurs ou drapeaux partagés entre threads
    - Comme blocs de construction pour des structures de données concurrentes
    - Pour éviter le coût d'un `Mutex` quand seule une valeur simple doit être partagée
2.  **Choix du bon ordre de mémoire :**

    - `Relaxed` : Seulement quand vous ne vous souciez pas de l'ordre des opérations
    - `Release`/`Acquire` : Pattern producteur/consommateur
    - `SeqCst` : Quand vous avez besoin de garanties fortes ou en cas de doute
3.  **Limitations :**

    - Les atomiques conviennent aux types primitifs, mais pas aux structures complexes
    - Les opérations atomiques peuvent être coûteuses sur certaines architectures
    - La compréhension des modèles de mémoire est nécessaire pour une utilisation correcte

### Conclusion

Les types atomiques offrent un mécanisme de synchronisation de bas niveau et efficace pour les opérations concurrentes sur des valeurs simples. Ils sont à la base de nombreuses abstractions de concurrence de plus haut niveau dans l'écosystème Rust.

Cependant, leur utilisation correcte nécessite une bonne compréhension des modèles de mémoire et des garanties offertes par chaque niveau d'ordonnancement. Pour des cas plus complexes, des abstractions de plus haut niveau comme `Mutex`, `RwLock` ou d'autres structures de la crate `crossbeam` peuvent être plus appropriées et plus sûres à utiliser.

L'utilisation des atomiques est un excellent exemple de la philosophie de Rust qui permet d'accéder à des primitives de bas niveau tout en maintenant un système de types fort et des garanties de sécurité.

⏭️ [III. Aller plus loin](/III-avance/README.md)
