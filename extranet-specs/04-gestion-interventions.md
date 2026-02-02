# Module Gestion des Interventions

## Objectif

Gerer les interventions realisees dans le cadre d'une demande.

---

## Concepts

### Intervention
Une intervention represente une action concrete sur le terrain :
- RDV de visite (constat)
- Recherche de fuite (RDF)
- Reparation ponctuelle
- Chantier de refection
- Entretien annuel

Chaque intervention peut generer **plusieurs comptes-rendus**.

---

## Hierarchie

```
DEMANDE
  └── INTERVENTION
        └── COMPTES-RENDUS (1 ou plusieurs)
```

---

## Liste des interventions

Vue tableau sur la fiche demande :

| Colonne | Description |
|---------|-------------|
| # | Numero unique |
| Date d'intervention | Date de realisation |
| Titre | Description courte |
| Statut | Etat actuel |
| Responsable | Personne en charge |
| Compte rendu | Lien vers le PDF |
| Action | Lien "Voir" |

---

## Fiche intervention

### Informations principales

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| Demande | lien | Oui | Reference a la demande |
| Bien | lien (auto) | Oui | Herite de la demande |
| Date d'intervention | date | Oui | Date de realisation |
| Titre | texte | Oui | Ex: "RDV de visite", "RDF appartement 3B" |
| Statut | liste | Oui | Voir statuts ci-dessous |
| Responsable | liste | Non | Technicien en charge |
| Consigne | zone texte | Non | Instructions pour les techniciens |

### Section : Fichiers
Zone de depot de fichiers (photos, documents).
- Checkbox "Document prive" (non visible par le syndic)

### Section : Liste des comptes-rendus
Tableau des CR lies a cette intervention.

---

## Statuts d'intervention

| Statut | Description |
|--------|-------------|
| Programmee | Intervention planifiee |
| En cours | Intervention en cours |
| Terminee | Intervention terminee, CR a rediger |
| Compte-rendu envoye | CR envoye au syndic |
| Annulee | Intervention annulee |

---

## Types d'intervention (dans le titre)

Pas de champ "type" formel, mais convention de nommage :

| Prefixe titre | Description |
|---------------|-------------|
| RDV de visite | Visite de constat |
| RDF | Recherche de fuite |
| Reparation | Travaux ponctuels |
| Chantier | Travaux importants |
| Entretien | Contrat d'entretien |
| Rdv d'intervention | Intervention generique |

---

## Schema de donnees

```sql
CREATE TABLE interventions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  numero SERIAL,
  demande_id UUID REFERENCES demandes(id) NOT NULL,
  date_intervention DATE NOT NULL,
  titre VARCHAR(500) NOT NULL,
  statut VARCHAR(50) DEFAULT 'Programmee',
  responsable_id UUID REFERENCES utilisateurs(id),
  consigne TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE intervention_fichiers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  intervention_id UUID REFERENCES interventions(id) ON DELETE CASCADE,
  nom VARCHAR(255) NOT NULL,
  url TEXT NOT NULL,
  prive BOOLEAN DEFAULT false,
  uploaded_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_interventions_demande ON interventions(demande_id);
CREATE INDEX idx_interventions_date ON interventions(date_intervention);
```

---

## Actions

| Action | Description |
|--------|-------------|
| Ajouter une intervention | Depuis la fiche demande |
| Modifier l'intervention | Mise a jour des infos |
| Supprimer l'intervention | Confirmation requise |
| Ajouter un compte-rendu | Cree un CR lie |
| Retourner a la demande | Navigation |

---

## Workflow

```
1. Depuis une demande, cliquer "Ajouter une intervention"
2. Remplir : date, titre, responsable, consigne
3. Statut = "Programmee"
4. Jour J : intervention realisee
5. Statut = "Terminee"
6. Technicien redige le compte-rendu
7. CR envoye au syndic
8. Statut = "Compte-rendu envoye"
```

---

## Lien avec Planitud

Dans Planitud (outil de planification), les interventions correspondent a :
- **Petites interventions** : RDV visite, RDF, reparations
- **Chantiers** : Travaux importants avec equipe

L'Extranet et Planitud doivent partager les memes donnees d'intervention.
