# Module Contrats d'Entretien

## Objectif

Gerer les contrats d'entretien annuels avec les syndics : dates, rappels, historique des interventions d'entretien.

---

## Fonctionnalites

### 1. Creation d'un contrat

**Champs du formulaire :**

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| bien_id | FK | Oui | Bien concerne |
| client_id | FK | Oui | Syndic/Client |
| type_contrat | select | Oui | Type d'entretien |
| date_debut | date | Oui | Debut du contrat |
| date_fin | date | Non | Fin (ou reconduit) |
| frequence | select | Oui | Annuel, Semestriel, Trimestriel |
| montant_ht | decimal | Non | Montant annuel |
| conditions | textarea | Non | Conditions particulieres |
| document_contrat | file | Non | PDF du contrat signe |

### 2. Types de contrats

| Type | Description |
|------|-------------|
| Etancheite terrasse | Entretien annuel toiture-terrasse |
| Etancheite parking | Entretien parking souterrain |
| Couverture | Controle annuel toiture |
| Multi-sites | Contrat global plusieurs biens |

### 3. Rappels automatiques

Notifications avant la date d'entretien :
- 1 mois avant : "Entretien a planifier"
- 1 semaine avant (si non planifie) : "Entretien urgent a planifier"
- Date depassee : "Entretien en retard"

### 4. Historique des entretiens

Pour chaque contrat, voir la liste des interventions d'entretien realisees avec :
- Date
- Technicien
- Compte rendu
- Prochaine date

---

## Schema de donnees (Supabase)

```sql
CREATE TABLE contrats_entretien (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  bien_id UUID REFERENCES biens(id),
  client_id UUID REFERENCES clients(id),
  type_contrat VARCHAR(50) NOT NULL,
  date_debut DATE NOT NULL,
  date_fin DATE,
  frequence VARCHAR(20) DEFAULT 'annuel',
  montant_ht DECIMAL(10,2),
  conditions TEXT,
  document_url TEXT,
  actif BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE entretiens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contrat_id UUID REFERENCES contrats_entretien(id),
  date_prevue DATE NOT NULL,
  date_realisee DATE,
  intervention_id UUID REFERENCES interventions(id),
  statut VARCHAR(20) DEFAULT 'a_planifier',
  notes TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Statuts entretien

| Statut | Description |
|--------|-------------|
| a_planifier | Date prevue, pas encore planifie |
| planifie | RDV pris |
| realise | Intervention effectuee |
| reporte | Reporte a une autre date |

---

## Vue tableau de bord

Ajouter sur le dashboard Planitud :
- Widget "Entretiens a planifier ce mois"
- Widget "Entretiens en retard"

---

## Workflow utilisateur

```
1. Admin cree un contrat d'entretien
2. Systeme genere automatiquement les dates d'entretien selon frequence
3. 1 mois avant : notification "a planifier"
4. Utilisateur cree une intervention liee
5. Apres intervention : entretien passe en "realise"
6. Prochaine date generee automatiquement
```

---

## Integration existante

- Lier aux tables `biens` et `clients` existantes
- Creer intervention depuis un entretien
- Lier compte rendu a l'entretien

---

## Estimation

**3-4 jours de developpement**
- Jour 1 : CRUD contrats
- Jour 2 : Gestion des entretiens + calcul dates
- Jour 3 : Notifications/rappels
- Jour 4 : Widgets dashboard + tests
