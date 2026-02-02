# Module Comptes Rendus d'Intervention

## Objectif

Permettre aux techniciens de rediger des comptes rendus d'intervention directement dans Planitud, puis de les envoyer automatiquement au syndic concerne.

---

## Fonctionnalites

### 1. Creation d'un compte rendu

**Acces :** Depuis une petite intervention ou un chantier

**Champs du formulaire :**

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| intervention_id | FK | Oui | Lien vers l'intervention |
| date_intervention | date | Oui | Date de l'intervention |
| techniciens | multi-select | Oui | Techniciens presents |
| duree | number | Non | Duree en heures |
| travaux_realises | textarea | Oui | Description des travaux |
| observations | textarea | Non | Observations complementaires |
| suite_a_donner | textarea | Non | Actions a prevoir |
| photos | file[] | Non | Photos (max 10) |
| signature_client | signature | Non | Signature sur place |

### 2. Templates de compte rendu

Proposer des templates pre-remplis selon le type d'intervention :
- Recherche de fuite
- Refection etancheite
- Reparation ponctuelle
- Entretien annuel
- Reunion de chantier

### 3. Generation PDF

Le compte rendu genere un PDF avec :
- Logo Enerpur
- Informations du bien (adresse, syndic)
- Contenu du compte rendu
- Photos integrees
- Signature si presente

### 4. Envoi automatique

Une fois valide, le PDF est envoye par email au :
- Gestionnaire du syndic (principal)
- Contacts en copie (optionnel)

---

## Schema de donnees (Supabase)

```sql
CREATE TABLE comptes_rendus (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  intervention_id UUID REFERENCES interventions(id),
  date_intervention DATE NOT NULL,
  duree_heures DECIMAL(4,1),
  travaux_realises TEXT NOT NULL,
  observations TEXT,
  suite_a_donner TEXT,
  signature_base64 TEXT,
  statut VARCHAR(20) DEFAULT 'brouillon',
  envoye_le TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE comptes_rendus_techniciens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  compte_rendu_id UUID REFERENCES comptes_rendus(id),
  technicien_id UUID REFERENCES techniciens(id)
);

CREATE TABLE comptes_rendus_photos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  compte_rendu_id UUID REFERENCES comptes_rendus(id),
  url TEXT NOT NULL,
  legende TEXT,
  ordre INT DEFAULT 0
);
```

---

## Statuts

| Statut | Description |
|--------|-------------|
| brouillon | En cours de redaction |
| a_valider | Pret pour validation |
| valide | Valide, pret a envoyer |
| envoye | Envoye au syndic |

---

## Workflow utilisateur

```
1. Technicien termine intervention
2. Clique "Rediger CR" depuis l'intervention
3. Remplit le formulaire (peut sauvegarder en brouillon)
4. Ajoute photos
5. Valide le CR
6. Preview PDF
7. Clique "Envoyer au syndic"
8. Email envoye automatiquement
9. CR passe en statut "envoye"
```

---

## Integration existante

- Lier a la table `interventions` existante
- Utiliser la table `clients` pour les emails syndic
- Recuperer les infos du bien depuis `biens`

---

## Estimation

**3-5 jours de developpement**
- Jour 1-2 : CRUD + formulaire
- Jour 2-3 : Generation PDF
- Jour 4 : Envoi email
- Jour 5 : Tests et ajustements
