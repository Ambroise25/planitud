# Module Journal d'Activites

## Objectif

Tracer automatiquement toutes les modifications apportees aux demandes et interventions pour assurer la tracabilite.

---

## Fonctionnement

Chaque action significative est enregistree avec :
- Date et heure
- Utilisateur (email)
- Action effectuee

---

## Affichage

Section "Activites" sur la fiche demande, ordonnee par date decroissante :

```
● 30/01/2026 15:14 - ambroise@enerpur.fr
  a modifie une intervention

● 30/01/2026 15:10 - ambroise@enerpur.fr
  a cree une intervention

● 29/01/2026 16:08 - ambroise@enerpur.fr
  a modifie un compte-rendu

● 29/01/2026 16:08 - ambroise@enerpur.fr
  a cree un compte-rendu
```

---

## Actions tracees

### Sur les demandes

| Action | Message |
|--------|---------|
| Creation | "a cree la demande" |
| Modification | "a modifie la demande" |
| Changement statut | "a change le statut en [nouveau statut]" |
| Suppression | "a supprime la demande" |

### Sur les interventions

| Action | Message |
|--------|---------|
| Creation | "a cree une intervention" |
| Modification | "a modifie une intervention" |
| Ajout fichier | "a ajoute un fichier a l'intervention" |
| Suppression | "a supprime une intervention" |

### Sur les comptes-rendus

| Action | Message |
|--------|---------|
| Creation | "a cree un compte-rendu" |
| Modification | "a modifie un compte-rendu" |
| Envoi | "a envoye le compte-rendu au syndic" |
| Suppression | "a supprime un compte-rendu" |

---

## Schema de donnees

```sql
CREATE TABLE activites (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  demande_id UUID REFERENCES demandes(id) ON DELETE CASCADE,
  intervention_id UUID REFERENCES interventions(id) ON DELETE SET NULL,
  compte_rendu_id UUID REFERENCES comptes_rendus(id) ON DELETE SET NULL,
  utilisateur_email VARCHAR(255) NOT NULL,
  action VARCHAR(100) NOT NULL,
  details TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_activites_demande ON activites(demande_id);
CREATE INDEX idx_activites_date ON activites(created_at DESC);
```

---

## Implementation technique

### Option 1 : Triggers SQL (recommande)

```sql
CREATE OR REPLACE FUNCTION log_demande_changes()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    INSERT INTO activites (demande_id, utilisateur_email, action)
    VALUES (NEW.id, current_setting('app.current_user'), 'a cree la demande');
  ELSIF TG_OP = 'UPDATE' THEN
    INSERT INTO activites (demande_id, utilisateur_email, action)
    VALUES (NEW.id, current_setting('app.current_user'), 'a modifie la demande');
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER demande_audit
AFTER INSERT OR UPDATE ON demandes
FOR EACH ROW EXECUTE FUNCTION log_demande_changes();
```

### Option 2 : Code applicatif

Ajouter un appel a `logActivity()` apres chaque operation CRUD.

---

## Retention

- Conserver les activites **indefiniment** (tracabilite legale)
- Optionnel : archiver les activites > 2 ans dans une table separee

---

## Acces

- **Visible par** : Utilisateurs Enerpur (Admin, Collaborateur)
- **Non visible par** : Clients (gestionnaires syndic)
