# LAB 3 : Observation du trafic HTTP(S) Android avec Burp Suite

## 📌 Objectif du lab

Comprendre comment un proxy d'interception (Burp Suite) capture et analyse le trafic réseau entre un émulateur Android et Internet. Apprendre à configurer un proxy, intercepter des requêtes HTTP/HTTPS, et documenter les observations.

---

## 🎯 Ce que nous avons appris

| Concept | Définition |
|---------|-------------|
| **Proxy** | Intermédiaire qui se place entre le client et le serveur pour observer/modifier le trafic |
| **Burp Suite** | Outil professionnel d'analyse de sécurité (proxy d'interception) |
| **HTTP history** | Mode passif : Burp enregistre les requêtes sans les bloquer |
| **Intercept** | Mode actif : Burp bloque la requête pour analyse avant de la transmettre |
| **Listener** | "Serveur" Burp qui attend les connexions sur un port (ex: 8888) |
| **Certificat CA** | Autorité de certification "maison" pour déchiffrer le HTTPS |

---

## 🛠️ Environnement de test

| Élément | Valeur |
|---------|--------|
| **Proxy** | Burp Suite Community (dans Mobexler) |
| **Listener** | `0.0.0.0:8888` (toutes interfaces) |
| **IP_HOTE** | `192.168.137.102` |
| **Client** | Genymotion (émulateur Android) |
| **Proxy client** | Manuel : `192.168.137.102:8888` |

---

## 🔧 Configuration de Burp Suite

### Proxy Listener

| Paramètre | Valeur |
|-----------|--------|
| **Interface** | `0.0.0.0:8888` |
| **Statut** | Running |
| **Port** | 8888 |

**Pourquoi `0.0.0.0` ?** Permet à l'émulateur Genymotion (sur le réseau Host-Only `192.168.137.x`) de se connecter à Burp.

### Configuration du proxy dans Genymotion

| Paramètre | Valeur |
|-----------|--------|
| **Proxy** | Manuel |
| **Nom d'hôte** | `192.168.137.102` |
| **Port** | `8888` |

---

## 📊 HTTP history vs Intercept (différence fondamentale)

| Mode | Rôle | Blocage ? | Modification ? | Utilisation |
|------|------|-----------|----------------|-------------|
| **HTTP history** | Journalisation | ❌ Non | ❌ Non | Observer le trafic normal |
| **Intercept** | Blocage + analyse | ✅ Oui | ✅ Oui | Tester/modifier des requêtes |


### Explication détaillée

| Mode | Ce qui se passe |
|------|-----------------|
| **HTTP history** | Burp reçoit la requête, l'enregistre dans l'historique, et la transmet **immédiatement** au serveur. L'utilisateur n'a pas d'interaction. |
| **Intercept** | Burp reçoit la requête, la **bloque**, et attend une action de l'analyste. L'utilisateur peut examiner, modifier, supprimer (Drop) ou laisser passer (Forward). |


---

## 🔐 HTTPS et certificat CA

### Pourquoi le navigateur bloque le HTTPS avec Burp ?

> HTTPS chiffre la communication. Burp agit comme un "homme du milieu" (MITM). Le navigateur ne fait pas confiance au certificat de Burp → erreur de sécurité.

### Solution : installer le certificat CA de Burp

| Sans certificat CA | Avec certificat CA installé |
|-------------------|------------------------------|
| Navigateur refuse la connexion | Navigateur accepte (CA connue) |
| Erreur `NET::ERR_CERT_AUTHORITY_INVALID` | Connexion établie |
| Burp ne voit rien (HTTPS) | Burp peut déchiffrer |


### Tableau récapitulatif des clés/certificats utilisés

| Segment | Chiffré avec | Certificat utilisé | Signé par |
|---------|--------------|-------------------|-----------|
| **Client → Burp** | Clé publique de Burp | Certificat BURP | CA Burp (installée dans émulateur) |
| **Burp → Serveur** | Clé publique du serveur | Certificat RÉEL du serveur | Autorité réelle (DigiCert, Let's Encrypt, etc.) |
| **Serveur → Burp** | Clé privée du serveur | Certificat RÉEL du serveur | Autorité réelle |
| **Burp → Client** | Clé privée de Burp | Certificat BURP | CA Burp |

### Rôle de la clé privée de Burp

| Action | Rôle de la clé privée Burp |
|--------|---------------------------|
| Déchiffrer requête du client | ✅ Oui (car client utilise clé publique Burp) |
| Rechiffrer réponse pour client | ✅ Oui |
| Communiquer avec le serveur | ❌ Non (utilise clé publique du serveur) |

### Ce qu'il faut absolument retenir

> **Burp ne signe JAMAIS les requêtes envoyées au serveur avec son propre certificat.** 
> 
> Pour communiquer avec le serveur, Burp utilise le **certificat RÉEL du serveur**. Le serveur voit une connexion HTTPS normale, comme si Burp n'existait pas.
>
> Le **certificat CA de Burp** sert uniquement à faire **accepter la connexion par le client Android**. Il n'est jamais présenté au serveur.


---

## 📸 Screenshots du laboratoire

| # | Description | Capture |
|---|-------------|---------|
| 1 | Proxy Listener Burp (0.0.0.0:8888) | <img width="1477" height="752" alt="Etape2_1" src="https://github.com/user-attachments/assets/14487e0d-1e70-48ae-8e27-a056a139c98a" /> |
| 2 | Configuration proxy dans Genymotion | <img width="536" height="849" alt="Etape4" src="https://github.com/user-attachments/assets/77789c30-804b-4988-a2c9-00de6eec3e8c" />|
| 3 | HTTP history avec requêtes capturées | <img width="1152" height="427" alt="Etape5" src="https://github.com/user-attachments/assets/91763ab0-f58a-41f4-9648-3064775ed37c" />|
| 4 | Analyse d'une requête (Raw/Headers/Params) | <img width="1157" height="755" alt="Etape6" src="https://github.com/user-attachments/assets/be5d3d50-6e0d-438e-889d-4b7a8e1d13ed" />|
| 5 | Interception activée (requête bloquée) | <img width="1181" height="513" alt="Etape7" src="https://github.com/user-attachments/assets/3ee56db5-d141-4e97-9444-b6797c657482" />|


---

## 📋 Observations du trafic HTTP

### Requête analysée (exemple)

| Élément | Valeur |
|---------|--------|
| **Méthode** | GET |
| **URL** | `/complete/search?hl=en&client=android&q=h` |
| **Host** | `www.google.com` |
| **User-Agent** | `Android/9` |
| **Status** | 200 OK |

### Éléments identifiés

| Type | Valeur |
|------|--------|
| Paramètres | `hl=en`, `client=android`, `q=h` |
| Cookies | Absents |
| Données sensibles | Aucune (trafic HTTP non chiffré) |

### Risques potentiels

- Trafic en clair (HTTP) → interception possible (MITM)
- User-Agent expose le modèle d'appareil

### Recommandations

- Utiliser HTTPS pour chiffrer les communications
- Minimiser les informations dans le User-Agent

---

## ✅ Checklist

### Début de séance

| Vérification | Statut |
|--------------|--------|
| Burp Suite lancé (projet temporaire) | ✅ |
| Proxy Listener actif (0.0.0.0:8888) | ✅ |
| IP_HOTE notée (192.168.137.102) | ✅ |
| Proxy configuré dans Genymotion | ✅ |
| Interception désactivée (off) | ✅ |

### Pendant la séance

| Vérification | Statut |
|--------------|--------|
| Requête HTTP capturée dans HTTP history | ✅ |
| Analyse d'une requête (Raw/Headers/Params) | ✅ |
| Interception activée et testée | ✅ |
| Compréhension du certificat CA | ✅ |

### Fin de séance

| Vérification | Statut |
|--------------|--------|
| Proxy désactivé dans Genymotion | ✅ |
| Certificat CA retiré (si installé) | ⬜ |
| Capture d'écran sauvegardées | ✅ |


---

## 🧠 Résumé des concepts clés

| Concept | Explication |
|---------|-------------|
| **Burp Suite** | Proxy d'interception (pas un sniffer comme Wireshark) |
| **HTTP history** | Enregistrement passif des requêtes |
| **Intercept** | Blocage actif pour analyse/modification |
| **Listener 0.0.0.0** | Écoute sur toutes les interfaces (pour que Genymotion puisse joindre Burp) |
| **Certificat CA** | Nécessaire pour déchiffrer le HTTPS |
| **MITM** | Man-In-The-Middle (Burp se place entre client et serveur) |

---

## 👤 Auteur

**El Hachimi Abdelhamid**  
Date : 2026-04-18  
Cours : Sécurité des applications mobiles

---
