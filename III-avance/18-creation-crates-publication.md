## 18. **Création de crates et publication** - Comment publier sur crates.io

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

La publication de bibliothèques sur crates.io permet de partager votre code avec la communauté Rust. Dans cette section, nous allons explorer le processus complet de création d'une bibliothèque et sa publication.
### 18.1 Préparer un crate pour la publication
Commençons par créer une nouvelle bibliothèque :
``` bash
cargo new hash_utils --lib
cd hash_utils
```
Notre bibliothèque contiendra des fonctions utilitaires pour calculer différentes valeurs de hachage, en utilisant les crates que nous avons à disposition (sha2, sha1, md5, crc).
Modifions le fichier `Cargo.toml` pour ajouter les dépendances nécessaires :
``` toml
[package]
name = "hash_utils"
version = "0.1.0"
edition = "2024"
description = "Utilitaires de fonctions de hachage"
license = "MIT OR Apache-2.0"
repository = "https://github.com/votre-utilisateur/hash_utils"
documentation = "https://docs.rs/hash_utils"
readme = "README.md"
keywords = ["hash", "cryptography", "sha", "md5", "crc"]
categories = ["cryptography", "algorithms"]
rust-version = "1.85.0"  # Version minimale supportant l'édition 2024

[dependencies]
sha2 = "0.11.0"
sha1 = "0.11.0"
md5 = "0.7.0"
crc = "3.2.1"
```
Notez les modifications apportées :
- Changement de `edition = "2021"` à `edition = "2024"`
- Ajout de `rust-version = "1.85.0"` qui permet au résolveur de dépendances de Cargo d'être conscient de la version minimale requise [[4]](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html)
- Mise à jour des versions des dépendances pour être compatibles avec Rust 2024

### 18.2 Structure du crate
Créons maintenant la structure de notre bibliothèque dans `src/lib.rs` :
``` rust
//! # Hash Utils
//!
//! `hash_utils` est une bibliothèque simple qui facilite l'utilisation
//! de différentes fonctions de hachage.

mod sha;
mod md5_utils;
mod crc_utils;

pub use sha::{sha1_hash, sha256_hash};
pub use md5_utils::md5_hash;
pub use crc_utils::crc32;

/// Convertit un vecteur d'octets en représentation hexadécimale
pub fn bytes_to_hex(bytes: &[u8]) -> String {
    bytes.iter()
         .map(|b| format!("{:02x}", b))
         .collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_bytes_to_hex() {
        let bytes = [0x12, 0xAB, 0xCD, 0xEF];
        assert_eq!(bytes_to_hex(&bytes), "12abcdef");
    }
}
```
Ensuite, créons les modules pour chaque type de hachage. Commençons par `src/sha.rs` :
``` rust
use sha1::Sha1;
use sha2::{Sha256, Digest};

/// Calcule le hachage SHA-1 d'une chaîne et retourne le résultat en hexadécimal
///
/// # Exemples
///
/// ```
/// let hash = hash_utils::sha1_hash("Hello, world!");
/// assert_eq!(hash.len(), 40); // SHA-1 produit 20 octets (40 caractères en hex)
/// ```
pub fn sha1_hash(input: &str) -> String {
    let mut hasher = Sha1::new();
    hasher.update(input.as_bytes());
    let result = hasher.finalize();

    crate::bytes_to_hex(&result)
}

/// Calcule le hachage SHA-256 d'une chaîne et retourne le résultat en hexadécimal
///
/// # Exemples
///
/// ```
/// let hash = hash_utils::sha256_hash("Hello, world!");
/// assert_eq!(hash.len(), 64); // SHA-256 produit 32 octets (64 caractères en hex)
/// ```
pub fn sha256_hash(input: &str) -> String {
    let mut hasher = Sha256::new();
    hasher.update(input.as_bytes());
    let result = hasher.finalize();

    crate::bytes_to_hex(&result)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_sha1() {
        // Valeur SHA-1 connue pour "abc"
        assert_eq!(
            sha1_hash("abc"),
            "a9993e364706816aba3e25717850c26c9cd0d89d"
        );
    }

    #[test]
    fn test_sha256() {
        // Valeur SHA-256 connue pour "abc"
        assert_eq!(
            sha256_hash("abc"),
            "ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad"
        );
    }
}
```
Créons maintenant `src/md5_utils.rs` :
``` rust
/// Calcule le hachage MD5 d'une chaîne et retourne le résultat en hexadécimal
///
/// # Exemples
///
/// ```
/// let hash = hash_utils::md5_hash("Hello, world!");
/// assert_eq!(hash.len(), 32); // MD5 produit 16 octets (32 caractères en hex)
/// ```
pub fn md5_hash(input: &str) -> String {
    let digest = md5::compute(input.as_bytes());
    crate::bytes_to_hex(&digest.0)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_md5() {
        // Valeur MD5 connue pour "abc"
        assert_eq!(
            md5_hash("abc"),
            "900150983cd24fb0d6963f7d28e17f72"
        );
    }
}
```
Enfin, créons `src/crc_utils.rs` :
``` rust
/// Calcule le CRC32 d'une chaîne et retourne le résultat en hexadécimal
///
/// # Exemples
///
/// ```
/// let hash = hash_utils::crc32("Hello, world!");
/// assert_eq!(hash.len(), 8); // CRC32 produit 4 octets (8 caractères en hex)
/// ```
pub fn crc32(input: &str) -> String {
    let checksum = crc::Crc::<u32>::new(&crc::CRC_32_ISO_HDLC)
        .checksum(input.as_bytes());
    format!("{:08x}", checksum)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_crc32() {
        // Valeur CRC32 connue pour "abc"
        assert_eq!(
            crc32("abc"),
            "352441c2"
        );
    }
}
```
### 18.3 Créer un README.md
Le fichier README est essentiel pour présenter votre bibliothèque aux utilisateurs potentiels :
``` markdown
# hash_utils

Une bibliothèque simple pour calculer différentes valeurs de hachage en Rust.

## Fonctionnalités

- Calcul de hash SHA-1
- Calcul de hash SHA-256
- Calcul de hash MD5
- Calcul de CRC32

## Installation

Ajoutez cette dépendance à votre fichier `Cargo.toml` :

```
toml [dependencies] hash_utils = "0.1.0"
```

## Exemples d'utilisation

```
rust use hash_utils::{sha1_hash, sha256_hash, md5_hash, crc32};
fn main() { let input = "Hello, world!";
println!("SHA-1:   {}", sha1_hash(input));
println!("SHA-256: {}", sha256_hash(input));
println!("MD5:     {}", md5_hash(input));
println!("CRC32:   {}", crc32(input));
}
```

## Compatibilité

Cette bibliothèque utilise l'édition Rust 2024 et nécessite Rust 1.85.0 ou supérieur.

## Licence

Sous licence MIT ou Apache-2.0, au choix.
```
Notez l'ajout d'une section "Compatibilité" pour informer les utilisateurs des exigences de version pour l'édition 2024.
### 18.4 Vérification avant publication
Avant de publier, nous devons :
1. Vérifier que notre code est correctement formaté :
``` bash
cargo fmt
```
1. S'assurer que tous les tests passent :
``` bash
cargo test
```
1. Vérifier les avertissements du compilateur :
``` bash
cargo clippy
```
1. Générer et vérifier la documentation :
``` bash
cargo doc --open
```
1. Vérifier la compatibilité avec l'édition 2024 :
``` bash
# Si vous développez toujours avec l'édition 2021, activez les lints de compatibilité
RUSTFLAGS="-W rust-2024-compatibility" cargo check
```
### 18.5 Création d'un compte sur crates.io
Pour publier sur crates.io, vous devez d'abord créer un compte et générer une clé API :
1. Rendez-vous sur [crates.io](https://crates.io) et connectez-vous avec votre compte GitHub.
2. Allez dans les paramètres de compte, dans la section "API Tokens".
3. Créez un nouveau jeton et copiez-le.
4. Enregistrez votre jeton en utilisant la commande suivante :
``` bash
cargo login votre-jeton-api
```
### 18.6 Publication sur crates.io
Une fois que votre crate est prêt, vous pouvez le publier :
``` bash
cargo publish
```
Si la publication réussit, votre crate sera disponible sur crates.io !
### 18.7 Gestion des versions
Lorsque vous développez de nouvelles fonctionnalités ou corrigez des bugs, vous devez mettre à jour le numéro de version dans `Cargo.toml` selon les règles du versionnement sémantique (SemVer) :
- Incrémentez le premier chiffre (1.0.0 → 2.0.0) pour des changements incompatibles.
- Incrémentez le deuxième chiffre (1.0.0 → 1.1.0) pour des ajouts compatibles.
- Incrémentez le troisième chiffre (1.0.0 → 1.0.1) pour des corrections de bugs.

Pour chaque nouvelle version, publiez à nouveau :
``` bash
# Après avoir modifié la version dans Cargo.toml
cargo publish
```
### 18.8 Retirer une version
Si vous découvrez un problème grave dans une version publiée, vous pouvez la retirer (yanking) :
``` bash
cargo yank --vers 0.1.0
```
Cela n'effacera pas la version, mais empêchera les nouveaux projets de l'utiliser comme dépendance. Pour annuler ce retrait :
``` bash
cargo yank --vers 0.1.0 --undo
```
### 18.9 Bonnes pratiques pour les crates publics
Quelques recommandations pour maintenir une bibliothèque de qualité :
1. **Documentation complète** : Incluez des exemples et expliquez clairement chaque fonction.
2. **Tests approfondis** : Assurez-vous que toutes les fonctionnalités sont testées.
3. **Changelog** : Maintenez un fichier CHANGELOG.md pour lister les modifications.
4. **Compatibilité ascendante** : Évitez de casser la compatibilité ascendante.
5. **Versionnement sémantique** : Respectez les règles SemVer pour les numéros de version.
6. **CI/CD** : Configurez l'intégration continue pour tester automatiquement votre code.
7. **Spécification de version minimale de Rust** : Utilisez le champ `rust-version` pour indiquer la version minimale requise.

### 18.10 Profiter des fonctionnalités de l'édition 2024
L'édition 2024 apporte plusieurs améliorations dont vous pouvez tirer parti dans votre crate [[4]](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html) :
1. **Support pour les fermetures asynchrones** : Vous pouvez maintenant utiliser `async || {}` dans votre code.
2. **Nouvelles additions au prélude** : `Future` et `IntoFuture` sont désormais inclus dans le prélude, facilitant le travail avec du code asynchrone.
3. **Résolveur de dépendances sensible à la version Rust** : Le champ `rust-version` est maintenant pris en compte lors de la résolution des dépendances.

### 18.11 Conclusion
La publication de crates sur crates.io est un excellent moyen de contribuer à l'écosystème Rust. En suivant ces étapes et bonnes pratiques, vous pouvez créer des bibliothèques utiles, bien documentées et faciles à utiliser pour d'autres développeurs Rust. L'utilisation de l'édition 2024 vous permet de profiter des dernières fonctionnalités du langage tout en étant explicite sur les exigences de version pour les utilisateurs de votre crate.

⏭️ [Les attributs de compilation et optimisation](/III-avance/19-attributs-compilation-optimisation.md) - Flags de compilation, optimisations spécifiques
