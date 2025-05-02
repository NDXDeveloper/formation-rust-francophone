## 9\. Unsafe

Retour à la [Table des matières](/SOMMAIRE.md)

Le langage Rust est conçu pour garantir la sécurité mémoire, l'absence de conditions de course et d'autres comportements indéfinis. Cependant, ces garanties peuvent parfois être trop restrictives pour certaines opérations comme l'interaction avec du code C ou certaines optimisations de performance. C'est là qu'intervient le mot-clé `unsafe`.

### Qu'est-ce que le code unsafe?

Le code `unsafe` permet d'effectuer certaines opérations qui ne peuvent pas être vérifiées par le compilateur. Cependant, contrairement à une idée reçue, `unsafe` ne suspend pas les règles fondamentales de Rust comme l'ownership et le borrow checking. Ces règles restent pleinement en vigueur.

Voici les quatre opérations qui nécessitent le mot-clé `unsafe` :

1.  Déréférencer un pointeur brut
2.  Appeler une fonction ou une méthode déclarée comme `unsafe`
3.  Accéder à ou modifier une variable statique mutable
4.  Implémenter un trait `unsafe`
5.  Accéder aux champs d'une union

### Les pointeurs bruts

Les pointeurs bruts en Rust se notent `*const T` ou `*mut T` selon qu'ils sont constants ou mutables. Contrairement aux références, ils ne garantissent pas la validité de la mémoire pointée.

``` rust
fn main() {
    let nombre = 42;
    let ptr = &nombre as *const i32;  // Conversion d'une référence en pointeur brut

    // La déréférencement d'un pointeur brut requiert unsafe
    unsafe {
        println!("Valeur pointée: {}", *ptr);
    }
}
```

Un exemple plus concret serait l'utilisation d'un pointeur brut pour accéder à un tableau sans vérification des bornes :

``` rust
fn main() {
    let tableau = [1, 2, 3, 4, 5];
    let ptr = tableau.as_ptr();

    unsafe {
        // Accès direct sans vérification des bornes
        println!("Premier élément: {}", *ptr);
        println!("Deuxième élément: {}", *ptr.add(1));
        println!("Troisième élément: {}", *ptr.add(2));

        // DANGER: Accès à un index hors limites
        // Ceci pourrait causer un comportement indéfini
        // println!("Sixième élément: {}", *ptr.add(5));
    }
}
```

### Fonctions unsafe

Une fonction peut être déclarée comme `unsafe` lorsqu'elle requiert que l'appelant garantisse certaines conditions pour éviter des comportements indéfinis.

``` rust
// Cette fonction est unsafe car elle déréférence un pointeur brut
// sans vérifier sa validité
unsafe fn déréférencer_pointeur(ptr: *const i32) -> i32 {
    *ptr
}

fn main() {
    let nombre = 42;
    let ptr = &nombre as *const i32;

    // Appel d'une fonction unsafe
    unsafe {
        let valeur = déréférencer_pointeur(ptr);
        println!("Valeur obtenue: {}", valeur);
    }
}
```

### Meilleures pratiques avec unsafe

Même si vous devez utiliser `unsafe`, vous pouvez limiter sa portée :

1.  **Encapsuler le code unsafe** dans des fonctions sûres en ajoutant les vérifications nécessaires :

``` rust
fn accéder_index_sûr(tableau: &[i32], index: usize) -> Option<i32> {
    if index < tableau.len() {
        // unsafe limité à cette opération spécifique
        unsafe {
            Some(*tableau.as_ptr().add(index))
        }
    } else {
        None
    }
}

fn main() {
    let tableau = [1, 2, 3, 4, 5];

    // L'API publique est sûre
    match accéder_index_sûr(&tableau, 2) {
        Some(valeur) => println!("Valeur à l'index 2: {}", valeur),
        None => println!("Index hors limites"),
    }
}
```

``` rust
// Cette fonction est unsafe car elle déréférence un pointeur brut
// Elle retourne un Result pour une meilleure gestion des erreurs
unsafe fn déréférencer_pointeur(ptr: *const i32) -> Result<i32, &'static str> {
    if ptr.is_null() {
        return Err("Pointeur nul détecté");
    }
    // Bloc unsafe nécessaire même dans une fonction unsafe
    unsafe { Ok(*ptr) }
}

fn main() {
    let nombre = 42;
    let ptr = &nombre as *const i32;

    // Appel d'une fonction unsafe avec gestion propre des erreurs
    unsafe {
        match déréférencer_pointeur(ptr) {
            Ok(valeur) => println!("Valeur obtenue: {}", valeur),
            Err(err) => eprintln!("Erreur: {}", err),
        }
    }

    // Exemple plus idiomatique avec Option
    let valeur_sûre = if !ptr.is_null() {
        unsafe { Some(*ptr) }
    } else {
        None
    };

    if let Some(v) = valeur_sûre {
        println!("Valeur obtenue de façon plus sûre: {}", v);
    }
}
```

2.  **Documenter** les invariants que l'utilisateur doit respecter :

``` rust
/// Alloue un buffer de la taille demandée.
///
 /// # Safety
///
 /// Le pointeur retourné doit être libéré avec `libérer_buffer`
/// et ne doit pas être utilisé après cela.
unsafe fn allouer_buffer(taille: usize) -> *mut u8 {
    std::alloc::alloc(std::alloc::Layout::from_size_align_unchecked(taille, 1))
}

/// Libère un buffer alloué par `allouer_buffer`.
///
 /// # Safety
///
 /// Le pointeur doit provenir d'un appel à `allouer_buffer` et
/// ne doit pas être utilisé après cet appel.
unsafe fn libérer_buffer(ptr: *mut u8, taille: usize) {
    std::alloc::dealloc(ptr, std::alloc::Layout::from_size_align_unchecked(taille, 1));
}
```

### Traits unsafe

Un trait peut être déclaré `unsafe` lorsque ses implémentations doivent respecter des invariants que le compilateur ne peut pas vérifier.

``` rust
// Ce trait indique qu'il est sûr d'accéder à ce type
// depuis plusieurs threads
unsafe trait ThreadSafe {}

// Implémentation pour un type simple
struct MonType(i32);

// En déclarant cette implémentation, nous garantissons
// que MonType est sûr à utiliser dans un contexte multi-thread
unsafe impl ThreadSafe for MonType {}

// On peut implémenter de façon conditionnelle si les types contenus sont ThreadSafe
struct Wrapper<T>(T);

unsafe impl<T: ThreadSafe> ThreadSafe for Wrapper<T> {}
```

### Les traits Send et Sync

Deux traits `unsafe` fondamentaux dans Rust sont `Send` et `Sync` :

- `Send` : indique qu'un type peut être transféré entre threads
- `Sync` : indique qu'un type peut être partagé entre threads via des références

``` rust
use std::cell::Cell;
use std::marker::PhantomData;

// Cell<T> n'est pas Sync car il permet des mutations internes
// sans synchronisation
struct MonTypeNonSync {
    valeur: Cell<i32>
}

// Ce type ne doit pas être utilisé entre threads
struct NonThreadSafe<T>(PhantomData<*mut T>);

// Implémentation explicite qui marque ce type comme non-envoyable entre threads
impl<T> !Send for NonThreadSafe<T> {}
```

### FFI (Foreign Function Interface)

Un des cas d'utilisation les plus courants d'`unsafe` est l'interfaçage avec du code C :

``` rust
// Déclaration d'une interface avec une bibliothèque C
#[link(name = "c")]
unsafe extern "C" {
    fn strlen(s: *const i8) -> usize;
}

fn main() {
    // Création d'une chaîne C-compatible avec terminaison nulle
    let c_string = b"Bonjour\0";

    // Appel à la fonction C (unsafe car on ne peut garantir la sécurité)
    let longueur = unsafe {
        strlen(c_string.as_ptr() as *const i8)
    };

    println!("Longueur de la chaîne: {}", longueur);
}
```

⏭️ [Les unions](/II-specificites/10-unions.md)
