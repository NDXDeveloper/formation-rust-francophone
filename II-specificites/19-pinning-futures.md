# 19\. **Pinning et futures** - Concept de pinning pour les types non déplaçables

## Introduction au problème de l'auto-référencement

Dans la programmation asynchrone en Rust, nous rencontrons un problème fondamental : les futures peuvent contenir des références à d'autres données au sein de la même structure. Ces structures sont dites "auto-référencées" car elles contiennent des pointeurs vers leurs propres champs. Ce type de structure pose un problème particulier en Rust car lorsqu'un objet est déplacé en mémoire, ses adresses internes changent, mais les références internes continuent de pointer vers les anciennes adresses.

## Qu'est-ce que le Pinning ?

Le "pinning" (épinglage) est un concept qui permet de résoudre ce problème en garantissant qu'un objet ne sera pas déplacé en mémoire une fois qu'il a été "épinglé" (pinned). C'est crucial pour les futures qui contiennent des références internes.

Le pinning en Rust est implémenté via le type `Pin<P>` où `P` est généralement un pointeur comme `&mut T` ou `Box<T>`.

``` rust
use std::pin::Pin;
```

## Comment fonctionne Pin

?

`Pin<P>` est un wrapper autour d'un pointeur `P` qui garantit que l'objet pointé ne sera pas déplacé. Il offre une garantie essentielle: si `T` est un type qui n'implémente pas `Unpin`, alors les données derrière un `Pin<&mut T>` ou `Pin<Box<T>>` ne seront pas déplacées.

## Le trait Unpin

La plupart des types en Rust implémentent le trait `Unpin` par défaut, ce qui signifie qu'ils peuvent être déplacés même s'ils sont épinglés. Ce trait est un "auto trait" (implémenté automatiquement) pour la majorité des types.

``` rust
// Ce type implémente automatiquement Unpin
struct TypeNormal {
    valeur: i32,
}

// Ce type n'implémente pas Unpin grâce au PhantomPinned
use std::marker::PhantomPinned;
struct TypeNonDeplacable {
    valeur: i32,
    auto_reference: *const i32,
    _marker: PhantomPinned,
}

fn main() {
    let _ = TypeNormal { valeur: 1 };
    let _ = TypeNonDeplacable { valeur: 1, auto_reference: &1, _marker: PhantomPinned };
}
```

## Création d'un type auto-référencé

Voici un exemple simple d'un type auto-référencé qui a besoin de pinning :

``` rust

use std::marker::PhantomPinned;
use std::pin::Pin;

struct AutoRef {
    valeur: String,
    reference: *const String, // Pointeur brut vers valeur
    _marker: PhantomPinned,  // Empêche l'implémentation automatique de Unpin
}

impl AutoRef {
    fn new(texte: &str) -> Self {
        let mut resultat = AutoRef {
            valeur: texte.to_string(),
            reference: std::ptr::null(),
            _marker: PhantomPinned,
        };

        // ATTENTION : Ceci est dangereux sans pinning approprié !
        resultat.reference = &resultat.valeur;

        resultat
    }

    // Cette méthode nécessite un Pin pour être sûre
    fn afficher(self: Pin<&Self>) {
        let self_ptr: *const Self = &*self;
        let valeur_ptr = self.reference;

        // Ici, nous sommes sûrs que ces pointeurs sont valides
        // car l'objet est épinglé et ne sera pas déplacé
        unsafe {
            println!("Valeur auto-référencée: {}", *valeur_ptr);
            println!("Adresse de self: {:p}", self_ptr);
            println!("Adresse de valeur: {:p}", &(*self_ptr).valeur);
        }
    }
}

fn main() {
    let texte = "Hello, world!";
    let auto_ref = AutoRef::new(texte);
    // Épingler l'objet sur le stack pour obtenir Pin<&AutoRef>
    let pinned_auto_ref = unsafe { Pin::new_unchecked(&auto_ref) };

    // Maintenant nous pouvons appeler la méthode afficher()
    pinned_auto_ref.afficher();

}
```

## Épingler un objet avec Box

La façon la plus sûre d'épingler un objet est d'utiliser `Box::pin` :

``` rust
fn main() {
    // Créer et épingler un AutoRef
    let auto_ref = Box::pin(AutoRef::new("Hello, pinning!"));

    // Utilisation sûre puisque l'objet est épinglé
    auto_ref.afficher();
}
```

## Épingler sur la pile (stack)

Épingler des objets sur la pile est plus délicat car l'objet sera détruit à la fin du scope :

``` rust
use std::marker::PhantomPinned;
use std::pin::Pin;

struct AutoRef {
    reference: *const String, // Pointeur brut vers valeur
    valeur: String,

    _marker: PhantomPinned,  // Empêche l'implémentation automatique de Unpin
}

impl AutoRef {
    fn new(texte: &str) -> Self {
        let mut resultat = AutoRef {
            reference: std::ptr::null(),
            valeur: texte.to_string(),

            _marker: PhantomPinned,
        };

        // ATTENTION : Ceci est dangereux sans pinning approprié !
        resultat.reference = &resultat.valeur;

        resultat
    }

    // Cette méthode nécessite un Pin pour être sûre
    fn afficher(self: Pin<&Self>) {
        let self_ptr: *const Self = &*self;
        let valeur_ptr = self.reference;

        // Ici, nous sommes sûrs que ces pointeurs sont valides
        // car l'objet est épinglé et ne sera pas déplacé
        unsafe {
            println!("Valeur auto-référencée: {}", *valeur_ptr);
            println!("Adresse de self: {:p}", self_ptr);
            println!("Adresse de valeur: {:p}", &(*self_ptr).valeur);
        }
    }
}


fn manipuler_sur_pile() {
    let mut auto_ref = AutoRef::new("Sur la pile");

    // Épingler sur place avec des unsafe
    let mut auto_ref_epinglee = unsafe { Pin::new_unchecked(&mut auto_ref) };

    // Maintenant nous pouvons utiliser notre objet épinglé
    auto_ref_epinglee.as_ref().afficher();

    // DANGER: déplacer auto_ref ici serait très problématique
    // car il contient des auto-références.
    // Heureusement, auto_ref est maintenant épinglé, donc il ne peut
    // pas être déplacé tant que auto_ref_epinglee existe.
}

fn main() {
    manipuler_sur_pile();
}
```

## Les futures et le pinning

Le pinning est crucial pour les futures en Rust. Lorsqu'une future est "await-ée", elle est examinée (polled) plusieurs fois, et entre ces examens, elle doit conserver son état. Si la future contient des références à son propre état interne, elle doit être épinglée.

Voici un exemple simplifié d'une future qui aurait besoin de pinning :

``` rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};
use std::ops::Deref;

// Une future qui a besoin de pinning
struct MaFuture {
    valeur: String,
    reference: *const String,  // Auto-référence !
}

impl Future for MaFuture {
    type Output = ();

    fn poll(self: Pin<&mut Self>, _cx: &mut Context<'_>) -> Poll<Self::Output> {
        // Solution 1: Utiliser la méthode deref() comme suggéré par le compilateur
        let self_ref = self.deref();

        // Alternative: Utiliser le pattern de pinning explicite
        // let mut pinned = self;
        // let self_ref = unsafe { Pin::into_inner_unchecked(pinned.as_ref()) };

        unsafe {
            println!("Valeur dans future: {}", *(self_ref.reference));
        }

        Poll::Ready(())
    }
}

// Fonction auxiliaire pour créer notre future
fn creer_future(texte: &str) -> impl Future<Output = ()> {
    let mut future = MaFuture {
        valeur: texte.to_string(),
        reference: std::ptr::null(),
    };

    future.reference = &future.valeur;

    // Nous devons épingler cette future avant de la retourner
    Box::pin(future)
}

fn main() {
    let future = creer_future("Hello world");
}
```

## `Pin` et `async/await`

Avec la syntaxe `async/await`, Rust génère automatiquement des futures qui peuvent contenir des auto-références. Voici un exemple :

``` rust
use std::pin::Pin;
use std::time::Duration;
use tokio::time::sleep;

async fn fonction_async() {
    // Créons une variable que nous allons référencer
    let texte = String::from("Hello, async world!");

    // Référencer 'texte' dans une closure puis attendre
    let reference = &texte;

    // Attendre un peu
    sleep(Duration::from_millis(100)).await;

    // Utiliser la référence - ceci est sûr car la future générée
    // par async/await est correctement épinglée par l'exécuteur (runtime)
    println!("Texte après attente: {}", reference);
}

#[tokio::main]
async fn main() {
    fonction_async().await;
}
```

Dans cet exemple, la future générée par `fonction_async` contient une référence à `texte`, qui est stockée dans la même future. Lorsque nous utilisons `await`, l'exécuteur `tokio` épingle correctement la future pour nous, garantissant ainsi que la référence reste valide.

## Implémentation sécurisée avec `pin-project`

Travailler directement avec `Pin` peut être complexe et sujet aux erreurs. C'est pourquoi il existe une bibliothèque appelée `pin-project` qui simplifie ce travail :

``` rust
use pin_project::pin_project;
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

#[pin_project]
struct StreamFuture<S, F> {
    #[pin]
    stream: S,
    #[pin]
    future: Option<F>,
}

impl<S, F> Future for StreamFuture<S, F>
where
    S: Stream,
    F: Future,
{
    type Output = Option<F::Output>;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        let this = self.project();

        // Accéder aux champs épinglés en toute sécurité
        match this.future {
            Some(fut) => {
                let result = fut.poll(cx);
                // ...
            }
            None => {
                // Traiter le stream
                // ...
            }
        }

        // ...
    }
}
```

La macro `#[pin_project]` génère tout le code nécessaire pour assurer une manipulation sûre des champs marqués avec `#[pin]`.

## Considérations pratiques

1.  **Utilisez des outils comme `pin-project`** : Évitez de manipuler `Pin` directement lorsque c'est possible.

2.  **Épinglez les futures** : Si vous créez des futures manuellement, assurez-vous de les épingler correctement avant de les traiter.

3.  **Évitez l'auto-référencement lorsque possible** : Les structures auto-référencées sont complexes; concevez vos types pour éviter cette complexité si possible.

4.  **Comprenez quand le pinning est nécessaire** : Tous les types n'ont pas besoin d'être épinglés. Seuls les types qui ne sont pas `Unpin` en ont besoin.


## Conclusion

Le concept de pinning en Rust résout un problème fondamental des structures auto-référencées, particulièrement important dans le contexte des futures. Il garantit que les objets qui contiennent des références à leurs propres champs ne seront pas déplacés après avoir été épinglés, préservant ainsi la validité des références internes.

Bien que le pinning puisse sembler complexe au premier abord, il constitue une solution élégante et sûre du point de vue des types pour permettre la programmation asynchrone avec des futures qui peuvent s'auto-référencer. Les abstractions comme `async/await` et des outils comme `pin-project` rendent ce concept beaucoup plus accessible dans la pratique quotidienne.

Le pinning est un exemple de la façon dont Rust résout des problèmes complexes de sécurité mémoire tout en offrant des garanties fortes à la compilation, sans compromettre la performance.
