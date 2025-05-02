# 3\. Utiliser du code compilé en C

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

La capacité à interagir avec du code C est l'un des grands atouts de Rust. Cette fonctionnalité, connue sous le nom de Foreign Function Interface (FFI), permet à Rust de tirer parti de l'écosystème massif de bibliothèques C existantes tout en profitant des avantages de sécurité et de performance propres à Rust. Ce chapitre vous guidera à travers les étapes essentielles pour intégrer du code C dans vos projets Rust.

## Les bases de la FFI en Rust

### Préparation et dépendances

La première étape consiste généralement à ajouter la crate `libc` à votre projet. Cette bibliothèque fournit les définitions de types C standards pour diverses plateformes et architectures, facilitant considérablement l'interfaçage avec du code C.

``` toml
[dependencies]
libc = "0.2"
```

Bien que cette dépendance ne soit pas strictement obligatoire, elle est fortement recommandée car elle évite d'avoir à redéfinir manuellement les types C standard et garantit une compatibilité multiplateforme.

### Déclaration de fonctions C externes

Pour utiliser une fonction C, vous devez d'abord la déclarer dans votre code Rust. Voici un exemple avec la fonction `printf` de la bibliothèque standard C :

``` rust
use std::ffi::CString;
use libc::{c_char, c_int};

extern "C" {
    fn printf(format: *const c_char, ...) -> c_int;
}

fn main() {
    let message = CString::new("Bonjour depuis Rust vers C: %d\n").unwrap();
    unsafe {
        printf(message.as_ptr(), 42);
    }
}
```

Notez plusieurs éléments importants :

- Le bloc `extern "C"` indique à Rust que les fonctions déclarées suivent la convention d'appel C
- L'appel à la fonction est entouré d'un bloc `unsafe` car Rust ne peut pas garantir la sécurité du code C appelé
- Nous utilisons `CString` pour créer une chaîne de caractères compatible avec C (terminée par un octet nul)

### Exemple avec une fonction système: rename

Voici un exemple utilisant la fonction système `rename` pour renommer un fichier :

``` rust
use std::ffi::CString;
use libc::{c_char, c_int};

// Ajout du mot-clé "unsafe" devant le bloc extern
unsafe extern "C" {
    fn rename(old: *const c_char, new_p: *const c_char) -> c_int;
}

fn main() {
    let old_path = CString::new("ancien_fichier.txt").unwrap();
    let new_path = CString::new("nouveau_fichier.txt").unwrap();

    let result = unsafe {
        rename(old_path.as_ptr(), new_path.as_ptr())
    };

    if result != 0 {
        println!("Échec du renommage: code d'erreur {}", result);
    } else {
        println!("Fichier renommé avec succès!");
    }
}
```

### Pourquoi utiliser les types de la libc ?

Bien qu'il soit possible d'utiliser directement les types Rust comme équivalents aux types C, cette approche est déconseillée :

```
// À éviter
unsafe extern "C" {
    fn rename(old: *const i8, new_p: *const i8) -> i32;
}
```

Les raisons de privilégier les types de `libc` sont multiples :

1.  **Clarté et intention** : Les types comme `c_char` et `c_int` indiquent clairement que vous travaillez avec des types C
2.  **Compatibilité multiplateforme** : Les types C peuvent avoir des tailles différentes selon les plateformes (`int` peut être 16, 32 ou 64 bits), alors que les types Rust ont une taille fixe (`i32` fait toujours 32 bits)
3.  **Correspondance exacte** : Par exemple, `char` en C n'est pas nécessairement équivalent à `i8` en Rust sur toutes les plateformes

## Interfaçage avec une bibliothèque C

### Configuration du linkage

Pour utiliser une bibliothèque C externe, vous devez configurer le linkage dans votre code Rust. Cela se fait généralement avec l'attribut `#[link]` :

``` rust
#[cfg(target_os = "linux")]
mod platform {
    #[link(name = "sqlite3")]
    extern {}
}
```

Ce code indique au compilateur de lier votre programme avec la bibliothèque `libsqlite3.so` sur Linux. Vous pouvez spécifier différentes bibliothèques selon le système d'exploitation et l'architecture :

``` rust
#[cfg(target_os = "linux")]
mod platform {
    #[link(name = "sqlite3")]
    unsafe extern {}
}

#[cfg(target_os = "windows")]
mod platform {
    #[link(name = "sqlite3")]
    unsafe extern {}
}

#[cfg(target_os = "macos")]
mod platform {
    #[link(name = "sqlite3")]
    #[link(framework = "CoreFoundation")]  // Pour les frameworks macOS
    unsafe extern {}
}
```

Vous pouvez également spécifier différentes bibliothèques selon l'architecture :

``` rust
#[cfg(target_os = "linux")]
mod platform {
    #[cfg(target_arch = "x86_64")]
    #[link(name = "ma_lib_64")]
    unsafe extern {}

#[cfg(target_arch = "x86")]
    #[link(name = "ma_lib_32")]
    unsafe extern {}
}
```

### Options supplémentaires de linkage

L'attribut `#[link]` accepte plusieurs options utiles :

``` rust
#[link(name = "ma_bibliotheque",
kind = "static",          // Type de linkage (static, dylib, framework)
       modifiers = "-bundle")]    // Modificateurs spécifiques
unsafe extern {}
```

## Interfaçage complet avec une bibliothèque C

Prenons un exemple concret d'une bibliothèque C fictive qui manipule des fichiers audio :

``` c
// audiolib.h
#define AUDIO_ERROR 0
#define AUDIO_OK 1

typedef struct AudioContext AudioContext;  // Type opaque

AudioContext* audio_create(const char* device_name);
int audio_play_file(AudioContext* ctx, const char* filename);
int audio_set_volume(AudioContext* ctx, float volume);
int audio_register_callback(AudioContext* ctx, void (*callback)(int event_type, void* user_data), void* user_data);
const char* audio_get_last_error(AudioContext* ctx);
void audio_destroy(AudioContext* ctx);
```

### Étape 1 : Définir les bindings FFI

Créons un module `ffi.rs` qui déclare l'interface avec la bibliothèque C :

``` rust
// ffi.rs
use libc::{c_char, c_float, c_int, c_void};

// Constantes
pub const AUDIO_ERROR: c_int = 0;
pub const AUDIO_OK: c_int = 1;

// Type opaque pour représenter le contexte audio
#[repr(C)]
pub struct AudioContext {
    _private: [u8; 0],  // Empêche l'instanciation directe
}

// Type pour le callback
pub type AudioCallbackFn = unsafe extern "C" fn(event_type: c_int, user_data: *mut c_void);

// Déclaration des fonctions externes
unsafe extern "C" {
    #[link(name = "audiolib")]
    pub fn audio_create(device_name: *const c_char) -> *mut AudioContext;

    pub fn audio_play_file(ctx: *mut AudioContext, filename: *const c_char) -> c_int;

    pub fn audio_set_volume(ctx: *mut AudioContext, volume: c_float) -> c_int;

    pub fn audio_register_callback(
        ctx: *mut AudioContext,
        callback: AudioCallbackFn,
        user_data: *mut c_void,
    ) -> c_int;

    pub fn audio_get_last_error(ctx: *mut AudioContext) -> *const c_char;

    pub fn audio_destroy(ctx: *mut AudioContext);
}
```

### Étape 2 : Créer une interface Rust sûre

Maintenant, créons une API Rust sûre qui encapsule les appels C dangereux :

``` rust
// audio.rs
use std::ffi::{CStr, CString};
use std::ptr;
use std::sync::{Arc, Mutex};
use libc::{c_int, c_void};

use crate::ffi;

#[derive(Debug)]
pub enum AudioError {
    InitializationFailed(String),
    PlaybackFailed(String),
    VolumeError(String),
    CallbackError(String),
}

pub type Result<T> = std::result::Result<T, AudioError>;

/// Wrapper sûr autour du contexte audio C
pub struct Audio {
    context: *mut ffi::AudioContext,
    // Stockage des données de callback pour éviter les fuites mémoire
    _callback_data: Option<Arc<Mutex<CallbackData>>>,
}

// Structure pour stocker les données de callback
struct CallbackData {
    user_callback: Box<dyn Fn(i32) + Send + 'static>,
}

// Callback intermédiaire appelé par le code C
unsafe extern "C" fn c_callback_trampoline(event_type: c_int, user_data: *mut c_void) {
    if !user_data.is_null() {
        let callback_data = &*(user_data as *const Arc<Mutex<CallbackData>>);
        if let Ok(data) = callback_data.lock() {
            (data.user_callback)(event_type);
        }
    }
}

impl Audio {
    /// Crée un nouveau contexte audio
    pub fn new(device_name: &str) -> Result<Self> {
        let c_device_name = CString::new(device_name).unwrap();

        let context = unsafe { ffi::audio_create(c_device_name.as_ptr()) };

        if context.is_null() {
            return Err(AudioError::InitializationFailed(
                "Impossible de créer le contexte audio".to_string()
            ));
        }

        Ok(Audio {
            context,
            _callback_data: None,
        })
    }

    /// Joue un fichier audio
    pub fn play_file(&self, filename: &str) -> Result<()> {
        let c_filename = CString::new(filename).unwrap();

        let result = unsafe { ffi::audio_play_file(self.context, c_filename.as_ptr()) };

        if result == ffi::AUDIO_OK {
            Ok(())
        } else {
            Err(AudioError::PlaybackFailed(self.get_last_error()))
        }
    }

    /// Règle le volume (0.0 à 1.0)
    pub fn set_volume(&self, volume: f32) -> Result<()> {
        if volume < 0.0 || volume > 1.0 {
            return Err(AudioError::VolumeError(
                "Le volume doit être entre 0.0 et 1.0".to_string()
            ));
        }

        let result = unsafe { ffi::audio_set_volume(self.context, volume) };

        if result == ffi::AUDIO_OK {
            Ok(())
        } else {
            Err(AudioError::VolumeError(self.get_last_error()))
        }
    }

    /// Enregistre un callback qui sera appelé lors d'événements audio
    pub fn register_callback<F>(&mut self, callback: F) -> Result<()>
    where
        F: Fn(i32) + Send + 'static,
    {
        // Créer les données de callback
        let callback_data = Arc::new(Mutex::new(CallbackData {
            user_callback: Box::new(callback),
        }));

        // Obtenir un pointeur vers les données
        let callback_ptr = Arc::into_raw(callback_data.clone()) as *mut c_void;

        let result = unsafe {
            ffi::audio_register_callback(
                self.context,
                c_callback_trampoline,
                callback_ptr
            )
        };

        if result == ffi::AUDIO_OK {
            self._callback_data = Some(callback_data);
            Ok(())
        } else {
            // Récupérer l'Arc pour éviter les fuites mémoire
            let _ = unsafe { Arc::from_raw(callback_ptr as *const Mutex<CallbackData>) };
            Err(AudioError::CallbackError(self.get_last_error()))
        }
    }

    /// Récupère le dernier message d'erreur
    fn get_last_error(&self) -> String {
        unsafe {
            let error_ptr = ffi::audio_get_last_error(self.context);
            if error_ptr.is_null() {
                "Erreur inconnue".to_string()
            } else {
                CStr::from_ptr(error_ptr)
                    .to_string_lossy()
                    .into_owned()
            }
        }
    }
}

impl Drop for Audio {
    fn drop(&mut self) {
        if !self.context.is_null() {
            unsafe {
                ffi::audio_destroy(self.context);
            }
            self.context = ptr::null_mut();
        }
    }
}

// Implémenter Send et Sync car nous gérons manuellement la synchronisation
unsafe impl Send for Audio {}
unsafe impl Sync for Audio {}
```

### Étape 3 : Utilisation de l'API

Enfin, voici comment utiliser cette API dans votre code Rust :

``` rust
// main.rs
mod ffi;
mod audio;

use audio::Audio;

fn main() {
    match Audio::new("default") {
        Ok(mut audio) => {
            println!("Contexte audio créé avec succès");

            // Régler le volume
            if let Err(e) = audio.set_volume(0.5) {
                eprintln!("Erreur lors du réglage du volume: {:?}", e);
                return;
            }

            // Enregistrer un callback
            if let Err(e) = audio.register_callback(|event| {
                println!("Événement audio reçu: {}", event);
            }) {
                eprintln!("Erreur lors de l'enregistrement du callback: {:?}", e);
                return;
            }

            // Jouer un fichier
            match audio.play_file("musique.mp3") {
                Ok(_) => println!("Lecture démarrée"),
                Err(e) => eprintln!("Erreur de lecture: {:?}", e),
            }

            // La bibliothèque sera automatiquement nettoyée quand `audio` sortira de la portée
        },
        Err(e) => {
            eprintln!("Erreur lors de la création du contexte audio: {:?}", e);
        }
    }
}
```

## Techniques avancées de FFI

### Gestion des types opaques

Les types opaques sont des structures dont vous ne connaissez pas les détails d'implémentation. Il existe plusieurs façons de les gérer en Rust :

1.  **Structure unitaire** (comme montré précédemment) :

```
   #[repr(C)]
   pub struct AudioContext {
       _private: [u8; 0],
   }
```

2.  **Structure vide** :

```
   #[repr(C)]
   pub struct AudioContext;
```

3.  **Utilisation de pointeurs bruts** :

```
   pub type AudioContext = c_void;
```

### Callbacks et gestion de la mémoire

Les callbacks posent un défi particulier car ils créent souvent des références circulaires. L'approche présentée dans l'exemple (avec `Arc<Mutex<...>>`) est une solution robuste qui évite les fuites mémoire.

### Alignement mémoire et représentation des types

Lorsque vous communiquez avec du code C, l'alignement et la représentation des types sont critiques :

``` rust
// Garantit que la structure a exactement la même représentation mémoire qu'en C
#[repr(C)]
pub struct Point {
    x: f32,
    y: f32,
}

// Pour les énumérations, vous pouvez spécifier le type sous-jacent
#[repr(u32)]
pub enum AudioFormat {
    WAV = 1,
    MP3 = 2,
    FLAC = 3,
}
```

### Gestion des chaînes de caractères

La conversion entre chaînes Rust et C nécessite une attention particulière :

``` rust
use libc::{c_char, c_float, c_int, c_void};
use std::ffi::{CString, CStr};

// Rust vers C
fn rust_to_c_string(s: &str) -> CString {
    CString::new(s).expect("Chaîne contenant un octet nul")
}

// C vers Rust (chaîne possédée)
unsafe fn owned_c_string_to_rust(ptr: *mut c_char) -> String {
    let result = CStr::from_ptr(ptr).to_string_lossy().into_owned();
    libc::free(ptr as *mut c_void);  // Libérer la mémoire allouée en C
    result
}

// C vers Rust (chaîne empruntée)
unsafe fn borrowed_c_string_to_rust(ptr: *const c_char) -> &'static str {
    CStr::from_ptr(ptr).to_str().unwrap_or("Chaîne invalide")
}
```

## Conclusion et bonnes pratiques

L'interfaçage avec du code C est une fonctionnalité puissante de Rust, mais qui nécessite une attention particulière. Voici quelques bonnes pratiques à suivre :

1.  **Isolation des FFI** : Gardez tout le code `unsafe` dans un module dédié (comme `ffi.rs`)
2.  **Encapsulation** : Créez toujours une API Rust sûre et ergonomique par-dessus l'API C
3.  **Gestion de la mémoire** : Implémentez soigneusement `Drop` pour libérer les ressources
4.  **Tests** : Testez rigoureusement votre code d'interface pour éviter les fuites mémoire
5.  **Documentation** : Documentez clairement quelles parties de votre API dépendent de code C
6.  **Build scripts** : Utilisez les scripts de compilation de Cargo (`build.rs`) pour la configuration complexe de la liaison

En suivant ces principes, vous pouvez profiter de l'écosystème C tout en conservant les garanties de sécurité et d'ergonomie de Rust.

⏭️ [Documentation et rustdoc](/III-avance/04-documentation-rustdoc.md)
