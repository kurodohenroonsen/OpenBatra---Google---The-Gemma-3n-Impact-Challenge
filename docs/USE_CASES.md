# Analyse Fonctionnelle : Les Cas d'Utilisation (UML)

Ce document détaille les exigences fonctionnelles du projet OpenBatra à travers le prisme des Cas d'Utilisation (Use Cases) UML. Il décrit les interactions entre les utilisateurs (Acteurs) et le système pour atteindre des objectifs spécifiques.

---

## Acteurs du Système

Nos personas principaux ont été modélisés en deux acteurs distincts pour garantir que toutes les fonctionnalités, en particulier celles liées à l'accessibilité, sont prises en compte dès la conception.

| Acteur | Description | Illustration |
| :--- | :--- | :--- |
| **Utilisateur Consommateur** | Représente l'utilisateur général qui interagit avec le système via l'interface visuelle et/ou vocale. Son objectif est d'obtenir des informations claires rapidement. | <img src="../assets/actor-consumer.png" alt="generate image: a stylized icon of a person, curious and tech-savvy, interacting with a browser side panel on a computer screen. The style is 'Blue-Glow Infographic': futuristic, clean, 2D vector art. --s 250"> |
| **Utilisateur Accessibilité** | Représente un utilisateur ayant des besoins spécifiques (ex: malvoyant) qui dépend principalement de l'interface vocale (TTS/STT) pour toutes les interactions. | <img src="../assets/actor-accessibility.png" alt="generate image: a stylized icon of a person using a screen reader. Soundwaves emanate from the person's mouth towards a device, and from the device towards their ear, symbolizing a complete voice-only interaction. The style is 'Blue-Glow Infographic': futuristic, clean, 2D vector art. --s 250"> |

---

## Vue d'Ensemble des Cas d'Utilisation

Le diagramme suivant présente la vue d'ensemble des principales fonctionnalités offertes par OpenBatra et les acteurs qui les utilisent.

<p align="center">
  <img src="../assets/use-case-overview.png" alt="generate image: a comprehensive UML Use Case diagram showing two actors on the left ('Utilisateur Consommateur', 'Utilisateur Accessibilité') and the system 'OpenBatra' on the right. Inside the system box, three main use cases are visible: 'Identifier un Élément', 'Mémoriser une Préférence', and 'Créer un Pack d'Identité'. Lines connect the actors to the use cases they can perform. The style is 'Blue-Glow Infographic': clean, professional, and easy to read. --ar 16:9">
</p>

---

## Détail des Cas d'Utilisation

Chaque cas d'utilisation est détaillé dans un document séparé pour une analyse approfondie.

### 1. Identifier un Élément

C'est le cœur de l'expérience OpenBatra. L'utilisateur soumet une image et une question pour recevoir une analyse structurée et conversationnelle. Ce flux est conçu pour être le principal "wow factor" de l'application, démontrant la puissance de Gemma 3n en local.

- **Objectif :** Obtenir des informations fiables et contextuelles sur n'importe quel élément identifiable.
- **Acteurs :** Utilisateur Consommateur, Utilisateur Accessibilité.
- **Lien vers l'analyse détaillée :** **[Consulter les détails de UC1 : Identifier un Élément »](use-cases/UC1_Analyze_Element.md)**

<p align="center">
  <img src="../assets/use-case-1-visual.png" alt="generate image: a concept art illustration showing the 'Analyze Element' use case in action. A user is holding a phone, taking a picture of a complex service agreement. On the phone screen, the OpenBatra UI is visible, with the AI assistant highlighting a specific clause and providing a simplified explanation. The style is 'Blue-Glow Infographic'. --ar 16:9">
</p>

### 2. Mémoriser une Préférence

Cette fonctionnalité transforme OpenBatra d'un simple outil à un véritable compagnon intelligent. En mémorisant les préférences de l'utilisateur (allergies, régimes, centres d'intérêt), l'IA peut fournir des réponses proactives et personnalisées lors des analyses futures.

- **Objectif :** Personnaliser l'expérience en sauvegardant les préférences de l'utilisateur de manière privée et locale.
- **Acteurs :** Utilisateur Consommateur, Utilisateur Accessibilité.
- **Lien vers l'analyse détaillée :** **[Consulter les détails de UC2 : Mémoriser une Préférence »](use-cases/UC2_Memorize_Preference.md)**

<p align="center">
  <img src="../assets/use-case-2-visual.png" alt="generate image: a concept art illustration for the 'Memorize Preference' use case. The OpenBatra AI assistant is shown offering a 'save' icon to the user after identifying an allergen. In the background, a secure vault icon with the IndexedDB logo symbolizes local and private storage. The style is 'Blue-Glow Infographic'. --ar 16:9">
</p>

### 3. Créer un Pack d'Identité

Cette fonctionnalité démontre la capacité d'OpenBatra non seulement à comprendre, mais aussi à créer. L'utilisateur peut générer un "Pack d'Identité" numérique complet pour n'importe quel bien ou service, prêt à être partagé.

- **Objectif :** Créer un livrable numérique autonome (micro-site, JSON, images) pour un élément identifié.
- **Acteurs :** Utilisateur Consommateur (dans un rôle de créateur).
- **Lien vers l'analyse détaillée :** **[Consulter les détails de UC3 : Créer un Pack d'Identité »](use-cases/UC3_Create_Identity_Pack.md)**

<p align="center">
  <img src="../assets/use-case-3-visual.png" alt="generate image: a concept art illustration for the 'Create Identity Pack' use case. An artisan is shown holding a handmade vase. The OpenBatra AI assistant is generating a ZIP file icon, which expands to show a small website, a JSON file, and images, ready for export and sharing. The style is 'Blue-Glow Infographic'. --ar 16:9">
</p>