## 11\. **WebAssembly avec Rust** - Compilation vers WASM et interopérabilité avec JavaScript



## Introduction à WebAssembly
WebAssembly (ou WASM) est un format de code binaire qui permet d'exécuter du code compilé à des vitesses proches du natif dans les navigateurs web. Rust est particulièrement bien adapté pour cibler WebAssembly grâce à son système de compilation, sa gestion de la mémoire sans garbage collector, et ses performances élevées.
## Mise en place de l'environnement pour Rust 2024
Pour commencer à développer avec Rust 2024 et WebAssembly, nous avons besoin des outils suivants :
``` bash
# Installation de wasm-pack, l'outil principal pour compiler Rust vers WASM
cargo install wasm-pack

# Installation de cargo-generate pour utiliser des templates
cargo install cargo-generate

# Outils optionnels mais utiles
cargo install wasm-bindgen-cli
cargo install twiggy     # Analyseur de taille de code WASM
```
## Création d'un premier projet WASM avec Rust 2024
Créons un projet simple avec l'édition Rust 2024 :
``` bash
# Création d'un nouveau projet avec cargo
cargo new --lib mon-premier-wasm
cd mon-premier-wasm
```
Modifions le fichier `Cargo.toml` pour utiliser l'édition 2024 :
``` toml
[package]
name = "mon-premier-wasm"
version = "0.1.0"
authors = ["Votre Nom <votre.email@exemple.com>"]
edition = "2024"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
wasm-bindgen = "0.2.91"
console_error_panic_hook = { version = "0.1.7", optional = true }
wee_alloc = { version = "0.4.5", optional = true }

[dev-dependencies]
wasm-bindgen-test = "0.3.41"

[features]
default = ["console_error_panic_hook"]
```
> **Note**: L'édition Rust 2024 apporte plusieurs changements, notamment au niveau des fonctionnalités WebAssembly et de la configuration par défaut des cibles WebAssembly [[1]](https://blog.rust-lang.org/2024/09/24/webassembly-targets-change-in-default-target-features.html).
>

## Structure du projet
La structure du projet généré ressemble à ceci :
```
mon-premier-wasm/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
```
## Code source principal
Créons le fichier `src/lib.rs` avec le code suivant :
``` rust
use wasm_bindgen::prelude::*;

// Lors de la compilation vers WASM, utiliser `wee_alloc` comme allocateur global
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn saluer(nom: &str) {
    alert(&format!("Bonjour, {}!", nom));
}
```
## Compilation du projet pour WebAssembly
Pour compiler notre projet en WebAssembly :
``` bash
wasm-pack build --target web
```
Cette commande génère un répertoire `pkg` contenant :
- Le fichier binaire WebAssembly (`.wasm`)
- Des fichiers JavaScript d'emballage (pour l'interopérabilité)
- Des fichiers TypeScript (définitions de types)
- Un fichier `package.json` pour l'intégration avec npm

## Utilisation du module WASM dans une page web
Créons une page HTML simple pour tester notre module :
``` html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Test WASM avec Rust 2024</title>
</head>
<body>
    <h1>WebAssembly avec Rust 2024</h1>
    <button id="btn-saluer">Saluer</button>

    <script type="module">
        import init, { saluer } from './pkg/mon_premier_wasm.js';

        async function run() {
            // Initialisation du module WASM
            await init();

            // Configuration du bouton
            document.getElementById('btn-saluer').addEventListener('click', () => {
                saluer('ami du WebAssembly');
            });
        }

        run();
    </script>
</body>
</html>
```
## Interopérabilité avec JavaScript
La bibliothèque `wasm-bindgen` reste au cœur de l'interopérabilité entre Rust et JavaScript. Voici quelques exemples d'interaction :
### Passage de données simples
``` rust
use wasm_bindgen::prelude::*;

// Fonction exportée vers JavaScript
#[wasm_bindgen]
pub fn additionner(a: i32, b: i32) -> i32 {
    a + b
}

// Fonction qui reçoit une chaîne et renvoie une chaîne
#[wasm_bindgen]
pub fn inverser_chaine(texte: &str) -> String {
    texte.chars().rev().collect()
}
```
### Utilisation de fonctions JavaScript dans Rust
``` rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    // Importe la fonction console.log de JavaScript
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);

    // Importe Date de JavaScript
    type Date;

    #[wasm_bindgen(constructor)]
    fn new() -> Date;

    #[wasm_bindgen(method, js_name = toLocaleString)]
    fn to_locale_string(this: &Date) -> String;
}

// Utilisation des fonctions importées
#[wasm_bindgen]
pub fn journaliser_date() {
    let date = Date::new();
    let date_str = date.to_locale_string();
    log(&format!("Date actuelle: {}", date_str));
}
```
## Passage de structures complexes avec serde
Pour passer des structures de données complexes entre Rust et JavaScript, nous utilisons généralement la sérialisation JSON avec `serde` :
``` rust
use wasm_bindgen::prelude::*;
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
pub struct Personne {
    nom: String,
    age: u32,
    langages_preferes: Vec<String>,
}

#[wasm_bindgen]
pub fn creer_personne(nom: &str, age: u32) -> JsValue {
    let personne = Personne {
        nom: nom.to_string(),
        age,
        langages_preferes: vec!["Rust".to_string(), "WebAssembly".to_string()],
    };

    // Conversion de la structure Rust en objet JavaScript
    serde_wasm_bindgen::to_value(&personne).unwrap()
}

#[wasm_bindgen]
pub fn traiter_personne(js_personne: JsValue) -> String {
    // Conversion de l'objet JavaScript en structure Rust
    let personne: Personne = serde_wasm_bindgen::from_value(js_personne).unwrap();

    format!(
        "{} a {} ans et aime programmer en {}",
        personne.nom,
        personne.age,
        personne.langages_preferes.join(", ")
    )
}
```
## Utilisation des structures complexes dans JavaScript
Pour utiliser les fonctions de sérialisation/désérialisation dans un fichier JavaScript :
``` javascript
import init, { creer_personne, traiter_personne } from './pkg/mon_premier_wasm.js';

async function run() {
    await init();

    // Obtenir un objet JavaScript depuis Rust
    const personne = creer_personne("Alice", 28);
    console.log(personne);  // {nom: "Alice", age: 28, langages_preferes: ["Rust", "WebAssembly"]}

    // Modifier l'objet en JavaScript
    personne.langages_preferes.push("JavaScript");

    // Repasser l'objet à Rust
    const message = traiter_personne(personne);
    console.log(message);  // "Alice a 28 ans et aime programmer en Rust, WebAssembly, JavaScript"
}

run();
```
Cet exemple montre clairement comment:
1. Utiliser une fonction Rust pour créer un objet JavaScript
2. Manipuler cet objet côté JavaScript
3. Repasser l'objet modifié à une fonction Rust

Cette interopérabilité bidirectionnelle est l'un des grands avantages de l'utilisation de Rust avec WebAssembly et permet de combiner la puissance et la sécurité de Rust avec la flexibilité de JavaScript.


## Manipuler le DOM depuis Rust avec web-sys
`web-sys` est une crate qui fournit des bindings pour l'API Web et permet de manipuler le DOM :
``` rust
use wasm_bindgen::prelude::*;
use web_sys::{Document, Element, HtmlElement, Window};

#[wasm_bindgen]
pub fn ajouter_paragraphe(texte: &str) -> Result<(), JsValue> {
    // Obtenir la fenêtre
    let window = web_sys::window().expect("Pas de fenêtre globale trouvée");

    // Obtenir le document
    let document = window.document().expect("Pas de document trouvé dans la fenêtre");

    // Créer un nouvel élément
    let p = document.create_element("p")?;
    p.set_text_content(Some(texte));

    // Ajouter une classe CSS
    p.set_class_name("generated-by-rust");

    // Ajouter l'élément au corps du document
    let body = document.body().expect("Le document devrait avoir un corps");
    body.append_child(&p)?;

    Ok(())
}
```
## Utilisation de canvas pour les visualisations
Voici un exemple simple de dessin sur un canvas HTML :
``` rust
use wasm_bindgen::prelude::*;
use wasm_bindgen::JsCast;
use web_sys::{CanvasRenderingContext2d, HtmlCanvasElement};

#[wasm_bindgen]
pub fn dessiner_cercle(canvas_id: &str, x: f64, y: f64, rayon: f64) -> Result<(), JsValue> {
    // Obtenir le canvas depuis le DOM
    let document = web_sys::window()
        .unwrap()
        .document()
        .unwrap();

    let canvas = document
        .get_element_by_id(canvas_id)
        .unwrap()
        .dyn_into::<HtmlCanvasElement>()?;

    let context = canvas
        .get_context("2d")?
        .unwrap()
        .dyn_into::<CanvasRenderingContext2d>()?;

    // Configurer le style
    context.set_fill_style(&JsValue::from_str("red"));

    // Dessiner un cercle
    context.begin_path();
    context.arc(x, y, rayon, 0.0, 2.0 * std::f64::consts::PI)?;
    context.fill();

    Ok(())
}
```
## Création d'un jeu simple : Snake (adapté pour Rust 2024)
Voici un exemple complet d'un jeu Snake en Rust 2024 compilé vers WebAssembly :
``` rust
use wasm_bindgen::prelude::*;
use wasm_bindgen::JsCast;
use web_sys::{CanvasRenderingContext2d, HtmlCanvasElement, KeyboardEvent};
use std::cell::RefCell;
use std::rc::Rc;

// Définition de la direction
#[derive(Clone, Copy, PartialEq)]
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

// Définition d'un segment du serpent
#[derive(Clone, Copy)]
struct Segment {
    x: u32,
    y: u32,
}

// État du jeu
struct GameState {
    segments: Vec<Segment>,
    direction: Direction,
    fruit: Segment,
    grid_size: u32,
    cell_size: u32,
    canvas_width: u32,
    canvas_height: u32,
}

impl GameState {
    fn new(canvas_width: u32, canvas_height: u32) -> Self {
        let grid_size = 20;
        let cell_size = canvas_width / grid_size;

        // Créer le serpent initial
        let segments = vec![
            Segment { x: 10, y: 10 },
            Segment { x: 9, y: 10 },
            Segment { x: 8, y: 10 },
        ];

        // Placer le fruit
        let fruit = Segment { x: 15, y: 15 };

        GameState {
            segments,
            direction: Direction::Right,
            fruit,
            grid_size,
            cell_size,
            canvas_width,
            canvas_height,
        }
    }

    fn update(&mut self) -> bool {
        // Obtenir la position de la tête
        let head = self.segments[0];

        // Calculer la nouvelle position de la tête
        let new_head = match self.direction {
            Direction::Up => Segment { x: head.x, y: if head.y == 0 { self.grid_size - 1 } else { head.y - 1 } },
            Direction::Down => Segment { x: head.x, y: (head.y + 1) % self.grid_size },
            Direction::Left => Segment { x: if head.x == 0 { self.grid_size - 1 } else { head.x - 1 }, y: head.y },
            Direction::Right => Segment { x: (head.x + 1) % self.grid_size, y: head.y },
        };

        // Vérifier la collision avec le serpent lui-même
        for segment in &self.segments {
            if new_head.x == segment.x && new_head.y == segment.y {
                return false; // Game over
            }
        }

        // Ajouter la nouvelle tête
        self.segments.insert(0, new_head);

        // Vérifier si le serpent a mangé le fruit
        if new_head.x == self.fruit.x && new_head.y == self.fruit.y {
            // Générer un nouveau fruit
            self.place_new_fruit();
        } else {
            // Retirer le dernier segment
            self.segments.pop();
        }

        true // Le jeu continue
    }

    fn place_new_fruit(&mut self) {
        // Dans Rust 2024, nous pouvons utiliser la fonctionnalité de génération de nombres aléatoires optimisée
        use js_sys::Math;

        loop {
            let x = (Math::random() * (self.grid_size as f64)) as u32;
            let y = (Math::random() * (self.grid_size as f64)) as u32;

            // Vérifier que le fruit n'est pas sur le serpent
            let mut collision = false;
            for segment in &self.segments {
                if segment.x == x && segment.y == y {
                    collision = true;
                    break;
                }
            }

            if !collision {
                self.fruit = Segment { x, y };
                break;
            }
        }
    }

    fn render(&self, context: &CanvasRenderingContext2d) {
        // Effacer le canvas
        context.set_fill_style(&JsValue::from_str("black"));
        context.fill_rect(0.0, 0.0, self.canvas_width as f64, self.canvas_height as f64);

        // Dessiner le serpent
        context.set_fill_style(&JsValue::from_str("green"));
        for segment in &self.segments {
            context.fill_rect(
                (segment.x * self.cell_size) as f64,
                (segment.y * self.cell_size) as f64,
                self.cell_size as f64,
                self.cell_size as f64,
            );
        }

        // Dessiner le fruit
        context.set_fill_style(&JsValue::from_str("red"));
        context.fill_rect(
            (self.fruit.x * self.cell_size) as f64,
            (self.fruit.y * self.cell_size) as f64,
            self.cell_size as f64,
            self.cell_size as f64,
        );
    }

    fn change_direction(&mut self, key: &str) {
        match key {
            "ArrowUp" if self.direction != Direction::Down => self.direction = Direction::Up,
            "ArrowDown" if self.direction != Direction::Up => self.direction = Direction::Down,
            "ArrowLeft" if self.direction != Direction::Right => self.direction = Direction::Left,
            "ArrowRight" if self.direction != Direction::Left => self.direction = Direction::Right,
            _ => {}
        }
    }
}

#[wasm_bindgen]
pub fn start_snake_game(canvas_id: &str) -> Result<(), JsValue> {
    // Obtenir le canvas
    let document = web_sys::window().unwrap().document().unwrap();
    let canvas = document
        .get_element_by_id(canvas_id)
        .unwrap()
        .dyn_into::<HtmlCanvasElement>()?;

    let context = canvas
        .get_context("2d")?
        .unwrap()
        .dyn_into::<CanvasRenderingContext2d>()?;

    // Créer l'état du jeu
    let game_state = Rc::new(RefCell::new(GameState::new(
        canvas.width(),
        canvas.height(),
    )));

    // Gestion des entrées clavier
    let game_state_keydown = game_state.clone();
    let keydown_handler = Closure::wrap(Box::new(move |event: KeyboardEvent| {
        game_state_keydown.borrow_mut().change_direction(&event.key());
    }) as Box<dyn FnMut(KeyboardEvent)>);

    document.add_event_listener_with_callback(
        "keydown",
        keydown_handler.as_ref().unchecked_ref(),
    )?;
    keydown_handler.forget();

    // Boucle de jeu
    let f = Rc::new(RefCell::new(None));
    let g = f.clone();

    let game_state_animation = game_state.clone();
    *g.borrow_mut() = Some(Closure::wrap(Box::new(move || {
        let game_running = game_state_animation.borrow_mut().update();
        game_state_animation.borrow().render(&context);

        if game_running {
            // Continuer la boucle
            request_animation_frame(f.borrow().as_ref().unwrap());
        } else {
            // Game over
            context.set_font("30px Arial");
            context.set_fill_style(&JsValue::from_str("white"));
            context.fill_text(
                "Game Over!",
                (canvas.width() / 2 - 80) as f64,
                (canvas.height() / 2) as f64,
            ).unwrap();
        }
    }) as Box<dyn FnMut()>));

    request_animation_frame(g.borrow().as_ref().unwrap());

    Ok(())
}

fn request_animation_frame(f: &Closure<dyn FnMut()>) {
    web_sys::window()
        .unwrap()
        .request_animation_frame(f.as_ref().unchecked_ref())
        .unwrap();
}
```
## Framework pour développement web complet: Yew avec Rust 2024
Pour les applications plus complexes, le framework Yew permet de créer des applications SPA (Single Page Applications) complètes en Rust 2024 :
``` rust
use yew::prelude::*;

#[function_component]
fn App() -> Html {
    let compteur = use_state(|| 0);
    let onclick = {
        let compteur = compteur.clone();
        Callback::from(move |_| {
            compteur.set(*compteur + 1);
        })
    };

    html! {
        <div>
            <h1>{ "Compteur en Rust 2024+WASM avec Yew" }</h1>
            <button {onclick}>{ "Incrémenter" }</button>
            <p>{ "Valeur actuelle: " }{ *compteur }</p>
        </div>
    }
}

#[wasm_bindgen(start)]
pub fn run_app() {
    yew::Renderer::<App>::new().render();
}
```
## Optimisation de la taille et des performances pour Rust 2024
Pour les applications WebAssembly, la taille du binaire est souvent critique. Voici les optimisations spécifiques pour Rust 2024 :
``` toml
# Cargo.toml
[profile.release]
# Optimisations pour la taille du binaire
opt-level = "z"        # Optimisation pour la taille
lto = true             # Link time optimization
codegen-units = 1      # Meilleure optimisation mais compilation plus lente
panic = "abort"        # Élimination du code de gestion des paniques
strip = true           # Suppression des symboles de débogage

[dependencies]
# Utiliser wee_alloc comme allocateur pour réduire la taille
wee_alloc = "0.4.5"
```
## Débogage du code WebAssembly en Rust 2024
Le débogage du code WebAssembly avec Rust 2024 bénéficie de plusieurs améliorations :
1. **Console logging** amélioré :
``` rust
use web_sys::console;

// Dans votre code
console::log_1(&"Débogage: fonction appelée".into());
console::log_2(&"Valeur: ".into(), &JsValue::from(42));
```
1. **Utilisation du hook de panique** :
``` rust
use console_error_panic_hook;

#[wasm_bindgen]
pub fn init_panic_hook() {
    console_error_panic_hook::set_once();
}
```
1. **Débogage source-level** : avec les outils de développement des navigateurs modernes qui supportent le débogage WebAssembly avec le code source Rust.

## Conclusion
WebAssembly avec Rust 2024 offre plusieurs améliorations par rapport aux éditions précédentes :
1. **Performance optimisée** : exécution à vitesse proche du natif dans le navigateur
2. **Sécurité renforcée** : nouvelles garanties de sécurité mémoire de Rust 2024
3. **Interopérabilité améliorée** : communication bidirectionnelle avec JavaScript plus fluide
4. **Configuration par défaut optimisée** pour les cibles WebAssembly [[1]](https://blog.rust-lang.org/2024/09/24/webassembly-targets-change-in-default-target-features.html)
5. **Écosystème plus riche** : bibliothèques mises à jour comme `wasm-bindgen`, `web-sys`, et frameworks comme Yew

Pour aller plus loin :
- Explorez la documentation officielle de Rust pour WebAssembly
- Découvrez les nouvelles configurations pour les cibles WASI [[2]](https://blog.rust-lang.org/2024/04/09/updates-to-rusts-wasi-targets.html)
- Testez les dernières versions des frameworks comme Yew ou Leptos pour le développement d'applications web
- Analysez les performances de votre code WASM avec les outils de développement des navigateurs

WebAssembly continue son évolution et Rust 2024 renforce sa position comme langage de choix pour cette technologie, offrant un excellent compromis entre performances, sécurité et productivité.
