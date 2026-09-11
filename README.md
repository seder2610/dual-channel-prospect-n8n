# Dual-channel prospect — UN seul workflow

**Fichier à importer dans n8n :** `dual-channel-prospect-v1.json`

Ce README documente ce workflow unique. Il ne s'agit **pas** d'une chaîne de 3–4 workflows séparés.

---

## Ce que fait le workflow

```
API Recherche Entreprises (NAF 73.11Z, agences pub)
    → scrape site (mentions légales, contact, équipe)
    → score confiance email + vérif MX DNS
    → extraction URL LinkedIn (si présente sur le site)
    → qualification ICP douleur (Haiku)
    → création fiche Notion (statut « À valider »)
```

**Toi ensuite :** valider dans Notion → envoyer email + connexion LinkedIn (manuel).

---

## Peur des faux emails — comment on s'en protège

### Règle d'or

> **On n'envoie jamais un email qu'on n'a pas trouvé sur le site.**

Les patterns (`prenom.nom@domaine`) sont des **hypothèses**. Ils vont dans **Notes**, jamais dans la colonne Email.

### Niveaux de confiance

| Niveau | Signification | Colonne Email Notion | Tu peux envoyer ? |
|--------|---------------|----------------------|-------------------|
| `verifie_mentions_legales` | Trouvé sur /mentions-legales | Oui | Après ta validation |
| `verifie_site` | Trouvé sur contact/équipe/home | Oui | Après ta validation |
| `generique_site` | contact@ / hello@ (réel mais boîte partagée) | Oui + warning Notes | Vérifie le canal |
| `hypothese` | Deviné (prenom.nom@) | Non | Jamais sans vérif manuelle |
| `mx_invalide` | Domaine sans serveur mail | Non | Jamais — bounce garanti |
| `aucun` | Rien trouvé | Non | LinkedIn seulement |

### Contrôles automatiques

1. **Blacklist** : noreply, wixpress, sentry, dpo@, etc.
2. **MX DNS** (dns.google) : si pas de MX → email rejeté du champ Notion
3. **Dédup SIREN** : pas de doublon dans Notion
4. **Statut sortie** : toujours `À valider` — rien ne part sans toi
5. **LLM** : interdit d'inventer email ou LinkedIn

---

## Colonnes Notion attendues (à confirmer)

| Propriété | Type | Contenu |
|-----------|------|---------|
| **Nom** (title) | Title | Prénom Nom dirigeant |
| **Agence** | Rich text | Raison sociale |
| **Email** | Email | Uniquement email scrapé + MX OK |
| **LinkedIn** | URL | URL /in/ si trouvée sur le site |
| **Notes** | Rich text | confiance, hypothèses, messages |
| **Statut** | Select | À valider |
| **Vague** | Select | Dual-channel ops |

Envoie ton tableau Notion → j'adapte les clés du node création.

---

## Installation

1. n8n → Import → `dual-channel-prospect-v1.json`
2. Credentials : Notion API, Anthropic (Haiku)
3. Remplacer `notion_database_id` dans Configuration
4. Test manuel puis schedule 6h lun-ven
