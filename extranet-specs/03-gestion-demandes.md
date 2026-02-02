# Module Gestion des Demandes

## Objectif

Gerer les demandes d'intervention recues des syndics (emails, ordres de service).

---

## Concepts

### Demande
Une demande represente une sollicitation du syndic pour un bien donne. Elle peut provenir :
- D'un email recu
- D'un appel telephonique
- D'un ordre de service

Une demande peut generer **plusieurs interventions** (RDV visite → RDF → Chantier).

---

## Hierarchie

```
BIEN
  └── DEMANDE (1 ou plusieurs)
        └── INTERVENTIONS (1 ou plusieurs)
              └── COMPTES-RENDUS (1 ou plusieurs)
```

---

## Liste des demandes

Vue tableau sur la fiche bien :

| Colonne | Description |
|---------|-------------|
| # | Numero unique |
| Date de creation | Date de saisie dans le systeme |
| Date de la demande | Date de reception (email/appel) |
| Statut | Etat actuel de la demande |
| Metier | Etancheite, Plomberie, Couverture, Autre |
| Objet | Resume de la demande |
| Action | Lien "Voir" |

---

## Fiche demande

### Informations principales

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| Bien | lien | Oui | Reference au bien concerne |
| Date de creation | date auto | Oui | Date de saisie |
| Date de la demande | date | Oui | Date de reception |
| Objet | texte | Oui | Resume court |
| Metier | liste | Oui | Etancheite, Plomberie, Couverture, Autre |
| Statut | liste | Oui | Voir statuts ci-dessous |
| Responsable | liste | Non | Personne en charge |
| Commentaire | zone texte | Non | Notes internes |
| Details et informations | zone texte | Non | Infos complementaires (acces, contacts...) |

### Section : Liste des interventions

Tableau des interventions liees a cette demande.

### Section : Activites

Journal automatique des modifications (voir spec 05).

---

## Statuts de demande

| Statut | Description |
|--------|-------------|
| Nouvelle | Demande recue, pas encore traitee |
| Rendez-vous programme | RDV visite planifie |
| En cours | Intervention(s) en cours |
| En attente OS | Attente ordre de service du syndic |
| Compte-rendu envoye | Dernier CR envoye au syndic |
| Cloturee | Demande terminee |
| Annulee | Demande annulee |

---

## Schema de donnees

```sql
CREATE TABLE demandes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  numero SERIAL,
  bien_id UUID REFERENCES biens(id) NOT NULL,
  date_creation TIMESTAMP DEFAULT NOW(),
  date_demande DATE NOT NULL,
  objet VARCHAR(500) NOT NULL,
  metier VARCHAR(50) DEFAULT 'Etancheite',
  statut VARCHAR(50) DEFAULT 'Nouvelle',
  responsable_id UUID REFERENCES utilisateurs(id),
  commentaire TEXT,
  details TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_demandes_bien ON demandes(bien_id);
CREATE INDEX idx_demandes_statut ON demandes(statut);
```

---

## Actions

| Action | Description |
|--------|-------------|
| Ajouter une demande | Depuis la fiche bien |
| Modifier la demande | Mise a jour des infos |
| Supprimer la demande | Confirmation requise |
| Ajouter une intervention | Cree une intervention liee |
| Retourner au bien | Navigation |

---

## Workflow

```
1. Email syndic arrive (ou appel)
2. Creation demande (manuelle ou via Email Parser)
3. Statut = "Nouvelle"
4. Planification RDV visite
5. Statut = "Rendez-vous programme"
6. Intervention realisee → CR envoye
7. Statut = "Compte-rendu envoye"
8. Si suite necessaire : nouvelle intervention
9. Sinon : Statut = "Cloturee"
```
