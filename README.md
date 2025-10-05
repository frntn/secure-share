![Alt](https://repobeats.axiom.co/api/embed/dfafc0b414ecb7e84caccfb71a24fd91ee3b4cbf.svg "Repobeats analytics image")

# 🔒 Secure-Share

**Secure-Share** est un outil web minimaliste et autonome pour le partage sécurisé de courts textes. Il fonctionne entièrement dans le navigateur (côté client), ne nécessite aucun serveur et ne stocke aucune donnée.

Le principe est simple : vous écrivez un message, le chiffrez avec un mot de passe, définissez une durée de validité, et générez un lien unique. Seule une personne possédant à la fois le lien et le mot de passe pourra déchiffrer le message pendant sa période de validité.

## 🎯 Objectifs

Cet outil est conçu pour des partages d'informations sensibles à faible enjeu où la commodité et l'absence de dépendance à un serveur sont prioritaires.

**Cas d'usage typiques :**
- Partager un mot de passe temporaire, un token d'API ou une clé de licence.
- Envoyer une note confidentielle qui doit s'autodétruire.
- Transmettre une information sans laisser de trace sur un serveur de messagerie ou un service tiers.

**Ce n'est PAS un outil conçu pour :**
- Le stockage sécurisé à long terme.
- La protection contre des adversaires étatiques ou des menaces persistantes avancées (APT).
- Des environnements où le poste client de l'utilisateur pourrait être compromis.

---

## 🛡️ Analyse de Sécurité

### ✅ Points Forts (Garanties de sécurité)

1.  **Chiffrement de Bout-en-Bout (Client-Side Only)**
    - Toute opération de chiffrement et de déchiffrement s'exécute localement dans le navigateur de l'utilisateur.
    - Le mot de passe et le contenu en clair ne quittent jamais le poste client.
    - **Garantie :** Confidentialité des données vis-à-vis d'intercepteurs réseau passifs.

2.  **Primitives Cryptographiques Robustes**
    - **Chiffrement :** AES-256-GCM, une norme de chiffrement authentifié qui garantit à la fois la confidentialité et l'intégrité des données.
    - **Dérivation de clé :** PBKDF2 avec 100 000 itérations, ce qui ralentit considérablement les attaques par force brute sur des mots de passe faibles.
    - **Intégrité des métadonnées :** HMAC-SHA-256 est utilisé pour signer la charge utile (incluant le texte chiffré et les timestamps), empêchant toute modification des données ou de la période de validité sans invalider la signature.

3.  **Absence de Serveur (Serverless) grâce aux Fragments d'URL**
    - L'application utilise les fragments d'URL (la partie après le `#`) pour stocker les données chiffrées.
    - **Garantie fondamentale :** Les fragments d'URL ne sont jamais envoyés au serveur web. Ils restent exclusivement dans le navigateur du client. Cela signifie qu'aucune donnée sensible n'est jamais stockée ou même visible par le serveur qui héberge l'application.
    - La surface d'attaque est ainsi drastiquement réduite, car il n'y a pas de base de données ou de backend à compromettre.

### ⚠️ Limitations et Vecteurs d'Attaque

1.  **Confiance en l'Environnement Client**
    - **Le maillon faible est le poste client.** La sécurité du système s'effondre si le navigateur ou le système d'exploitation de l'utilisateur (l'émetteur ou le destinataire) est compromis.
    - **Vecteurs d'attaque :**
        - **Malware/Keylogger :** Un logiciel malveillant peut capturer le mot de passe ou le message en clair lors de leur saisie.
        - **Extensions de navigateur malveillantes :** Une extension peut lire et exfiltrer le contenu de la page, y compris le mot de passe et le message déchiffré.
        - **Attaques XSS (Cross-Site Scripting) :** Bien que non applicable pour un fichier local, si ce code était hébergé sur un site vulnérable, une attaque XSS pourrait compromettre la session.

2.  **Dépendance à la Force du Mot de Passe**
    - La sécurité de l'ensemble repose sur la robustesse du mot de passe choisi par l'utilisateur. Un mot de passe faible ("12345", "password") reste vulnérable aux attaques par dictionnaire ou force brute, bien que ralenties par PBKDF2.

3.  **Manipulation de l'Horloge Système (Time Skewing)**
    - La validation de l'expiration (`iat` et `exp`) dépend de l'horloge du système du destinataire. Un utilisateur averti peut contourner la restriction d'expiration en modifiant manuellement l'heure de son ordinateur pour la faire correspondre à la fenêtre de validité du lien.
    - **Conclusion :** La date d'expiration est une sécurité de "convenance", pas une garantie cryptographique absolue contre un acteur déterminé.

5.  **Attaques par Ingénierie Sociale (Phishing)**
    - Un attaquant pourrait héberger une version modifiée de cette page sur un domaine similaire et tromper un utilisateur pour qu'il y entre son mot de passe et son message. La page malveillante pourrait alors exfiltrer les secrets. Il est crucial de s'assurer de l'intégrité du code source utilisé.

## � Améliorations de Sécurité

Suite à un audit de sécurité approfondi, plusieurs améliorations ont été apportées pour renforcer la sécurité de l'application :

1. **Sels Cryptographiques Aléatoires**
   - **Avant :** Utilisation de sels fixes pour la dérivation des clés.
   - **Après :** Chaque message utilise désormais des sels aléatoires cryptographiquement forts (16 octets), générés via `crypto.getRandomValues()`.
   - **Bénéfice :** Protection contre les attaques par table pré-calculée (rainbow tables) et garantie que deux messages avec le même mot de passe auront des clés complètement différentes.

2. **Paramètres de Sécurité Configurables**
   - **Avant :** Nombre fixe d'itérations PBKDF2 (100 000).
   - **Après :** L'utilisateur peut choisir entre différents niveaux de sécurité (100 000, 200 000, ou 500 000 itérations).
   - **Bénéfice :** Adaptation du niveau de sécurité aux besoins et augmentation significative de la résistance aux attaques par force brute.

3. **Validation de la Complexité des Mots de Passe**
   - **Avant :** Aucune vérification de la force du mot de passe.
   - **Après :** Indicateur visuel de force du mot de passe et suggestions pour l'améliorer (longueur, diversité de caractères).
   - **Bénéfice :** Incitation à choisir des mots de passe plus robustes et prévention contre les mots de passe trivialement devinables.

4. **Informations Temporelles Détaillées et Mécanisme de Consensus Temporel**
   - **Avant :** Messages génériques sur l'expiration des liens, basés uniquement sur l'horloge locale.
   - **Après :** 
     * Affichage détaillé des informations temporelles, incluant l'heure exacte d'expiration et le temps restant.
     * Mise en place d'un mécanisme de consensus temporel qui consulte plusieurs sources de temps externes et fiables.
     * Utilisation d'un algorithme de médiane pour déterminer l'heure la plus précise, résistant aux valeurs aberrantes.
     * Hiérarchie de fiabilité : consensus de sources multiples > horloge du serveur GitHub Pages > horloge locale.
     * Notification à l'utilisateur en cas de différence significative entre les horloges.
   - **Bénéfice :** Protection renforcée contre la manipulation délibérée de l'horloge et fiabilité accrue des mécanismes d'expiration, même si une source de temps est compromise ou manipulée.

### Limitations Restantes

Malgré ces améliorations, certaines limitations fondamentales demeurent inhérentes à ce type d'architecture :

1. **Absence d'Authentification des Utilisateurs**
   - Sans un serveur d'authentification, il n'existe pas de moyen de vérifier l'identité de l'expéditeur ou du destinataire.

2. **Impossibilité de Révoquer un Message**
   - Une fois un lien créé, il reste valide jusqu'à son expiration. Il n'existe aucun mécanisme pour révoquer un lien avant sa date d'expiration.

3. **Vulnérabilité aux Malwares et Keyloggers**
   - La sécurité dépend entièrement de l'intégrité du poste client. Un système compromis peut capturer les données en clair.

4. **Vulnérabilité Réduite à la Manipulation de l'Horloge**
   - Le système utilise désormais un mécanisme de consensus temporel qui consulte plusieurs sources de temps externes, rendant la manipulation beaucoup plus difficile.
   - Ce mécanisme de consensus utilise plusieurs services de temps et l'horloge du serveur GitHub Pages, validant son authenticité.
   - L'algorithme de médiane filtre les valeurs aberrantes, rendant difficile la compromission de l'heure de référence.
   - Une manipulation de l'horloge locale n'aura généralement pas d'effet tant que l'application peut se connecter à au moins certaines des sources de temps externes.
   - Cependant, dans un contexte entièrement hors ligne, l'horloge locale reste utilisée comme dernier recours.

### Mesures Compensatoires

Pour atténuer ces risques, considérez les pratiques suivantes :

1. **Utilisation d'appareils fiables et à jour**
   - Utilisez uniquement des appareils dont vous maîtrisez la sécurité et maintenez-les à jour.

2. **Double Canal pour le Mot de Passe**
   - Partagez le lien et le mot de passe via des canaux de communication différents.

3. **Durées d'Expiration Courtes**
   - Privilégiez les durées d'expiration les plus courtes possibles pour limiter la fenêtre d'opportunité d'une attaque.
