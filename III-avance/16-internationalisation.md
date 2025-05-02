# 16. Internationalisation et localisation pour Rust 2024 edition

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Je vais analyser et mettre à jour ce tutoriel pour la Rust 2024 edition. Pour cela, je dois d'abord vérifier les versions actuelles des bibliothèques d'internationalisation pour Rust.# 16. Internationalisation et localisation

L'internationalisation (i18n) et la localisation (l10n) sont essentielles pour développer des applications accessibles à un public international. Dans cette section, nous allons explorer comment gérer les langues et formats internationaux en Rust 2024 edition.

## 16.1. Introduction à l'i18n et l10n

L'internationalisation consiste à concevoir votre application de manière à ce qu'elle puisse être adaptée à différentes langues et régions, tandis que la localisation est le processus d'adaptation de l'application à une région ou langue spécifique.

En Rust, plusieurs bibliothèques facilitent ce travail :

- `fluent-rs` : Pour la gestion des chaînes de caractères localisées
- `rust-i18n` : Une bibliothèque simple basée sur les macros
- `gettext-rs` : Une interface pour le système GNU gettext
- `icu` : Pour le formatage des dates, nombres, pluriels, etc. (remplace icu4x)

## 16.2. Installation des bibliothèques nécessaires

Ajoutons les dépendances nécessaires à notre fichier `Cargo.toml` :

```toml
[dependencies]
fluent = "0.16.1"
fluent-bundle = "0.15.2"
unic-langid = "0.9.5"
```


## 16.3. Utilisation de Fluent pour la traduction

Fluent est une solution moderne pour l'internationalisation développée par Mozilla. Voici comment l'utiliser :

```rust
use fluent::{FluentBundle, FluentResource};
use std::collections::HashMap;
use fluent::FluentValue;
use unic_langid::langid;

fn main() {
    // 1. Création d'une ressource Fluent avec des messages
    let ftl_string = r#"
welcome = Bienvenue, { $name } !
items = { $count ->
    [one] Vous avez 1 article
   *[other] Vous avez { $count } articles
}
    "#;

    let resource = FluentResource::try_new(ftl_string.to_string())
        .expect("Erreur de syntaxe dans les messages FTL");

    // 2. Création d'un bundle pour le français
    let langid_fr = langid!("fr");
    let mut bundle = FluentBundle::new(vec![langid_fr]);

    // 3. Ajout de la ressource au bundle
    bundle.add_resource(resource)
        .expect("Échec de l'ajout de la ressource au bundle");

    // 4. Formatage d'un message avec des paramètres
    let msg = bundle.get_message("welcome")
        .expect("Message 'welcome' non trouvé");

    let mut args = HashMap::new();
    args.insert("name".to_string(), FluentValue::from("Marie"));

    let pattern = msg.value().expect("Message sans valeur");

    let mut errors = vec![];
    let value = bundle.format_pattern(pattern, Some(&args), &mut errors);
    println!("{}", value); // Affiche: "Bienvenue, Marie !"

    // 5. Gestion du pluriel
    let msg_items = bundle.get_message("items")
        .expect("Message 'items' non trouvé");
    let pattern_items = msg_items.value().expect("Message sans valeur");

    let mut args_one = HashMap::new();
    args_one.insert("count".to_string(), FluentValue::from(1));

    let mut args_multiple = HashMap::new();
    args_multiple.insert("count".to_string(), FluentValue::from(5));

    let value_one = bundle.format_pattern(pattern_items, Some(&args_one), &mut errors);
    let value_multiple = bundle.format_pattern(pattern_items, Some(&args_multiple), &mut errors);

    println!("{}", value_one);      // Affiche: "Vous avez 1 article"
    println!("{}", value_multiple); // Affiche: "Vous avez 5 articles"
}
```


## 16.4. Organisation des fichiers de traduction

Pour une application réelle, il est préférable d'organiser les traductions dans des fichiers séparés :

```
locales/
  ├── fr/
  │    └── messages.ftl
  └── en/
       └── messages.ftl
```


Voici comment charger les traductions depuis des fichiers :

```rust
use std::fs;
use std::path::Path;
use fluent::{FluentBundle, FluentResource};
use unic_langid::LanguageIdentifier;

fn load_fluent_bundle(lang_code: &str) -> FluentBundle<FluentResource> {
    // Construire le chemin vers le fichier
    let file_path = format!("locales/{}/messages.ftl", lang_code);

    // Lire le contenu du fichier
    let source = fs::read_to_string(Path::new(&file_path))
        .expect(&format!("Impossible de lire le fichier {}", file_path));

    // Créer une ressource à partir du contenu
    let resource = FluentResource::try_new(source)
        .expect("Erreur de syntaxe dans les messages");

    // Créer le bundle pour la langue spécifiée
    let lang_id: LanguageIdentifier = lang_code.parse()
        .expect("Identifiant de langue invalide");

    let mut bundle = FluentBundle::new(vec![lang_id]);

    // Ajouter la ressource au bundle
    bundle.add_resource(resource)
        .expect("Échec de l'ajout de la ressource au bundle");

    bundle
}
```


## 16.5. Formatage des nombres et dates

Pour le formatage des nombres et des dates selon les conventions locales, la bibliothèque `icu` est recommandée (qui remplace `icu4x`) :

```toml
[dependencies]
icu = "2.0.0"
```


Exemple de formatage de nombres et dates :

```rust
use icu::decimal::{FixedDecimalFormatter, options::FixedDecimalFormatterOptions};
use icu::datetime::{DateTimeFormatter, options::DateTimeFormatterOptions};
use icu::locale::locale;

fn main() {
    // Formatage de nombres
    let locale_fr = locale!("fr");
    let locale_us = locale!("en-US");

    let formatter_fr = FixedDecimalFormatter::try_new(
        &locale_fr,
        FixedDecimalFormatterOptions::default()
    ).expect("Erreur de création du formateur de nombres");

    let formatter_us = FixedDecimalFormatter::try_new(
        &locale_us,
        FixedDecimalFormatterOptions::default()
    ).expect("Erreur de création du formateur de nombres");

    let number = 1234567.89;

    println!("Nombre en format français: {}", formatter_fr.format(number));
    // Affiche: 1 234 567,89

    println!("Nombre en format américain: {}", formatter_us.format(number));
    // Affiche: 1,234,567.89

    // Formatage de dates
    let date_formatter_fr = DateTimeFormatter::try_new(
        &locale_fr,
        DateTimeFormatterOptions::default()
    ).expect("Erreur de création du formateur de dates");

    let date_formatter_us = DateTimeFormatter::try_new(
        &locale_us,
        DateTimeFormatterOptions::default()
    ).expect("Erreur de création du formateur de dates");

    let now = time::OffsetDateTime::now_utc();

    println!("Date en format français: {}", date_formatter_fr.format(&now));
    // Exemple: 12 juin 2023 15:30:45

    println!("Date en format américain: {}", date_formatter_us.format(&now));
    // Exemple: June 12, 2023 3:30:45 PM
}
```


## 16.6. Détection de la langue du système

Pour détecter la langue préférée de l'utilisateur, vous pouvez utiliser la variable d'environnement `LANG` ou `LC_ALL` :

```rust
use std::env;

fn detect_user_language() -> String {
    // Essayer LANG d'abord, puis LC_ALL, avec une valeur par défaut "en"
    env::var("LANG")
        .or_else(|_| env::var("LC_ALL"))
        .unwrap_or_else(|_| String::from("en"))
        .split('.')
        .next()
        .unwrap_or("en")
        .split('_')
        .next()
        .unwrap_or("en")
        .to_string()
}

fn main() {
    let lang = detect_user_language();
    println!("Langue détectée : {}", lang);
}
```


## 16.7. Création d'une structure d'internationalisation

Créons une structure pour gérer l'internationalisation de manière plus organisée :

```rust
use std::collections::HashMap;
use fluent::{FluentBundle, FluentResource, FluentValue};
use unic_langid::LanguageIdentifier;
use std::fs;

pub struct I18n {
    bundles: HashMap<String, FluentBundle<FluentResource>>,
    current_lang: String,
}

impl I18n {
    pub fn new(default_lang: &str, available_langs: &[&str]) -> Self {
        let mut bundles = HashMap::new();

        for lang in available_langs {
            let file_path = format!("locales/{}/messages.ftl", lang);
            let source = fs::read_to_string(&file_path)
                .expect(&format!("Impossible de lire le fichier {}", file_path));

            let resource = FluentResource::try_new(source)
                .expect("Erreur de syntaxe dans les messages");

            let lang_id: LanguageIdentifier = lang.parse()
                .expect("Identifiant de langue invalide");

            let mut bundle = FluentBundle::new(vec![lang_id]);
            bundle.add_resource(resource)
                .expect("Échec de l'ajout de la ressource au bundle");

            bundles.insert(lang.to_string(), bundle);
        }

        I18n {
            bundles,
            current_lang: default_lang.to_string(),
        }
    }

    pub fn set_language(&mut self, lang: &str) -> bool {
        if self.bundles.contains_key(lang) {
            self.current_lang = lang.to_string();
            true
        } else {
            false
        }
    }

    pub fn get_message(&self, id: &str, args: Option<HashMap<String, FluentValue>>) -> String {
        if let Some(bundle) = self.bundles.get(&self.current_lang) {
            if let Some(msg) = bundle.get_message(id) {
                if let Some(pattern) = msg.value() {
                    let mut errors = vec![];
                    let value = bundle.format_pattern(pattern, args.as_ref(), &mut errors);
                    return value.to_string();
                }
            }
        }

        // Fallback : retourner l'identifiant si le message n'est pas trouvé
        format!("#{}", id)
    }
}
```


Utilisation :

```rust
fn main() {
    // Initialiser avec le français comme langue par défaut
    let mut i18n = I18n::new("fr", &["fr", "en"]);

    // Message simple
    println!("{}", i18n.get_message("welcome", None));

    // Message avec paramètres
    let mut args = HashMap::new();
    args.insert("name".to_string(), FluentValue::from("Pierre"));
    println!("{}", i18n.get_message("hello", Some(args)));

    // Changer de langue
    i18n.set_language("en");

    // Même message en anglais
    let mut args = HashMap::new();
    args.insert("name".to_string(), FluentValue::from("Peter"));
    println!("{}", i18n.get_message("hello", Some(args)));
}
```


## 16.8. Internationalisation dans une application CLI

Voici un exemple d'application CLI simple avec internationalisation :

```rust
use std::io::{self, Write};
use std::collections::HashMap;
use fluent::FluentValue;

fn main() {
    // Initialiser l'i18n
    let mut i18n = I18n::new("fr", &["fr", "en"]);

    // Demander la langue préférée
    println!("Choisissez une langue / Choose a language (fr/en):");

    io::stdout().flush().unwrap();
    let mut input = String::new();
    io::stdin().read_line(&mut input).unwrap();

    let lang = input.trim().to_lowercase();
    if !i18n.set_language(&lang) {
        println!("Langue non disponible, utilisation du français par défaut");
    }

    // Afficher un message de bienvenue
    println!("\n{}", i18n.get_message("welcome", None));

    // Demander le nom
    print!("{} ", i18n.get_message("ask_name", None));
    io::stdout().flush().unwrap();

    let mut name = String::new();
    io::stdin().read_line(&mut name).unwrap();
    let name = name.trim();

    // Saluer l'utilisateur
    let mut args = HashMap::new();
    args.insert("name".to_string(), FluentValue::from(name));
    println!("{}", i18n.get_message("hello", Some(args)));

    // Montrer différents formats de nombres
    let number = 1234.56;
    let mut args = HashMap::new();
    args.insert("number".to_string(), FluentValue::from(number));
    println!("{}", i18n.get_message("number_format", Some(args)));
}
```


Les fichiers de traduction pourraient ressembler à ceci :

**locales/fr/messages.ftl**:

```
welcome = Bienvenue dans notre application!
ask_name = Comment vous appelez-vous?
hello = Bonjour, { $name } !
number_format = En français, { $number } s'écrit: { NUMBER($number, style: "currency", currency: "EUR") }
```


**locales/en/messages.ftl**:

```
welcome = Welcome to our application!
ask_name = What is your name?
hello = Hello, { $name }!
number_format = In English, { $number } is written: { NUMBER($number, style: "currency", currency: "USD") }
```


## 16.9. Support de l'async dans Rust 2024

Avec l'édition 2024 de Rust, vous pouvez utiliser l'internationalisation dans un contexte asynchrone plus facilement grâce au support intégré des fermetures asynchrones (`async closures`) et de `Future` dans le prélude :

```rust
use fluent::{FluentBundle, FluentResource};
use std::collections::HashMap;
use fluent::FluentValue;
use unic_langid::langid;

async fn load_translations(lang_code: &str) -> FluentBundle<FluentResource> {
    // Cette fonction pourrait charger les traductions depuis un serveur
    // ou une base de données de manière asynchrone
    let ftl_string = match lang_code {
        "fr" => r#"
            welcome = Bienvenue, { $name } !
            items = { $count ->
                [one] Vous avez 1 article
               *[other] Vous avez { $count } articles
            }
        "#.to_string(),
        _ => r#"
            welcome = Welcome, { $name }!
            items = { $count ->
                [one] You have 1 item
               *[other] You have { $count } items
            }
        "#.to_string(),
    };

    let resource = FluentResource::try_new(ftl_string)
        .expect("Erreur de syntaxe dans les messages FTL");

    let lang_id = lang_code.parse()
        .expect("Identifiant de langue invalide");

    let mut bundle = FluentBundle::new(vec![lang_id]);
    bundle.add_resource(resource)
        .expect("Échec de l'ajout de la ressource au bundle");

    bundle
}

async fn main() {
    let bundle = load_translations("fr").await;

    // Utiliser une fermeture asynchrone (nouveauté Rust 2024)
    let translate = async |msg_id: &str, name: &str| {
        let msg = bundle.get_message(msg_id).expect("Message non trouvé");
        let pattern = msg.value().expect("Message sans valeur");

        let mut args = HashMap::new();
        args.insert("name".to_string(), FluentValue::from(name));

        let mut errors = vec![];
        bundle.format_pattern(pattern, Some(&args), &mut errors).to_string()
    };

    let greeting = translate("welcome", "Sophie").await;
    println!("{}", greeting);
}
```


## 16.10. Meilleures pratiques

Pour créer une application bien internationalisée :

1. **Séparation des préoccupations** : Gardez toutes les chaînes de caractères dans des fichiers de ressources séparés.
2. **Conception pour la pluralisation** : Utilisez les fonctionnalités de pluralisation pour gérer correctement le singulier et le pluriel.
3. **Contexte** : Fournissez un contexte pour les traducteurs afin qu'ils comprennent comment une chaîne est utilisée.
4. **Tests** : Testez votre application avec différentes langues et paramètres régionaux.
5. **Formats adaptables** : Assurez-vous que vos interfaces utilisateur peuvent s'adapter à des chaînes de caractères de longueurs variables.
6. **Abstraction** : Utilisez une couche d'abstraction pour faciliter le changement de système d'internationalisation si nécessaire.
7. **Utilisez les nouveautés de Rust 2024** : Profitez des fonctionnalités asynchrones pour une gestion plus efficace des ressources.

## 16.11. Conclusion

L'internationalisation et la localisation sont des aspects importants pour rendre votre application accessible à un public international. Rust 2024 édition offre plusieurs bibliothèques robustes et à jour pour gérer ces aspects, permettant de créer des applications vraiment multilingues et adaptées aux conventions locales.

En suivant les bonnes pratiques d'internationalisation dès le début du développement, vous pourrez éviter de nombreux problèmes liés à l'adaptation de votre application pour différentes régions du monde.

⏭️ [Sérialisation et désérialisation](/III-avance/17-serialisation-deserialisation.md) - Utilisation de serde pour JSON, YAML, etc.
