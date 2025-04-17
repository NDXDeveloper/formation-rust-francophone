# 8\. Le réseau

## Introduction aux réseaux en Rust

Rust offre une excellente bibliothèque standard pour le développement réseau. Dans cette section, nous explorerons principalement les communications réseau en mode "connecté", communément appelé TCP. Cette connaissance vous permettra ensuite d'utiliser d'autres protocoles comme UDP (mode "non-connecté") sans difficulté majeure.

Nous nous concentrerons sur le code synchrone, l'approche asynchrone en Rust fera l'objet d'une discussion séparée.

## Client TCP : fondamentaux

Commençons par comprendre comment créer un client TCP simple en Rust:

``` rust
use std::net::TcpStream;
use std::io::{Read, Write};

fn main() {
    println!("Tentative de connexion au serveur...");
    match TcpStream::connect("127.0.0.1:1234") {
        Ok(mut stream) => {
            println!("Connexion au serveur réussie !");

            // Envoi d'un message au serveur
            let message = "Bonjour, serveur!";
            stream.write(message.as_bytes()).expect("Échec lors de l'envoi du message");
            println!("Message envoyé: {}", message);

            // Réception de la réponse
            let mut buffer = [0; 1024];
            match stream.read(&mut buffer) {
                Ok(size) => {
                    if size > 0 {
                        println!("Réponse reçue: {}", String::from_utf8_lossy(&buffer[..size]));
                    } else {
                        println!("Le serveur a fermé la connexion");
                    }
                },
                Err(e) => println!("Erreur lors de la réception: {}", e),
            }
        },
        Err(e) => {
            println!("La connexion au serveur a échoué: {}", e);
        }
    }
}
```

Si vous exécutez ce code sans serveur correspondant, vous obtiendrez probablement l'erreur "Connection refused", indiquant qu'aucun serveur n'accepte la connexion sur ce port.

### Structure TcpStream

La structure clé utilisée ici est `TcpStream`, qui permet de lire et d'écrire sur un flux réseau. Elle implémente les traits `Read` et `Write`, offrant un ensemble complet de méthodes pour manipuler les données:

- `read()` - Lecture des données depuis le flux
- `write()` - Écriture des données vers le flux
- `flush()` - Garantit que toutes les données tamponnées ont été transmises
- `peek()` - Permet de consulter les données sans les retirer du flux

### Méthodes de connexion

La méthode `connect` accepte tout objet implémentant le trait `ToSocketAddrs`. Voici différentes façons d'utiliser cette méthode:

```
use std::net::{TcpStream, Ipv4Addr, SocketAddrV4};

fn main() {
    // Différentes façons de spécifier une adresse
    let ip = Ipv4Addr::new(127, 0, 0, 1);
    let port = 1234;

    // Méthode 1: Utilisation de SocketAddrV4
    let tcp_s = TcpStream::connect(SocketAddrV4::new(ip, port));

    // Méthode 2: Utilisation d'un tuple (Ipv4Addr, port)
    let tcp_s = TcpStream::connect((ip, port));

    // Méthode 3: Utilisation d'un tuple (chaîne IP, port)
    let tcp_s = TcpStream::connect(("127.0.0.1", port));

    // Méthode 4: Utilisation d'un tuple (nom d'hôte, port)
    let tcp_s = TcpStream::connect(("localhost", port));

    // Méthode 5: Chaîne combinée IP:port
    let tcp_s = TcpStream::connect("127.0.0.1:1234");

    // Méthode 6: Chaîne combinée nom d'hôte:port
    let tcp_s = TcpStream::connect("localhost:1234");
}
```

"localhost" est équivalent à "127.0.0.1" et représente l'adresse de la machine locale.

## Serveur TCP : fondamentaux

Voici maintenant comment créer un serveur TCP simple:

``` rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};

fn main() {
    // Création d'un écouteur TCP lié à l'adresse et au port spécifiés
    let listener = TcpListener::bind("127.0.0.1:1234").expect("Échec de la liaison au port");
    println!("Serveur démarré. En attente de connexions...");

    // Accepter une connexion
    match listener.accept() {
        Ok((mut stream, addr)) => {
            println!("Nouveau client connecté [adresse: {}]", addr);

            // Lire les données envoyées par le client
            let mut buffer = [0; 1024];
            match stream.read(&mut buffer) {
                Ok(size) => {
                    if size > 0 {
                        println!("Message reçu: {}", String::from_utf8_lossy(&buffer[..size]));

                        // Envoyer une réponse
                        let response = "Message bien reçu!";
                        stream.write(response.as_bytes()).expect("Échec de l'envoi de la réponse");
                        println!("Réponse envoyée: {}", response);
                    }
                },
                Err(e) => println!("Erreur lors de la lecture: {}", e)
            }
        },
        Err(e) => println!("Échec de l'acceptation de la connexion: {}", e)
    }
}
```

### Structure TcpListener

La structure `TcpListener` permet d'écouter les connexions TCP entrantes. La méthode statique `bind` spécifie l'adresse et le port sur lesquels le serveur écoutera. Elle accepte les mêmes types de paramètres que la méthode `connect` que nous avons vue précédemment.

La méthode `accept` attend qu'un client se connecte et renvoie un tuple contenant un `TcpStream` (pour communiquer avec le client) et un `SocketAddr` (l'adresse du client).

### Test de la connexion

Pour tester, lancez d'abord le serveur puis le client. Vous devriez obtenir des résultats similaires à:

Côté serveur:

```
Serveur démarré. En attente de connexions...
Nouveau client connecté [adresse: 127.0.0.1:45982]
Message reçu: Bonjour, serveur!
Réponse envoyée: Message bien reçu!
```

Côté client:

```
Tentative de connexion au serveur...
Connexion au serveur réussie !
Message envoyé: Bonjour, serveur!
Réponse reçue: Message bien reçu!
```

## Gestion de multiples clients

Pour gérer plusieurs clients, nous avons deux approches principales :

### Approche 1: Utilisation de la méthode `incoming()`

``` rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:1234").expect("Échec de la liaison au port");
    println!("Serveur démarré. En attente de connexions...");

    // Itérer sur les connexions entrantes
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                let adresse = match stream.peer_addr() {
                    Ok(addr) => format!("[adresse: {}]", addr),
                    Err(_) => "inconnue".to_owned()
                };
                println!("Nouveau client {}", adresse);

                // Traitement du client ici...
                // Note: Ce code traite les clients de façon séquentielle
            },
            Err(e) => {
                println!("Échec de la connexion du client: {}", e);
            }
        }
        println!("En attente d'un autre client...");
    }
}
```

### Approche 2: Traitement multi-client avec threads

Pour traiter plusieurs clients simultanément, nous pouvons utiliser des threads:

``` rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};
use std::thread;

fn handle_client(mut stream: TcpStream) {
    // Identifions le client
    let addr = match stream.peer_addr() {
        Ok(addr) => addr.to_string(),
        Err(_) => "adresse inconnue".to_string()
    };

    println!("Traitement du client {}", addr);

    // Buffer pour stocker les données reçues
    let mut buffer = [0; 1024];

    // Lecture des données envoyées par le client
    match stream.read(&mut buffer) {
        Ok(size) => {
            if size > 0 {
                let message = String::from_utf8_lossy(&buffer[..size]);
                println!("Message reçu de {}: {}", addr, message);

                // Réponse au client
                let response = format!("Merci pour votre message: {}", message);
                stream.write(response.as_bytes()).expect("Échec de l'envoi de la réponse");
            }
        },
        Err(e) => println!("Erreur lors de la lecture depuis {}: {}", addr, e)
    }
}

fn main() {
    let listener = TcpListener::bind("127.0.0.1:1234").expect("Échec de la liaison au port");
    println!("Serveur multi-client démarré. En attente de connexions...");

    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                // Création d'un thread dédié pour chaque client
                thread::spawn(move || {
                    handle_client(stream)
                });
            },
            Err(e) => {
                println!("Échec de la connexion du client: {}", e);
            }
        }
        println!("En attente d'un autre client...");
    }
}
```

## Gestion de la perte de connexion

Un aspect crucial du développement réseau est la détection de la déconnexion des clients. Deux cas principaux indiquent une déconnexion:

1.  Une erreur est retournée lors de la lecture/écriture
2.  La lecture réussit mais renvoie 0 octet (indiquant une fermeture normale de la connexion)

Voici un exemple de code robuste pour gérer ces situations:

``` rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};
use std::thread;
use std::time::Duration;

fn handle_client(mut stream: TcpStream) {
    let addr = match stream.peer_addr() {
        Ok(addr) => addr.to_string(),
        Err(_) => "adresse inconnue".to_string()
    };

    println!("Client connecté: {}", addr);

    // Définir un timeout pour les opérations de lecture
    stream.set_read_timeout(Some(Duration::from_secs(30)))
          .expect("Échec de la définition du timeout");

    let mut buffer = [0; 1024];

    // Boucle de communication
    loop {
        match stream.read(&mut buffer) {
            Ok(0) => {
                // Le client a fermé proprement la connexion
                println!("Client déconnecté normalement: {}", addr);
                break;
            },
            Ok(size) => {
                // Traitement du message reçu
                let message = String::from_utf8_lossy(&buffer[..size]);
                println!("Message de {}: {}", addr, message);

                // Vérification si le client demande à quitter
                if message.trim() == "quit" {
                    println!("Client {} a demandé à quitter", addr);
                    let goodbye = "Au revoir!";
                    let _ = stream.write(goodbye.as_bytes());
                    break;
                }

                // Réponse au client
                match stream.write(b"Message recu!\n") {
                    Ok(_) => {},
                    Err(e) => {
                        println!("Erreur d'écriture vers {}: {}", addr, e);
                        break;
                    }
                }
            },
            Err(e) => {
                // Erreur de lecture - probablement une déconnexion
                println!("Erreur avec le client {}: {}", addr, e);
                break;
            }
        }
    }

    println!("Fin de la gestion du client {}", addr);
}
```

## Exemple complet d'un chat server/client

Voici un exemple plus complet montrant un serveur de chat simple et son client correspondant:

### Serveur de chat

``` rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};
use std::thread;
use std::sync::{Arc, Mutex};
use std::collections::HashMap;

type ClientMap = Arc<Mutex<HashMap<String, TcpStream>>>;

fn handle_client(stream: TcpStream, clients: ClientMap) {
    let addr = match stream.peer_addr() {
        Ok(addr) => addr.to_string(),
        Err(_) => "adresse inconnue".to_string()
    };

    // Enregistrer le client
    {
        let mut stream_clone = stream.try_clone().expect("Échec du clonage du stream");
        let mut clients_map = clients.lock().unwrap();
        clients_map.insert(addr.clone(), stream_clone);

        // Informer les autres clients de la nouvelle connexion
        let connect_msg = format!(">>> Utilisateur {} a rejoint le chat\n", addr);
        broadcast_message(&connect_msg, &addr, &clients_map);
    }

    println!("Nouveau participant: {}", addr);

    let mut stream = stream;
    let mut buffer = [0; 1024];

    // Boucle de lecture des messages
    loop {
        match stream.read(&mut buffer) {
            Ok(0) => break, // Déconnexion
            Ok(size) => {
                let message = String::from_utf8_lossy(&buffer[..size]).to_string();
                println!("{}: {}", addr, message);

                // Diffuser le message à tous les autres clients
                let formatted_msg = format!("{}: {}", addr, message);
                let clients_map = clients.lock().unwrap();
                broadcast_message(&formatted_msg, &addr, &clients_map);
            },
            Err(_) => break, // Erreur de lecture
        }
    }

    // Client déconnecté, le retirer de la liste
    {
        let mut clients_map = clients.lock().unwrap();
        clients_map.remove(&addr);

        // Informer les autres de la déconnexion
        let disconnect_msg = format!(">>> Utilisateur {} a quitté le chat\n", addr);
        broadcast_message(&disconnect_msg, &addr, &clients_map);
    }

    println!("Client déconnecté: {}", addr);
}

fn broadcast_message(message: &str, sender: &str, clients: &HashMap<String, TcpStream>) {
    for (addr, stream) in clients.iter() {
        if addr != sender {
            let mut stream = stream.try_clone().unwrap();
            let _ = stream.write(message.as_bytes());
        }
    }
}

fn main() {
    let listener = TcpListener::bind("127.0.0.1:1234").expect("Échec de la liaison au port");
    println!("Serveur de chat démarré sur 127.0.0.1:1234");

    // Structure partagée pour stocker les clients connectés
    let clients: ClientMap = Arc::new(Mutex::new(HashMap::new()));

    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                let clients_clone = Arc::clone(&clients);
                thread::spawn(move || {
                    handle_client(stream, clients_clone)
                });
            },
            Err(e) => {
                println!("Erreur lors de la connexion du client: {}", e);
            }
        }
    }
}
```

### Client de chat

``` rust
use std::net::TcpStream;
use std::io::{Read, Write, stdin, stdout};
use std::thread;
use std::sync::mpsc;

fn main() {
    println!("Client de chat - Connexion au serveur...");

    // Se connecter au serveur
    match TcpStream::connect("127.0.0.1:1234") {
        Ok(stream) => {
            println!("Connecté au serveur!");
            println!("Tapez vos messages et appuyez sur Entrée pour envoyer");
            println!("Tapez 'quit' pour quitter");

            // Créer un canal pour la communication entre les threads
            let (tx, rx) = mpsc::channel::<String>();

            // Clone du stream pour le thread de lecture
            let mut read_stream = stream.try_clone().expect("Échec du clonage du stream");

            // Thread pour recevoir les messages
            thread::spawn(move || {
                let mut buffer = [0; 1024];
                loop {
                    match read_stream.read(&mut buffer) {
                        Ok(0) => {
                            println!("\nServeur déconnecté");
                            tx.send("quit".to_string()).unwrap(); // Signal pour quitter
                            break;
                        },
                        Ok(size) => {
                            let message = String::from_utf8_lossy(&buffer[..size]);
                            println!("\nReçu: {}", message);
                            print!("> ");
                            stdout().flush().unwrap();
                        },
                        Err(_) => {
                            println!("\nErreur lors de la réception");
                            tx.send("quit".to_string()).unwrap();
                            break;
                        }
                    }
                }
            });

            // Thread principal pour envoyer les messages
            let mut write_stream = stream;
            loop {
                print!("> ");
                stdout().flush().unwrap();

                // Vérifier si le thread de réception a signalé une déconnexion
                if let Ok(signal) = rx.try_recv() {
                    if signal == "quit" {
                        break;
                    }
                }

                // Lire l'entrée utilisateur
                let mut input = String::new();
                stdin().read_line(&mut input).expect("Échec de la lecture de l'entrée");
                let input = input.trim();

                if input == "quit" {
                    println!("Déconnexion...");
                    break;
                }

                // Envoyer le message au serveur
                match write_stream.write(input.as_bytes()) {
                    Ok(_) => {},
                    Err(e) => {
                        println!("Erreur lors de l'envoi: {}", e);
                        break;
                    }
                }
            }
        },
        Err(e) => println!("Échec de la connexion au serveur: {}", e)
    }

    println!("Client terminé");
}
```

## Sockets UDP

Bien que nous nous soyons concentrés sur TCP, voici un aperçu rapide de l'utilisation des sockets UDP en Rust:

### Client UDP

``` rust
use std::net::UdpSocket;

fn main() {
    // Créer un socket UDP lié à n'importe quelle adresse disponible
    let socket = UdpSocket::bind("0.0.0.0:0").expect("Échec de la création du socket");

    // Message à envoyer
    let message = "Bonjour en UDP!";

    // Envoi du message
    socket.send_to(message.as_bytes(), "127.0.0.1:1234")
          .expect("Échec de l'envoi du message");

    println!("Message envoyé: {}", message);

    // Buffer pour la réponse
    let mut buffer = [0; 1024];

    // Réception de la réponse
    match socket.recv_from(&mut buffer) {
        Ok((size, src)) => {
            println!("Réponse de {}: {}",
                     src,
                     String::from_utf8_lossy(&buffer[..size]));
        },
        Err(e) => println!("Erreur de réception: {}", e)
    }
}
```

### Serveur UDP

``` rust
use std::net::UdpSocket;

fn main() {
    // Lier le socket à une adresse et un port
    let socket = UdpSocket::bind("127.0.0.1:1234")
                           .expect("Échec de la liaison du socket");

    println!("Serveur UDP démarré sur 127.0.0.1:1234");

    let mut buffer = [0; 1024];

    loop {
        // Attendre un datagramme
        match socket.recv_from(&mut buffer) {
            Ok((size, src)) => {
                let message = String::from_utf8_lossy(&buffer[..size]);
                println!("Message reçu de {}: {}", src, message);

                // Envoyer une réponse
                let response = "Message UDP reçu!";
                socket.send_to(response.as_bytes(), src)
                      .expect("Échec de l'envoi de la réponse");
            },
            Err(e) => println!("Erreur de réception: {}", e)
        }
    }
}
```

## Bonnes pratiques réseau en Rust

Voici quelques conseils pour écrire du code réseau robuste en Rust:

1.  **Gestion des erreurs**: Utilisez toujours un traitement d'erreur approprié pour les opérations réseau.
2.  **Timeouts**: Configurez des timeouts avec `set_read_timeout()` et `set_write_timeout()` pour éviter les blocages indéfinis.
3.  **Buffering**: Utilisez la taille de buffer appropriée selon vos besoins (1024 octets est généralement un bon point de départ).
4.  **Fermeture propre**: Utilisez `shutdown()` pour fermer proprement les connexions.
5.  **Non-blocage**: Pour les applications plus complexes, envisagez d'utiliser des sockets non-bloquants avec `set_nonblocking(true)`.

## Conclusion

Cette introduction au réseau en Rust vous donne les outils de base pour créer des applications client-serveur robustes. Pour des applications plus complexes ou à haute performance, vous pourriez envisager des bibliothèques comme Tokio ou async-std qui offrent des capacités asynchrones avancées.

Le modèle de sécurité mémoire strict de Rust est particulièrement bénéfique pour les applications réseau, où les erreurs peuvent avoir des conséquences graves sur la sécurité et la stabilité.
