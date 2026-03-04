Doc : https://github.com/SAP-samples/btp-sac-forecast/tree/main/documentation
# Cloud CAP Risk Management (POC)

POC **Risk Management** sur **SAP BTP** (Cloud Foundry) basé sur **SAP Cloud Application Programming Model (CAP)**, **UI5 Fiori elements** et **HANA HDI container**.  
L’application expose des risques (dataset historique) via CAP/OData, affiche des UI Fiori (Risks, Mitigations) et s’intègre à **SAP Build Work Zone**. Un **mock-server** remplace SAP S/4HANA pour l’API Business Partner.

> Référence générale SAP (exemples et tutoriels CAP/BTP) : [SAP-samples/cloud-cap-risk-management](https://github.com/SAP-samples/cloud-cap-risk-management/) — repo d’exemples (archivé, lecture seule).  
> Ces exemples illustrent le packaging CAP+UI5+HANA et des déploiements CF/Kyma.  
> ⚠️ Le présent repo (`selmzah/cloud-cap-risk-management`) contient **mon POC** adapté aux besoins (mock BUPA, Work Zone, etc.).
  
## Architecture

- **Backend** : CAP (Node.js), service `RiskService` (OData v4).
- **Frontend** : UI5 Fiori elements (`nsrisks`, `nsmitigations`) packagées en HTML5 apps.
- **Base de données** : **HDI container** (HANA service CF, plan `hdi-shared`) — pas d’instance HANA Cloud dédiée.
- **Sécurité** : XSUAA (rôles `RiskManager`, `RiskViewer`).
- **Connectivité ERP** : Destination **`cpapp-bupa`** pointant vers un **mock-server** S/4 BUPA.
- **Work Zone** : exposé via **HTML5 Applications** (site Work Zone).

## Décisions clés

- **Node.js buildpack CF** : épingle la version **Node 20.19.3** dans `package.json` →  
  `"engines": { "node": "20.19.3" }`. Le buildpack n’expose plus Node 18.
- **CAP build** : compat CAP v6 → forcer **`@sap/cds-dk@7`** dans le hook `before-all` du `mta.yaml`:  
  `npx -p @sap/cds-dk@7 cds build --production`.
- **S/4 API** : ressource `s4-hana-cloud / api-access` **désactivée** dans `mta.yaml` (commentée).  
  En **production**, CAP lit via la **destination `cpapp-bupa`** (vers le mock).
- **Données d’exemple** : CSV auto‑chargés par le **module DB deployer** au premier déploiement.  
  > Si besoin de “livrer vide”, activer en fin de POC la suppression des CSV :  
  > `npx rimraf gen/db/src/gen/data` (à éviter durant le POC sinon plus d’auto‑load).

## Déploiement (Cloud Foundry)

1. **Build CAP** (BAS)  
   ```bash
   npx -p @sap/cds-dk@7 cds build --production
mbt build -t mta_archives
cf login
cf deploy mta_archives/<nom_fichier>.mtar

   Post‑deploy

Vérifier apps cpapp-srv, nsrisks, nsmitigations (cf apps).
Dans le Subaccount BTP → HTML5 Applications → ouvrir les UI.
(Option) Intégrer dans SAP Build Work Zone.
Configuration
package.json
engines.node = "20.19.3" (aligné sur le Node.js buildpack CF).
cds.requires.API_BUSINESS_PARTNER :
[sandbox].credentials.url = URL du mock
[production].credentials.destination = "cpapp-bupa".
mta.yaml
Hook build CAP : npx -p @sap/cds-dk@7 cds build --production
Ressources : cpapp-db (hdi-shared), cpapp-uaa, cpapp-destination, cpapp-html5-repo-host, cpapp-logs.
S/4 service commenté (usage du mock uniquement).
Données
Schéma HDI généré automatiquement (nom technique aléatoire).
Tables CAP (générées) remplies par les CSV du projet au premier déploiement.
L’UI Fiori lit via CAP ; Work Zone affiche les UI sans stocker de données.
Rôles & Sécurité
Rôles RiskManager (manage) et RiskViewer (view) provisionnés par XSUAA.
Assigner les collections « RiskManager-<space> » et « RiskViewer-<space> » aux utilisateurs BTP.
Dépannage (quick wins)
Staging CF échoue avec Node 18 → fixer engines.node à une version supportée (ex : 20.19.3).
Build CAP échoue (cds-dk incompatible) → utiliser @sap/cds-dk@7 avec CAP v6.
Pas de données → vérifier que les CSV existent dans gen/db/src/gen/data et que le db-deployer s’est exécuté.
Pas de lecture SQL directe → tables en HDI ; utiliser Database Explorer as HDI user ou les services OData CAP.

https://github.com/SAP-samples/btp-sac-forecast/tree/main/documentation

Licence
Projet POC ; se base sur les concepts présentés par SAP (exemple public archivé). Voir licences applicables dans le repo SAP-samples si nécessaire.
