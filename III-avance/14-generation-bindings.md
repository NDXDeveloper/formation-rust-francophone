## 14. **Génération de bindings** - Création d'interfaces pour d'autres langages

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

La puissance de Rust peut être exploitée par d'autres langages de programmation grâce à la génération de bindings. Cette technique permet de créer des interfaces entre Rust et d'autres langages, rendant possible l'utilisation de code Rust depuis C, Python, JavaScript et bien d'autres.
### 14.1 Pourquoi créer des bindings ?
Plusieurs raisons peuvent motiver la création d'interfaces entre Rust et d'autres langages :
- Profiter des performances et de la sécurité mémoire de Rust dans un projet existant
- Réutiliser une bibliothèque Rust dans un écosystème différent
- Optimiser les parties critiques d'une application en les écrivant en Rust
- Faciliter la transition progressive d'un projet vers Rust

### 14.2 Bindings pour C/C++ avec `cbindgen`
L'outil `cbindgen` est essentiel pour générer des en-têtes C/C++ à partir de code Rust.
#### Installation de cbindgen
``` bash
cargo install cbindgen
```
#### Exemple de bibliothèque Rust avec API C
Commençons par créer une bibliothèque Rust simple contenant une fonction de hachage MD5 :
``` rust
// lib.rs
use md5::{Digest, Md5};

#[no_mangle]
pub extern "C" fn rust_md5_hash(input: *const u8, len: usize, output: *mut u8) {
    // Vérification de sécurité
    if input.is_null() || output.is_null() {
        return;
    }

    // Conversion des pointeurs en tranches sûres
    let input_slice = unsafe { std::slice::from_raw_parts(input, len) };

    // Calcul du hachage MD5
    let mut hasher = Md5::new();
    hasher.update(input_slice);
    let result = hasher.finalize();

    // Copie du résultat dans le buffer de sortie
    unsafe {
        std::ptr::copy_nonoverlapping(result.as_ptr(), output, 16); // MD5 = 16 bytes
    }
}
```
Créons maintenant un fichier `cbindgen.toml` pour configurer la génération des en-têtes :
``` toml
language = "C"
header = "/* Bindings Rust MD5 générés automatiquement */"
include_guard = "RUST_MD5_H"
```
Génération de l'en-tête C :
``` bash
cbindgen --config cbindgen.toml --crate my_hash_lib --output include/rust_md5.h
```
L'en-tête C généré ressemblera à ceci :
``` c
/* Bindings Rust MD5 générés automatiquement */
#ifndef RUST_MD5_H
#define RUST_MD5_H

#include <stdint.h>
#include <stdlib.h>
#include <stdbool.h>

void rust_md5_hash(const uint8_t *input, size_t len, uint8_t *output);

#endif /* RUST_MD5_H */
```
### 14.3 Bindings pour Python avec PyO3
PyO3 est un framework qui facilite la création de modules Python natifs en Rust.
#### Configuration du projet
Créons un module Python qui expose des fonctions de hachage avec des performances optimales :
``` toml
# Cargo.toml
[package]
name = "rusthash"
version = "0.1.0"
edition = "2024"

[lib]
name = "rusthash"
crate-type = ["cdylib"]

[dependencies]
pyo3 = { version = "0.20", features = ["extension-module"] }
sha2 = "0.11.0-pre.5"
sha1 = "0.11.0-pre.5"
md5 = "0.7.0"
```
#### Implémentation du module Python
``` rust
// lib.rs
use pyo3::prelude::*;
use sha2::{Sha256, Digest};
use sha1::Sha1;
use md5::Md5;

#[pyfunction]
fn md5_hash(py: Python<'_>, data: &[u8]) -> PyResult<PyObject> {
    let mut hasher = Md5::new();
    hasher.update(data);
    let result = hasher.finalize();

    // Conversion en bytes Python
    Ok(PyBytes::new(py, &result).into())
}

#[pyfunction]
fn sha1_hash(py: Python<'_>, data: &[u8]) -> PyResult<PyObject> {
    let mut hasher = Sha1::new();
    hasher.update(data);
    let result = hasher.finalize();

    Ok(PyBytes::new(py, &result).into())
}

#[pyfunction]
fn sha256_hash(py: Python<'_>, data: &[u8]) -> PyResult<PyObject> {
    let mut hasher = Sha256::new();
    hasher.update(data);
    let result = hasher.finalize();

    Ok(PyBytes::new(py, &result).into())
}

#[pymodule]
fn rusthash(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(md5_hash, m)?)?;
    m.add_function(wrap_pyfunction!(sha1_hash, m)?)?;
    m.add_function(wrap_pyfunction!(sha256_hash, m)?)?;

    m.add("__doc__", "Module de hachage rapide implémenté en Rust")?;
    Ok(())
}
```
#### Compilation et utilisation
Compilation du module :
``` bash
cargo build --release
```
Exemple d'utilisation en Python :
``` python
import rusthash
import hashlib
import time

# Test de performance
data = b"Test de performance des fonctions de hachage" * 100000

# MD5 en Python natif
start = time.time()
hashlib.md5(data).hexdigest()
python_time = time.time() - start

# MD5 avec notre module Rust
start = time.time()
rusthash.md5_hash(data).hex()
rust_time = time.time() - start

print(f"Python: {python_time:.6f}s, Rust: {rust_time:.6f}s")
print(f"Accélération: {python_time/rust_time:.2f}x")
```
### 14.4 Bindings pour WebAssembly avec wasm-bindgen
Pour le web, `wasm-bindgen` permet de créer des interfaces entre Rust et JavaScript.
#### Configuration du projet
``` toml
# Cargo.toml
[package]
name = "rust-hash-wasm"
version = "0.1.0"
edition = "2024"

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"
sha2 = "0.11.0-pre.5"
sha1 = "0.11.0-pre.5"
md5 = "0.7.0"
js-sys = "0.3"
```
#### Implémentation du module WASM
``` rust
// lib.rs
use wasm_bindgen::prelude::*;
use sha2::{Sha256, Digest};
use sha1::Sha1;
use md5::Md5;

#[wasm_bindgen]
pub fn md5_hash(data: &[u8]) -> Vec<u8> {
    let mut hasher = Md5::new();
    hasher.update(data);
    hasher.finalize().to_vec()
}

#[wasm_bindgen]
pub fn sha1_hash(data: &[u8]) -> Vec<u8> {
    let mut hasher = Sha1::new();
    hasher.update(data);
    hasher.finalize().to_vec()
}

#[wasm_bindgen]
pub fn sha256_hash(data: &[u8]) -> Vec<u8> {
    let mut hasher = Sha256::new();
    hasher.update(data);
    hasher.finalize().to_vec()
}

#[wasm_bindgen]
pub fn bytes_to_hex(bytes: &[u8]) -> String {
    bytes.iter()
         .map(|b| format!("{:02x}", b))
         .collect()
}
```
#### Compilation et utilisation
Compilation avec wasm-pack :
``` bash
wasm-pack build --target web
```
Utilisation dans une page web :
``` html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Démo Hash WASM</title>
    <script type="module">
        import init, { md5_hash, sha1_hash, sha256_hash, bytes_to_hex } from './pkg/rust_hash_wasm.js';

        async function run() {
            await init();

            const textEncoder = new TextEncoder();
            const data = textEncoder.encode("Hello, WebAssembly!");

            // Utilisation des fonctions de hachage
            const md5Result = md5_hash(data);
            const sha1Result = sha1_hash(data);
            const sha256Result = sha256_hash(data);

            document.getElementById('md5').textContent = bytes_to_hex(md5Result);
            document.getElementById('sha1').textContent = bytes_to_hex(sha1Result);
            document.getElementById('sha256').textContent = bytes_to_hex(sha256Result);
        }

        run();
    </script>
</head>
<body>
    <h1>Démonstration de hachage avec Rust + WebAssembly</h1>
    <p>Texte : "Hello, WebAssembly!"</p>
    <p>MD5 : <span id="md5"></span></p>
    <p>SHA-1 : <span id="sha1"></span></p>
    <p>SHA-256 : <span id="sha256"></span></p>
</body>
</html>
```
### 14.5 Bindings FFI manuels
Pour des cas plus spécifiques, il est parfois nécessaire de créer manuellement des interfaces FFI (Foreign Function Interface).
#### Exemple avec un calcul CRC32
``` rust
// lib.rs
use crc::{Crc, CRC_32_ISO_HDLC};

#[no_mangle]
pub extern "C" fn calculate_crc32(data: *const u8, len: usize) -> u32 {
    if data.is_null() {
        return 0;
    }

    // Conversion en slice Rust
    let slice = unsafe { std::slice::from_raw_parts(data, len) };

    // Calcul du CRC32
    let crc = Crc::<u32>::new(&CRC_32_ISO_HDLC);
    crc.checksum(slice)
}
```
#### Utilisation depuis C
``` c
// main.c
#include <stdio.h>
#include <stdint.h>

// Déclaration de la fonction externe
extern uint32_t calculate_crc32(const uint8_t* data, size_t len);

int main() {
    const char* text = "Test CRC32";
    uint32_t crc = calculate_crc32((const uint8_t*)text, 9);
    printf("CRC32: %08x\n", crc);
    return 0;
}
```
Pour compiler et lier :
``` bash
# Compiler la bibliothèque Rust
cargo build --release

# Compiler le programme C et le lier à la bibliothèque Rust
gcc -o crc_test main.c -L./target/release -lrust_crc -Wl,-rpath,./target/release
```
### 14.6 Gestion de la mémoire entre langages
La gestion de la mémoire est un aspect critique lors de la création de bindings. Voici quelques principes à respecter :
1. **Propriété des données** : Définir clairement quel langage est responsable de l'allocation et de la libération de la mémoire
2. **Conversion des types** : Créer des interfaces de conversion sûres entre les types Rust et ceux du langage cible
3. **Libération des ressources** : Implémenter des fonctions pour libérer correctement les ressources allouées par Rust

Exemple de gestion de chaînes de caractères :
``` rust
use std::ffi::{CStr, CString};
use std::os::raw::c_char;

#[no_mangle]
pub extern "C" fn rust_uppercase(input: *const c_char) -> *mut c_char {
    // Vérification de sécurité
    if input.is_null() {
        return std::ptr::null_mut();
    }

    // Conversion en &str
    let c_str = unsafe { CStr::from_ptr(input) };
    let rust_str = match c_str.to_str() {
        Ok(s) => s,
        Err(_) => return std::ptr::null_mut(),
    };

    // Transformation
    let uppercase = rust_str.to_uppercase();

    // Conversion en CString et transfert de propriété
    match CString::new(uppercase) {
        Ok(c_string) => c_string.into_raw(),
        Err(_) => std::ptr::null_mut(),
    }
}

#[no_mangle]
pub extern "C" fn rust_free_string(ptr: *mut c_char) {
    if !ptr.is_null() {
        unsafe {
            // Reprendre la propriété et laisser CString libérer la mémoire
            let _ = CString::from_raw(ptr);
        }
    }
}
```
### 14.7 Outils et ressources complémentaires
Plusieurs autres outils sont disponibles pour faciliter la création de bindings :
- **bindgen** : Génère des bindings Rust à partir de code C/C++
- **rust-cpython** : Alternative à PyO3 pour l'intégration avec Python
- **juniper** : Pour créer des API GraphQL en Rust
- **neon** : Pour créer des modules Node.js en Rust

### 14.8 Bonnes pratiques
1. **Documentation** : Documenter clairement l'interface et les conventions de mémoire
2. **Tests inter-langages** : Écrire des tests qui vérifient le comportement correct des bindings
3. **Gestion d'erreurs** : Implémenter un mécanisme cohérent de propagation des erreurs entre les langages
4. **Versions** : Maintenir une compatibilité stricte des ABI entre les versions
5. **Sécurité** : Vérifier soigneusement les entrées provenant d'autres langages pour éviter les vulnérabilités

### 14.9 Conclusion
La création de bindings permet d'exploiter la puissance de Rust depuis n'importe quel environnement de programmation. Cette approche hybride combine l'efficacité et la sécurité de Rust avec la facilité d'utilisation et l'écosystème d'autres langages.
Les techniques présentées dans ce chapitre vous permettront d'intégrer du code Rust à vos projets existants ou de créer des bibliothèques utilisables par plusieurs langages, élargissant ainsi la portée de vos développements Rust.
### Notes sur les changements pour Rust 2024 Edition
Pour adapter ce tutoriel à la Rust 2024 Edition, les modifications suivantes ont été apportées [[1]](https://codeandbitters.com/rust-2024-upgrade/) [[5]](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html) :
1. **Edition dans Cargo.toml** : Modification de `edition = "2021"` à `edition = "2024"` dans les exemples de configuration
2. **Durées de vie explicites pour Python** : Ajout de l'annotation de durée de vie dans les fonctions PyO3 (`Python<'_>`) pour respecter les nouvelles règles de capture de durée de vie pour `impl Trait`
3. **Mise à jour des versions de dépendances** :
    - Mise à jour de PyO3 vers la version 0.20
    - Utilisation des versions préliminaires correctes pour les bibliothèques sha1 et sha2 (0.11.0-pre.5)

4. **Annotations explicites** : Ajustement des annotations pour garantir la compatibilité avec les nouvelles règles de sécurité et de capture de durée de vie

Ces modifications permettent de tirer profit des nouvelles fonctionnalités de Rust 2024 Edition tout en maintenant la compatibilité avec le code existant.

⏭️ [Écosystème Rust pour la cryptographie](/III-avance/15-cryptographie.md) - Utilisation des crates de hachage et cryptographie (sha2, sha1, md5)
