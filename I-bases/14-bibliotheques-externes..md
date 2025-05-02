## 14\. Utiliser des bibliothèques externes

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Trouver des bibliothèques

Les bibliothèques Rust (appelées "crates") peuvent être trouvées principalement via :

1.  [crates.io](https://crates.io/) - Le registre officiel
2.  [lib.rs](https://lib.rs/) - Une alternative avec des fonctionnalités de recherche avancées
3.  [docs.rs](https://docs.rs/) - Documentation automatique de toutes les crates publiées

## Ajouter des dépendances

### Depuis crates.io

Pour utiliser une bibliothèque du registre officiel, ajoutez-la à votre `Cargo.toml` :

```  toml
[dependencies]
sha2 = "0.10.8"
serde = { version = "1.0", features = ["derive"] }
rand = "0.8"
```

### Depuis un dépôt Git

Vous pouvez également utiliser du code directement depuis un dépôt Git :

```  toml
[dependencies]
ma_crate = { git = "https://github.com/utilisateur/ma_crate", branch = "main" }
# Ou spécifier un tag ou un commit précis
autre_crate = { git = "https://github.com/utilisateur/autre_crate", tag = "v1.0.0" }
encore_une = { git = "https://github.com/utilisateur/encore_une", rev = "abc123" }
```

### Depuis un chemin local

Pendant le développement, il est souvent utile de référencer une bibliothèque locale :

```  toml
[dependencies]
ma_lib_locale = { path = "../chemin/vers/ma_lib_locale" }
```

## Importer et utiliser les bibliothèques

Une fois les dépendances ajoutées à votre `Cargo.toml`, vous pouvez les utiliser dans votre code :

``` rust
// Importation simple
use sha2::{Sha256, Digest};

// Importation avec alias
use crossbeam_channel as channel;

// Importation de plusieurs éléments
use serde::{Serialize, Deserialize};

// Importation avec chemin complet
fn calculer_hash(données: &[u8]) -> String {
    let mut hasher = sha2::Sha256::new();
    hasher.update(données);
    format!("{:x}", hasher.finalize())
}
```

## Exemple complet avec plusieurs bibliothèques

Voici un exemple qui utilise plusieurs bibliothèques externes pour créer un petit outil qui calcule différents types de hashes :

```  toml
[dependencies]
sha2 = "0.11.0-pre.5"
hex = "0.4.3"
sha1 = "0.11.0-pre.5"
crossbeam-channel = "0.5.15"
crc = "3.2.1"
```

``` rust
use std::io::{self, Read};
use sha2::{Sha256, Digest as Sha2Digest};
use sha1::{Sha1, Digest as Sha1Digest};
use crossbeam_channel::{bounded, select};
use std::thread;

#[derive(Debug)]
enum HashType {
    SHA1(String),
    SHA256(String),
    CRC32(String),
}

fn main() -> io::Result<()> {
    // Lecture des données depuis stdin
    let mut buffer = Vec::new();
    io::stdin().read_to_end(&mut buffer)?;

    // Création des canaux pour la communication entre threads
    let (sha1_s, sha1_r) = bounded(1);
    let (sha256_s, sha256_r) = bounded(1);
    let (crc_s, crc_r) = bounded(1);


    // Calcul du SHA-1 dans un thread séparé
    let sha1_data = buffer.clone();
    thread::spawn(move || {
        let mut hasher = Sha1::new();
        hasher.update(&sha1_data);
        let hash_result = hasher.finalize();
        let result = hex::encode(hash_result);
        sha1_s.send(HashType::SHA1(result)).unwrap();
    });

    // Calcul du SHA-256 dans un thread séparé
    let sha256_data = buffer.clone();
    thread::spawn(move || {
        let mut hasher = Sha256::new();
        hasher.update(&sha256_data);
        let hash_result = hasher.finalize();
        let result = hex::encode(hash_result);
        sha256_s.send(HashType::SHA256(result)).unwrap();
    });

    // Calcul du CRC32 dans un thread séparé
    let crc_data = buffer.clone();
    thread::spawn(move || {
        let checksum = crc::Crc::<u32>::new(&crc::CRC_32_ISO_HDLC).checksum(&crc_data);
        crc_s.send(HashType::CRC32(format!("{:08x}", checksum))).unwrap();
    });

    // Collecte des résultats
    let mut results = Vec::new();
    for _ in 0..4 {
        select! {
            recv(sha1_r) -> hash => results.push(hash.unwrap()),
            recv(sha256_r) -> hash => results.push(hash.unwrap()),
            recv(crc_r) -> hash => results.push(hash.unwrap()),
        }
    }

    // Affichage des résultats
    for hash in results {
        match hash {
            HashType::SHA1(hash) => println!("SHA1: {}", hash),
            HashType::SHA256(hash) => println!("SHA256: {}", hash),
            HashType::CRC32(hash) => println!("CRC32: {}", hash),
        }
    }

    Ok(())
}
```

## Documentation des bibliothèques

Pour consulter la documentation d'une bibliothèque utilisée dans votre projet :

```bash
cargo doc --open
```

Cette commande génère et ouvre la documentation de votre projet et de toutes ses dépendances.

## Fonctionnalités (features) des bibliothèques

Beaucoup de bibliothèques offrent des fonctionnalités optionnelles que vous pouvez activer selon vos besoins :

``` rust
[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }
```

Vous pouvez également désactiver les fonctionnalités par défaut et activer uniquement celles dont vous avez besoin :

``` rust
[dependencies]
tokio = { version = "1.0", default-features = false, features = ["rt", "macros"] }
```

## Gestion des conflits de version

Parfois, différentes bibliothèques peuvent dépendre de versions différentes d'une même crate. Cargo tente de résoudre automatiquement ces conflits en sélectionnant la version la plus compatible, mais parfois cela peut causer des problèmes.

Pour visualiser l'arbre de dépendances et identifier les conflits :

``` bash
cargo install cargo-tree
cargo tree
```

Pour identifier des problèmes de version spécifiques :

``` bash
cargo tree -i nom_crate
```

## Tester avant d'adopter

Avant d'adopter une bibliothèque pour un projet critique, considérez ces critères :

1.  **Maturité** : Vérifiez le nombre de téléchargements, la date de dernière mise à jour, et l'historique des versions
2.  **Documentation** : Une bonne documentation est essentielle
3.  **Tests** : Vérifiez la présence de tests unitaires et d'intégration
4.  **Maintenance** : Regardez l'activité du dépôt et la réactivité aux issues
5.  **Licence** : Assurez-vous que la licence est compatible avec votre projet

## Création d'un bac à sable pour tester des bibliothèques

Pour tester rapidement une bibliothèque sans créer un projet complet :

``` bash
mkdir test-lib && cd test-lib
cargo init
# Modifiez Cargo.toml pour ajouter vos dépendances
# Modifiez src/main.rs pour tester la bibliothèque
cargo run
```

L'utilisation judicieuse des bibliothèques externes vous permet d'éviter de "réinventer la roue" et d'accélérer considérablement le développement de vos projets Rust.

⏭️ [Jeu de devinette de mots](/I-bases/15-jeu-devinette.md)
