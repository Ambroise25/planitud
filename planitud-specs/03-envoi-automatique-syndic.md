# Envoi Automatique aux Syndics

## Objectif

Envoyer automatiquement les documents (comptes rendus, devis, factures) aux gestionnaires de syndic par email.

---

## Fonctionnalites

### 1. Configuration email

**Service recommande :** Resend (simple, pas cher, bonne delivrabilite)
Alternative : SendGrid, Postmark

**Configuration Supabase :**
```
RESEND_API_KEY=re_xxxx
EMAIL_FROM=rapports@enerpur.fr
EMAIL_FROM_NAME=Enerpur Etancheite
```

### 2. Types d'envoi

| Type | Declencheur | Destinataires |
|------|-------------|---------------|
| Compte rendu | Validation CR | Gestionnaire syndic |
| Rapport entretien | Fin d'entretien | Gestionnaire + CS |
| Devis | Validation devis | Gestionnaire |
| Alerte | Probleme detecte | Gestionnaire |

### 3. Template email

```html
Objet: [Enerpur] Compte rendu intervention - {adresse_bien}

Bonjour {nom_gestionnaire},

Veuillez trouver ci-joint le compte rendu de notre intervention 
du {date} concernant le bien situe au :

{adresse_complete}

Travaux realises : {resume_travaux}

Cordialement,
L'equipe Enerpur

---
Enerpur Etancheite
Tel: 01 XX XX XX XX
www.enerpur.fr
```

### 4. Pieces jointes

- PDF du compte rendu (obligatoire)
- Photos en pieces jointes (optionnel, selon preference)

### 5. Suivi des envois

| Champ | Description |
|-------|-------------|
| email_id | ID unique de l'envoi |
| destinataires | Liste des emails |
| objet | Sujet de l'email |
| envoye_le | Timestamp |
| statut | envoye, delivre, erreur |
| document_type | Type de document |
| document_id | Reference au document |

---

## Schema de donnees (Supabase)

```sql
CREATE TABLE emails_envoyes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  destinataires TEXT[] NOT NULL,
  cc TEXT[],
  objet VARCHAR(255) NOT NULL,
  corps TEXT NOT NULL,
  pieces_jointes TEXT[],
  document_type VARCHAR(50),
  document_id UUID,
  statut VARCHAR(20) DEFAULT 'envoye',
  envoye_le TIMESTAMP DEFAULT NOW(),
  erreur TEXT
);
```

---

## Edge Function Supabase

```typescript
// supabase/functions/send-email/index.ts

import { Resend } from 'resend';

const resend = new Resend(Deno.env.get('RESEND_API_KEY'));

Deno.serve(async (req) => {
  const { to, cc, subject, html, attachments } = await req.json();

  const { data, error } = await resend.emails.send({
    from: 'Enerpur <rapports@enerpur.fr>',
    to,
    cc,
    subject,
    html,
    attachments
  });

  if (error) {
    return new Response(JSON.stringify({ error }), { status: 400 });
  }

  return new Response(JSON.stringify({ success: true, id: data.id }));
});
```

---

## Workflow utilisateur

```
1. Utilisateur valide un compte rendu
2. Clique "Envoyer au syndic"
3. Preview de l'email (destinataires, objet, PJ)
4. Confirme l'envoi
5. Email envoye via Resend
6. Statut mis a jour dans la base
7. Historique visible sur l'intervention
```

---

## Configuration domaine

Pour une bonne delivrabilite :
1. Configurer domaine enerpur.fr dans Resend
2. Ajouter enregistrements DNS (SPF, DKIM, DMARC)
3. Utiliser email @enerpur.fr comme expediteur

---

## Estimation

**2-3 jours de developpement**
- Jour 1 : Setup Resend + Edge Function
- Jour 2 : Templates email + generation PDF
- Jour 3 : UI d'envoi + historique
