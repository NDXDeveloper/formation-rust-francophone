# 17\. Sérialisation et désérialisation

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

La sérialisation est le processus de conversion d'objets en mémoire vers un format de données qui peut être stocké ou transmis. La désérialisation est le processus inverse. Dans Rust, l'écosystème `serde` fournit des outils puissants pour ces opérations.

## 17.1. Introduction à Serde
Serde (SERialization/DEserialization) reste le framework de référence pour la sérialisation et la désérialisation de données en Rust. Ses points forts demeurent :
- Performances optimales avec l'approche sans allocation (zero-copy) quand possible
- Support de nombreux formats (JSON, TOML, etc.)
- Code efficace généré à la compilation
- Absence de dépendance à un système de réflexion à l'exécution

## 17.2. Configuration de base
Pour utiliser Serde avec Rust 2024 Edition, ajoutez les dépendances suivantes dans votre fichier `Cargo.toml` :
``` toml
[package]
name = "mon_projet"
version = "0.1.0"
edition = "2024"

[dependencies]
serde = { version = "1.0.219", features = ["derive"] }
serde_json = "1.0"
toml = "0.8"  # Version plus récente disponible
```
À noter que le support YAML via `serde_yaml` est désormais déprécié [[8]](https://users.rust-lang.org/t/serde-yaml-deprecation-alternatives/108868), mais il existe des alternatives que nous verrons plus loin.
## 17.3. Sérialisation et désérialisation avec JSON
Les principes fondamentaux restent les mêmes, mais nous pouvons mettre à jour l'exemple en utilisant les fonctionnalités de Rust 2024 :
``` rust
use serde::{Deserialize, Serialize};
use std::error::Error;
use std::fs::File;
use std::io::{Read, Write};

// Définition d'une structure que nous voulons sérialiser
#[derive(Serialize, Deserialize, Debug)]
struct Utilisateur {
    nom: String,
    age: u32,
    email: String,
    actif: bool,
    preferences: Vec<String>,
}

async fn enregistrer_utilisateur(utilisateur: &Utilisateur) -> Result<(), Box<dyn Error>> {
    // Utilisation de async/await, disponible dans Rust 2024
    let json = serde_json::to_string_pretty(utilisateur)?;
    // Reste du code pour écrire dans un fichier...
    Ok(())
}

fn main() -> Result<(), Box<dyn Error>> {
    // Création d'un utilisateur
    let utilisateur = Utilisateur {
        nom: String::from("Sophie Martin"),
        age: 28,
        email: String::from("sophie.martin@example.com"),
        actif: true,
        preferences: vec![
            String::from("Programmation"),
            String::from("Lecture"),
            String::from("Randonnée"),
        ],
    };

    // Sérialisation en JSON formaté
    let json_pretty = serde_json::to_string_pretty(&utilisateur)?;
    println!("JSON (formaté):\n{}", json_pretty);

    // Sérialisation en JSON compact
    let json = serde_json::to_string(&utilisateur)?;
    println!("JSON (compact): {}", json);

    // Écriture dans un fichier
    let mut file = File::create("utilisateur.json")?;
    file.write_all(json_pretty.as_bytes())?;

    // Lecture depuis un fichier
    let mut file = File::open("utilisateur.json")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;

    // Désérialisation depuis une chaîne JSON
    let utilisateur_charge: Utilisateur = serde_json::from_str(&contents)?;
    println!("Utilisateur chargé: {:?}", utilisateur_charge);

    Ok(())
}
```
## 17.4. Personnalisation avec les attributs de Serde
Les attributs de personnalisation restent identiques, mais nous pouvons ajouter quelques exemples de cas d'utilisation plus modernes :
``` rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Produit {
    // Renomme le champ en "product_id" dans le JSON
    #[serde(rename = "product_id")]
    id: u32,

    nom: String,

    // Skip si la valeur est None
    #[serde(skip_serializing_if = "Option::is_none")]
    description: Option<String>,

    // Utilise une valeur par défaut si le champ est absent
    #[serde(default = "prix_defaut")]
    prix: f64,

    // Ignore complètement ce champ
    #[serde(skip)]
    stock_interne: u32,

    // Versionnement du format
    #[serde(rename = "created_at")]
    date_creation: String,
}

fn prix_defaut() -> f64 {
    0.0
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let produit = Produit {
        id: 42,
        nom: String::from("Clavier mécanique"),
        description: Some(String::from("Clavier à switches Cherry MX Brown")),
        prix: 89.99,
        stock_interne: 150,
        date_creation: String::from("2024-07-15T14:30:00Z"),
    };

    let json = serde_json::to_string_pretty(&produit)?;
    println!("JSON:\n{}", json);

    // Désérialisation d'un JSON avec des champs manquants
    let json_incomplet = r#"{
        "product_id": 43,
        "nom": "Souris ergonomique"
    }"#;

    let produit_incomplet: Produit = serde_json::from_str(json_incomplet)?;
    println!("Produit incomplet: {:?}", produit_incomplet);

    Ok(())
}
```
## 17.5. Alternatives à serde_yaml
Comme mentionné, `serde_yaml` est désormais déprécié. Voici une alternative avec `serde-yaml-core` :
``` rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Configuration {
    serveur: ServeurConfig,
    base_de_donnees: BDDConfig,
    journalisation: JournalisationConfig,
}

#[derive(Serialize, Deserialize, Debug)]
struct ServeurConfig {
    hote: String,
    port: u16,
    ssl: bool,
}

#[derive(Serialize, Deserialize, Debug)]
struct BDDConfig {
    url: String,
    utilisateur: String,
    mot_de_passe: String,
    max_connexions: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct JournalisationConfig {
    niveau: String,
    fichier: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Note: Remplacez serde_yaml par serde_yaml_core
    // Pour utiliser serde_yaml_core, ajoutez-le à votre Cargo.toml:
    // serde_yaml_core = "0.1"

    let config = Configuration {
        serveur: ServeurConfig {
            hote: String::from("127.0.0.1"),
            port: 8080,
            ssl: true,
        },
        base_de_donnees: BDDConfig {
            url: String::from("postgres://localhost/app"),
            utilisateur: String::from("admin"),
            mot_de_passe: String::from("secure_password"),
            max_connexions: 20,
        },
        journalisation: JournalisationConfig {
            niveau: String::from("info"),
            fichier: String::from("/var/log/app.log"),
        },
    };

    // La syntaxe reste similaire
    // let yaml = serde_yaml_core::to_string(&config)?;

    // Pour l'instant, conservons l'exemple avec des commentaires
    println!("YAML (utiliserait serde_yaml_core avec la même API)");

    Ok(())
}
```
## 17.6. Utilisation de TOML pour les configurations
TOML reste un excellent choix pour les configurations. La bibliothèque a évolué, utilisez maintenant des versions plus récentes : `toml`
``` rust
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::fs::{self, File};
use std::io::Write;

#[derive(Serialize, Deserialize, Debug)]
struct PackageConfig {
    name: String,
    version: String,
    authors: Vec<String>,
    edition: String, // Maintenant "2024"
    description: Option<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct CargoToml {
    package: PackageConfig,
    dependencies: HashMap<String, String>,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let cargo_config = CargoToml {
        package: PackageConfig {
            name: String::from("mon_application"),
            version: String::from("0.1.0"),
            authors: vec![String::from("Sophie Martin <sophie.martin@example.com>")],
            edition: String::from("2024"),
            description: Some(String::from("Une application exemple pour démontrer serde")),
        },
        dependencies: {
            let mut deps = HashMap::new();
            deps.insert(String::from("serde"), String::from("1.0.219"));
            deps.insert(String::from("serde_derive"), String::from("1.0"));
            deps.insert(String::from("toml"), String::from("0.8"));
            deps
        },
    };

    // Sérialisation en TOML
    let toml_str = toml::to_string(&cargo_config)?;
    println!("TOML:\n{}", toml_str);

    // Écriture dans un fichier
    let mut file = File::create("projet_exemple/Cargo.toml")?;
    file.write_all(toml_str.as_bytes())?;

    // Lecture d'un Cargo.toml existant
    let cargo_content = fs::read_to_string("Cargo.toml")?;
    let parsed_cargo: CargoToml = toml::from_str(&cargo_content)?;

    println!("Cargo.toml parsé:");
    println!("  Package: {}", parsed_cargo.package.name);
    println!("  Version: {}", parsed_cargo.package.version);
    println!("  Dépendances:");
    for (dep, version) in &parsed_cargo.dependencies {
        println!("    {} = \"{}\"", dep, version);
    }

    Ok(())
}
```
## 17.7. Sérialisation de structures complexes avec Rust 2024
En Rust 2024, nous pouvons profiter du trait `Future` ajouté au prelude [[3]](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html) et des améliorations sur les closures asynchrones :
``` rust
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::error::Error;

#[derive(Serialize, Deserialize, Debug)]
enum Statut {
    Actif,
    Inactif,
    Suspendu,
}

#[derive(Serialize, Deserialize, Debug)]
struct Adresse {
    rue: String,
    ville: String,
    code_postal: String,
    pays: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct Contact {
    email: String,
    telephone: Option<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Client {
    id: u32,
    nom: String,
    prenom: String,
    statut: Statut,
    adresse: Adresse,
    contact: Contact,
    tags: Vec<String>,
    metadonnees: HashMap<String, String>,
    // Champ avec une structure personnalisée de sérialisation
    #[serde(with = "chrono::serde::ts_seconds")]
    date_creation: chrono::DateTime<chrono::Utc>,
}

// Nouveau dans Rust 2024: closures asynchrones
async fn traiter_client(client: &Client) -> Result<(), Box<dyn Error>> {
    // Simuler un traitement asynchrone
    tokio::time::sleep(std::time::Duration::from_millis(100)).await;
    println!("Client traité: {} {}", client.prenom, client.nom);
    Ok(())
}

fn main() -> Result<(), Box<dyn Error>> {
    // Création d'un client avec des données complexes
    let mut metadonnees = HashMap::new();
    metadonnees.insert("source".to_string(), "site_web".to_string());
    metadonnees.insert("referral".to_string(), "ami".to_string());

    let client = Client {
        id: 12345,
        nom: String::from("Dubois"),
        prenom: String::from("Jean"),
        statut: Statut::Actif,
        adresse: Adresse {
            rue: String::from("123 Rue de la République"),
            ville: String::from("Lyon"),
            code_postal: String::from("69001"),
            pays: String::from("France"),
        },
        contact: Contact {
            email: String::from("jean.dubois@example.com"),
            telephone: Some(String::from("+33 6 12 34 56 78")),
        },
        tags: vec![
            String::from("premium"),
            String::from("fidèle"),
        ],
        metadonnees,
        date_creation: chrono::Utc::now(),
    };

    // Sérialisation en JSON
    let json = serde_json::to_string_pretty(&client)?;
    println!("JSON:\n{}", json);

    // Utilisation d'une closure asynchrone (nouvelle dans Rust 2024)
    let process = async || {
        let client_clone: Client = serde_json::from_str(&json)?;
        traiter_client(&client_clone).await
    };

    // Dans un contexte asynchrone, on pourrait exécuter:
    // tokio::runtime::Runtime::new()?.block_on(process());

    println!("\nRust 2024 permet d'utiliser des closures asynchrones pour traiter les données sérialisées");

    Ok(())
}
```
## 17.8. Gestion des formats binaires avec bincode
``` rust
use serde::{Deserialize, Serialize};
use std::fs::File;
use std::io::{Read, Write};

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct Mesure {
    timestamp: u64,
    temperature: f32,
    humidite: f32,
    pression: f32,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Création d'un ensemble de mesures
    let mesures = vec![
        Mesure {
            timestamp: 1629884234,
            temperature: 22.5,
            humidite: 65.3,
            pression: 1013.2,
        },
        Mesure {
            timestamp: 1629884294,
            temperature: 22.6,
            humidite: 65.0,
            pression: 1013.1,
        },
        Mesure {
            timestamp: 1629884354,
            temperature: 22.7,
            humidite: 64.8,
            pression: 1013.0,
        },
    ];

    // Sérialisation en binaire avec bincode
    let encoded: Vec<u8> = bincode::serialize(&mesures)?;
    println!("Taille des données binaires: {} octets", encoded.len());

    // Sérialisation en JSON pour comparaison
    let json = serde_json::to_string(&mesures)?;
    println!("Taille des données JSON: {} octets", json.len());

    // Écriture dans un fichier binaire
    let mut file = File::create("mesures.bin")?;
    file.write_all(&encoded)?;

    // Lecture depuis un fichier binaire
    let mut file = File::open("mesures.bin")?;
    let mut buffer = Vec::new();
    file.read_to_end(&mut buffer)?;

    // Désérialisation
    let decoded: Vec<Mesure> = bincode::deserialize(&buffer)?;

    // Vérification
    assert_eq!(mesures, decoded);
    println!("Première mesure après désérialisation: {:?}", decoded[0]);

    Ok(())
}
```
## 17.9. Sérialisation conditionnelle basée sur le format cible
Une nouveauté intéressante : Serde permet désormais d'adapter la sérialisation en fonction du format cible [[7]](https://stackoverflow.com/questions/73650684/can-i-change-how-a-rust-data-structure-serializes-based-on-the-target-serializat) :
``` rust
use serde::{Deserialize, Serialize, Serializer};
use std::error::Error;

#[derive(Deserialize, Debug)]
struct Configuration {
    nom: String,
    valeur: f64,
    metadata: Option<String>,
}

impl Serialize for Configuration {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        // Vérifier le format de sérialisation
        let is_human_readable = serializer.is_human_readable();

        // Utiliser différentes structures selon le format
        if is_human_readable {
            // Pour les formats lisibles comme JSON ou YAML
            #[derive(Serialize)]
            struct HumanReadableConfig<'a> {
                nom: &'a str,
                valeur: f64,
                #[serde(skip_serializing_if = "Option::is_none")]
                metadata: &'a Option<String>,
            }

            HumanReadableConfig {
                nom: &self.nom,
                valeur: self.valeur,
                metadata: &self.metadata,
            }.serialize(serializer)
        } else {
            // Pour les formats binaires comme bincode
            #[derive(Serialize)]
            struct BinaryConfig<'a> {
                n: &'a str,  // Noms plus courts
                v: f64,
                m: &'a Option<String>,
            }

            BinaryConfig {
                n: &self.nom,
                v: self.valeur,
                m: &self.metadata,
            }.serialize(serializer)
        }
    }
}

fn main() -> Result<(), Box<dyn Error>> {
    let config = Configuration {
        nom: String::from("app_config"),
        valeur: 42.0,
        metadata: Some(String::from("Production")),
    };

    // Sérialisation en JSON (format lisible)
    let json = serde_json::to_string_pretty(&config)?;
    println!("JSON (format lisible):\n{}", json);

    // Sérialisation en bincode (format binaire)
    let binary = bincode::serialize(&config)?;
    println!("Binaire (longueur): {} octets", binary.len());

    Ok(())
}
```
## 17.10. Versionnement des structures pour maintenir la compatibilité
Avec l'introduction de Rust 2024, le versionnement des structures devient encore plus important pour maintenir la compatibilité [[4]](https://github.com/serde-rs/serde/issues/1137) [[9]](https://users.rust-lang.org/t/how-do-i-future-proof-de-serialization-of-types-that-might-change-across-versions/116392) :
``` rust
use serde::{Deserialize, Serialize};
use std::error::Error;

// Version 1 de notre structure
#[derive(Serialize, Deserialize, Debug)]
struct UserV1 {
    id: u32,
    username: String,
}

// Version 2 avec des champs supplémentaires
#[derive(Serialize, Deserialize, Debug)]
struct UserV2 {
    id: u32,
    username: String,
    email: Option<String>,
    #[serde(default)]
    active: bool,
}

// Wrapper qui gère les différentes versions
#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "version")]
enum VersionedUser {
    #[serde(rename = "1")]
    V1(UserV1),
    #[serde(rename = "2")]
    V2(UserV2),
}

// Fonction utilitaire pour convertir d'une version à l'autre
impl VersionedUser {
    fn to_latest(&self) -> UserV2 {
        match self {
            VersionedUser::V1(user) => UserV2 {
                id: user.id,
                username: user.username.clone(),
                email: None,
                active: true, // Valeur par défaut
            },
            VersionedUser::V2(user) => user.clone(),
        }
    }
}

impl Clone for UserV2 {
    fn clone(&self) -> Self {
        UserV2 {
            id: self.id,
            username: self.username.clone(),
            email: self.email.clone(),
            active: self.active,
        }
    }
}

fn main() -> Result<(), Box<dyn Error>> {
    // Données au format V1
    let v1_json = r#"
    {
        "version": "1",
        "V1": {
            "id": 42,
            "username": "utilisateur1"
        }
    }
    "#;

    // Données au format V2
    let v2_json = r#"
    {
        "version": "2",
        "V2": {
            "id": 43,
            "username": "utilisateur2",
            "email": "user2@example.com",
            "active": true
        }
    }
    "#;

    // Désérialisation des deux formats
    let user1: VersionedUser = serde_json::from_str(v1_json)?;
    let user2: VersionedUser = serde_json::from_str(v2_json)?;

    // Conversion vers la dernière version
    let latest1 = user1.to_latest();
    let latest2 = user2.to_latest();

    println!("User1 (converti à la dernière version): {:?}", latest1);
    println!("User2 (déjà à la dernière version): {:?}", latest2);

    Ok(())
}
```
## 17.11. Bonnes pratiques actualisées pour Rust 2024
1. **Utilisez les types asynchrones** : Avec l'ajout de `Future` au prelude et le support pour les closures asynchrones, intégrez ces fonctionnalités pour traiter les données sérialisées de manière asynchrone.
2. **Versionnez vos structures de données** : Particulièrement important pour les applications à long terme.
3. **Choisissez des alternatives maintenues** : Avec la dépréciation de certaines bibliothèques comme `serde_yaml`, préférez des alternatives maintenues.
4. **Gestion des erreurs modernes** : Utilisez ou `anyhow` pour une gestion plus élégante des erreurs. `thiserror`
``` rust
use serde::{Deserialize, Serialize};
use thiserror::Error;

#[derive(Error, Debug)]
enum ConfigError {
    #[error("Erreur de désérialisation: {0}")]
    DeserializeError(#[from] serde_json::Error),

    #[error("Erreur d'IO: {0}")]
    IoError(#[from] std::io::Error),

    #[error("Configuration invalide: {0}")]
    ValidationError(String),
}

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    port: u16,
    max_connections: u32,
}

impl Config {
    fn validate(self) -> Result<Self, ConfigError> {
        if self.port < 1024 {
            return Err(ConfigError::ValidationError(
                "Le port doit être > 1024".to_string()
            ));
        }
        if self.max_connections == 0 {
            return Err(ConfigError::ValidationError(
                "max_connections doit être > 0".to_string()
            ));
        }
        Ok(self)
    }

    fn from_json(json: &str) -> Result<Self, ConfigError> {
        let config: Self = serde_json::from_str(json)?;
        config.validate()
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let json = r#"{"port": 8080, "max_connections": 100}"#;

    let config = match Config::from_json(json) {
        Ok(config) => config,
        Err(e) => {
            eprintln!("Erreur de configuration: {}", e);
            return Err(Box::new(e));
        }
    };

    println!("Configuration valide: {:?}", config);

    Ok(())
}
```
## 17.12. Conclusion
Serde reste l'outil de référence pour la sérialisation et la désérialisation en Rust, y compris dans la nouvelle édition 2024. Les principes fondamentaux sont toujours valables, mais l'écosystème évolue avec :
- De nouvelles versions des formats et bibliothèques
- Des alternatives à certaines bibliothèques dépréciées
- Une meilleure intégration avec les fonctionnalités asynchrones de Rust
- Des approches plus avancées pour le versionnement et la compatibilité

Ces améliorations permettent de profiter pleinement de la sécurité et des performances de Rust tout en bénéficiant d'un écosystème mature pour la sérialisation et la désérialisation de données.

⏭️ [Création de crates et publication](/III-avance/18-creation-crates-publication.md) - Comment publier sur crates.io
