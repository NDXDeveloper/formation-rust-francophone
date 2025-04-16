## 8\. Sized et String vs str

La compréhension des types de taille connue ("sized") et inconnue ("unsized") est fondamentale en Rust, particulièrement pour manipuler correctement les chaînes de caractères.

### Le trait `Sized`

Le trait `Sized` est un trait spécial en Rust qui indique au compilateur que la taille d'un type est connue à la compilation. Par défaut, tous les types génériques en Rust sont considérés comme implémentant `Sized` implicitement, sauf indication contraire.

Un type qui n'implémente pas `Sized` est dit "dynamically sized type" (DST) ou type de taille dynamique. Les exemples les plus courants sont :

- `str` (chaîne de caractères sans allocation)
- `[T]` (slice, ou tableau sans taille fixe)
- `dyn Trait` (objets de traits)

### Pourquoi `str` n'est pas `Sized`

Le type `str` représente une séquence de caractères UTF-8 valides, mais sa taille n'est pas connue à la compilation. En effet, une chaîne comme `"bonjour"` et une chaîne comme `"un exemple très long"` n'ont évidemment pas la même taille.

Essayons de déclarer une variable de type `str` :

``` rust
// Ceci ne compile pas !
let x: str = "bonjour";
```

L'erreur obtenue est :

```
error[E0277]: the size for values of type `str` cannot be known at compilation time
  |
  | let x: str = "bonjour";
  |     ^ doesn't have a size known at compile-time
  |
  = help: the trait `Sized` is not implemented for `str`
```

### Manipulation des types non-Sized

Pour manipuler des types de taille dynamique en Rust, nous devons utiliser :

1.  Des références (`&str`, `&[T]`)
2.  Des types d'indirection sur le tas (`Box<str>`, `Box<[T]>`)
3.  Des wrappers spécifiques (`String` pour `str`, `Vec<T>` pour `[T]`)

### La différence entre `String` et `str`

#### Le type `str`

`str` est un type primitif qui représente une séquence de caractères UTF-8. C'est un type de taille dynamique qui :

- ne peut pas être utilisé directement comme variable
- est généralement manipulé via une référence `&str`
- est immuable (non modifiable)
- n'alloue pas de mémoire supplémentaire
- peut être une vue (slice) d'une plus grande chaîne

``` rust
// Une référence à une str litérale (stockée dans la section data du programme)
let salutation: &str = "Bonjour";

// Une slice d'une chaîne plus grande
let texte = "Bonjour à tous";
let partie: &str = &texte[0..7]; // "Bonjour"
```

#### Le type `String`

`String` est une structure qui possède, alloue et gère une chaîne de caractères UTF-8. Elle :

- implémente `Deref<Target=str>`, ce qui permet le déréférencement implicite en `&str`
- est modifiable (si mutable)
- alloue de la mémoire sur le tas (heap)
- peut grandir, rétrécir ou être modifiée
- contient trois champs internes : un pointeur vers les données, une longueur et une capacité

``` rust
// Création d'une String
let mut salutation = String::from("Bonjour");

// Modification
salutation.push_str(" à tous");
salutation.push('!');

// Conversion de &str vers String
let chaine: String = "Une chaîne".to_owned();
// ou
let chaine: String = "Une chaîne".to_string();

// Déréférencement de String vers &str
let vue: &str = &salutation;
```

### Structure interne

Pour mieux comprendre la différence, voici un aperçu de leur structure interne :

```
str:
┌───────────────────────────┐
│ Contenu UTF-8             │
└───────────────────────────┘

&str:
┌─────────┬─────────┐
│ Pointeur│ Longueur│ → Référence à un contenu UTF-8
└─────────┴─────────┘

String:
┌─────────┬─────────┬──────────┐
│ Pointeur│ Longueur│ Capacité │ → Données UTF-8 allouées sur le tas
└─────────┴─────────┴──────────┘
```

### La relation avec `Vec<T>` et les slices `[T]`

La relation entre `String` et `str` est très similaire à celle entre `Vec<T>` et `[T]` :

- `[T]` est une séquence d'éléments de type `T` de taille inconnue à la compilation (non-Sized)
- `Vec<T>` est une structure qui possède et gère un tableau d'éléments `T` sur le tas
- `Vec<T>` implémente `Deref<Target=[T]>`

``` rust
// Démonstration de la relation similaire
let nombres_vec: Vec<i32> = vec![1, 2, 3, 4, 5];
let nombres_slice: &[i32] = &nombres_vec;  // Déréférencement implicite

// Conversion slice -> Vec
let clone_vec: Vec<i32> = nombres_slice.to_vec();

// Approche équivalente avec les chaînes
let texte_string: String = String::from("Bonjour");
let texte_str: &str = &texte_string;  // Déréférencement implicite

// Conversion str -> String
let clone_string: String = texte_str.to_owned();
```

### Implémentation interne de `String`

Il est important de noter que `String` est essentiellement un wrapper autour de `Vec<u8>` avec des garanties supplémentaires :

``` rust
// Création d'une String à partir d'un Vec<u8>
let octets = vec![72, 101, 108, 108, 111]; // ASCII pour "Hello"
let resultat = String::from_utf8(octets);

match resultat {
    Ok(s) => println!("String valide: {}", s),
    Err(e) => println!("Erreur UTF-8: {}", e),
}
```

Le type `String` garantit que son contenu est toujours un UTF-8 valide, contrairement à `Vec<u8>` qui peut contenir n'importe quelle séquence d'octets.

### Quand utiliser `String` vs `&str`

En règle générale :

- Utilisez `&str` pour les paramètres de fonction lorsque vous avez uniquement besoin de lire une chaîne
- Utilisez `String` lorsque vous devez posséder ou modifier une chaîne
- Utilisez `&str` pour les chaînes littérales ou les vues dans d'autres chaînes
- Utilisez `String` pour les chaînes construites ou modifiées à l'exécution

La compréhension de ces deux types et de leur relation est essentielle pour une programmation efficace en Rust.
