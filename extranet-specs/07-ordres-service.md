# Module Ordres de Service

## Objectif

Stocker et suivre les ordres de service (OS) recus des syndics pour valider les travaux.

---

## Concepts

### Ordre de Service (OS)
Document officiel envoye par le syndic pour :
- Valider un devis
- Autoriser le demarrage des travaux
- Servir de reference pour la facturation

---

## Informations d'un OS

Basees sur l'exemple reel (Plisson Immobilier) :

| Champ | Description | Exemple |
|-------|-------------|---------|
| N° OS | Reference unique du syndic | BDCE26383 |
| Date | Date d'emission | 26/01/2026 |
| Syndic | Societe emettrice | Plisson Immobilier |
| Gestionnaire | Personne qui signe | Eva Marie LE BOUYONNEC |
| Reference devis | Lien vers le devis valide | DEV26141 |
| Objet | Description des travaux | Mesure conservatoire |
| Bien | Adresse concernee | 24 rue Hegesippe Moreau, 75018 Paris |
| Montant TTC | Montant autorise | 3 120,00 € |
| Contacts | Gardien, coproprietaire, occupant | Voir document |
| Acces | Moyens d'acces au bien | Digicode, cle... |
| PDF | Document original | Piece jointe |

---

## Stockage

L'OS est lie a une **demande** :
- Une demande peut avoir 0 ou plusieurs OS
- Chaque OS valide une etape (visite → RDF → chantier)

---

## Fiche OS

### Champs du formulaire

| Champ | Type | Obligatoire | Description |
|-------|------|-------------|-------------|
| Demande | lien | Oui | Reference a la demande |
| Numero OS | texte | Oui | Reference du syndic |
| Date reception | date | Oui | Date de reception |
| Date emission | date | Non | Date sur le document |
| Reference devis | texte | Non | N° du devis valide |
| Objet | texte | Oui | Description courte |
| Montant TTC | nombre | Non | Montant autorise |
| Commentaire | zone texte | Non | Notes internes |
| Document PDF | fichier | Oui | Upload du PDF original |

---

## Schema de donnees

```sql
CREATE TABLE ordres_service (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  demande_id UUID REFERENCES demandes(id) NOT NULL,
  numero_os VARCHAR(100) NOT NULL,
  date_reception DATE NOT NULL,
  date_emission DATE,
  reference_devis VARCHAR(100),
  objet VARCHAR(500) NOT NULL,
  montant_ttc DECIMAL(10,2),
  commentaire TEXT,
  document_url TEXT NOT NULL,
  document_nom VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_os_demande ON ordres_service(demande_id);
CREATE INDEX idx_os_numero ON ordres_service(numero_os);
CREATE INDEX idx_os_date ON ordres_service(date_reception);
```

---

## Affichage

### Sur la fiche demande

Section "Ordres de service" avec tableau :

| Colonne | Description |
|---------|-------------|
| N° OS | Reference |
| Date | Date de reception |
| Objet | Description |
| Montant | Montant TTC |
| PDF | Lien telechargement |
| Actions | Voir, Modifier, Supprimer |

### Liste globale (optionnel)

Vue de tous les OS recus, avec filtres par :
- Syndic
- Date
- Demande

---

## Workflow

### Reception manuelle
```
1. Email recu du syndic avec OS en PJ
2. Ouvrir la demande concernee
3. Cliquer "Ajouter un ordre de service"
4. Remplir les infos + upload PDF
5. OS enregistre et visible sur la demande
```

### Reception automatique (Email Parser)
```
1. Email recu avec OS detecte
2. Email Parser extrait les infos
3. OS cree automatiquement sur la demande
4. PDF stocke
5. Notification a l'equipe
```

---

## Lien avec le flux de travail

```
1. Demande recue (email syndic)
2. RDV de visite → Rapport + Recommandations + Devis
3. ★ Reception OS (validation du devis) ★
4. Intervention (RDF, reparation, chantier)
5. Compte-rendu envoye
6. Si nouveau devis → Nouveau cycle (retour etape 3)
```

---

## Extraction automatique (futur)

L'Email Parser pourrait extraire automatiquement :
- N° OS (pattern : lettres + chiffres)
- Montant (pattern : X XXX,XX €)
- Reference devis (pattern : DEV + chiffres)
- Adresse du bien (matching avec base existante)

---

## Actions

| Action | Description |
|--------|-------------|
| Ajouter un OS | Depuis la fiche demande |
| Modifier | Mise a jour des infos |
| Telecharger PDF | Acces au document original |
| Supprimer | Confirmation requise |
