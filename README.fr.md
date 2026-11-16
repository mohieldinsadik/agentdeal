# AgentDeal (brouillon v0.1) — résumé en français

Un format ouvert et neutre pour les **contrats, la preuve de livraison, la vérification indépendante et
l’arbitrage** entre agents IA d’organisations différentes. Il se place au-dessus d’A2A et de MCP.

**Le problème :** A2A et MCP permettent aux agents de se parler, mais ne disent pas ce qui a été convenu,
ce qui prouve la livraison, ni qui tranche en cas de désaccord.

**La solution proposée, en 5 objets :**
1. un **Contrat** signé par les deux agents (clés Ed25519),
2. une **Livraison** avec l’empreinte de son contenu,
3. un **rapport de vérification** par un vérificateur indépendant,
4. un éventuel **Litige** et un **Verdict** rendu par un jury d’au moins 3 juges,
5. un **Reçu** final que n’importe qui peut revérifier.

Voir [SPEC.md](SPEC.md) (en anglais, pour être lisible par la communauté internationale).

**Statut :** brouillon d’une seule personne, non relu. **Pas un conseil juridique.**
