## 10\. Les unions

Retour à la [Table des matières](/SOMMAIRE.md)

Les unions constituent un concept avancé en Rust qui permet de stocker différents types de données dans le même espace mémoire. Contrairement aux enums qui sont des types somme (un champ à la fois est actif), les unions permettent d'interpréter le même espace mémoire de différentes façons.

### Définition et propriétés

Une union en Rust est définie comme suit :

``` rust
union MonUnion {
    entier: u32,
    flottant: f32,
}
```

Propriétés importantes des unions :

- Tous les champs partagent le même espace mémoire
- La taille d'une union est égale à la taille de son plus grand champ
- L'accès aux champs est toujours `unsafe` car le compilateur ne peut pas savoir quel champ contient la valeur valide
- Seuls les types qui implémentent `Copy` ou sont entourés de `ManuallyDrop` peuvent être utilisés dans une union

### Exemple d'utilisation basique

``` rust
union IntOuFloat {
    entier: u32,
    flottant: f32,
}

fn main() {
    // Initialisation avec le champ entier
    let valeur = IntOuFloat { entier: 42 };

    unsafe {
        println!("Comme entier: {}", valeur.entier);
        // Le même espace mémoire interprété comme flottant
        println!("Comme flottant: {}", valeur.flottant);
    }

    // Initialisation avec un flottant
    let valeur2 = IntOuFloat { flottant: 3.14 };

    unsafe {
        println!("Comme flottant: {}", valeur2.flottant);
        // Bit pattern du flottant interprété comme entier
        println!("Comme entier: {}", valeur2.entier);
    }
}
```

### Utilisation pratique : manipulation de bits

Les unions sont particulièrement utiles pour des opérations de bas niveau comme la manipulation de bits ou l'interfaçage avec des structures C.

``` rust
// Représentation d'une couleur RGBA
union RGBAColor {
    rgba: u32,            // Format compact sur 32 bits
    components: [u8; 4],  // Composants individuels R,G,B,A
    named: NamedColors,   // Structure nommée
}

#[repr(C)]
#[derive(Copy, Clone)]
struct NamedColors {
    r: u8,
    g: u8,
    b: u8,
    a: u8,
}

fn main() {
    // Création d'une couleur rouge opaque
    let mut couleur = RGBAColor { rgba: 0 };

    unsafe {
        couleur.named.r = 255;   // Rouge au maximum
        couleur.named.a = 255;   // Opaque

        // Affichage de la valeur comme entier 32 bits
        println!("Couleur RGBA comme entier: 0x{:08X}", couleur.rgba);

        // Manipulation des composants individuels
        println!("Composants: R={}, G={}, B={}, A={}",
            couleur.components[0],
            couleur.components[1],
            couleur.components[2],
            couleur.components[3]);
    }
}
```

### Unions avec types complexes

Pour utiliser des types non-Copy dans une union, il faut les envelopper dans `ManuallyDrop` :

``` rust
use std::mem::ManuallyDrop;

union StringOrInt {
    entier: u64,
    chaine: ManuallyDrop<String>,
}

fn main() {
    let mut union = StringOrInt { entier: 0 };

    unsafe {
        // Écrit une String dans l'union
        *(&mut union.chaine as *mut ManuallyDrop<String>) =
            ManuallyDrop::new(String::from("Bonjour Rust"));

        // Lecture de la String
        println!("Comme String: {}", &*union.chaine);

        // Lecture comme entier (probablement une adresse mémoire)
        println!("Comme entier: {}", union.entier);

        // IMPORTANT: il faut manuellement libérer la String avant de réutiliser l'union
        // pour un autre type, afin d'éviter les fuites mémoire
        ManuallyDrop::drop(&mut union.chaine);
    }

    // Maintenant on peut utiliser le champ entier sans problème
    union.entier = 42;
}
```

### Pattern matching avec les unions

Le pattern matching avec les unions est possible mais limité, car le compilateur ne peut pas savoir quel champ est actuellement valide :

```
union IntOuBool {
    entier: u32,
    booléen: bool,
}

fn main() {
    let valeur = IntOuBool { entier: 1 };

    unsafe {
        match valeur {
            IntOuBool { entier: 0 } => println!("C'est zéro"),
            IntOuBool { entier: 1 } => println!("C'est un"),
            _ => println!("C'est autre chose")
        }
    }
}
```

### Cas d'utilisation réel : interprétation de f32 comme u32

Voici un exemple pratique où l'on utilise une union pour examiner la représentation binaire d'un nombre flottant :

``` rust
union FloatBits {
    f: f32,
    bits: u32,
}

fn main() {
    let pi = FloatBits { f: std::f32::consts::PI };

    unsafe {
        // Extraire le signe, l'exposant et la mantisse
        let bits = pi.bits;
        let signe = (bits >> 31) & 1;
        let exposant = (bits >> 23) & 0xFF;
        let mantisse = bits & 0x7FFFFF;

        println!("Pi (f32): {}", pi.f);
        println!("Représentation binaire: {:032b}", bits);
        println!("Signe: {}", signe);
        println!("Exposant: {} (biaisé, réel: {})", exposant, exposant as i32 - 127);
        println!("Mantisse: 0x{:X}", mantisse);
    }
}
```

### Considérations de sécurité

Les unions demandent une attention particulière :

1.  **Responsabilité du développeur** : C'est au développeur de s'assurer que le champ accédé contient une valeur valide.

2.  **Violation d'aliasing** : Attention à ne pas créer de références mutables à plusieurs champs d'une même union simultanément.

3.  **Undefined Behavior** : L'accès à un champ qui ne contient pas la dernière valeur écrite peut causer un comportement indéfini, surtout avec des types non-triviaux.


``` rust
fn comportement_risqué() {
    union Risquée {
        entier: u32,
        flottant: f32,
    }

    let mut val = Risquée { entier: 42 };

    unsafe {
        // Écriture d'un entier
        val.entier = 0xDEADBEEF;

        // Lecture comme flottant - comportement défini tant que les
        // bits forment un flottant valide
        let f = val.flottant;

        // Écriture d'un flottant NaN (Not a Number)
        val.flottant = std::f32::NAN;

        // Vérifier les bits - flottant NaN a un motif particulier
        assert_ne!(val.entier, 0xDEADBEEF);
    }
}
```

### Cas d'utilisation typiques

Les unions sont particulièrement utiles dans les situations suivantes :

1.  **Interfaçage avec du code C** qui utilise des unions
2.  **Manipulation de bits** pour des formats de données spécifiques
3.  **Optimisation mémoire** dans des structures de données très contraintes
4.  **Interprétation de données binaires** provenant de fichiers ou du réseau

En résumé, les unions sont un outil puissant mais qui demande beaucoup de précautions. La plupart des développeurs Rust n'auront que rarement besoin d'y recourir dans le code courant.

⏭️ [Closure](/II-specificites/11-closure.md)
