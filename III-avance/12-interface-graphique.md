# 12\. **Interface graphique** - Introduction aux frameworks GUI disponibles (GTK, Egui, etc.)

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction
L'écosystème Rust continue d'offrir plusieurs options pour développer des interfaces graphiques (GUI). Contrairement à d'autres langages, Rust n'a toujours pas de bibliothèque GUI "officielle", mais dispose d'un écosystème diversifié de frameworks qui évoluent rapidement pour répondre aux différents besoins des développeurs.
## Les principaux frameworks GUI en Rust (2024)
### 1. Egui
Egui reste un framework GUI immédiat (immediate mode) léger, simple à utiliser et multiplateforme, qui a gagné en maturité.
#### Caractéristiques principales:
- Mode immédiat (pas de stockage d'état interne)
- Très simple à démarrer
- Rendu basé sur des primitives vectorielles
- Support multiplateforme incluant le web (via WebAssembly)
- Idéal pour les outils, applications de développement et prototypes

#### Exemple simple avec Egui (version mise à jour):
``` rust
use eframe::{App, Frame, NativeOptions, run_native, CreationContext};
use egui::{Context, CentralPanel, DragValue, Ui};

struct MyApp {
    nom: String,
    age: i32,
}

impl Default for MyApp {
    fn default() -> Self {
        Self {
            nom: "Jean Dupont".to_owned(),
            age: 42,
        }
    }
}

impl App for MyApp {
    fn update(&mut self, ctx: &Context, _frame: &mut Frame) {
        CentralPanel::default().show(ctx, |ui| {
            ui.heading("Bonjour Egui!");

            ui.horizontal(|ui| {
                ui.label("Nom: ");
                ui.text_edit_singleline(&mut self.nom);
            });

            ui.horizontal(|ui| {
                ui.label("Âge: ");
                ui.add(DragValue::new(&mut self.age).speed(0.1));
            });

            if ui.button("Cliquez-moi").clicked() {
                // Action lors du clic
                self.age += 1;
            }

            ui.label(format!("Bonjour, {} ! Vous avez {} ans.", self.nom, self.age));
        });
    }
}

fn main() -> Result<(), eframe::Error> {
    let options = NativeOptions::default();
    run_native(
        "Ma première application Egui",
        options,
        Box::new(|_cc: &CreationContext| Box::new(MyApp::default()))
    )
}
```
Pour utiliser Egui, ajoutez ces dépendances à votre `Cargo.toml`:
``` toml
[dependencies]
eframe = "0.26.2"
egui = "0.26.2"
```
### 2. GTK-rs
GTK-rs reste une liaison (binding) Rust robuste pour la bibliothèque GTK, qui continue d'évoluer avec les nouvelles versions de GTK.
#### Caractéristiques principales:
- Riche en fonctionnalités et mature
- Apparence native sur Linux (et utilisable sur Windows/macOS)
- Interface orientée objet traditionnelle
- Grande communauté et documentation étendue

#### Exemple simple avec GTK-rs (mise à jour pour GTK4):
``` rust
use gtk4::prelude::*;
use gtk4::{Application, ApplicationWindow, Button, Box, Orientation, Entry, Label};

fn main() -> glib::ExitCode {
    let application = Application::new(
        Some("com.exemple.monapp"),
        Default::default(),
    ).expect("Échec de l'initialisation de l'application");

    application.connect_activate(|app| {
        // Créer une fenêtre
        let window = ApplicationWindow::new(app);
        window.set_title(Some("Application GTK en Rust"));
        window.set_default_size(400, 200);

        // Créer un conteneur vertical
        let vbox = Box::new(Orientation::Vertical, 5);
        window.set_child(Some(&vbox));

        // Ajouter un champ texte
        let nom_label = Label::new(Some("Nom:"));
        vbox.append(&nom_label);

        let nom_entry = Entry::new();
        nom_entry.set_text("Jean Dupont");
        vbox.append(&nom_entry);

        // Ajouter un bouton
        let button = Button::with_label("Cliquez-moi");
        vbox.append(&button);

        // Ajouter un label pour afficher le résultat
        let resultat_label = Label::new(Some(""));
        vbox.append(&resultat_label);

        // Connecter le clic du bouton
        button.connect_clicked(clone!(@strong nom_entry, @strong resultat_label => move |_| {
            let nom = nom_entry.text();
            resultat_label.set_text(&format!("Bonjour, {} !", nom));
        }));

        // Afficher tous les widgets
        window.present();
    });

    application.run()
}
```
Pour utiliser GTK-rs, ajoutez ces dépendances à votre `Cargo.toml`:
``` toml
[dependencies]
gtk4 = "0.8.1"
glib = "0.19.2"
```
### 3. Iced
Iced continue de s'améliorer comme framework orienté utilisateur pour créer des applications GUI multiplateformes avec une API déclarative.
#### Caractéristiques principales:
- API déclarative et orientée état
- Architecture inspirée d'Elm (modèle, vue, messages)
- Focus sur la simplicité et une expérience développeur agréable
- Support natif et web (via WebAssembly)

#### Exemple simple avec Iced (mise à jour):
``` rust
use iced::widget::{button, column, text, Container};
use iced::{Alignment, Element, Sandbox, Settings};

struct Compteur {
    valeur: i32,
}

#[derive(Debug, Clone, Copy)]
enum Message {
    IncrementPressed,
    DecrementPressed,
}

impl Sandbox for Compteur {
    type Message = Message;

    fn new() -> Self {
        Self {
            valeur: 0,
        }
    }

    fn title(&self) -> String {
        String::from("Compteur - Application Iced")
    }

    fn update(&mut self, message: Message) {
        match message {
            Message::IncrementPressed => {
                self.valeur += 1;
            }
            Message::DecrementPressed => {
                self.valeur -= 1;
            }
        }
    }

    fn view(&self) -> Element<Message> {
        let contenu = column![
            text(format!("Valeur actuelle: {}", self.valeur)).size(30),
            button("+").on_press(Message::IncrementPressed),
            button("-").on_press(Message::DecrementPressed),
        ]
        .padding(20)
        .spacing(20)
        .align_items(Alignment::Center);

        Container::new(contenu)
            .center_x()
            .center_y()
            .width(iced::Length::Fill)
            .height(iced::Length::Fill)
            .into()
    }
}

fn main() -> iced::Result {
    Compteur::run(Settings::default())
}
```
Pour utiliser Iced, ajoutez cette dépendance à votre `Cargo.toml`:
``` toml
[dependencies]
iced = "0.12.1"
```
### 4. Autres frameworks notables
#### Tauri
Tauri a considérablement gagné en popularité comme alternative à Electron pour créer des applications de bureau multiplateformes en utilisant des technologies web pour l'interface utilisateur et Rust pour la logique backend. [[1]](https://blog.logrocket.com/state-rust-gui-libraries/)
``` toml
[dependencies]
tauri = "2.0.0"
```
#### Slint
Slint a beaucoup évolué et est devenu un toolkit UI moderne de plus en plus populaire, conçu pour les applications embarquées et de bureau. [[1]](https://blog.logrocket.com/state-rust-gui-libraries/)
``` toml
[dependencies]
slint = "1.4.1"
```
#### Xilem
Nouveau venu dans l'écosystème, Xilem est un framework de rendu développé par l'équipe derrière Druid, qui prend de l'ampleur pour son architecture unique. [[1]](https://blog.logrocket.com/state-rust-gui-libraries/)
## Comparaison et choix d'un framework (2024)

| Framework | Type | Popularité | Maturité | Facilité d'apprentissage | Multiplateforme |
| --- | --- | --- | --- | --- | --- |
| Egui | Immédiat | Élevée | Élevée | Très facile | Excellente (incl. Web) |
| GTK-rs | Widget traditionnel | Haute | Très haute | Moyenne | Bonne (meilleure sur Linux) |
| Iced | Déclarative | Haute | Élevée | Facile | Très bonne |
| Tauri | Hybride (Web) | Très haute | Élevée | Moyenne (nécessite HTML/CSS/JS) | Excellente |
| Slint | DSL + Bindings | Élevée | Élevée | Moyenne | Très bonne |
| Xilem | Composants | Croissante | En développement | Moyenne | En développement |
## Conseils pour choisir en 2024
1. **Pour les débutants** ou les petits projets : Egui reste probablement le plus simple pour commencer, avec une communauté grandissante.
2. **Pour les applications professionnelles Linux** : GTK-rs (particulièrement avec GTK4) offre la meilleure intégration native.
3. **Pour les applications multiplateformes complexes** : Tauri est maintenant souvent considéré comme le meilleur choix pour les applications de bureau professionnelles, surtout si votre équipe possède déjà des compétences en développement web. [[1]](https://blog.logrocket.com/state-rust-gui-libraries/)
4. **Pour les interfaces utilisateur embarquées** : Slint a mûri et est devenu un excellent choix pour ce cas d'usage.
5. **Pour les développeurs cherchant une API purement Rust** : Iced offre une expérience déclarative élégante qui a continué à s'améliorer.

## Conclusion
L'écosystème GUI de Rust a considérablement mûri depuis 2021, avec des frameworks établis qui ont gagné en stabilité et en fonctionnalités. Tauri en particulier s'est imposé comme une solution de premier plan pour les applications de bureau multiplateformes en entreprise, tandis que les développeurs travaillant sur des applications embarquées ou des outils spécialisés disposent maintenant d'options plus matures comme Slint et Egui.
Le choix d'un framework GUI en Rust dépend toujours de vos besoins spécifiques, mais la bonne nouvelle est que l'écosystème offre désormais des options plus stables et mieux documentées pour presque tous les cas d'utilisation.

⏭️ [Base de données](/III-avance/13-base-donnees.md) - Connexion et manipulation de bases de données
