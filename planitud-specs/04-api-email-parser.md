# API pour Email Parser

## Objectif

Permettre au service Email Parser (heberge sur Replit) de creer des demandes d'intervention dans Planitud.

---

## Architecture

```
┌─────────────────┐         ┌─────────────────┐
│  Email Parser   │  POST   │    Planitud     │
│    (Replit)     │ ──────> │   (Supabase)    │
│                 │  JSON   │                 │
└─────────────────┘         └─────────────────┘
                                    │
                                    ▼
                            ┌───────────────┐
                            │   Database    │
                            │   (Postgres)  │
                            └───────────────┘
```

---

## Endpoints

### 1. Creer une demande

```
POST /functions/v1/create-demande
```

**Headers :**
```
Authorization: Bearer <SUPABASE_SERVICE_KEY>
Content-Type: application/json
```

**Body :**
```json
{
  "source": {
    "type": "email",
    "email_id": "msg-123456",
    "date_reception": "2026-02-02T08:30:00Z",
    "expediteur": "gestionnaire@foncia.fr"
  },
  "bien": {
    "adresse": "12 rue de la Paix",
    "code_postal": "75002",
    "ville": "Paris",
    "nom_copropriete": "Residence Les Lilas"
  },
  "demande": {
    "objet": "Infiltration terrasse",
    "detail": "Fuite importante au niveau de la terrasse du 5eme",
    "metier": "Etancheite",
    "urgence": "Urgent",
    "ref_syndic": "OS-2026-12345"
  },
  "contacts": [
    {
      "nom": "Mme Martin",
      "telephone": "0698765432",
      "email": "martin@foncia.fr",
      "qualite": "gestionnaire"
    }
  ],
  "syndic": {
    "nom": "Foncia Vaucelles",
    "email_gestionnaire": "martin@foncia.fr"
  },
  "codes_acces": "Digicode 1234, Gardien M. Dupont",
  "pieces_jointes": [
    {
      "nom": "photo_fuite.jpg",
      "url": "https://storage.replit.com/...",
      "type": "image/jpeg"
    }
  ],
  "confiance_globale": 0.89
}
```

**Reponse (201 Created) :**
```json
{
  "success": true,
  "demande_id": "uuid-xxx",
  "bien_id": "uuid-yyy",
  "bien_cree": false,
  "message": "Demande creee avec succes"
}
```

**Reponse (422 Validation Error) :**
```json
{
  "success": false,
  "error": "Validation failed",
  "details": {
    "bien.code_postal": "Format invalide"
  }
}
```

---

### 2. Rechercher un bien

```
GET /functions/v1/search-bien?adresse=12+rue&code_postal=75002
```

**Reponse :**
```json
{
  "total": 1,
  "biens": [
    {
      "id": "uuid-xxx",
      "adresse": "12 rue de la Paix",
      "code_postal": "75002",
      "ville": "Paris",
      "client": {
        "id": "uuid-yyy",
        "nom": "Foncia"
      }
    }
  ]
}
```

---

### 3. Lister les clients (syndics)

```
GET /functions/v1/clients
```

**Reponse :**
```json
{
  "clients": [
    {
      "id": "uuid-xxx",
      "nom": "Foncia Vaucelles",
      "email": "contact@foncia.fr"
    }
  ]
}
```

---

## Edge Function Supabase

```typescript
// supabase/functions/create-demande/index.ts

import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  Deno.env.get('SUPABASE_URL')!,
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
);

Deno.serve(async (req) => {
  // Verifier authentification
  const authHeader = req.headers.get('Authorization');
  if (!authHeader?.startsWith('Bearer ')) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), { 
      status: 401 
    });
  }

  const data = await req.json();

  // 1. Chercher ou creer le bien
  let bien = await findBien(data.bien);
  let bienCree = false;
  
  if (!bien) {
    bien = await createBien(data.bien, data.syndic);
    bienCree = true;
  }

  // 2. Chercher ou creer le client
  let client = await findClient(data.syndic);
  if (!client) {
    client = await createClient(data.syndic);
  }

  // 3. Creer la demande (petite intervention)
  const demande = await createDemande({
    bien_id: bien.id,
    client_id: client.id,
    ...data.demande,
    source: data.source,
    contacts: data.contacts,
    codes_acces: data.codes_acces,
    confiance: data.confiance_globale
  });

  // 4. Traiter les pieces jointes
  if (data.pieces_jointes?.length) {
    await attachFiles(demande.id, data.pieces_jointes);
  }

  return new Response(JSON.stringify({
    success: true,
    demande_id: demande.id,
    bien_id: bien.id,
    bien_cree: bienCree
  }), { status: 201 });
});
```

---

## Securite

1. **Authentification** : Service Role Key Supabase (secret)
2. **Validation** : Verifier tous les champs entrants
3. **Rate limiting** : Limiter a 100 requetes/minute
4. **Logging** : Logger toutes les demandes recues

---

## Configuration Email Parser (Replit)

```
PLANITUD_API_URL=https://xxxx.supabase.co/functions/v1
PLANITUD_API_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Workflow complet

```
1. Email arrive dans la boite mail
2. Email Parser le detecte et l'analyse (LLM)
3. Email Parser envoie POST /create-demande
4. Planitud cree/trouve le bien
5. Planitud cree la demande (petite intervention)
6. Demande apparait dans le tableau de bord Planitud
7. Email archive dans "Email traite"
```

---

## Estimation

**1-2 jours de developpement**
- Jour 1 : Edge Functions + logique
- Jour 2 : Tests + integration Email Parser
