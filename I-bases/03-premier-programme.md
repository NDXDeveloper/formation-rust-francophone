## 3\. Premier programme

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

Un premier programme est toujours une étape importante pour commencer à apprendre un nouveau langage. En Rust, c'est particulièrement intéressant car on peut déjà observer certaines particularités du langage.

### 3.1 Création et compilation d'un programme simple

Commençons par créer un fichier nommé `hello.rs` contenant le code suivant :

``` rust
fn main() {
    println!("Bonjour, monde!");
}
```

Quelques observations importantes sur ce code :

- La fonction `main()` est le point d'entrée de tout programme Rust
- `println!` se termine par un point d'exclamation car c'est une macro et non une fonction
- Les instructions se terminent par des points-virgules (`;`)
- Les blocs de code sont délimités par des accolades `{}`

Pour compiler ce programme, utilisez la commande suivante dans votre terminal :

``` bash
rustc hello.rs
```

Cette commande génère un exécutable `hello` (ou `hello.exe` sous Windows). Pour l'exécuter :

**Sous Windows :**

``` bash
.\hello.exe
```

**Sous Linux/macOS :**

``` bash
./hello
```

Vous devriez alors voir s'afficher :

```
Bonjour, monde!
```

### 3.2 Options de compilation

Vous pouvez personnaliser le processus de compilation avec diverses options :

- Pour changer le nom de l'exécutable généré :

``` bash
rustc hello.rs -o mon_programme
```

- Pour compiler avec des optimisations de performance :

``` bash
rustc -O hello.rs
```

- Pour afficher les avertissements supplémentaires lors de la compilation :

``` bash
rustc -W warnings hello.rs
```

### 3.3 Utilisation de Cargo

Dans la pratique, la plupart des développeurs Rust utilisent Cargo pour gérer leurs projets. Voici comment créer un projet "Hello World" avec Cargo :

``` bash
cargo new hello_cargo
cd hello_cargo
cargo run
```

Cette dernière commande compile et exécute automatiquement votre programme. Le code se trouve dans `src/main.rs`.

⏭️ [Variables](/I-bases/04-variables.md)
