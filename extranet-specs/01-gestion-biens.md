# Module Gestion des Biens

## Objectif

Permettre de creer, modifier et consulter les biens immobiliers (coproprietes) geres par Enerpur.

---

## Fonctionnalites

### 1. Liste des biens

Vue tableau avec :
- Adresse complete
- Nom copropriete (si renseigne)
- Gestionnaire (nom + syndic)
- Nombre d'interventions
- Derniere intervention

Filtres :
- Par syndic
- Par ville/code postal
- Recherche texte (adresse)

### 2. Fiche bien

Consultation et modification des informations d'un bien.

---

## Champs du formulaire

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| Adresse | texte | Oui | Numero et rue |
| Complement d'adresse | texte | Non | Batiment, escalier, etc. |
| Code postal | texte | Oui | 5 chiffres |
| Ville | texte | Oui | - |
| Gestionnaire | liste (email) | Oui | Recherche par email |
| Information | zone de texte | Non | Digicode, gardien, acces... |
| Travaux effectues par Enerpur | checkbox | Non | Historique travaux Enerpur |

---

## Section : Membres du Conseil Syndical

Possibilite d'ajouter plusieurs membres.

| Champ | Type | Description |
|-------|------|-------------|
| Prenom | texte | - |
| Nom | texte | - |
| Email | texte | - |
| Telephone | texte | - |
| President | checkbox | Indique le president du CS |

---

## Lien avec le Gestionnaire

### Comportement actuel (probleme)
- Selection par nom du gestionnaire
- Si la personne change de syndic, l'email devient obsolete

### Comportement cible (amelioration)
- Selection par **email** du gestionnaire
- Recherche autocomplete : taper l'email affiche nom + syndic
- Si l'email n'existe pas : bouton "Creer un nouveau gestionnaire"

---

## Schema de donnees

```sql
CREATE TABLE biens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  adresse VARCHAR(255) NOT NULL,
  complement_adresse VARCHAR(255),
  code_postal VARCHAR(10) NOT NULL,
  ville VARCHAR(100) NOT NULL,
  gestionnaire_id UUID REFERENCES gestionnaires(id),
  information TEXT,
  travaux_enerpur BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE membres_conseil_syndical (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  bien_id UUID REFERENCES biens(id) ON DELETE CASCADE,
  prenom VARCHAR(100),
  nom VARCHAR(100),
  email VARCHAR(255),
  telephone VARCHAR(20),
  est_president BOOLEAN DEFAULT false
);
```

---

## Historique du bien

Depuis la fiche bien, afficher :
- Liste de toutes les interventions (petites + chantiers)
- Ordonnees par date (plus recente en premier)
- Avec statut et type

Voir spec "Historique par bien" pour le detail.

---

## Actions

| Action | Description |
|--------|-------------|
| Creer un bien | Formulaire vide |
| Modifier un bien | Pre-rempli avec donnees existantes |
| Supprimer un bien | Confirmation requise, garde historique |
| Voir historique | Affiche toutes les interventions |
| Creer intervention | Depuis le bien, pre-remplit le lien |
