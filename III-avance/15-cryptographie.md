## 15. **Écosystème Rust pour la cryptographie** - Utilisation des crates de hachage et cryptographie
### 15.1 Introduction à la cryptographie en Rust
Rust est particulièrement bien adapté aux applications cryptographiques grâce à sa sécurité mémoire, ses performances et son modèle de concurrence. L'écosystème Rust pour la cryptographie est riche et mature, offrant des implémentations soigneusement vérifiées d'algorithmes standard et modernes.
Dans ce chapitre, nous explorerons les bibliothèques cryptographiques les plus utilisées, en nous concentrant particulièrement sur les fonctions de hachage comme SHA-2, SHA-1 et MD5, tout en abordant d'autres aspects importants de la cryptographie moderne. Avec l'édition Rust 2024, certaines API sont désormais stables dans les contextes const, ce qui peut améliorer les performances des opérations cryptographiques.
### 15.2 Les fonctions de hachage
Les fonctions de hachage transforment des données de taille arbitraire en une empreinte de taille fixe. Elles sont essentielles pour :
- La vérification d'intégrité des données
- Les signatures numériques
- Le stockage sécurisé des mots de passe
- Les structures de données comme les tables de hachage

#### 15.2.1 Utilisation de la crate MD5
Bien que MD5 soit considéré comme cryptographiquement faible et ne devrait pas être utilisé pour des applications de sécurité, il reste utile pour certaines applications comme la vérification d'intégrité non-critique.
``` rust
use md5::{Md5, Digest};

fn main() {
    // Méthode simple pour une chaîne
    let hash = md5::compute("Hello, world!");
    println!("MD5 (simple): {:x}", hash);

    // Méthode avec le trait Digest, adaptée aux flux de données
    let mut hasher = Md5::new();
    hasher.update("Hello, ");
    hasher.update("world!");
    let result = hasher.finalize();

    println!("MD5 (digest): {:x}", result);

    // Traitement de fichiers volumineux par blocs
    let data = vec![0u8; 1024]; // Simulons un bloc de données
    let mut file_hasher = Md5::new();

    // On pourrait traiter plusieurs blocs dans une boucle
    file_hasher.update(&data);
    let file_hash = file_hasher.finalize();

    println!("MD5 (fichier): {:x}", file_hash);
}
```
#### 15.2.2 Utilisation de la crate SHA-1
SHA-1 reste utilisé dans certains contextes, bien qu'il ne soit plus recommandé pour les applications cryptographiques sensibles.
``` rust
use sha1::{Sha1, Digest};

fn main() {
    // Création d'un nouveau hasher SHA-1
    let mut hasher = Sha1::new();

    // Ajout de données à hacher
    hasher.update("Un exemple");
    hasher.update(" de texte à hacher");

    // Finalisation et récupération du résultat
    let result = hasher.finalize();

    println!("SHA-1: {:x}", result);

    // Utilisation avec des données binaires
    let binary_data = vec![0xDE, 0xAD, 0xBE, 0xEF];
    let hash = Sha1::digest(&binary_data);

    println!("SHA-1 de données binaires: {:x}", hash);
}
```
#### 15.2.3 Utilisation de la crate SHA-2 (SHA-256, SHA-512)
SHA-2 est une famille d'algorithmes considérés comme sûrs pour les applications cryptographiques modernes.
``` rust
use sha2::{Sha256, Sha512, Digest};

fn main() {
    // SHA-256
    let mut hasher_256 = Sha256::new();
    hasher_256.update("Donnée sensible à protéger");
    let hash_256 = hasher_256.finalize();

    println!("SHA-256: {:x}", hash_256);

    // SHA-512 (empreinte plus longue)
    let mut hasher_512 = Sha512::new();
    hasher_512.update("Donnée sensible à protéger");
    let hash_512 = hasher_512.finalize();

    println!("SHA-512: {:x}", hash_512);

    // Méthode plus concise avec digest
    let texte = b"Exemple de texte";
    let hash_rapide = Sha256::digest(texte);

    println!("SHA-256 (concis): {:x}", hash_rapide);
}
```
### 15.3 Vérification d'intégrité avec CRC32
Le CRC32 est un code de détection d'erreurs, non pas un algorithme cryptographique, mais il est souvent utilisé pour la vérification d'intégrité rapide.
``` rust
use crc::{Crc, CRC_32_ISO_HDLC};

fn main() {
    // Définir le polynôme CRC-32 standard
    let crc = Crc::<u32>::new(&CRC_32_ISO_HDLC);

    // Calculer le CRC d'une chaîne
    let data = b"Vérifier l'intégrité de ces données";
    let checksum = crc.checksum(data);

    println!("CRC-32: {:08x}", checksum);

    // Vérification d'intégrité
    let data_reçu = b"Vérifier l'intégrité de ces données";
    let checksum_reçu = crc.checksum(data_reçu);

    if checksum == checksum_reçu {
        println!("Les données sont intactes!");
    } else {
        println!("Les données ont été modifiées!");
    }

    // Utilisation avec un builder pour traiter des données en plusieurs étapes
    let mut digest = crc.digest();
    digest.update(b"Première partie des données");
    digest.update(b", seconde partie");
    let checksum_parties = digest.finalize();

    println!("CRC-32 (multi-parties): {:08x}", checksum_parties);
}
```
### 15.4 Cryptographie symétrique avec RustCrypto
L'écosystème Rust dispose d'excellentes bibliothèques pour le chiffrement symétrique. Voyons comment utiliser AES, l'un des algorithmes les plus répandus.
``` rust
// Cargo.toml:
// [dependencies]
// aes-gcm = "0.10.3"  // Version mise à jour
// rand = "0.8.5"

use aes_gcm::{
    aead::{Aead, KeyInit},
    Aes256Gcm, Nonce
};
use rand::{rngs::OsRng, RngCore};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Génération de clé aléatoire
    let key = Aes256Gcm::generate_key(OsRng);

    // Création du chiffreur AES-GCM
    let cipher = Aes256Gcm::new(&key);

    // Nonce (IV) - doit être unique pour chaque message avec la même clé
    let mut nonce_bytes = [0u8; 12]; // 96 bits pour AES-GCM
    OsRng.fill_bytes(&mut nonce_bytes);
    let nonce = Nonce::from_slice(&nonce_bytes);

    // Message à chiffrer
    let message = b"Message secret à protéger";

    // Chiffrement
    let ciphertext = cipher.encrypt(nonce, message.as_ref())?;

    println!("Message chiffré: {:?}", ciphertext);

    // Déchiffrement
    let plaintext = cipher.decrypt(nonce, ciphertext.as_ref())?;

    println!("Message déchiffré: {}", String::from_utf8_lossy(&plaintext));

    Ok(())
}
```
### 15.5 Cryptographie à clé publique
L'écosystème Rust propose plusieurs bibliothèques pour les opérations de cryptographie asymétrique.
``` rust
// Cargo.toml:
// [dependencies]
// rsa = "0.9.2"  // Version mise à jour
// rand = "0.8.5"

use rsa::{
    pkcs8::{EncodePublicKey, LineEnding},
    RsaPrivateKey, RsaPublicKey, Pkcs1v15Encrypt
};
use rand::rngs::OsRng;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Génération d'une paire de clés RSA (peut prendre un moment)
    let bits = 2048;
    println!("Génération d'une clé RSA de {} bits...", bits);

    let private_key = RsaPrivateKey::new(&mut OsRng, bits)?;
    let public_key = RsaPublicKey::from(&private_key);

    // Export des clés au format PEM
    let pem_public = public_key.to_public_key_pem(LineEnding::LF)?;
    println!("Clé publique PEM:\n{}", pem_public);

    // Chiffrement d'un message avec la clé publique
    let message = b"Message confidentiel pour le destinataire";
    let ciphertext = public_key.encrypt(&mut OsRng, Pkcs1v15Encrypt, message)?;

    println!("Message chiffré: {:?}", ciphertext);

    // Déchiffrement avec la clé privée
    let decrypted = private_key.decrypt(Pkcs1v15Encrypt, &ciphertext)?;

    println!("Message déchiffré: {}", String::from_utf8_lossy(&decrypted));

    Ok(())
}
```
### 15.6 Signature numérique
Les signatures numériques sont essentielles pour vérifier l'authenticité et l'intégrité des données.
``` rust
// Cargo.toml:
// [dependencies]
// rsa = "0.9.2"  // Version mise à jour
// rand = "0.8.5"
// sha2 = "0.10.8"

use rsa::{
    pkcs8::{EncodePrivateKey, EncodePublicKey, LineEnding},
    RsaPrivateKey, RsaPublicKey, Pkcs1v15Sign
};
use rand::rngs::OsRng;
use sha2::{Sha256, Digest};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Génération de clés
    let private_key = RsaPrivateKey::new(&mut OsRng, 2048)?;
    let public_key = RsaPublicKey::from(&private_key);

    // Document à signer
    let document = b"Contrat important à signer électroniquement";

    // Calcul du hachage du document
    let mut hasher = Sha256::new();
    hasher.update(document);
    let hashed_doc = hasher.finalize();

    // Signature du hachage avec la clé privée
    let signature = private_key.sign(Pkcs1v15Sign::new::<Sha256>(), &hashed_doc)?;

    println!("Document: {}", String::from_utf8_lossy(document));
    println!("Signature: {:?}", signature);

    // Vérification de la signature
    let result = public_key.verify(
        Pkcs1v15Sign::new::<Sha256>(),
        &hashed_doc,
        &signature
    );

    match result {
        Ok(()) => println!("Signature valide - document authentique!"),
        Err(e) => println!("Signature invalide: {}", e),
    }

    // Essayons avec un document modifié
    let document_modifié = b"Contrat modifié après signature";
    let mut hasher = Sha256::new();
    hasher.update(document_modifié);
    let hashed_modified = hasher.finalize();

    let result = public_key.verify(
        Pkcs1v15Sign::new::<Sha256>(),
        &hashed_modified,
        &signature
    );

    match result {
        Ok(()) => println!("Signature valide pour le document modifié!"),
        Err(_) => println!("Signature invalide - document altéré détecté!"),
    }

    Ok(())
}
```
### 15.7 Stockage sécurisé de mots de passe
La sécurisation des mots de passe est une application critique de la cryptographie.
``` rust
// Cargo.toml:
// [dependencies]
// argon2 = "0.5.2"  // Version mise à jour
// rand_core = { version = "0.6.4", features = ["std"] }

use argon2::{
    password_hash::{
        rand_core::OsRng,
        PasswordHash, PasswordHasher, PasswordVerifier, SaltString
    },
    Argon2
};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Mot de passe en clair (venant de l'utilisateur)
    let password = b"mot_de_passe_utilisateur";

    // Génération d'un sel aléatoire
    let salt = SaltString::generate(&mut OsRng);

    // Configuration de l'algorithme Argon2
    let argon2 = Argon2::default();

    // Hashage du mot de passe
    let password_hash = argon2.hash_password(password, &salt)?
                             .to_string();

    println!("Mot de passe haché à stocker: {}", password_hash);

    // Lors d'une tentative de connexion ultérieure:
    let parsed_hash = PasswordHash::new(&password_hash)?;

    // Vérification avec le bon mot de passe
    let result = argon2.verify_password(password, &parsed_hash);
    match result {
        Ok(()) => println!("Mot de passe correct!"),
        Err(_) => println!("Mot de passe incorrect!"),
    }

    // Essai avec un mauvais mot de passe
    let wrong_password = b"mauvais_mot_de_passe";
    let result = argon2.verify_password(wrong_password, &parsed_hash);
    match result {
        Ok(()) => println!("Mot de passe correct!"),
        Err(_) => println!("Mot de passe incorrect!"),
    }

    Ok(())
}
```
### 15.8 Communication sécurisée inter-threads avec cryptographie et async
Avec l'édition Rust 2024, nous pouvons exploiter les nouvelles fonctionnalités d'async, notamment les closures async qui sont désormais disponibles [[4]](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html).
``` rust
use sha2::{Sha256, Digest};
use tokio::sync::mpsc;
use std::sync::Arc;

// Structure pour les messages sécurisés
struct SecureMessage {
    payload: Vec<u8>,
    hmac: Vec<u8>,
}

// Fonction pour calculer un HMAC simple basé sur SHA-256
fn calculate_hmac(key: &[u8], message: &[u8]) -> Vec<u8> {
    let mut hasher = Sha256::new();
    hasher.update(key);
    hasher.update(message);
    hasher.finalize().to_vec()
}

#[tokio::main]
async fn main() {
    // Clé partagée pour l'authentification des messages
    let shared_key = b"clé_secrète_partagée";

    // Canal de communication async entre tâches
    let (sender, mut receiver) = mpsc::channel::<SecureMessage>(10);

    // Tâche émettrice (utilisant closure async, nouveauté de Rust 2024)
    let sender_key = shared_key.to_vec();
    let sender_task = tokio::spawn(async move {
        for i in 0..5 {
            let message = format!("Message sécurisé n°{}", i);
            let payload = message.as_bytes().to_vec();

            // Calcul du HMAC pour authentifier le message
            let hmac = calculate_hmac(&sender_key, &payload);

            // Envoi du message sécurisé
            sender.send(SecureMessage { payload, hmac }).await.unwrap();

            println!("Émetteur: message n°{} envoyé", i);
            tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
        }
    });

    // Tâche réceptrice
    let receiver_key = Arc::new(shared_key.to_vec());
    let receiver_task = tokio::spawn(async move {
        while let Some(secure_msg) = receiver.recv().await {
            // Vérification de l'authenticité avec HMAC
            let calculated_hmac = calculate_hmac(&receiver_key, &secure_msg.payload);

            if calculated_hmac == secure_msg.hmac {
                // Message authentique
                let message = String::from_utf8_lossy(&secure_msg.payload);
                println!("Récepteur: message authentique reçu: {}", message);
            } else {
                println!("Récepteur: ALERTE! Message compromis détecté!");
            }
        }
    });

    // Attendre la fin des tâches
    let _ = tokio::join!(sender_task, receiver_task);
}
```
### 15.9 Analyse comparative des performances avec Rayon
Comparons les performances des différents algorithmes de hachage en utilisant les capacités de parallélisme de Rayon.
``` rust
use std::time::{Duration, Instant};
use sha2::{Sha256, Sha512, Digest};
use sha1::Sha1;
use md5::Md5;
use rayon::prelude::*;

fn bench_hash<D: Digest + Send + Sync>(name: &str, data: &[u8], iterations: usize)
where
    D: Digest,
{
    let start = Instant::now();

    // Utilisation de Rayon pour paralléliser les opérations (plus efficace avec l'édition 2024)
    (0..iterations).into_par_iter().for_each(|_| {
        let mut hasher = D::new();
        hasher.update(data);
        let _result = hasher.finalize();
    });

    let duration = start.elapsed();
    let throughput = (data.len() * iterations) as f64 / duration.as_secs_f64() / 1_000_000.0;

    println!("{}: {:?} ({:.2f} MB/s)", name, duration, throughput);
}

fn main() {
    // Données de test (1 MB)
    let data = vec![0xAA; 1_000_000];
    let iterations = 100;

    println!("Benchmark avec {} itérations sur {} MB de données:",
             iterations, data.len() / 1_000_000);

    bench_hash::<Md5>("MD5", &data, iterations);
    bench_hash::<Sha1>("SHA-1", &data, iterations);
    bench_hash::<Sha256>("SHA-256", &data, iterations);
    bench_hash::<Sha512>("SHA-512", &data, iterations);
}
```
### 15.10 Bonnes pratiques en cryptographie
Voici quelques principes à respecter pour l'utilisation sécurisée de la cryptographie :
1. **Ne pas inventer ses propres algorithmes** - Utilisez des implémentations éprouvées
2. **Utilisez des algorithmes modernes** - MD5 et SHA-1 sont obsolètes pour les usages de sécurité
3. **Attention aux oracles temporels** - Utilisez des comparaisons à temps constant
4. **Gérez soigneusement les clés** - Ne les codez jamais en dur dans le code
5. **Utilisez des générateurs d'aléa cryptographiques** - Pas de `rand::random()` pour la crypto

Exemple de comparaison à temps constant :
``` rust
use subtle::ConstantTimeEq;

fn main() {
    let hmac1 = [0x01, 0x02, 0x03, 0x04];
    let hmac2 = [0x01, 0x02, 0x03, 0x04];
    let hmac3 = [0x01, 0x02, 0x03, 0x05];

    // Comparaison vulnérable aux attaques temporelles
    let vulnerable_eq = hmac1 == hmac2;

    // Comparaison à temps constant (sécurisée)
    let secure_eq = hmac1.ct_eq(&hmac2).into();
    let secure_neq = hmac1.ct_eq(&hmac3).into();

    println!("Égalité (vulnérable): {}", vulnerable_eq);
    println!("Égalité (sécurisée): {}", secure_eq);
    println!("Inégalité (sécurisée): {}", !secure_neq);
}
```
### 15.11 Projet pratique : Service d'authentification sécurisé avec async
Voici un exemple plus élaboré combinant plusieurs aspects cryptographiques et utilisant les nouvelles fonctionnalités async de l'édition Rust 2024 :
``` rust
// Cargo.toml:
// [dependencies]
// argon2 = "0.5.2"
// sha2 = "0.10.8"
// tokio = { version = "1.44.2", features = ["full"] }
// uuid = { version = "1.16.0", features = ["v4"] }
// chrono = "0.4.40"

use argon2::{
    password_hash::{
        rand_core::OsRng,
        PasswordHash, PasswordHasher, PasswordVerifier, SaltString
    },
    Argon2
};
use sha2::{Sha256, Digest};
use std::collections::HashMap;
use std::sync::{Arc, Mutex};
use tokio::sync::RwLock;
use uuid::Uuid;
use chrono::{DateTime, Duration, Utc};

// Structure de token de session
struct SessionToken {
    token: String,
    username: String,
    expiry: DateTime<Utc>,
}

// Structure pour notre service d'authentification
struct AuthService {
    users: HashMap<String, String>, // username -> password_hash
    sessions: HashMap<String, SessionToken>, // token -> session_info
}

impl AuthService {
    fn new() -> Self {
        Self {
            users: HashMap::new(),
            sessions: HashMap::new(),
        }
    }

    // Enregistrement d'un nouvel utilisateur
    async fn register(&mut self, username: &str, password: &[u8]) -> Result<(), String> {
        if self.users.contains_key(username) {
            return Err("Utilisateur déjà existant".to_string());
        }

        // Hachage du mot de passe avec Argon2
        let salt = SaltString::generate(&mut OsRng);
        let argon2 = Argon2::default();

        let password_hash = argon2.hash_password(password, &salt)
            .map_err(|e| format!("Erreur de hachage: {}", e))?
            .to_string();

        // Enregistrement de l'utilisateur
        self.users.insert(username.to_string(), password_hash);

        Ok(())
    }

    // Connexion et génération de token
    async fn login(&mut self, username: &str, password: &[u8]) -> Result<String, String> {
        let password_hash = self.users.get(username)
            .ok_or("Utilisateur inconnu")?;

        // Vérification du mot de passe
        let parsed_hash = PasswordHash::new(password_hash)
            .map_err(|_| "Format de hachage invalide")?;

        Argon2::default().verify_password(password, &parsed_hash)
            .map_err(|_| "Mot de passe incorrect")?;

        // Génération d'un token de session
        let token = Uuid::new_v4().to_string();
        let expiry = Utc::now() + Duration::hours(1);

        // Enregistrement de la session
        self.sessions.insert(token.clone(), SessionToken {
            token: token.clone(),
            username: username.to_string(),
            expiry,
        });

        Ok(token)
    }

    // Vérification d'un token de session
    async fn verify_session(&self, token: &str) -> Option<String> {
        let session = self.sessions.get(token)?;

        // Vérification de l'expiration
        if session.expiry < Utc::now() {
            return None;
        }

        Some(session.username.clone())
    }

    // Déconnexion (révocation de token)
    async fn logout(&mut self, token: &str) -> bool {
        self.sessions.remove(token).is_some()
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Créer un service d'authentification thread-safe
    let auth_service = Arc::new(RwLock::new(AuthService::new()));

    // Enregistrement d'utilisateurs
    {
        let mut service = auth_service.write().await;
        service.register("alice", b"mot_de_passe_secret").await?;
        service.register("bob", b"autre_mot_de_passe").await?;
        println!("Utilisateurs enregistrés!");
    }

    // Connexion d'Alice (avec async closure, nouveauté de Rust 2024)
    let alice_token = {
        let mut service = auth_service.write().await;
        service.login("alice", b"mot_de_passe_secret").await?
    };
    println!("Alice connectée avec le token: {}", alice_token);

    // Simulation de requêtes simultanées (démontrant la concurrence)
    let auth_service_clone = Arc::clone(&auth_service);
    let alice_token_clone = alice_token.clone();

    // Utilisation d'une async closure (fonctionnalité de Rust 2024)
    let verify_task = tokio::spawn(async move {
        let service = auth_service_clone.read().await;
        match service.verify_session(&alice_token_clone).await {
            Some(username) => println!("Session valide pour: {}", username),
            None => println!("Session invalide!"),
        }
    });

    // Tentative de connexion avec un mauvais mot de passe
    let bad_login_task = tokio::spawn({
        let auth_service = Arc::clone(&auth_service);
        async move {
            let mut service = auth_service.write().await;
            match service.login("bob", b"mauvais_mot_de_passe").await {
                Ok(_) => println!("Bob connecté (ne devrait pas arriver)"),
                Err(e) => println!("Échec de connexion (attendu): {}", e),
            }
        }
    });

    // Attendre la fin des tâches
    let _ = tokio::join!(verify_task, bad_login_task);

    // Déconnexion
    {
        let mut service = auth_service.write().await;
        if service.logout(&alice_token).await {
            println!("Alice déconnectée avec succès");
        }
    }

    Ok(())
}
```
### 15.12 Conclusion
L'écosystème cryptographique de Rust continue d'évoluer avec l'édition Rust 2024, offrant des outils encore plus performants et sécurisés pour développer des applications nécessitant des fonctionnalités de sécurité avancées. Les bibliothèques comme `sha2`, `sha1`, `md5`, et d'autres crates de l'écosystème RustCrypto, permettent de mettre en œuvre facilement :
- Des fonctions de hachage cryptographiques
- Du chiffrement symétrique et asymétrique
- Des signatures numériques
- Le stockage sécurisé de mots de passe
- La vérification d'intégrité

Les nouvelles fonctionnalités de l'édition Rust 2024, comme les closures asynchrones et les améliorations des blocs `unsafe extern`, contribuent à améliorer encore davantage la robustesse et la flexibilité des applications cryptographiques en Rust.
La sécurité mémoire et la gestion des ressources de Rust contribuent également à réduire les risques de vulnérabilités cryptographiques courantes, ce qui en fait un excellent choix pour les applications sensibles à la sécurité.
En suivant les bonnes pratiques présentées dans ce chapitre, vous pourrez implémenter des solutions cryptographiques robustes dans vos projets Rust.
