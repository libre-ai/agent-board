[English](README.md) · **Français**

> [!NOTE]
> **Réservé · futur foyer de Missions** — reconstruit dans le dépôt de base canonique [`libre-ai/libre-ai`](https://github.com/libre-ai/libre-ai) ([topologie multi-dépôts, ADR-0008](https://github.com/libre-ai/libre-ai/blob/main/docs/adr/0008-multi-repo-target-topology-and-brand.md)).
> Ce dépôt rouvrira comme dépôt applicatif réel lorsque le propriétaire l'activera, consommant la base comme dépendance versionnée. Les fondations décrites ci-dessous sont **en cours de construction** — avec des liens vers le code qui existe déjà.

# Missions

**L'autorité qui gouverne les missions bornées d'agents.** Un demandeur propose une mission — un travail borné confié à des agents ; deux agents relecteurs indépendants approuvent le même plan immuable ; Missions l'autorise ; un orchestrateur l'exécute ; et le résultat est validé par quorum avant de compter. **L'activité rapportée reste distincte des résultats validés.**

Missions est l'autorité humaine de la couche-2 (Polaris) sur l'orchestration d'agents : elle décide ce qui est permis, consigne les preuves, et ne laisse jamais l'affirmation d'un agent tenir lieu de résultat validé à elle seule.

## Ce qui la distingue

- **Quorum à deux agents.** Aucune identité — humaine ou agent — n'autorise son propre travail. Deux relecteurs éligibles, distincts de tout contributeur, approuvent le même **digest de plan immuable**, puis le même **digest de résultat immuable**.
- **Rapporté ≠ validé.** Ce qu'un agent rapporte avoir fait est tenu distinct de ce qui a été validé par quorum. L'observation ne devient jamais silencieusement une vérité.
- **Digests immuables.** L'autorisation se lie à un digest de plan exact ; un résultat modifié exige de nouvelles relectures. La preuve passée n'est jamais réécrite.
- **Porte de contrôle humaine conservée.** Les contrats canoniques protégés, l'auth, les migrations, les releases et les déploiements conservent une porte de contrôle humaine supplémentaire au-dessus du quorum d'agents.
- **Refus par défaut.** Un relecteur inconnu ou qui s'auto-relit, un digest périmé, ou une autorisation expirée est refusé, fermé par défaut.

## État — spécifié publiquement, fondations en construction

Missions est reconstruite à partir de contrats verrouillés. Elle **n'est pas encore publiée** ; la ligne de base d'autorité v1 existe déjà et est prouvée dans le dépôt de base :

| Fondation                                                 | État         | Preuve                                                                                                                                                          |
| --------------------------------------------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Machine à états du domaine v1**                         | ✅ construit | Cycle de vie des missions comme machine à états pure ([#151](https://github.com/libre-ai/libre-ai/pull/151))                                                    |
| **Persistance RLS par locataire**                         | ✅ construit | Isolation append-only, row-level-security ([#152](https://github.com/libre-ai/libre-ai/pull/152))                                                               |
| **Matrice d'autorisation**                                | ✅ construit | Autorisation côté app conforme au contrat missions-v1 ([#153](https://github.com/libre-ai/libre-ai/pull/153))                                                   |
| **Cockpit accessible**                                    | ✅ construit | Vue en lecture rendue côté serveur, accessible au clavier ([#154](https://github.com/libre-ai/libre-ai/pull/154))                                               |
| **Service de commandes**                                  | ✅ construit | Compose autorisation → domaine → persistance ([#155](https://github.com/libre-ai/libre-ai/pull/155))                                                            |
| **Qualification adversariale du chemin d'écriture**       | ✅ construit | Qualification de release du chemin d'écriture ([#157](https://github.com/libre-ai/libre-ai/pull/157))                                                           |
| **Contrats d'orchestration + brique moteur**              | ✅ présent   | Schémas `mission-record.v2`, `orchestrator-control/event`, `agent-review-quorum` + vecteurs golden (quorum, transitions, digests) ; `crates/agent-orchestrator` |
| **Quorum à deux agents v2**                               | ⏳ suite     | Contrat verrouillé, non implémenté : relectures signées par agent, isolation des relecteurs, nonce/expiration                                                   |
| **Liaison des events orchestrateur + portes de décision** | ⏳ suite     | Accepter les events signés par l'orchestrateur ; bloquer/reprendre sur les demandes de décision humaine                                                         |
| **Agent Board** — projection opérationnelle               | ⏳ suite     | Le tableau de bord en lecture seule (flotte, task board, avancement live) qui projette les events de Missions                                                   |

Ce dépôt est public et réservé. **Cible de référence :** les plateformes d'agents managés (p. ex. Multica) — atteinte par une autorisation gouvernée et des preuves de quorum plutôt que par le débit brut de tâches.

## Comment ça fonctionne

1. **Proposer / relire** — un demandeur crée une mission à partir d'un handoff de planification accepté ; un corps de plan déterministe est relu en aveugle par deux agents éligibles sur le même digest.
2. **Autoriser / observer** — Missions vérifie le quorum du plan et émet une **autorisation expirante** ; l'orchestrateur rapporte des events causaux tandis qu'un opérateur peut suspendre ou annuler dans les limites de la politique.
3. **Valider** — le résultat est approuvé par un second quorum sur le digest de résultat immuable avant de compter comme validé ; l'activité rapportée qui n'a jamais atteint le quorum n'est jamais promue en vérité.

## Architecture — assemblée à partir de briques interopérables

Missions est l'autorité de la surface humaine de la couche-2. Autour d'elle, chaque brique est versionnée indépendamment et interopérable (la cible multi-dépôts de [l'ADR-0008](https://github.com/libre-ai/libre-ai/blob/main/docs/adr/0008-multi-repo-target-topology-and-brand.md)).

| Brique                                         | Rôle                                   | Interface exposée / consommée                                                                                          |
| ---------------------------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Missions** (`apps/missions`)                 | L'autorité — control plane             | Propose, relit, autorise, valide ; émet autorisation + events de mission                                               |
| **Agent Board** (`apps/agent-board`, ⏳)       | La projection — observability plane    | Vue en lecture seule des events de Missions : flotte, task board, avancement live ; n'écrit jamais l'état des missions |
| **Orchestrator** (`crates/agent-orchestrator`) | Le moteur d'exécution                  | Planifie, exécute, applique le budget, rapporte les events causaux ; consomme une autorisation expirante               |
| **Contrats**                                   | Surface d'interopérabilité verrouillée | Schémas `mission-record.v2`, `orchestrator-control/event.v1`, `agent-review-quorum.v1` + vecteurs golden               |

L'autorité (Missions) **écrit** l'état de vérité ; le tableau de bord le **lit** pour rendre la flotte observable ; l'orchestrateur **exécute** le travail entre les deux. Le tableau de bord ne détient aucune autorité — il ne peut pas autoriser, seulement afficher.

## Où se déroule le travail

Tout le développement actif est dans le dépôt de base, sous :

- `apps/missions` — l'autorité (domaine, autorisation, persistance, cockpit, service de commandes)
- `crates/agent-orchestrator` — le moteur d'exécution
- `contracts/` — les schémas verrouillés, les contrats d'orchestration et les vecteurs golden
- [`docs/apps/missions.md`](https://github.com/libre-ai/libre-ai/blob/main/docs/apps/missions.md) — le cahier des charges de l'autorité Missions
- [`docs/apps/agent-board.md`](https://github.com/libre-ai/libre-ai/blob/main/docs/apps/agent-board.md) — le cahier des charges de la projection Agent Board

Pour suivre l'avancement ou contribuer, ouvrez issues et pull requests dans [`libre-ai/libre-ai`](https://github.com/libre-ai/libre-ai). Ce dépôt reste réservé jusqu'à son activation.

## Licence

EUPL-1.2.
