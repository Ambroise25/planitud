# Module Doublons et Relances

## Objectif

Detecter automatiquement les emails en doublon et les relances des syndics pour eviter les demandes en double et suivre les relances.

---

## Deux cas distincts

### 1. Doublon = email identique
Meme email recu deux fois (envoi multiple, copie a plusieurs personnes).

**Action :** Ignorer l'email, ne pas creer de nouvelle demande.

### 2. Relance = nouveau message, meme sujet
Le syndic renvoie un email pour demander ou en est sa demande.

**Action :** Ajouter une note sur la demande existante + notifier l'equipe.

---

## Detection des doublons

### Criteres (TOUS doivent correspondre)

| Critere | Comparaison |
|---------|-------------|
| Expediteur | Email identique |
| Objet | Identique |
| Bien | Meme adresse |
| Contenu | Identique ou quasi-identique |

### Logique

```
SI expediteur identique
ET objet identique
ET contenu identique (ou >95% similaire)
ET delai < 48h
ALORS → Doublon detecte
```

### Action
- Email ignore (pas de creation de demande)
- Log : "Doublon detecte - email ignore"
- Notification optionnelle : "Email en doublon detecte et ignore pour {adresse_bien}"

---

## Detection des relances

### Criteres

| Critere | Comparaison |
|---------|-------------|
| Expediteur | Meme email ou meme syndic |
| Bien | Meme adresse |
| Contenu | Different (le syndic demande des nouvelles) |
| Demande existante | Statut non cloturee |

### Logique

```
SI expediteur meme syndic
ET bien identifie = bien d'une demande existante non cloturee
ET contenu different du premier email
ALORS → Relance detectee
```

### Indices de relance dans le contenu
Le LLM peut detecter des formulations typiques :
- "Suite a notre demande..."
- "Nous n'avons pas eu de retour..."
- "Merci de nous faire un point..."
- "Ou en est l'intervention..."
- "Relance" dans l'objet
- "RE:" ou "FW:" dans l'objet

### Action
1. **Ne pas creer de nouvelle demande**
2. **Ajouter une note** sur la demande existante :
   - Date de la relance
   - Contenu resume
   - Lien vers l'email original
3. **Incrementer le compteur** de relances sur la demande
4. **Notifier l'equipe** : "Relance #X recue pour demande #XXXX - {adresse_bien}"

---

## Schema de donnees

### Compteur de relances sur la demande

```sql
ALTER TABLE demandes ADD COLUMN nb_relances INT DEFAULT 0;
ALTER TABLE demandes ADD COLUMN derniere_relance TIMESTAMP;
```

### Table des relances

```sql
CREATE TABLE relances (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  demande_id UUID REFERENCES demandes(id) NOT NULL,
  email_id VARCHAR(255),
  date_reception TIMESTAMP NOT NULL,
  expediteur VARCHAR(255) NOT NULL,
  resume TEXT,
  email_brut TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_relances_demande ON relances(demande_id);
```

### Table des doublons ignores (log)

```sql
CREATE TABLE doublons_ignores (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email_id VARCHAR(255),
  expediteur VARCHAR(255) NOT NULL,
  objet VARCHAR(500),
  raison VARCHAR(100) DEFAULT 'doublon_exact',
  demande_existante_id UUID REFERENCES demandes(id),
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Affichage sur la fiche demande

### Indicateur de relance

Si `nb_relances > 0`, afficher un badge :
- **1 relance** : Badge orange "1 relance"
- **2+ relances** : Badge rouge "X relances"

### Section relances

Liste des relances recues :

| Date | Expediteur | Resume |
|------|-----------|--------|
| 15/02/2026 | martin@foncia.fr | Demande de nouvelles sur l'intervention |
| 22/02/2026 | martin@foncia.fr | Relance urgente |

---

## Workflow Email Parser

```
1. Email recu
2. Email Parser extrait les infos (LLM)
3. Recherche demande existante :
   a. Meme bien + meme syndic + demande non cloturee ?
   b. OUI → Verifier si doublon ou relance
      - Contenu identique → DOUBLON → Ignorer
      - Contenu different → RELANCE → Ajouter note + notifier
   c. NON → Nouvelle demande → Creer normalement
```

---

## Role du LLM

Le LLM analyse l'email et retourne un champ supplementaire :

```json
{
  "type_email": "nouvelle_demande | relance | doublon",
  "relance_indices": [
    "Objet contient RE:",
    "Formulation : suite a notre demande"
  ],
  "confiance_detection": 0.92
}
```

---

## Notifications

| Evenement | Destinataires | Message |
|-----------|--------------|---------|
| Doublon ignore | Log uniquement | "Doublon detecte et ignore" |
| 1ere relance | Responsable demande | "Relance recue pour demande #XXXX" |
| 2eme+ relance | Responsable + Admin | "Relance #X recue - demande #XXXX" |
