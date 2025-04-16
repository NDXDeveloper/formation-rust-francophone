## 6\. Durée de vie (ou lifetime)

La durée de vie (lifetime) est l'un des concepts fondamentaux de Rust qui renforce sa sécurité mémoire. Il s'agit d'un mécanisme par lequel le compilateur vérifie que les références restent valides pendant toute la durée de leur utilisation. Bien que le concept puisse sembler complexe au premier abord, sa compréhension est essentielle pour maîtriser Rust.

### Les types de durées de vie

En Rust, il existe principalement deux catégories de durées de vie :

#### 1\. Les durées de vie statiques (`'static`)

Une durée de vie statique signifie que la référence pointe vers une donnée qui vivra pendant toute la durée d'exécution du programme. Il y a plusieurs façons d'obtenir des références avec une durée de vie statique :

``` rust
// À partir de variables statiques
static NOMBRE: i32 = 42;
let reference_statique: &'static i32 = &NOMBRE;

// À partir de constantes
const PI: f64 = 3.14159;
let reference_pi: &'static f64 = &PI;

// Littéraux de chaînes de caractères
let message: &'static str = "Ce message est stocké dans la mémoire du programme";
```

Les références avec une durée de vie statique sont particulièrement utiles pour les données qui ne changent jamais et qui doivent être accessibles tout au long de l'exécution du programme.

#### 2\. Les durées de vie temporaires

La plupart des références en Rust ont une durée de vie temporaire, limitée à un bloc de code spécifique. Le compilateur Rust utilise un système sophistiqué appelé "borrow checker" pour suivre ces durées de vie et s'assurer qu'aucune référence ne pointe jamais vers des données invalidées.

### La syntaxe des durées de vie

Les durées de vie sont généralement indiquées par une apostrophe suivie d'un identifiant : `'a`, `'b`, etc. Dans de nombreux cas, grâce à l'élision (inférence automatique des durées de vie), vous n'avez pas besoin de les spécifier explicitement.

Voici un exemple où nous devons spécifier les durées de vie :

``` rust
// Retourne la plus longue des deux chaînes
fn plus_longue<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("longue chaîne est longue");
    let string2 = String::from("courte");

    let resultat = plus_longue(&string1, &string2);
    println!("La plus longue chaîne est: {}", resultat);
}
```

Dans cet exemple, `'a` représente la durée de vie commune des deux références d'entrée. La fonction retourne une référence avec cette même durée de vie, garantissant ainsi que le résultat ne vivra pas plus longtemps que les données référencées.

### L'élision des durées de vie

Pour simplifier le code, Rust utilise l'élision : des règles qui permettent au compilateur de déduire automatiquement les durées de vie dans certains cas. Voici les règles principales :

1.  Chaque paramètre de référence obtient sa propre durée de vie
2.  S'il n'y a qu'un seul paramètre de référence, sa durée de vie est attribuée à tous les paramètres de retour
3.  Si l'un des paramètres est `&self` ou `&mut self`, sa durée de vie est attribuée aux paramètres de retour

Exemple où l'élision fonctionne :

``` rust
// Ces deux fonctions sont équivalentes
fn première_ligne(s: &str) -> &str {
    s.lines().next().unwrap_or("")
}

// Version avec durées de vie explicites
fn première_ligne_explicite<'a>(s: &'a str) -> &'a str {
    s.lines().next().unwrap_or("")
}
```

### Durées de vie dans les structures

Lorsqu'une structure contient des références, vous devez explicitement spécifier les durées de vie :

``` rust
struct Extrait<'a> {
    texte: &'a str,
    début: usize,
    fin: usize,
}

impl<'a> Extrait<'a> {
    fn nouveau(texte: &'a str, mot_clé: &str) -> Option<Extrait<'a>> {
        let position = texte.find(mot_clé)?;

        // Trouver le début du contexte (max 20 caractères avant)
        let début = texte[..position].char_indices()
            .rev()
            .take(20)
            .last()
            .map_or(0, |(i, _)| i);

        // Trouver la fin du contexte (max 20 caractères après)
        let fin = texte[position..].char_indices()
            .take(20 + mot_clé.chars().count())
            .last()
            .map_or(texte.len(), |(i, c)| position + i + c.len_utf8());

        Some(Extrait {
            texte,
            début,
            fin,
        })
    }

    fn contenu(&self) -> &str {
        &self.texte[self.début..self.fin]
    }
}

fn main() {
    let texte = "Rust est un langage de programmation performant et sécurisé.";

    if let Some(extrait) = Extrait::nouveau(texte, "langage") {
        println!("Extrait trouvé: \"{}\"", extrait.contenu());
    }
}
```

Dans cet exemple, la structure `Extrait` contient une référence vers une chaîne de caractères. La durée de vie `'a` indique que l'instance de `Extrait` ne peut pas vivre plus longtemps que la chaîne référencée.

### Implémentation d'un itérateur avec des durées de vie

Voici un exemple amélioré d'itérateur qui parcourt les mots d'une chaîne de caractères avec une longueur minimale :

``` rust
struct MotMinimum<'a> {
    texte: &'a str,
    position: usize,
    longueur_min: usize,
}

impl<'a> MotMinimum<'a> {
    fn nouveau(texte: &'a str, longueur_min: usize) -> MotMinimum<'a> {
        MotMinimum {
            texte,
            position: 0,
            longueur_min,
        }
    }
}

impl<'a> Iterator for MotMinimum<'a> {
    type Item = &'a str;

    fn next(&mut self) -> Option<Self::Item> {
        // Si nous avons atteint la fin du texte
        if self.position >= self.texte.len() {
            return None;
        }

        // Ignorer les caractères qui ne sont pas alphabétiques
        let début = self.texte[self.position..].find(|c: char| c.is_alphabetic())
            .map(|i| self.position + i)
            .unwrap_or(self.texte.len());

        if début >= self.texte.len() {
            self.position = self.texte.len();
            return None;
        }

        // Trouver la fin du mot
        let fin = self.texte[début..].find(|c: char| !c.is_alphabetic())
            .map(|i| début + i)
            .unwrap_or(self.texte.len());

        // Mettre à jour la position pour le prochain appel
        self.position = fin;

        // Récupérer le mot
        let mot = &self.texte[début..fin];

        // Si le mot est trop court, passer au suivant
        if mot.chars().count() < self.longueur_min {
            return self.next();
        }

        Some(mot)
    }
}

fn main() {
    let texte = "Rust est un langage de programmation très performant et sûr!";
    let mots_longs = MotMinimum::nouveau(texte, 4);

    println!("Mots d'au moins 4 caractères:");
    for mot in mots_longs {
        println!("- {}", mot);
    }
}
```

Cet itérateur utilise une référence au texte d'origine et retourne des références aux mots individuels. La durée de vie `'a` garantit que ces références restent valides tant que le texte d'origine existe.

### Contraintes sur les durées de vie

Dans des cas plus complexes, vous pouvez avoir besoin d'exprimer des relations entre différentes durées de vie. Rust permet de spécifier des contraintes qui indiquent qu'une durée de vie doit être au moins aussi longue qu'une autre.

``` rust
fn fusion<'a, 'b, T>(x: &'a [T], y: &'b [T]) -> Vec<&'a T>
where
    'b: 'a,  // 'b vit au moins aussi longtemps que 'a
    T: PartialOrd,
{
    // Implémentation simplifiée
    let mut résultat = Vec::new();
    for élément in x {
        résultat.push(élément);
    }
    // Seuls les éléments de y qui vivent au moins aussi longtemps que 'a peuvent être inclus
    for élément in y {
        if *élément > *x.last().unwrap_or(&x[0]) && élément as *const _ as usize % 2 == 0 {
            // Ce code est artificiel pour illustrer le concept
            // En pratique, vous ne pourriez pas ajouter directement les éléments de y
            // sans cette contrainte 'b: 'a
        }
    }
    résultat
}
```

Dans cet exemple, nous utilisons la contrainte `'b: 'a` pour indiquer que la durée de vie `'b` doit être au moins aussi longue que `'a`. Cela permet au compilateur de vérifier que nous ne stockons pas de références qui pourraient devenir invalides.

### Le mot-clé `where` pour améliorer la lisibilité

Pour les signatures de fonctions complexes avec plusieurs durées de vie et contraintes, le mot-clé `where` améliore considérablement la lisibilité :

``` rust
fn analyse_complexe<'a, 'b, 'c, T>(
    données_a: &'a [T],
    données_b: &'b [T],
    données_c: &'c [T],
) -> Vec<&'a T>
where
    'b: 'a,          // 'b vit au moins aussi longtemps que 'a
    'c: 'a,          // 'c vit au moins aussi longtemps que 'a
    T: PartialOrd,
{
    // Implémentation...
    Vec::new()
}
```

### Cas pratique : une structure de cache avec références

Voici un exemple plus concret d'utilisation des durées de vie pour implémenter un cache simple qui stocke des références vers des données :

``` rust
// Un cache qui conserve des références vers des résultats calculés
struct Cache<'a, T, U>
where
    T: Eq + std::hash::Hash,
    U: 'a,
{
    calcul: Box<dyn Fn(&T) -> U + 'a>,
    données: std::collections::HashMap<&'a T, &'a U>,
}

impl<'a, T, U> Cache<'a, T, U>
where
    T: Eq + std::hash::Hash,
    U: 'a,
{
    fn nouveau(calcul: impl Fn(&T) -> U + 'a) -> Self {
        Cache {
            calcul: Box::new(calcul),
            données: std::collections::HashMap::new(),
        }
    }

    fn obtenir<'b>(&'b mut self, entrée: &'a T) -> &'b U
    where
        'a: 'b,
    {
        if !self.données.contains_key(entrée) {
            let résultat = (self.calcul)(entrée);
            // Cette ligne est complexe et nécessiterait une implémentation plus sophistiquée
            // pour gérer correctement les durées de vie et le stockage des résultats
            // Le concept est illustratif
        }

        // Cette ligne est simplifiée pour l'exemple
        self.données.get(entrée).unwrap()
    }
}
```

Ce code est intentionnellement incomplet car une implémentation complète nécessiterait une utilisation plus sophistiquée de la gestion mémoire, mais il illustre comment les durées de vie permettent de construire des structures de données sûres qui travaillent avec des références.

### Conclusion

Les durées de vie en Rust peuvent sembler complexes au premier abord, mais elles constituent un mécanisme puissant pour garantir la sécurité mémoire sans avoir recours à un ramasse-miettes. La plupart du temps, l'élision des durées de vie vous évitera d'avoir à les spécifier explicitement, mais comprendre leur fonctionnement vous aidera à résoudre les problèmes lorsque le compilateur ne peut pas les déduire automatiquement.

Avec la pratique, vous deviendrez plus à l'aise avec les durées de vie et apprécierez la sécurité qu'elles apportent à votre code Rust.
