Parfait, Kurodo. On continue de remplir notre arborescence.

Voici le contenu complet pour le fichier docs/architecture/Data_Models.md, prêt à être copié.

Generated markdown
# Architecture : Modélisation des Données

**[◀ Retour à l'Analyse d'Architecture Principale](../SEQUENCE_DIAGRAM.md)**

Ce document fournit la définition formelle et détaillée de nos principaux modèles de données. Avoir des contrats de données (schemas) clairs et bien définis est essentiel pour garantir la cohérence des échanges entre les [composants du système](Components.md) et la fiabilité de l'application.

Nous utilisons le standard **JSON Schema** pour décrire nos structures de données.

---

## 1. Schéma `AnalysisResult`

Cet objet est le livrable le plus important de notre système. Il est produit par l'`AI_Service` (Agent Socrates) après une analyse multimodale et est consommé par l'`AppController` pour orchestrer la réponse. Il contient toutes les informations nécessaires pour que l'UI et le `VoiceOrchestrator` puissent présenter un résultat complet à l'utilisateur.

-   **Producteur :** `AI_Service`
-   **Consommateurs :** `AppController`, `UI`, `VoiceOrchestrator`

<p align="center">
  <img src="../../assets/data-model-analysis.png" alt="generate image: a visual representation of a JSON data structure for 'AnalysisResult'. It shows key fields like 'humanReadableAnswer', 'foundTerm', 'confidenceScore', and a nested 'boundingBox' object, all clearly laid out as if on a blueprint. The style is 'Blue-Glow Infographic': clean, professional, and technical. --ar 16:9">
</p>

### Définition JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "AnalysisResult",
  "description": "Objet de données structuré retourné par l'AI_Service après l'analyse d'un produit. C'est le contrat de données principal produit par Gemma 3n.",
  "type": "object",
  "properties": {
    "humanReadableAnswer": {
      "description": "La réponse directe, naturelle et conversationnelle à la question de l'utilisateur. Conçue pour être lue à voix haute.",
      "type": "string",
      "examples": ["Oui, des fruits à coque sont mentionnés.", "Non, je ne vois aucune mention de gluten."]
    },
    "foundTerm": {
      "description": "Le terme exact ou l'entité textuelle qui a été identifié sur l'image et qui justifie la réponse.",
      "type": "string",
      "examples": ["traces de fruits à coque", "sans gluten"]
    },
    "contextSnippet": {
      "description": "Un court extrait du texte original (OCR) montrant le terme trouvé dans son contexte pour une meilleure compréhension.",
      "type": "string",
      "examples": ["...farine de blé, sucre, beurre, traces de fruits à coque, sel..."]
    },
    "confidenceScore": {
      "description": "Le score de confiance de l'IA pour cette détection spécifique, normalisé entre 0.0 (incertain) et 1.0 (très certain).",
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "boundingBox": {
      "description": "Un objet décrivant le rectangle qui entoure le 'foundTerm' sur l'image. Les coordonnées sont normalisées (de 0.0 à 1.0) pour être indépendantes de la résolution de l'image.",
      "type": "object",
      "properties": {
        "x": { "type": "number", "description": "Position X (horizontale) du coin supérieur gauche." },
        "y": { "type": "number", "description": "Position Y (verticale) du coin supérieur gauche." },
        "width": { "type": "number", "description": "Largeur du rectangle." },
        "height": { "type": "number", "description": "Hauteur du rectangle." }
      },
      "required": ["x", "y", "width", "height"]
    }
  },
  "required": [
    "humanReadableAnswer",
    "foundTerm",
    "confidenceScore",
    "boundingBox"
  ]
}

2. Schéma UserPreference

Cet objet représente une préférence unique de l'utilisateur. Il est conçu pour être stocké efficacement dans une base de données locale IndexedDB par le DataStore. Une collection de ces objets constitue le profil de personnalisation de l'utilisateur.

Producteur : AppController (suite à une interaction avec l'utilisateur)

Gestionnaire : DataStore

Consommateur : AI_Service (pour personnaliser les futures analyses)

<p align="center">
<img src="../../assets/data-model-preference.png" alt="generate image: a visual representation of a JSON data structure for 'UserPreference'. It shows key fields like 'preferenceId', 'preferenceType', and 'value', arranged neatly in a secure-looking data container icon, symbolizing storage in IndexedDB. The style is 'Blue-Glow Infographic': clean, professional, and technical. --ar 16:9">
</p>

Définition JSON Schema
Generated json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "UserPreference",
  "description": "Objet de données représentant une préférence utilisateur unique à stocker dans la base de données locale (IndexedDB).",
  "type": "object",
  "properties": {
    "preferenceId": {
      "description": "Identifiant unique de la préférence, généré côté client (ex: UUID).",
      "type": "string",
      "format": "uuid"
    },
    "preferenceType": {
      "description": "Le type de préférence, pour permettre le regroupement et un traitement différencié.",
      "type": "string",
      "enum": ["allergy", "diet", "intolerance", "interest", "language", "other"]
    },
    "value": {
      "description": "La valeur de la préférence, normalisée en minuscules pour faciliter les recherches (ex: 'nuts', 'gluten-free', 'lactose', 'fr').",
      "type": "string"
    },
    "createdAt": {
      "description": "Timestamp ISO 8601 de la création de la préférence.",
      "type": "string",
      "format": "date-time"
    },
    "source": {
      "description": "Comment cette préférence a été ajoutée : soit proactivement par l'IA, soit manuellement par l'utilisateur.",
      "type": "string",
      "enum": ["proactive_ia", "manual_user"],
      "default": "proactive_ia"
    }
  },
  "required": [
    "preferenceId",
    "preferenceType",
    "value",
    "createdAt"
  ]
}
