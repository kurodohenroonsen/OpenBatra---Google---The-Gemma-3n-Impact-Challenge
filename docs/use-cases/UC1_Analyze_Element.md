# UC1: Identifier un Élément

**[◀ Retour à la Vue d'Ensemble des Cas d'Utilisation](../USE_CASES.md)**

Ce document fournit une analyse détaillée du cas d'utilisation principal et le plus critique du projet OpenBatra.

---

## 1. Description et Objectif

Ce cas d'utilisation décrit le cœur de l'interaction. Un utilisateur soumet une image d'un élément (bien, service, document, etc.) et pose une question en langage naturel pour obtenir une analyse instantanée, privée et fonctionnant hors-ligne. Le système doit être entièrement pilotable à la voix pour garantir l'accessibilité.

<p align="center">
  <img src="../../assets/uc1-detail-banner.png" alt="generate image: a detailed illustration of the 'Analyze Element' use case. A user's hand holds a phone pointed at a bottle of medicine. On the phone screen, the OpenBatra side panel is open, showing a conversation with an AI. The AI has highlighted the 'Dosage' section on the bottle's label and is providing a clear, concise summary. The style is 'Blue-Glow Infographic': futuristic, clean, professional and clear. --ar 16:9">
</p>

## 2. Diagramme UML du Cas d'Utilisation

<p align="center">
  <img src="../../assets/uc1-diagram.png" alt="generate image: a clean and professional UML Use Case diagram for 'Identifier un Élément'. It shows two actors, 'Utilisateur Consommateur' and 'Utilisateur Accessibilité', connected to a central use case. This use case <<includes>> sub-tasks like 'Capturer une image', 'Poser une question vocale', 'Analyser (OCR + IA)', and 'Répondre vocalement'. The style is 'Blue-Glow Infographic'. --ar 16:9">
</p>

## 3. Informations Détaillées

| Champ | Description |
| :--- | :--- |
| **ID** | UC1 |
| **Nom** | Identifier un Élément |
| **Acteurs** | `Utilisateur Consommateur`, `Utilisateur Accessibilité` |
| **Déclencheur** | L'utilisateur active l'agent et soumet une image et/ou une question vocale. |
| **Préconditions** | L'extension OpenBatra est active dans le side panel. Les permissions pour le microphone et la caméra ont été accordées. |
| **Postconditions (Succès)** | Le système a fourni une réponse vocale et visuelle à la question de l'utilisateur. Le résultat de l'analyse est affiché dans l'UI. |
| **Postconditions (Échec)** | Le système a notifié l'utilisateur de l'échec (ex: image illisible) et est retourné à un état d'attente. |

## 4. Scénario Nominal (Happy Path)

Cette séquence décrit le flux d'interaction idéal, comme présenté dans la vidéo de démonstration.

1.  **Utilisateur :** Active l'agent et prend une photo de l'élément à analyser.
    - *API :* `navigator.mediaDevices.getUserMedia()`
2.  **Système (Agent Orion) :** Annonce vocalement la bonne réception de l'image. "Photo capturée. Quelle est votre question ?"
    - *API :* `SpeechSynthesis`
3.  **Utilisateur :** Pose sa question vocalement. "Quelles sont les conditions de garantie de ce service ?"
    - *API :* `SpeechRecognition`
4.  **Système (Agent Orion) :** Confirme la réception et le début de l'analyse. "Analyse en cours..."
    - *Accessibilité :* Un indicateur de chargement visuel est accompagné de ce feedback vocal pour ne pas laisser l'utilisateur dans le silence.
5.  **Système (Agent Lens/Socrates) :**
    a. Le texte est extrait de l'image via OCR.
    b. Le texte et la question sont envoyés au modèle Gemma 3n.
    c. Le modèle analyse et génère un objet JSON `AnalysisResult` structuré.
    - *IA :* C'est ici que le prompt `analyzeProduct` est exécuté.
6.  **Système (Agent Orion & UI) :** Le résultat est présenté à l'utilisateur de manière multimodale.
    - *UI :* Le `humanReadableAnswer` est affiché et le `boundingBox` est utilisé pour surligner la zone pertinente sur l'image.
    - *Voix :* Le `humanReadableAnswer` et le `contextSnippet` sont annoncés vocalement pour décrire le résultat et sa localisation.
    - *Exemple d'annonce :* "La garantie est de 2 ans. Je surligne cette mention pour vous dans le troisième paragraphe du document."

## 5. Flux Alternatifs et Gestion des Erreurs

### 5.1. Image Illisible

- **Déclencheur :** Le score de confiance de l'OCR est en dessous d'un seuil prédéfini.
- **Flux :**
    1. L'Agent Lens retourne un échec d'analyse.
    2. L'Agent Orion annonce : "Je n'arrive pas à lire correctement le texte sur cette image. Pourriez-vous en prendre une nouvelle, plus nette et avec un meilleur éclairage ?"
    3. Le système retourne à l'étape de capture de photo.

### 5.2. Erreur de Compréhension Vocale (Directive #3)

- **Déclencheur :** La transcription de la `SpeechRecognition` a un score de confiance faible ou ne correspond à aucune intention claire.
- **Flux :**
    1. L'Agent Orion initie un dialogue de clarification.
    2. **Orion :** "Pardon, je ne suis pas sûr d'avoir compris. Vouliez-vous parler de la 'garantie' ? Vous pouvez dire oui, non, ou 'corriger'."
    3. Si **"Oui"**, le flux nominal reprend avec le terme clarifié.
    4. Si **"Non"**, Orion demande : "Pouvez-vous reformuler votre question ?"
    5. Si **"Corriger"**, le système se remet en état d'écoute pour une nouvelle saisie de la question.

### 5.3. Aucune Information Pertinente Trouvée

- **Déclencheur :** Gemma 3n analyse l'image et la question mais ne trouve aucune information correspondante.
- **Flux :**
    1. L'Agent Socrates retourne un `AnalysisResult` avec une réponse indiquant l'échec de la recherche.
    2. **Orion :** "J'ai bien analysé l'élément, mais je n'ai trouvé aucune information concernant '[terme recherché]'. Souhaitez-vous que je vous fasse un résumé de ce que j'ai pu identifier ?"
    3. Le système propose des actions alternatives, comme une analyse générale.