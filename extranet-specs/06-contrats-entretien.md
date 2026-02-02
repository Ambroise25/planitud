# Module Contrats d'Entretien

## Objectif

Gerer les contrats d'entretien annuels proposes aux syndics apres la realisation d'un chantier.

---

## Concepts

### Contrat d'entretien
Un contrat d'entretien est une offre de maintenance annuelle pour un bien, comprenant une ou plusieurs prestations a prix fixe.

### Caracteristiques
- Propose apres un chantier (idealement automatiquement avec le PV de reception)
- Envoye par email au syndic
- Reconduit automatiquement chaque annee
- Rappels avant la date d'intervention prevue

---

## Acces

- Depuis la **fiche bien** : onglet "Contrat d'entretien"
- Depuis le **menu principal** : "Contrats d'entretien" (liste globale)

---

## Liste des contrats

Vue tableau avec :

| Colonne | Description |
|---------|-------------|
| Titre | Objet du contrat |
| Etat | Proposition, Signe, Solde, Resilie |
| Bien | Adresse de la propriete |
| Numero | Reference du contrat |
| Date maintenance | Prochaine intervention |
| Debut | Date de debut du contrat |
| Fin | Date de fin du contrat |
| Moyen de paiement | Virement, Cheque... |
| TVA | Taux applique |
| Date de creation | - |
| Total | Montant TTC |
| Actions | Voir, Modifier, Supprimer |

Filtres par etat disponibles.

---

## Fiche contrat

### Champs du formulaire

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| Objet | texte | Oui | Ex: "Entretien des terrasses inaccessibles" |
| Details | zone texte | Non | Description des surfaces, zones concernees |
| Propriete | lien bien | Oui | Adresse du bien |
| N° de contrat | texte | Oui | Reference unique (ex: 45CBPE) |
| Date de l'intervention | date | Oui | Prochaine date d'intervention |
| Date de debut | date | Oui | Debut du contrat |
| Date de fin | date | Oui | Fin du contrat |
| Moyen de paiement | liste | Oui | Virement, Cheque, Prelevement |
| Etat | liste | Oui | Voir statuts ci-dessous |
| TVA | nombre | Oui | Taux (10, 20...) |

### Section : Prestations

Tableau des lignes de prestation :

| Champ | Type | Description |
|-------|------|-------------|
| Prestation | texte | Description de la prestation |
| Prix | nombre | Montant HT |

Bouton "Ajouter une prestation" pour ajouter des lignes.

**Total** calcule automatiquement.

---

## Etats du contrat

| Etat | Description | Couleur |
|------|-------------|---------|
| Proposition | Envoye au syndic, en attente | Jaune |
| Signe | Contrat accepte et actif | Vert |
| Solde | Facture payee | Bleu |
| Resilie | Contrat annule | Rouge |

---

## Schema de donnees

```sql
CREATE TABLE contrats_entretien (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  bien_id UUID REFERENCES biens(id) NOT NULL,
  numero VARCHAR(50) NOT NULL UNIQUE,
  objet VARCHAR(255) NOT NULL,
  details TEXT,
  date_intervention DATE NOT NULL,
  date_debut DATE NOT NULL,
  date_fin DATE NOT NULL,
  moyen_paiement VARCHAR(50) DEFAULT 'Virement',
  etat VARCHAR(50) DEFAULT 'Proposition',
  tva DECIMAL(4,2) DEFAULT 10.00,
  reconduction_auto BOOLEAN DEFAULT true,
  envoye_le TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE contrat_prestations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contrat_id UUID REFERENCES contrats_entretien(id) ON DELETE CASCADE,
  description VARCHAR(500) NOT NULL,
  prix_ht DECIMAL(10,2) NOT NULL,
  ordre INT DEFAULT 0
);

CREATE INDEX idx_contrats_bien ON contrats_entretien(bien_id);
CREATE INDEX idx_contrats_etat ON contrats_entretien(etat);
CREATE INDEX idx_contrats_date ON contrats_entretien(date_intervention);
```

---

## Rappels automatiques

| Delai | Action |
|-------|--------|
| 1 mois avant | Notification "Intervention a planifier" |
| 1 semaine avant | Notification "Intervention imminente" |
| Date depassee | Alerte "Intervention en retard" |

---

## Reconduction automatique

A la date de fin du contrat :
1. Si `reconduction_auto = true`
2. Creer une nouvelle periode : date_debut = ancienne date_fin + 1 jour
3. date_fin = date_debut + 1 an
4. Mettre a jour date_intervention
5. Envoyer notification interne

---

## Workflow

### Proposition initiale
```
1. Fin de chantier → PV de reception valide
2. Creation automatique d'une proposition de contrat
3. Etat = "Proposition"
4. Envoi par email au syndic
5. Syndic accepte → Etat = "Signe"
```

### Cycle annuel
```
1. Rappel 1 mois avant date intervention
2. Planification intervention d'entretien
3. Realisation intervention → Compte-rendu
4. Facturation → Etat = "Solde"
5. Reconduction automatique
6. Retour a l'etape 1
```

---

## Envoi automatique apres chantier

Declencheur : PV de reception valide

Email envoye au gestionnaire :
```
Objet: [Enerpur] Proposition de contrat d'entretien - {adresse}

Bonjour {nom_gestionnaire},

Suite a la realisation des travaux au {adresse}, nous vous proposons 
un contrat d'entretien annuel pour maintenir votre ouvrage en bon etat.

Prestations incluses :
- {liste_prestations}

Montant annuel : {total} € TTC

Veuillez trouver ci-joint notre proposition de contrat.

Cordialement,
L'equipe Enerpur
```

---

## Actions

| Action | Description |
|--------|-------------|
| Ajouter un contrat | Depuis la fiche bien |
| Modifier le contrat | Mise a jour des infos |
| Envoyer au syndic | Envoi par email |
| Supprimer | Confirmation requise |
