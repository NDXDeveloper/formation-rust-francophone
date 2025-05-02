## 10\. Rust embarqué - Introduction à la programmation sur microcontrôleurs

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

### Introduction au Rust embarqué

Le Rust est particulièrement bien adapté à la programmation embarquée grâce à son contrôle précis de la mémoire, sa sécurité sans surcoût d'exécution et ses abstractions à coût zéro. Cette section introduit les concepts fondamentaux pour utiliser Rust sur des microcontrôleurs.

### Configuration de l'environnement

Pour commencer à développer sur des microcontrôleurs en Rust, nous devons installer quelques outils spécifiques :

``` bash
# Installation des outils cross-compilation
rustup target add thumbv7m-none-eabi  # Pour ARM Cortex-M3
rustup target add thumbv6m-none-eabi  # Pour ARM Cortex-M0/M0+
rustup target add thumbv7em-none-eabi # Pour ARM Cortex-M4 sans FPU
rustup target add thumbv7em-none-eabihf # Pour ARM Cortex-M4 avec FPU

# Installation de cargo-binutils
cargo install cargo-binutils
rustup component add llvm-tools-preview
```

### Structure d'un projet embarqué

Un projet Rust embarqué typique ressemble à ceci :

``` toml
# Cargo.toml
[package]
name = "mon_projet_embarque"
version = "0.1.0"
edition = "2021"

[dependencies]
cortex-m = "0.7.7"        # Support de base pour les processeurs Cortex-M
cortex-m-rt = "0.7.3"     # Support runtime minimal
panic-halt = "0.2.0"      # Gestion simple des panics
embedded-hal = "0.2.7"    # Abstractions de matériel

[profile.release]
opt-level = "z"           # Optimisation pour la taille
lto = true                # Link-time optimization
codegen-units = 1         # Meilleure optimisation mais compilation plus lente
debug = false             # Pas d'informations de débogage pour réduire la taille
```

### Premier programme : Clignotement de LED

Voici un exemple simple pour faire clignoter une LED sur une carte STM32F3 Discovery :

``` rust
#![no_std]                // Pas de bibliothèque standard
#![no_main]               // Pas de fonction main standard

use cortex_m_rt::entry;   // Point d'entrée pour notre application
use panic_halt as _;      // Gestion des panics
use stm32f3xx_hal::{
    prelude::*,           // Import des traits utiles
    pac,                  // Peripheral Access Crate
    delay::Delay,
};

#[entry]
fn main() -> ! {          // La fonction ne renvoie jamais (boucle infinie)
    // Accès aux périphériques
    let cp = cortex_m::Peripherals::take().unwrap();
    let dp = pac::Peripherals::take().unwrap();

    // Configuration de l'horloge
    let mut flash = dp.FLASH.constrain();
    let mut rcc = dp.RCC.constrain();
    let clocks = rcc.cfgr.freeze(&mut flash.acr);

    // Création d'un délai basé sur l'horloge système
    let mut delay = Delay::new(cp.SYST, clocks);

    // Configuration du port GPIO
    let mut gpioe = dp.GPIOE.split(&mut rcc.ahb);
    let mut led = gpioe
        .pe9
        .into_push_pull_output(&mut gpioe.moder, &mut gpioe.otyper);

    // Boucle principale
    loop {
        led.set_high().unwrap();  // Allume la LED
        delay.delay_ms(500_u16);  // Attend 500ms
        led.set_low().unwrap();   // Éteint la LED
        delay.delay_ms(500_u16);  // Attend 500ms
    }
}
```

### Gestion des périphériques avec `embedded-hal`

L'écosystème Rust embarqué est centré autour de la bibliothèque `embedded-hal`, qui fournit des traits pour interagir avec les périphériques matériels de manière abstraite.

``` rust
use embedded_hal::digital::v2::{InputPin, OutputPin};

// Fonction générique qui fonctionne avec n'importe quel microcontrôleur
 // supportant embedded-hal
fn lire_bouton_et_allumer_led<B, L>(bouton: &mut B, led: &mut L) -> Result<(), ()>
where
    B: InputPin,
    L: OutputPin,
{
    if bouton.is_high()? {
        led.set_high()?;
    } else {
        led.set_low()?;
    }
    Ok(())
}
```

### Communication série (UART)

Voici un exemple de communication UART sur un microcontrôleur :

``` rust
use core::fmt::Write;
use stm32f3xx_hal::{
    pac,
    prelude::*,
    serial::{Serial, Tx},
};

fn configurer_uart(dp: pac::Peripherals, clocks: stm32f3xx_hal::rcc::Clocks)
    -> Tx<pac::USART1>
{
    // Configuration des broches
    let mut gpioc = dp.GPIOC.split();
    let tx_pin = gpioc.pc4.into_af7();
    let rx_pin = gpioc.pc5.into_af7();

    // Configuration de l'UART
    let serial = Serial::usart1(
        dp.USART1,
        (tx_pin, rx_pin),
        9600.bps(),
        clocks,
    );

    // Séparation des canaux TX et RX
    let (tx, _rx) = serial.split();

    // Retourne le transmetteur
    tx
}

// Utilisation
let mut tx = configurer_uart(dp, clocks);
writeln!(tx, "Hello from Rust on a microcontroller!").unwrap();
```

### Communication I2C

Exemple de lecture d'un capteur de température I2C :

``` rust
use stm32f3xx_hal::{
    prelude::*,
    i2c::{I2c, Mode},
    pac,
};

fn lire_temperature_i2c(dp: pac::Peripherals, clocks: stm32f3xx_hal::rcc::Clocks) -> Result<f32, ()> {
    // Configuration des broches I2C
    let mut gpiob = dp.GPIOB.split();
    let scl = gpiob.pb6.into_af4_open_drain();
    let sda = gpiob.pb7.into_af4_open_drain();

    // Configuration du bus I2C
    let i2c = I2c::i2c1(
        dp.I2C1,
        (scl, sda),
        Mode::Standard { frequency: 100_000.hz() },
        clocks,
    );

    // Adresse du capteur de température (exemple avec LM75)
    const ADRESSE_CAPTEUR: u8 = 0x48;

    // Buffer pour la lecture
    let mut buffer = [0u8; 2];

    // Lecture du registre de température
    match i2c.read(ADRESSE_CAPTEUR, &mut buffer) {
        Ok(_) => {
            // Conversion des données
            let temp_raw = ((buffer[0] as i16) << 8) | (buffer[1] as i16);
            let temperature = (temp_raw >> 5) as f32 * 0.125;
            Ok(temperature)
        },
        Err(_) => Err(()),
    }
}
```

### Interruptions

La gestion des interruptions en Rust embarqué est sûre et expressive :

``` rust
use cortex_m::peripheral::NVIC;
use stm32f3xx_hal::{
    gpio::{Edge, ExtiPin, Input, PullUp},
    pac::{self, interrupt, Interrupt},
};

// Variable globale pour communiquer avec l'ISR
static mut BOUTON_PRESSE: bool = false;

// Configuration du bouton avec interruption
fn configurer_bouton_interruption(dp: &mut pac::Peripherals) -> impl ExtiPin {
    let mut syscfg = dp.SYSCFG.constrain();
    let mut exti = dp.EXTI;

    // Configuration du GPIO
    let mut gpioa = dp.GPIOA.split();
    let mut bouton = gpioa.pa0.into_pull_up_input(&mut gpioa.moder, &mut gpioa.pupdr);

    // Configuration de l'interruption
    bouton.make_interrupt_source(&mut syscfg);
    bouton.trigger_on_edge(&mut exti, Edge::FALLING);
    bouton.enable_interrupt(&mut exti);

    // Activation de l'interruption dans le NVIC
    unsafe {
        NVIC::unmask(Interrupt::EXTI0);
    }

    bouton
}

#[interrupt]
fn EXTI0() {
    // Marqueur que le bouton a été pressé
    unsafe {
        BOUTON_PRESSE = true;
    }

    // Acquittement de l'interruption
    let mut exti = unsafe { pac::Peripherals::steal().EXTI };
    exti.pr.write(|w| w.pr0().set_bit());
}
```

### Gestion de la mémoire en environnement `no_std`

En programmation embarquée sans système d'exploitation, nous devons faire attention à la gestion de la mémoire :

``` rust
use core::mem::MaybeUninit;
use heapless::Vec;

// Tableau de taille fixe sur la pile
fn collecter_donnees() -> Vec<u16, 64> {  // Max 64 éléments
    let mut donnees = Vec::<u16, 64>::new();

    // Simulation de collecte de données
    for i in 0..10 {
        if donnees.push(i).is_err() {
            // Gestion du dépassement de capacité
            break;
        }
    }

    donnees
}

// Buffer statique
static mut BUFFER: [MaybeUninit<u8>; 1024] = [MaybeUninit::uninit(); 1024];

fn utiliser_buffer_statique() {
    unsafe {
        // Initialisation partielle du buffer
        for i in 0..100 {
            BUFFER[i].write(i as u8);
        }

        // Utilisation des données
        let premier_octet = BUFFER[0].assume_init();
    }
}
```

### Débogage sur matériel

Le débogage est crucial en développement embarqué. Voici comment configurer un projet pour le débogage avec probe-run :

``` toml
# Cargo.toml
[dependencies]
# ...autres dépendances
defmt = "0.3.4"           # Format de journalisation optimisé
defmt-rtt = "0.4.0"       # Transmission via RTT
panic-probe = { version = "0.3.1", features = ["print-defmt"] }

[dev-dependencies]
defmt-test = "0.3.0"

# Configuration pour probe-run
[target.'cfg(all(target_arch = "arm", target_os = "none"))']
runner = "probe-run --chip STM32F303VCT6"
```

Exemple de code avec journalisation :

``` rust
#![no_std]
#![no_main]

use defmt_rtt as _;       // Implémentation RTT pour defmt
use panic_probe as _;     // Gestion des panics
use stm32f3xx_hal::prelude::*;

#[defmt::panic_handler]
fn panic() -> ! {
    cortex_m::asm::udf()  // Undefined instruction pour déclencher un arrêt
}

#[cortex_m_rt::entry]
fn main() -> ! {
    defmt::info!("Application démarrée!");

    // Configuration matérielle
    defmt::debug!("Configuration des périphériques...");

    // Logique principale
    let compteur = 42;
    defmt::info!("Valeur du compteur: {}", compteur);

    // Boucle infinie
    loop {
        defmt::trace!("Itération de la boucle");
        // ...
    }
}
```

### Conclusion

La programmation embarquée en Rust offre de nombreux avantages par rapport aux langages traditionnels comme C ou C++ :

1.  **Sécurité mémoire** sans coût d'exécution
2.  **Abstractions puissantes** via l'écosystème `embedded-hal`
3.  **Détection d'erreurs à la compilation** plutôt qu'à l'exécution
4.  **Outillage moderne** avec cargo et rustup

Pour aller plus loin dans le développement embarqué avec Rust :

- Explorez le [livre de référence sur Rust embarqué](https://docs.rust-embedded.org/book/)
- Découvrez la collection de [crates découvrables](https://github.com/rust-embedded/awesome-embedded-rust)
- Rejoignez la communauté Rust Embedded sur [Matrix](https://matrix.to/#/#rust-embedded:matrix.org)

La prochaine étape dans votre parcours d'apprentissage pourrait être d'explorer WebAssembly avec Rust, qui est le sujet de notre prochaine section.

⏭️ [WebAssembly avec Rust](/III-avance/11-webassembly.md) - Compilation vers WASM et interopérabilité avec JavaScript
