# Cockpit local de product readiness

Date canonique : 2026-07-14  
Snapshot : `main@36fd7fa`

## Lecture rapide

- Maturité : **contract-first**
- Disponibilité : **discovery**
- Journey locale : **partielle** — cycle de contrat seulement
- À ne pas annoncer : **board disponible**

## Preuves établies

- `MissionRecord v1` est contractuellement défini et validé par schéma.
- Fixtures positives/négatives présentes sous `fixtures/mission-record/`.
- Le moteur de transition local est side-effect-free et applique des mises à jour immuables.
- Les révisions optimistes sont vérifiées avant toute transition.
- Le lifecycle couvre block / recovery / result / verdict humain.
- Les refus sont fail-closed.

## Absences confirmées

- projection board locale absente
- persistence absente
- execution / routing agent absent
- adapter live Agent Factory absent
- collaboration hébergée absente

`0 issue` ne signifie pas `readiness`.

## Gates

### P0

- board locale en lecture seule sur le moteur existant

### P1

- adapter simulé
- preuve locale mission-governance

### P2

- intégrations explicites
- persistance
- tenancy
- prérequis d’hébergement avant toute mise en ligne

## Liens utiles

- [README](../README.md)
- [ROADMAP](../ROADMAP.md)

## Remarque

Ce cockpit est un état documentaire, pas une promesse de produit livré.