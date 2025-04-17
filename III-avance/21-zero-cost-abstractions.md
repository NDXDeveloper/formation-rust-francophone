## 21. **Zero-cost abstractions avancées**
Les zero-cost abstractions sont l'un des principes fondamentaux de Rust, permettant d'écrire du code de haut niveau sans impact sur les performances à l'exécution. C'est l'un des aspects les plus puissants du langage : la capacité à créer des abstractions élégantes qui disparaissent lors de la compilation.
### 21.1. Principe des zero-cost abstractions
Une zero-cost abstraction est une abstraction dont l'utilisation n'entraîne aucun surcoût à l'exécution par rapport à une implémentation manuelle de bas niveau. En d'autres termes, vous ne payez que pour ce que vous utilisez, et vous ne payez pas plus cher que si vous l'aviez écrit vous-même en code de bas niveau.
``` rust
// Cette abstraction (iterator) se compile en code aussi efficace
// qu'une boucle for classique avec des indices
let sum: u32 = (0..1000).filter(|x| x % 2 == 0).sum();

// Équivalent conceptuel à cette boucle manuelle
let mut sum = 0;
for i in 0..1000 {
    if i % 2 == 0 {
        sum += i;
    }
}
```
### 21.2. Inlining et optimisations du compilateur
Le compilateur Rust utilise LLVM pour effectuer des optimisations agressives, notamment l'inlining des fonctions. Cela permet d'éliminer les appels de fonction pour des abstractions simples.
``` rust
#[inline]
fn double(x: u32) -> u32 {
    x * 2
}

fn main() {
    let value = 5;
    let doubled = double(value);
    // Après compilation, c'est comme si on avait écrit:
    // let doubled = value * 2;
}
```
### 21.3. Monomorphisation des génériques
Rust utilise la monomorphisation pour les types génériques, générant du code spécifique pour chaque instanciation concrète d'un type générique, ce qui évite le coût du dynamic dispatch.
``` rust
struct Wrapper<T> {
    value: T,
}

impl<T> Wrapper<T> {
    fn new(value: T) -> Self {
        Wrapper { value }
    }

    fn get(&self) -> &T {
        &self.value
    }
}

fn main() {
    // Le compilateur génère du code spécifique pour Wrapper<i32>
    let w1 = Wrapper::new(42);

    // Et du code séparé pour Wrapper<String>
    let w2 = Wrapper::new(String::from("hello"));
}
```
### 21.4. Utilisation stratégique des traits bounds
Les trait bounds permettent de créer des abstractions performantes en contraignant les types génériques à implémenter certaines fonctionnalités.
``` rust
// Un trait pour les types qui peuvent être copiés bit à bit
trait BitCopy: Copy {
    fn bit_copy(&self) -> Self;
}

impl BitCopy for u32 {
    fn bit_copy(&self) -> Self {
        *self
    }
}

// Cette fonction est optimisée pour les types qui implémentent Copy
fn duplicate<T: BitCopy>(value: T) -> (T, T) {
    (value, value.bit_copy())
}
```
### 21.5. Les types fantômes (Phantom Types)
Les types fantômes permettent d'ajouter des contraintes de type au moment de la compilation sans coût à l'exécution.
``` rust
use std::marker::PhantomData;

// États d'une connexion
struct Open;
struct Closed;

// Connexion typée par son état
struct Connection<State> {
    // Aucune donnée réelle de type State n'est stockée
    _state: PhantomData<State>,
    socket_fd: i32,
}

impl Connection<Closed> {
    fn new(socket_fd: i32) -> Self {
        Connection {
            _state: PhantomData,
            socket_fd,
        }
    }

    fn open(self) -> Connection<Open> {
        // Logique d'ouverture...
        Connection {
            _state: PhantomData,
            socket_fd: self.socket_fd,
        }
    }
}

impl Connection<Open> {
    fn send_data(&self, data: &[u8]) -> Result<(), String> {
        // Implémentation...
        Ok(())
    }

    fn close(self) -> Connection<Closed> {
        // Logique de fermeture...
        Connection {
            _state: PhantomData,
            socket_fd: self.socket_fd,
        }
    }
}
```
Ce code garantit à la compilation qu'une connexion fermée ne peut pas envoyer de données, sans aucun coût à l'exécution.
### 21.6. Combinateurs de fonctions sans allocation
Les combinateurs de fonctions permettent de composer des comportements sans créer d'allocations supplémentaires. Avec Rust 2024, nous pouvons désormais utiliser des fermetures asynchrones [[1]](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html), ce qui rend cette approche encore plus puissante :
``` rust
// Implémentation simplifiée d'un Result-like pour illustration
enum MyResult<T, E> {
    Ok(T),
    Err(E),
}

impl<T, E> MyResult<T, E> {
    fn map<U, F>(self, f: F) -> MyResult<U, E>
    where
        F: FnOnce(T) -> U,
    {
        match self {
            MyResult::Ok(t) => MyResult::Ok(f(t)),
            MyResult::Err(e) => MyResult::Err(e),
        }
    }

    // Nouvelle version pour Rust 2024 supportant l'async
    async fn map_async<U, F, Fut>(self, f: F) -> MyResult<U, E>
    where
        F: FnOnce(T) -> Fut,
        Fut: std::future::Future<Output = U>,
    {
        match self {
            MyResult::Ok(t) => MyResult::Ok(f(t).await),
            MyResult::Err(e) => MyResult::Err(e),
        }
    }

    fn and_then<U, F>(self, f: F) -> MyResult<U, E>
    where
        F: FnOnce(T) -> MyResult<U, E>,
    {
        match self {
            MyResult::Ok(t) => f(t),
            MyResult::Err(e) => MyResult::Err(e),
        }
    }
}

// Exemple d'utilisation avec async closures (Rust 2024)
async fn process_data(input: MyResult<String, String>) -> MyResult<usize, String> {
    input.map_async(async |s| {
        // Traitement asynchrone
        tokio::time::sleep(std::time::Duration::from_millis(10)).await;
        s.len()
    }).await
}
```
### 21.7. Itérateurs personnalisés sans allocation
Les itérateurs sont l'un des meilleurs exemples de zero-cost abstractions en Rust. En 2024 Edition, nous bénéficions de l'ajout de et `Extend` pour les tuples de paires, rendant les opérations sur les paires encore plus efficaces [[1]](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html) : `FromIterator`
``` rust
struct Fibonacci {
    current: u64,
    next: u64,
}

impl Fibonacci {
    fn new() -> Self {
        Fibonacci { current: 0, next: 1 }
    }
}

impl Iterator for Fibonacci {
    type Item = u64;

    fn next(&mut self) -> Option<Self::Item> {
        let current = self.current;
        self.current = self.next;
        self.next = current + self.next;
        Some(current)
    }
}

fn main() {
    // Cette séquence n'alloue aucune mémoire sur le tas
    let fib_sum: u64 = Fibonacci::new()
        .take(10)
        .filter(|&x| x % 2 == 0)
        .sum();

    // Nouveau en Edition 2024 - utilisation facilitée des itérateurs de paires
    let pairs = vec![(1, "un"), (2, "deux"), (3, "trois")];

    // Extraction des deux collections en une seule opération
    let (numbers, words): (Vec<i32>, Vec<&str>) = pairs.iter()
        .map(|&(num, word)| (num, word))
        .collect(); // Utilise FromIterator pour (T, U)
}
```
### 21.8. Utilisation des const generics pour les tableaux de taille fixe
Les const generics permettent de créer des abstractions sur des tableaux de taille fixe sans surcoût à l'exécution:
``` rust
struct Matrix<const N: usize, const M: usize> {
    data: [[f64; M]; N],
}

impl<const N: usize, const M: usize> Matrix<N, M> {
    fn new() -> Self {
        Matrix {
            data: [[0.0; M]; N],
        }
    }

    fn get(&self, i: usize, j: usize) -> Option<f64> {
        if i < N && j < M {
            Some(self.data[i][j])
        } else {
            None
        }
    }
}

fn main() {
    // Matrices de tailles différentes partageant la même API
    let matrix_3x3: Matrix<3, 3> = Matrix::new();
    let matrix_4x2: Matrix<4, 2> = Matrix::new();
}
```
### 21.9. Expression Builder Pattern
Le modèle Builder peut être implémenté de manière à ce que toutes les vérifications soient effectuées à la compilation. Avec Rust 2024, nous pouvons également tirer parti du trait `Future` désormais disponible dans le prelude [[1]](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html) pour faciliter les builders asynchrones :
``` rust
struct Request {
    method: String,
    url: String,
    headers: Vec<(String, String)>,
    body: Option<Vec<u8>>,
}

struct RequestBuilder {
    method: String,
    url: String,
    headers: Vec<(String, String)>,
    body: Option<Vec<u8>>,
}

impl RequestBuilder {
    fn new(method: &str, url: &str) -> Self {
        RequestBuilder {
            method: method.to_string(),
            url: url.to_string(),
            headers: Vec::new(),
            body: None,
        }
    }

    fn header(mut self, name: &str, value: &str) -> Self {
        self.headers.push((name.to_string(), value.to_string()));
        self
    }

    fn body(mut self, body: Vec<u8>) -> Self {
        self.body = Some(body);
        self
    }

    fn build(self) -> Request {
        Request {
            method: self.method,
            url: self.url,
            headers: self.headers,
            body: self.body,
        }
    }

    // Nouvelle méthode pour Rust 2024: construction et envoi asynchrone
    async fn send(self) -> Result<reqwest::Response, reqwest::Error> {
        let client = reqwest::Client::new();
        let mut request_builder = match self.method.as_str() {
            "GET" => client.get(&self.url),
            "POST" => client.post(&self.url),
            // Autres méthodes...
            _ => client.request(
                reqwest::Method::from_bytes(self.method.as_bytes()).unwrap(),
                &self.url
            ),
        };

        // Ajouter les headers
        for (name, value) in &self.headers {
            request_builder = request_builder.header(name, value);
        }

        // Ajouter le body si présent
        if let Some(body) = self.body {
            request_builder = request_builder.body(body);
        }

        // Envoyer la requête
        request_builder.send().await
    }
}

// Exemple d'utilisation avec async/await
async fn fetch_data() -> Result<String, Box<dyn std::error::Error>> {
    let response = RequestBuilder::new("GET", "https://api.example.com")
        .header("Accept", "application/json")
        .send()
        .await?;

    Ok(response.text().await?)
}
```
### 21.10. Utilisation des changements de règles de capture de durée de vie RPIT
Avec Rust 2024, les règles de capture des paramètres par les types impl Trait ont été modifiées lorsque `use<..>` n'est pas présent [[1]](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html), ce qui peut être utilisé pour créer des API plus expressives :
``` rust
// En Rust 2024, cette fonction peut retourner un itérateur qui emprunte le slice
// sans avoir besoin d'une annotation de durée de vie explicite
fn iter_filtered(nums: &[i32]) -> impl Iterator<Item = &i32> {
    nums.iter().filter(|&&x| x > 0)
}

fn main() {
    let numbers = vec![1, -2, 3, -4, 5];

    // Utilisation de l'itérateur
    for &num in iter_filtered(&numbers) {
        println!("Nombre positif: {}", num);
    }

    // Avec Rust 2024, les règles de capture des durées de vie sont améliorées
    // pour les types `impl Trait` dans les types de retour
}
```
### 21.11. Bilan et considérations pratiques
Les zero-cost abstractions de Rust vous permettent d'écrire du code expressif et sûr sans compromis sur les performances. Avec Rust 2024, cette philosophie s'étend encore davantage. Voici quelques conseils pratiques:
1. Utilisez les génériques au lieu du dynamic dispatch lorsque possible
2. Profitez des traits et trait bounds pour exprimer des contraintes
3. Laissez le compilateur optimiser votre code plutôt que de l'optimiser prématurément
4. Utilisez les types fantômes pour ajouter de la sécurité au moment de la compilation
5. Préférez les itérateurs aux boucles explicites pour un code plus expressif
6. Tirez parti des fermetures asynchrones (async closures) introduites dans Rust 2024
7. Utilisez les nouveaux traits ajoutés au prelude (Future, IntoFuture) pour des API plus ergonomiques
``` rust
// Un exemple combinant plusieurs techniques de Rust 2024
async fn process_data<T, F, Fut, R>(data: &[T], transform: F) -> Vec<R>
where
    F: Fn(&T) -> Fut,
    Fut: std::future::Future<Output = Option<R>>,
{
    let mut results = Vec::new();

    for item in data {
        if let Some(result) = transform(item).await {
            results.push(result);
        }
    }

    results
}

// Utilisation avec async closures
async fn example() {
    let data = vec![1, 2, 3, 4, 5];

    let results = process_data(&data, async |&x| {
        if x % 2 == 0 {
            Some(x * 2)
        } else {
            None
        }
    }).await;

    println!("Résultats: {:?}", results);
}
```
Ces abstractions disparaissent complètement après compilation, produisant un code aussi performant que si vous aviez écrit chaque opération manuellement, tout en profitant des nouvelles fonctionnalités de Rust 2024.
