# Specifications Extranet

## Objectif

Documenter toutes les fonctionnalites de l'Extranet actuel pour les reproduire dans Planitud.

---

## Modules

| # | Module | Fichier | Statut |
|---|--------|---------|--------|
| 1 | Gestion des biens | [01-gestion-biens.md](01-gestion-biens.md) | ✓ Termine |
| 2 | Gestion des gestionnaires | [02-gestion-gestionnaires.md](02-gestion-gestionnaires.md) | ✓ Termine |
| 3 | Gestion des demandes | [03-gestion-demandes.md](03-gestion-demandes.md) | ✓ Termine |
| 4 | Gestion des interventions | [04-gestion-interventions.md](04-gestion-interventions.md) | ✓ Termine |
| 5 | Journal d'activites | [05-journal-activites.md](05-journal-activites.md) | ✓ Termine |
| 6 | Contrats d'entretien | [06-contrats-entretien.md](06-contrats-entretien.md) | ✓ Termine |
| 7 | Ordres de service | [07-ordres-service.md](07-ordres-service.md) | ✓ Termine |
| 8 | Rapports/CR avec photos | Voir planitud-specs/01-comptes-rendus-intervention.md | ✓ Termine |
| 9 | Envoi email syndic | Voir planitud-specs/03-envoi-automatique-syndic.md | ✓ Termine |

---

## Architecture de donnees

```
┌─────────────┐       ┌─────────────────┐
│   Syndics   │──────<│  Gestionnaires  │
└─────────────┘       └────────┬────────┘
                               │
                               │ gere
                               ▼
                        ┌──────────────┐
                        │    Biens     │
                        └──────┬───────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
   │Interventions│     │  Chantiers  │     │  Contrats   │
   │  (petites)  │     │             │     │ Entretien   │
   └─────────────┘     └─────────────┘     └─────────────┘
          │                    │
          ▼                    ▼
   ┌─────────────┐     ┌─────────────┐
   │   Rapports  │     │  Rapports   │
   │     CR      │     │  Chantier   │
   └─────────────┘     └─────────────┘
```

---

## Workflow principal

```
1. Email syndic arrive
2. Email Parser extrait les infos
3. Creation bien (si nouveau) + intervention dans Extranet
4. Visite constat → Rapport envoye au syndic
5. Reception ordre de service
6. Intervention (RDF, reparation, chantier)
7. Rapport envoye au syndic
8. Proposition contrat entretien
```

---

## Prochaines etapes

1. Valider les specs avec le collegue
2. Developper dans Planitud
3. Integrer avec Email Parser
