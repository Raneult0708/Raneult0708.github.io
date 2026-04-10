# 🚩 Projet : CTF "Operation NightOwl" & Challenges Divers

Ce document répertorie 20 scénarios de challenges de cybersécurité conçus pour être résistants aux IA et testant des compétences variées (Web, Pwn, Reverse, Forensics, Stegano, Misc).

---

## 🔗 SÉRIE — "Operation NightOwl" (#1 à #5)

Un agent infiltré dans une organisation fictive a laissé des traces sur plusieurs systèmes. Les joueurs doivent reconstituer l'opération complète. Chaque challenge fournit un artefact indispensable pour le suivant.

---

### #1 — Web | "NightOwl : Footprint"
**Difficulté :** Medium  

**Scénario :**  
Un portail web d'une ONG fictive "AfriNet Solutions" expose une API REST non documentée. En énumérant les endpoints, le joueur découvre `/api/v0/legacy/export` qui retourne un fichier de log chiffré. Ce log contient une clé partielle **K1** encodée en base62 dans les métadonnées JSON.

**Résistance IA :**  
L'endpoint est absent du JS frontend. Nécessite un fuzzing avec une wordlist personnalisée (noms de villes africaines).

**Guessing :**  
Le paramètre secret de l'endpoint est un nom de ville namibienne.

**Artefact transmis :**  
Clé K1 + fichier `.pcap` (pièce jointe API).

---

### #2 — Forensics | "NightOwl : Intercept"
**Difficulté :** Medium-Hard  

**Scénario :**  
Le `.pcap` contient du trafic réseau d'une session SSH tunnelée. À l'intérieur du tunnel, des paquets DNS encodent des données dans les champs QNAME (DNS exfiltration). Le joueur doit reconstruire le flux pour obtenir un binaire ELF partiel et une clé partielle **K2**.

**Résistance IA :**  
Flux bruité avec du trafic DNS légitime. Encodage via un alphabet custom (différent du Base64 standard).

**Guessing :**  
La longueur de chaque chunk DNS est un indice sur l'alignement des données.

**Artefact transmis :**  
Binaire ELF partiel + clé K2.

---

### #3 — Reverse + Crypto | "NightOwl : Decryptor"
**Difficulté :** Hard  

**Scénario :**  
Le binaire ELF est un outil de chiffrement custom. Il attend **K1** et **K2** en entrée pour déchiffrer `payload.enc`. Le chiffrement est un XOR à clé glissante avec une dérivation PBKDF2 maison. Le payload révèle une image PNG corrompue et une clé **K3**.

**Résistance IA :**  
Binaire strippé. Logique PBKDF2 réimplémentée manuellement. Nombre d'itérations stocké dans une section ELF personnalisée.

**Artefact transmis :**  
Image PNG corrompue + clé K3.

---

### #4 — Stegano | "NightOwl : Whisper"
**Difficulté :** Medium-Hard  

**Scénario :**  
L'image PNG possède un chunk custom (`stXX`) injecté entre les chunks IDAT. Ce chunk contient un message stéganographique encodé en LSB sur un canal alpha fictif. **K3** sert de seed pour le pattern LSB non-linéaire.

**Résistance IA :**  
Le chunk n'est pas reconnu par les outils standards (zsteg, stegsolve). Nécessite un parsing manuel de la structure PNG.

**Guessing :**  
Le nom du chunk (`stXX`) est un indice sur la technique de stockage.

**Artefact transmis :**  
Credentials SSH + clé **K4** (hash partiel).

---

### #5 — Pwn | "NightOwl : Exfil"
**Difficulté :** Hard  

**Scénario :**  
Les credentials donnent accès à un serveur où tourne un daemon C gérant des rapports. Le binaire présente un Buffer Overflow protégé par un canary dérivé de **K4**. Le joueur doit reconstruire le canary, exploiter le BOF et lire le flag via ret2libc.

**Résistance IA :**  
Canary non aléatoire ; sa dérivation implique le PID du processus et K4 (formule non triviale).

**Fichiers fournis :**  
Binaire + Libc + Dockerfile.

---

## 🌐 WEB (Hors série)

### #6 — "TokenBlind"
**Difficulté :** Medium-Hard  

JWT signés en HS256 avec une clé dérivée d'un timestamp UNIX (arrondi à la minute). Le joueur doit estimer la fenêtre temporelle via les headers HTTP, bruteforcer la clé, forger un token admin et exploiter un IDOR sur `/api/admin/export`.

**Résistance IA :**  
Clé dynamique variant selon le contexte serveur. Guessing sur le schéma de dérivation via un header HTTP custom obfusqué.

---

### #7 — "CachePoison"
**Difficulté :** Hard  

Empoisonnement de cache via un reverse proxy Varnish. Identification d'un header non-keyed (`X-Forwarded-Scheme`) pour voler les cookies d'un bot admin (Puppeteer).

**Guessing :**  
Indice caché dans un commentaire JS : *"le schéma compte"*.

---

### #8 — "SSRF Inception"
**Difficulté :** Medium  

Convertisseur HTML → PDF (headless Chrome). Exploitation d'un SSRF via  
`<iframe src="file:///etc/passwd">` puis rebond sur un Redis interne pour obtenir une RCE.

**Résistance IA :**  
Filtres bloquant `127.0.0.1` et `localhost` (contournement via `0x7f000001` ou `[::]`).

---

## 🔬 REVERSE + CRYPTO (Hors série)

### #9 — "ObsidianVM"
**Difficulté :** Hard  

Binaire ELF implémentant une mini-VM custom. Les opcodes implémentent un schéma LFSR pour chiffrer l'input. Le polynôme est caché dans une section `.note` (timestamp factice).

**Résistance IA :**  
Jeu d'instructions 100% custom sans noms d'opcodes. Polynôme non standard.

---

### #10 — "PyroShield"
**Difficulté :** Medium  

Fichier `.pyc` (Python 3.12) avec bytecode corrompu manuellement pour faire planter les décompilateurs. Analyse manuelle via `dis` pour trouver l'algorithme de validation (CRC32 custom).

---

### #11 — "EmojiLang"
**Difficulté :** Medium  

Interpréteur de langage ésotérique où chaque instruction est un emoji (thème animaux africains). Reverse de l'interpréteur pour comprendre le jeu d'instructions.

---

## 💥 PWN (Hors série)

### #12 — "SafeHeap"
**Difficulté :** Hard  

Gestionnaire de notes en C avec allocateur custom. Use-After-Free et contournement de canary custom (XOR avec seed dérivé du PID). Fuite du PID via un message d'erreur.

---

### #13 — "EchoBreak"
**Difficulté :** Medium  

Format String Vulnerability dans un serveur de messagerie. Leak de la stack, contournement PIE et redirection vers une fonction `win()` cachée dans la section `.__hidden`.

---

## 🔍 FORENSICS (Hors série)

### #14 — "GhostPacket"
**Difficulté :** Medium  

Exfiltration via champs TTL et ID ICMP. Message chiffré en XOR avec une clé encodée en base85 dans un enregistrement DNS TXT.

---

### #15 — "MemoryLeak"
**Difficulté :** Hard  

Dump mémoire Linux (`.lime`). Malware fileless. Utilisation de Volatility (profil à reconstruire) pour extraire le shellcode et le flag XOR-encodé.

---

## 🖼️ STEGANO (Hors série)

### #16 — "SilentFreq"
**Difficulté :** Medium  

Fichier WAV avec message caché dans la phase du spectre FFT.

**Guessing :**  
*"Ce qui se cache n'est pas dans ce que tu entends, mais dans comment tu l'entends".*

---

### #17 — "PixelPath"
**Difficulté :** Medium  

Le flag est encodé dans l'ordre de parcours des pixels (coordonnées dont la valeur Rouge est paire).

**Guessing :**  
*"Le chemin est dans les pairs".*

---

## 🎲 MISC (Hors série)

### #18 — "BabelScript"
**Difficulté :** Medium  

Langage ésotérique (type Brainfuck) utilisant des caractères Tifinagh/Arabes. Spec partielle fournie en PDF.

---

### #19 — "ChessBlind"
**Difficulté :** Medium  

Message binaire encodé dans une partie d'échecs PGN (Prise = 1, Mouvement simple = 0). Nécessite une normalisation de la notation longue.

---

### #20 — "RFCHunter"
**Difficulté :** Medium-Hard  

Identifier 5 violations subtiles de RFC dans un trafic réseau custom. Le flag est composé des numéros de RFC concernés.

---

## 📊 Tableau Récapitulatif

| #  | Catégorie     | Titre                    | Difficulté   | Série   | Guessing |
|----|--------------|--------------------------|--------------|--------|----------|
| 1  | Web          | NightOwl : Footprint     | Medium       | ✅ 1/5 | ✅ |
| 2  | Forensics    | NightOwl : Intercept     | Medium-Hard  | ✅ 2/5 | ✅ |
| 3  | Rev+Crypto   | NightOwl : Decryptor     | Hard         | ✅ 3/5 | ❌ |
| 4  | Stegano      | NightOwl : Whisper       | Medium-Hard  | ✅ 4/5 | ✅ |
| 5  | Pwn          | NightOwl : Exfil         | Hard         | ✅ 5/5 | ❌ |
| 6  | Web          | TokenBlind               | Medium-Hard  | ❌ | ✅ |
| 7  | Web          | CachePoison              | Hard         | ❌ | ✅ |
| 8  | Web          | SSRF Inception           | Medium       | ❌ | ❌ |
| 9  | Rev+Crypto   | ObsidianVM               | Hard         | ❌ | ✅ |
| 10 | Rev          | PyroShield               | Medium       | ❌ | ❌ |
| 11 | Rev          | EmojiLang                | Medium       | ❌ | ✅ |
| 12 | Pwn          | SafeHeap                 | Hard         | ❌ | ❌ |
| 13 | Pwn          | EchoBreak                | Medium       | ❌ | ❌ |
| 14 | Forensics    | GhostPacket              | Medium       | ❌ | ✅ |
| 15 | Forensics    | MemoryLeak               | Hard         | ❌ | ❌ |
| 16 | Stegano      | SilentFreq               | Medium       | ❌ | ✅ |
| 17 | Stegano      | PixelPath                | Medium       | ❌ | ✅ |
| 18 | Misc         | BabelScript              | Medium       | ❌ | ✅ |
| 19 | Misc         | ChessBlind               | Medium       | ❌ | ❌ |
| 20 | Misc         | RFCHunter                | Medium-Hard  | ❌ | ❌ |
