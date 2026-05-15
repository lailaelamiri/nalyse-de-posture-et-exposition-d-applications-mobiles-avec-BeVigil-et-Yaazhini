# Notes d'analyse BeVigil
## App analysée : com.cerdillac.filmmaker v3.4.3

## Ce qui est certain
- Firebase URL exposée : https://bff-test.firebaseio.com
- 1 Firebase Storage Bucket détecté (exposition cloud potentielle)
- 1 Firebase URL détectée
- 31 adresses IP hardcodées dans le code
- 4 IP URLs directes
- 8 adresses email dans le code source
- 50 endpoints REST détectés
- 50 URLs détectées
- 48 domaines associés

## Ce qui est hypothèse
- Les IP hardcodées pourraient être des serveurs backend de production
- Le Firebase bucket pourrait être accessible publiquement sans authentification
- Les emails pourraient appartenir aux développeurs (info de contact)

## Points d'intérêt
- Firebase Storage Bucket : vérifier si accès public possible
- 31 IPs hardcodées : surface d'attaque étendue
- Shaders depuis chemins utilisateur (ChenXinHeng0430/...) : ressources tierces non contrôlées

## Domaines et sous-domaines
- 48 domaines détectés (voir export CSV)

## Endpoints et APIs
- 50 Relative Endpoints (ex: /adMuted, /default/token)
- 50 REST API endpoints
- Pattern observé : chemins de shaders GLSL et assets média

## URLs HTTP/HTTPS
- 50 URLs détectées (voir export CSV)
- 4 IP URLs directes (non chiffrées potentiellement)

## Emails et identifiants
- 8 adresses email trouvées dans le code

## Technologies détectées
- Firebase (Storage + Realtime DB)
- OpenGL ES / GLSL (shaders vidéo)

## Résultat test Firebase
- URL testée : https://bff-test.firebaseio.com/.json
- Réponse : {"error" : "The Firebase database 'bff-test' has been deactivated."}
- Interprétation : Base désactivée, aucune donnée exposée actuellement
- Statut : FAIBLE RISQUE (historique d'usage Firebase confirmé)
