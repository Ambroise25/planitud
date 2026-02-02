# Specifications Planitud - Nouvelles fonctionnalites

## Contexte

Ces specifications decrivent les fonctionnalites a ajouter a Planitud pour :
1. Remplacer l'Extranet actuel
2. Integrer Email2Extranet

## Documents

| Document | Description | Effort estime |
|----------|-------------|---------------|
| [01-comptes-rendus-intervention.md](01-comptes-rendus-intervention.md) | Module de redaction et envoi des CR | 3-5 jours |
| [02-contrats-entretien.md](02-contrats-entretien.md) | Gestion des contrats d'entretien | 3-4 jours |
| [03-envoi-automatique-syndic.md](03-envoi-automatique-syndic.md) | Envoi email automatique | 2-3 jours |
| [04-api-email-parser.md](04-api-email-parser.md) | API pour Email Parser | 1-2 jours |

## Effort total estime

**10-14 jours de developpement**

## Ordre de developpement recommande

1. **API Email Parser** (1-2 jours) - Permet de brancher Email Parser rapidement
2. **Envoi automatique** (2-3 jours) - Infrastructure email reutilisable
3. **Comptes rendus** (3-5 jours) - Fonctionnalite principale
4. **Contrats entretien** (3-4 jours) - Peut attendre

## Stack technique

- **Backend** : Supabase (Edge Functions, Postgres)
- **Frontend** : Existant Planitud (Lovable/React)
- **Email** : Resend (recommande)
- **PDF** : @react-pdf/renderer ou jsPDF

## Integration Email2Extranet

```
┌─────────────────┐         ┌─────────────────┐
│  Email Parser   │  POST   │    Planitud     │
│    (Replit)     │ ──────> │   (Supabase)    │
└─────────────────┘         └─────────────────┘
        │                           │
        │ Lit emails                │ Affiche demandes
        ▼                           ▼
┌─────────────────┐         ┌─────────────────┐
│   Boite mail    │         │   Dashboard     │
│   syndics       │         │   utilisateur   │
└─────────────────┘         └─────────────────┘
```

## Prochaines etapes

1. Ton collegue lit ces specs
2. Validation/ajustements si necessaire
3. Developpement dans Lovable
4. Integration Email Parser
