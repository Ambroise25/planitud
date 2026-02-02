# Module Gestion des Gestionnaires (Clients)

## Objectif

Gerer les gestionnaires de syndic qui sont les interlocuteurs principaux pour chaque bien.

---

## Concepts

### Syndic (Societe)
La societe de gestion immobiliere : Foncia, Nexity, A2BCD, etc.

### Gestionnaire (Client)
La personne physique qui travaille pour le syndic et gere un portefeuille de biens.

---

## Fonctionnalites

### 1. Liste des gestionnaires

Vue tableau avec :
- Nom complet
- Syndic (societe)
- Email
- Telephone
- Nombre de biens geres
- Statut (actif/inactif)

Filtres :
- Par syndic
- Par statut
- Recherche texte

### 2. Fiche gestionnaire

Consultation et modification des informations.

---

## Champs du formulaire

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| Prenom | texte | Oui | - |
| Nom de famille | texte | Oui | - |
| Societe/Syndic | liste ou texte | Oui | Nom du syndic |
| **Adresse email** | texte | Oui | **Cle unique d'identification** |
| Telephone | texte | Oui | - |
| Adresse | texte | Oui | Adresse du bureau |
| Complement d'adresse | texte | Non | - |
| Code postal | texte | Oui | - |
| Ville | texte | Oui | - |
| Email assistant(e) | texte | Non | Email secondaire |
| Active | checkbox | - | Compte actif ou non |
| Type utilisateur | liste | Oui | Admin / Client / Collaborateur |

---

## Identification par email

### Probleme actuel
Un gestionnaire est identifie par son nom. Si "Marie Dupont" quitte Foncia pour Nexity :
- Son email change (marie.dupont@foncia.fr → m.dupont@nexity.fr)
- Les biens lies a "Marie Dupont" pointent vers le mauvais email

### Solution
- L'email devient la **cle unique** d'identification
- Changer de syndic = creer un nouveau gestionnaire avec le nouvel email
- L'ancien gestionnaire reste dans l'historique (desactive)

---

## Schema de donnees

```sql
CREATE TABLE syndics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  nom VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE gestionnaires (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  prenom VARCHAR(100) NOT NULL,
  nom VARCHAR(100) NOT NULL,
  syndic_id UUID REFERENCES syndics(id),
  email VARCHAR(255) NOT NULL UNIQUE,
  telephone VARCHAR(20),
  adresse VARCHAR(255),
  complement_adresse VARCHAR(255),
  code_postal VARCHAR(10),
  ville VARCHAR(100),
  email_assistant VARCHAR(255),
  actif BOOLEAN DEFAULT true,
  type_utilisateur VARCHAR(20) DEFAULT 'Client',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_gestionnaires_email ON gestionnaires(email);
CREATE INDEX idx_gestionnaires_syndic ON gestionnaires(syndic_id);
```

---

## Types d'utilisateurs

| Type | Description | Acces |
|------|-------------|-------|
| Admin | Equipe Enerpur | Tout |
| Client | Gestionnaire syndic | Ses biens uniquement |
| Collaborateur | Technicien Enerpur | Interventions assignees |

---

## Workflow de creation

```
1. Recherche par email dans le formulaire "Bien"
2. Email trouve → Selection automatique
3. Email non trouve → Bouton "Creer nouveau gestionnaire"
4. Formulaire de creation rapide :
   - Email (pre-rempli)
   - Prenom, Nom
   - Syndic (liste ou nouveau)
   - Telephone
5. Gestionnaire cree et selectionne automatiquement
```

---

## Actions

| Action | Description |
|--------|-------------|
| Creer | Nouveau gestionnaire |
| Modifier | Mise a jour des infos |
| Desactiver | Passe actif = false (ne supprime pas) |
| Voir biens | Liste des biens geres par ce gestionnaire |
