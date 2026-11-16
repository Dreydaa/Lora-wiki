@@ page decisions | I | 13 | Décisions et compromis | Chaque décision structurante, l'alternative écartée, la raison et le coût accepté.

| Décision | Alternative écartée | Pourquoi | Coût accepté |
| --- | --- | --- | --- |
| **QStash** pour planifier briefing et bilan | Crons Vercel | Le plan gratuit limite à 1 exécution par jour avec ±59 min d'imprécision ; QStash a une API qui permet de changer l'heure *depuis une conversation*, sans redéployer | Un service tiers de plus |
| **Webhook + file + worker** | Tout traiter dans le webhook | Un appel IA lent ne doit pas faire expirer ni rejouer le webhook | Plus de pièces mobiles |
| **Notion comme source de vérité, y compris la config** | Variables d'environnement, base dédiée | Changer l'heure du briefing et du bilan en conversation naturelle, sans toucher au déploiement | Latence et limites de l'API Notion |
| **Un bloc Notion par jour de shift** | Un évènement récurrent | L'API Notion ne gère pas la récurrence : un bloc unique s'afficherait comme un évènement continu | Beaucoup de pages créées |
| **Redis limité à l'éphémère** | Y mettre aussi l'état métier | Deux sources de vérité = divergences | Rien d'autre que du cache en Redis |
| **La pause dans Redis, pas dans Notion** | Nouvelle propriété Notion | Évite de modifier le schéma de la base de production pour un état transitoire ; expire seule 3 jours après la fin | Pas visible dans Notion |
| **Dashboard branché en dernier** | Interface d'abord | Ne pas construire une interface contre un backend mouvant : elle a tourné sur des données factices jusqu'à ce que le backend soit éprouvé | Un temps sans vraies données à l'écran |
| **Un seul fournisseur de modèle** | Garder deux fournisseurs « au cas où » | Code de branchement, dépendance et variables supprimés plutôt que laissés en jachère | Dépendance à Groq |
| **Pas de vidéo pour l'intro** | Export vidéo depuis Blender | Une vidéo qui bascule vers un canvas crée un raccord visible et n'est pas réactive | Intro codée à la main |
| **Rendu du ruban en three.js, pas depuis Blender** | Embarquer une scène Blender | Une scène Blender ne s'embarque pas comme fond d'une app Tauri ; l'équation paramétrique est plus légère et variable en temps réel | Blender ne sert qu'au prototypage |
| **Fenêtre verrouillée en 1440 ou 1920 de large** | Mise en page responsive complète | Une app de bureau n'a pas besoin d'un mode mobile ; la grille est mise à l'échelle | Pas d'usage sur petit écran |
| **Temps d'écran mesuré en Rust, gardé sur la machine** | L'envoyer à Notion ou au backend | Ce qu'on fait sur son ordinateur est une donnée sensible : elle n'a pas à quitter la machine, et un fichier local évite tout réseau | Visible seulement dans l'app, Windows uniquement |
| **Fermer la fenêtre la masque** | Quitter l'application | Le suivi et le démarrage automatique supposent une app qui reste vivante ; « Quitter » est explicite dans la zone de notification | L'app reste en mémoire |
| **Le PC va chercher ses étapes** | Ouvrir un port sur le PC, ou l'allumer à distance | Aucun port ouvert ; le PC interroge le backend toutes les 30 s ; l'allumage est fait par l'utilisateur avec l'application de sa prise connectée | Environ 2 900 lectures Redis par jour ; un PC éteint ne reprend le protocole qu'au démarrage de l'application |
| **Reconnaissance vocale locale (Whisper) avec mot d'activation tout de suite** | Un raccourci à maintenir (plus fiable, sans faux déclenchements : c'était la recommandation) ; un service de reconnaissance en ligne | Choix de l'utilisateur ; l'audio ne quitte pas le PC | Faux déclenchements possibles (noms proches, télévision) ; installation à la main, 1,7 Go |
| **Une voix entraînée sur des enregistrements consentis (Piper)** | Un clonage sans entraînement (jugé « très médiocre »), des voix cloud payantes, la transformation de sa propre voix | Pas de clonage d'une voix sans accord écrit ; Piper est local et gratuit ; les conditions d'un service cloud interdisent d'entraîner un autre modèle avec ses sorties | Environ 6 h d'entraînement, 45 minutes à enregistrer ; attribution à prévoir si la voix est distribuée |
| **Alléger les appels plutôt que payer** | Un palier payant de Groq ou une API payante | L'utilisateur refuse de payer au jeton ; le goulot est la limite de 8 000 jetons par minute | Un routage par mots-clés à maintenir ; un second modèle qui suit moins bien la personnalité |
| **Confirmer à voix haute avant d'écrire dans Notion** | Écrire directement, comme sur Telegram | Une phrase mal comprise ne doit pas écrire des données | Un échange de plus |
| **La zone de texte de la barre reste** | Tout passer à la voix | Un micro en panne ou une impossibilité de parler ne doivent pas enfermer l'utilisateur dehors | Deux entrées à maintenir |
| **Répondre sans modèle quand le code le peut** | Tout envoyer au modèle (la recherche web coûte 19 000 à 41 000 jetons) | Heure, minuteurs, météo de demain, conversions, titres, compte à rebours se fabriquent en code : zéro jeton, réponse immédiate ; une demande reconnue ne retombe jamais sur le gros modèle (si un service est en panne, la réponse le dit) | Des reconnaissances à maintenir ; une phrase proche d'une demande reconnue peut être mal captée |
| **Aucun ordre ne descend vers le PC, sauf les protocoles** | Piloter le PC depuis Telegram (« verrouille le PC ») | Garder la surface d'attaque petite : seuls la présence du PC et « envoie-moi ça » existent | Pas de verrouillage à distance |
| **Une confirmation vocale pour ce qui éteint ou efface** | Exécuter directement | Une phrase mal comprise ne doit pas éteindre le PC ni vider la corbeille : question, « oui » dans les 30 secondes, délai de 30 secondes avant l'extinction | Un échange de plus |
| **Les moteurs de voix hors du dossier effaçable** | Les laisser dans le dossier de données de l'application | La case « supprimer les données » du désinstalleur avait tout effacé (1,9 Go posés à la main) ; un dossier voisin n'est pas touché | Un dossier de plus à connaître |
| **Abandonner une fonction qui ne tient pas** (« qu'est-ce que je regarde ? ») | La garder « au cas où » | Inutile pour l'utilisateur, coût en jetons non rentable, et le modèle de vision inventait le contenu de l'écran | Du code écrit puis retiré |
| **Gestes de la main : un seul mode, Blender** | Des commandes de base aux gestes (scroll, son, déplacement de fenêtre) | Un geste par action sans conflit ; le scroll a été codé puis retiré le jour même, le son et la fenêtre abandonnés par l'utilisateur (le son reste possible à la voix) | Les gestes ne servent plus que dans Blender |
| **Caméra éteinte et fenêtre cachée par défaut** | Un aperçu ouvert par un menu | Une caméra qui tourne sans qu'on y pense est une surface sensible ; la voix est le seul accès | Première utilisation : « affiche la caméra » pour autoriser WebView2 |
| **Deux index pointés pour le zoom** | Le dos des mains ou les paumes face à face, avec un cliquet | Le dos de la main cache le poignet et l'orientation de la paume de profil est instable : mesuré, puis remplacé par un signal qui reste visible | Un geste moins naturel, pas de cliquet |
| **Deux poings fermés pour valider** | Une paume qui se ferme | La paume d'un poing est instable : elle relançait l'embrayage en boucle | Un geste de plus à apprendre |
| **Des tests sans aucun vrai service** | Tester contre les vrais Notion, Redis, Groq | Rapides (environ une seconde pour 547 tests), reproductibles, sans risque pour les vraies données ; tout appel réseau non prévu fait échouer le test | Un seul test (`test:live`), lancé à la main, valide les vraies APIs |

---

@@ page retraits | I | 14 | Ce qui a été retiré | Savoir retirer est autant une compétence que savoir ajouter : chaque retrait est daté et motivé.

Chaque retrait est daté et motivé dans l'état du projet (`docs/ai/STATUS.md`).

| Retiré | Date | Pourquoi et comment |
| --- | --- | --- |
| **Le « Proof of Work »** | 26/09/2026 | Preuve tangible d'une journée (lien, capture, export 3D, commit) : jugée trop lourde à l'usage, remplacée par un contrôle de cohérence conversationnel, puis supprimée. À reconsidérer en V2 seulement. |
| **Le streak** (compteur de jours consécutifs) | 30/09/2026 | Retiré *en entier*, pas seulement son affichage. La machinerie qui le justifiait (contrôle d'horaire, vérification de cohérence) est partie avec lui ; `record_progress` est redevenu un simple enregistrement ; la colonne a été supprimée de la base Notion de production plutôt que laissée morte. |
| **Le mode ACTIF/PAUSE et les deux personnalités** | 07/10/2026 | Un mode qui basculait le ton du modèle selon les horaires de travail. Retiré en entier : le module de détection, le type, le champ dans Notion et dans l'API, l'outil qui le pilotait, les composants du dashboard. L'utilisateur tenait sa cadence de travail : le mécanisme n'avait plus d'utilité pour lui, et un mécanisme inutilisé reste du code à maintenir et à tester. |
| **L'ancien dashboard plat et son code mort** | 07/10/2026 | 20 fichiers, repérés par une cartographie des imports à partir du point d'entrée plutôt qu'« à l'œil ». |
| **Les réglages de shift** | 08/10/2026 | Le code, la réponse de l'API, le type du dashboard et les colonnes Notion correspondantes, par un script de migration qui **simule d'abord** puis s'applique avec `--apply`. |
| **5 dépendances inutilisées** | 08/10/2026 | Le build du front reste identique à l'octet près. |
| **La dégradation visuelle de l'interface après un jalon manqué** | 08/10/2026 | Envisagée, puis abandonnée, hors V1 comme hors V2. Une décision explicite vaut mieux qu'une idée qui traîne dans la liste des « à faire ». |
| **Le second fournisseur de modèle** (Gemini), la refonte visuelle « éditoriale », le parallax du fond, une première version du fond en poussière à quatre couches, la mise en page mobile | 24/09 → 06/10/2026 | Essayés puis abandonnés. |
| **La voix de Windows, puis son retour** | 18 → 19/10/2026 | Faite le 18/10, retirée le jour même quand l'utilisateur a cherché une voix plus naturelle, remise le 19/10 « en attendant » ; elle reste le repli de la voix entraînée. |
| **Quatre pistes de voix** (Piper avec ses voix de série, voix cloud payantes, transformation de sa propre voix, clonage sans entraînement) | 18/10 → 02/11/2026 | Essayées ou étudiées puis écartées ; tout ce qui avait été installé pour les essais a été supprimé (environ 37 Go sur deux séances). |
| **L'allumage du PC par Løra** (prise connectée, réveil par le réseau) | 17/10/2026 | Écarté : l'utilisateur démarre son PC lui-même avec l'application de sa prise. |
| **Les premières formes du retour visuel de la voix** (ruban réduit de moitié, étoile à six branches) | 06/11/2026 | Remplacées à la demande de l'utilisateur par un huit à l'écoute et un spectre circulaire en parole. |
| **« Qu'est-ce que je regarde ? »** (capture d'écran et modèle de vision) | 08/11/2026 | Décision de l'utilisateur : fonction inutile, coût en jetons non rentable. Code retiré (capture, commande, champ image de `api/ask`, deux dépendances, tests) ; le modèle de vision inventait le contenu de l'écran. Il n'existe plus que dans l'historique git. |
| **Scroll, son et déplacement de fenêtre aux gestes** | 13/11/2026 | Décision de l'utilisateur : il n'y a plus aucune commande de base aux gestes. Le scroll avait été codé puis retiré le jour même sans jamais être essayé ; le son reste possible à la voix. |
| **Les poses « dos contre dos » et « paumes face à face » pour le zoom, et leur cliquet** | 13/11/2026 | Mesurées puis remplacées par les deux index pointés : le dos de la main cache le poignet et la normale de la paume est instable de profil. |

<span class="st st-repl">Retiré</span> Ne pas réintroduire le streak : c'est une décision de l'utilisateur.

---

@@ page bugs | I | 15 | Bugs et causes racines | Vingt-sept bugs : symptôme, cause, correction, leçon.

Chaque entrée suit le même plan : **symptôme → cause → correction → leçon**.

## 1. QStash livrait vers une URL protégée

Les jobs échouaient en 401 silencieux et étaient retentés sans fin. **Cause** : ils visaient `VERCEL_URL`, l'URL propre à chaque déploiement, protégée par défaut par Vercel Deployment Protection. **Correction** : une URL fixe (`APP_URL`) pointant sur l'alias de production. *Leçon : la protection par défaut d'une plateforme est facile à oublier tant qu'on ne teste pas un vrai appel externe.*

## 2. Le décalage horaire invisible

La fenêtre « pause » (celle de l'ancien mode ACTIF/PAUSE, retiré depuis) se déclenchait avec 1 à 2 heures d'écart selon la saison. **Cause** : Vercel exécute en UTC ; `getHours()` et `toISOString()` donnaient l'heure UTC, pas celle de Paris. **Correction** : une conversion centralisée via `Intl` dans `lib/date.ts`, utilisée partout. *Leçon : ne jamais laisser chaque appelant refaire son propre calcul d'heure.*

## 3. Un `null` explicite qui fait échouer toute une action

Groq renvoie parfois `null` pour un paramètre optionnel. **Cause** : `.optional()` n'accepte que « absent ». **Correction** : `.nullish()`. *Leçon : une validation trop stricte sur la sortie d'un modèle fait échouer l'action de façon invisible au lieu de la rejeter proprement.*

## 4. Des secrets corrompus par un espace

La plupart des valeurs `.env` étaient écrites `CLE= valeur`. Sans effet en local, mais le secret poussé tel quel vers Vercel était invalide. **Correction** : retirer l'espace et repousser. *Leçon : la copie de secrets entre environnements mérite une vérification explicite.*

## 5. Un flux RSS qui dupliquait son titre

Le parsing positionnel des balises `<title>` lisait deux fois le titre du flux (répété dans un bloc `<image>`). **Correction** : isoler chaque titre à l'intérieur de sa balise `<item>`.

## 6. Un déploiement qui envoyait 1,8 Go

Le premier déploiement tentait d'envoyer `node_modules` et la cible de compilation Rust. **Correction** : un `.vercelignore`.

## 7. Le briefing qui tournait en boucle sur un ancien bilan

**Symptôme** : sans bilan la veille, le briefing du matin répétait le dernier bilan enregistré comme s'il datait d'hier. **Cause racine** : la requête renvoyait « le dernier bilan », quelle que soit sa date, et le texte envoyé au modèle disait « Hier : … ». Corriger le *prompt* n'aurait pas suffi : le modèle n'avait aucun moyen de savoir que cette information était périmée. **Correction** : la requête renvoie aussi la **date** du bilan ; le code décide si c'est « hier » ; sinon le modèle reçoit explicitement « Bilan d'hier : AUCUN, le dernier date du … » et doit le signaler. Le même correctif limite les rappels d'agenda aux 7 prochains jours avec une échéance écrite en toutes lettres, car un rendez-vous à trois semaines était présenté comme s'il avait lieu aujourd'hui. *Leçon : ne demandez pas à un modèle de déduire une absence ou une date relative ; calculez-la et donnez-lui le fait.*

## 8. Un correctif écrit n'est pas un correctif livré

Le lendemain du correctif ci-dessus, le briefing avait encore l'ancien comportement. **Cause** : les commits étaient sur une branche de travail, et la production se déploie depuis `main` ; la liste des déploiements montrait un dernier déploiement de production vieux de 7 jours. Indice décisif dans le texte lui-même : il citait un évènement à 23 jours, que le nouveau code aurait filtré. **Résolu le 07/10** : la fusion de la pull request sur `main` a déclenché un déploiement de production, observé « Ready ». *Leçon : « terminé » veut dire déployé et observé en production, pas seulement fusionné ; et un symptôme vous dit quelle version de votre code tourne.*

## 9. Un écran blanc quand le backend est plus ancien que l'app

Le front lit `eveningBriefing` dans la réponse de `/api/dashboard`. Avec un backend déployé avant l'ajout de ce champ, le composant lisait `undefined.split` et tout l'écran devenait blanc après l'heure du bilan. **Correction** : un texte de repli quand le briefing manque. *Leçon : le front et le backend se déploient séparément ; un champ absent d'une réponse ne doit jamais faire planter l'affichage.*

## 10. Des liens qui ne faisaient rien dans l'application

Sans dossier de capacités, Tauri 2 refuse toute commande de plugin (`opener.open_url not allowed`) : les liens d'articles ne faisaient rien. **Correction** : une capacité donnant à la fenêtre principale la seule permission d'ouvrir des URL `https://`. *Leçon : un refus par défaut est un choix de sécurité, pas un bug ; on accorde le minimum.*

## 11. Une trace blanche à l'ouverture d'une carte

Une carte naissait à quelques pixels de haut et recevait un reflet de bord généré à cette taille, étiré ensuite sur toute sa hauteur : des bords blancs épaissis. **Correction** : aucune carte n'est générée pendant un redimensionnement, le reflet d'une carte étirée de plus de 12 % est masqué, et la réfraction n'est appliquée qu'une fois la carte de déplacement prête. Vérifié image par image (à 120 ms, 450 ms, 800 ms, 1,6 s). *Leçon : un effet calculé à une taille n'est pas valable à une autre ; une animation se vérifie sur ses images intermédiaires, pas sur son état final.*

## 12. Un build qui échoue sur des versions décalées

`cargo add` avait monté la crate Tauri à 2.12.1 alors que les paquets npm restaient en 2.11. **Correction** : aligner les deux paquets npm sur 2.12.1. *Leçon : les dépendances jumelles côté Rust et côté JavaScript doivent suivre la même version.*

## 13. Des entités HTML visibles dans les titres d'articles

Certains flux RSS laissent `&#8216;` dans leurs titres, que React affiche tel quel. **Correction** : une fonction `decodeEntities` appliquée juste après l'extraction, qui laisse intacte une entité inconnue. *Leçon : une donnée externe se nettoie à l'entrée, pas à l'affichage.*

## 14. Une animation saccadée dans l'application installée

Le ruban avait été plafonné à 30 images/s au repos pour économiser le CPU (mesuré le 08/10 : environ 8 à 10 % d'un cœur, contre environ 140 % sans plafond), et ce réglage avait été validé sur la consommation, pas sur le ressenti. Dans l'app installée, l'animation était saccadée. **Correction appliquée le 09/10** : 60 images/s au repos. **Cause établie le 12/10/2026** : le `backdrop-filter` à réfraction des cartes de verre, recalculé à chaque image du canvas (~120 % d'un cœur et ~20 images/s mesurés dans l'app installée, contre ~30 % et 60 images/s sans la réfraction). L'utilisateur a choisi de la couper (`REFRACTIVE = false`, réversible). *Leçon : un réglage de performance validé par une mesure de ressources peut dégrader l'expérience ; un seuil de fluidité se juge à l'œil dans l'application réelle.* **Effet de bord** : deux tests d'acceptation figeaient la valeur de 30 images/s et échouaient depuis le passage à 60 ; corrigé le 12/10/2026 (ils lisent `REST_FPS`). *Leçon : un test qui recopie la valeur d'un réglage au lieu de la lire casse à chaque changement du réglage.*

## 15. Un prompt qui impose un mot par défaut le répète partout

Le prompt de personnalité donnait « Boss » comme appellation par défaut, et ses exemples commençaient par ce mot. **Correction** : trois surnoms alternés, jamais le même deux messages de suite, et pas de surnom dans la plupart des messages (un seul, pour saluer, marquer une remarque ou adoucir un rappel) ; la question de repli du briefing du soir n'a plus de surnom en dur. **Suite, 14/10/2026** : la règle « jamais le même deux fois de suite » ne tenait pas, car le modèle n'a aucune mémoire d'un appel à l'autre et retombait sur « patron ». Le code tire désormais au hasard un surnom pour les briefings du matin et du soir et le donne au modèle ; le prompt interdit tout surnom ailleurs. <span class="st st-prop">Non vérifié</span> : le texte réel du modèle (seul le contenu du prompt est testé). *Leçon : une valeur « par défaut » écrite dans un prompt tend à devenir un tic ; mieux vaut décrire quand l'employer. Et une règle qui suppose une mémoire que le modèle n'a pas se déplace dans le code.*

## 16. Le worker se bloque sur la limite de débit du modèle

Deux messages du soir (18/10) restent sans réponse sur Telegram et chacun est traité plusieurs fois, en écrivant à chaque fois dans Notion. **Cause** : trois pièces qui s'enchaînent mal. (1) Le palier gratuit de Groq limite à 8 000 jetons par minute, et un appel pèse au moins 3 748 jetons avant le message de l'utilisateur : une question avec outils en demande 11 000 à 14 000. Les journaux de Groq montrent 12 appels réussis et 16 refus « tokens per minute » sur une heure. (2) Le worker est coupé par Vercel après 25 s sans réponse, et la bibliothèque d'IA attend puis réessaie après chaque refus. (3) QStash relance tout ce qui n'est pas un 2xx, et chaque relance refaisait toute la conversation, écritures Notion comprises, avec des titres un peu différents : des doublons. En cas d'échec, rien n'était envoyé à l'utilisateur. **Correction** : plus de relance automatique (`retries: 0`), le worker répond 200 après avoir prévenu sur Telegram (« vérifie avant de le renvoyer »), échec à 20 s au lieu de 25 ; puis l'allègement des appels (voir [Le cœur IA](#coeur-ia)). *Leçon : un système de relances qui rejoue des effets de bord a besoin d'idempotence, ou de ne rien rejouer ; et un goulot de débit se mesure dans les journaux du fournisseur, pas dans le code.* Les doublons créés ont été supprimés à la main par l'utilisateur. <span class="st st-prop">Non vérifié</span> Un vrai repli de modèle sur un vrai refus.

## 17. Un « demain » recopié du bilan, corrigé deux fois

Le briefing du matin reprenait le « demain » du bilan écrit la veille et parlait de « demain matin » au lieu de « ce matin ». **Première correction (13/10)** : une consigne au modèle (« les mots relatifs du bilan sont décalés d'un jour »), notée non vérifiée. Le bug est revenu le 04/11 sous la même forme. **Correction de fond** : une fonction convertit les mots relatifs en code, en un seul passage (« demain matin » → « ce matin », « demain » → « aujourd'hui », « après-demain » → « demain », « hier » → « avant-hier »…), seulement pour un bilan daté d'hier, sans toucher « ce matin-là » ni un mot qui contient ces mots. *Leçon : une consigne à un modèle n'est pas un correctif ; ce qui doit être exact se calcule dans le code.* Limites : un jour en toutes lettres, « cette semaine » et « le week-end » ne sont pas décalés. **Vérifié sur de vrais briefings le 08/11/2026** (textes lus dans Redis) : le « demain » du bilan du 07/11 est bien devenu « aujourd'hui ».

## 18. Un projet dit terminé mais jamais terminé dans Notion

Le bilan du 01/11 disait qu'un site était terminé ; il restait « En cours » (donc dans les projets du briefing et sur la carte Projets) et son jalon « À faire ». **Causes** : (1) une consigne héritée d'un autre projet demandait une preuve concrète pour valider un jalon ; (2) la recherche d'une idée cherchait le texte tel quel (« le site d'untel » ne trouvait pas « Site pour untel ») ; (3) un seul jalon lié était validé alors que les relances du bug 16 avaient pu en créer plusieurs ; (4) rien ne posait le statut « Manqué ». **Corrections** : « la parole de l'utilisateur suffit » ; `record_progress` accepte les projets terminés et les termine après l'écriture du bilan ; les noms sont rapprochés des projets en cours (les petits mots sont ignorés) ; tous les jalons ouverts liés sont validés ; au début du cron du matin, les jalons « À faire » en retard passent en « Manqué ». *Leçon : une règle de prompt héritée d'un autre projet peut bloquer en silence une fonction ; et un seul chemin d'écriture pour « terminer » évite deux comportements.* <span class="st st-prop">Non vérifié</span> Le comportement réel du modèle (il peut oublier le champ) ; au premier cron après le déploiement, tous les jalons déjà en retard passent en « Manqué » d'un coup.

## 19. Une voix qui prononce tous les accents de travers

Sur la voix entraînée, « prêt, à demain » était mal dit. **Cause** : l'exécutable de la synthèse lisait son entrée en cp1252 (l'encodage de Windows pour un tube) au lieu d'UTF-8 : « prêt » arrivait sous la forme « prÃªt ». L'essai de départ (« Bonjour, je suis Løra. ») ne contenait aucun accent. **Correction** : lecture en UTF-8, vérifiée aux phonèmes produits. *Leçon : tester avec une phrase qui contient les caractères difficiles, pas la plus simple ; et regarder ce que le programme a reçu, pas ce qu'on lui a envoyé.* L'utilisateur a jugé le rendu des accents satisfaisant après la correction.

## 20. Un tirage au sort qui ne tire jamais « patron »

Les phrases « Claude Code a terminé » reçoivent un surnom une fois sur deux, tiré parmi trois. **Cause**, trouvée à la relecture du 06/11 : le tirage utilisait les nanosecondes de l'horloge ; or l'horloge de Windows tourne à 100 ns, donc les nanosecondes sont toujours un multiple de 100 et, modulo 6, que des valeurs paires : « patron » ne sortait jamais. Le test ne pouvait pas le voir, il fournit lui-même le tirage. **Correction** : les microsecondes. *Leçon : un test qui injecte l'aléa ne teste pas la source de l'aléa.*

## 21. Une réinstallation qui efface ce qui avait été posé à la main

Après la réinstallation du 06/11, la reconnaissance vocale, la synthèse (1,9 Go) et plusieurs fichiers de réglage avaient disparu. **Cause** (déduite des dates, la désinstallation n'a pas été vue) : la case « supprimer les données de l'application » du désinstalleur efface les deux dossiers de l'application, et la voix avait été posée à la main dans l'un d'eux ; les deux dossiers ont été recréés dans la même seconde au premier lancement. **Correction** : les moteurs de voix sont déplacés dans un dossier voisin que la case ne touche pas, une confirmation (« non » par défaut, « non » aussi en mode silencieux) est ajoutée à la désinstallation, la liste des applications est restaurée à partir d'une transcription ; les identifiants de la musique, l'historique du temps d'écran et d'éventuels réglages de voix sont perdus. *Leçon : ce qui est installé à la main ne doit pas vivre dans un dossier qu'un désinstalleur a le droit de vider ; et une sauvegarde qui n'existe pas se découvre le jour où on en a besoin.* <span class="st st-prop">Non vérifié</span> Aucune vraie désinstallation essayée ; l'installeur n'est pas reconstruit, le désinstalleur actuel n'a pas la confirmation.

## 22. La connexion au compte de musique ouvre le dossier Documents

La phrase « connecte… » ouvrait l'explorateur de fichiers sur le dossier Documents au lieu du navigateur. **Cause** : l'adresse d'autorisation, longue et pleine de `%` et de `&`, était confiée à l'explorateur de fichiers, qui ne la comprenait pas. **Correction** : les adresses web passent par le gestionnaire de protocole de Windows ; seules les adresses propres au lecteur de musique restent sur l'explorateur. *Leçon : un programme qui ouvre « n'importe quoi » ne se comporte pas pareil selon ce qu'on lui passe ; tester avec la vraie adresse, pas une courte.* Trouvé à l'usage par l'utilisateur le 10/11 ; la reconnexion a été refaite.

## 23. « Mets » lance une musique quand on veut autre chose

« Mets la page en plein écran » lançait une musique, puis « mets en fenêtré » aussi. **Cause** : dans la reconnaissance de la musique, « mets X » signifie « joue X » : toute phrase qui commence par « mets » partait en recherche de morceau. **Correction** : « plein écran », « agrandis » et « en fenêtré » sont reconnus avant la musique. **Limite** : la touche F11 bascule sans connaître l'état de la fenêtre. *Leçon : un verbe courant est un piège ; l'ordre dans lequel le code essaie ses reconnaissances est une décision de conception, pas un détail.* Trouvé deux fois par l'utilisateur à l'usage (09 et 10/11).

## 24. Les gestes de la main : trois défauts qu'aucun test ne pouvait voir

(a) **Gauche et droite inversés** : j'avais supposé, d'après la documentation du modèle de suivi, que l'image brute de la webcam inversait les étiquettes « gauche » et « droite ». Sur de vraies mains, les deux sortaient fausses (et, par ricochet, paume et dos). *Cause* : la logique était cohérente avec l'hypothèse, pas avec le modèle ; les sept tests passaient. *Correction* : constat en réel, hypothèse retirée. (b) **Les deux poings fermés relançaient l'embrayage en boucle** : la paume d'un poing est instable et sortait « paume », donc après la validation les poings valaient « deux paumes » et armaient un nouveau geste. *Correction* : un poing n'arme rien, et il faut quitter la pose d'embrayage 0,6 s après une fin de geste. (c) **Le zoom à deux mains dos contre dos** : le dos de la main cache le poignet et l'orientation de la paume de profil est instable ; une mesure de la normale 3D (paumes face à face +0,9 à +1,0, dos face à face −0,4 à −0,9) a servi à comprendre, puis le geste a été remplacé par la distance entre les bouts de deux index pointés. *Leçon : mesurer sur de vraies mains avant de figer un seuil, et préférer le geste dont le signal reste visible.* Les deux premiers ont été trouvés en réel (le premier par moi sur des images réelles, le deuxième par l'utilisateur).

## 25. Un bilan non enregistré : ce n'était pas le quota

*Symptôme* : aucun bilan dans Notion pour les 12 et 13/11. *Hypothèse écartée par la mesure* : les en-têtes de Groq, lus avec une requête de 80 jetons, montrent 999 requêtes restantes sur 1 000 par jour et 7 920 jetons sur 8 000 par minute. *Vraies causes*, dans les journaux de production (qui ne remontent qu'environ une heure, donc quelques minutes seulement ici) : une requête qui échoue en 400 parce que le modèle appelle un outil absent de la liste envoyée, et une requête de 6 675 jetons qui, rejouée sur le second modèle puis sans recherche, dépasse les 8 000 jetons par minute. *Non prouvé* : laquelle de ces deux demandes était le bilan. *Leçon : la limite par minute se heurte à la taille d'une seule requête bien avant le quota du jour, et les journaux de l'offre gratuite s'effacent vite : relever les erreurs le jour même.* **Pistes sans paiement, non codées** : réduire l'historique, retirer un outil du groupe « bilan » du routage, plafonner la réponse, ne pas rejouer dans la même minute, réessayer avec tous les outils après un 400. En attendant : dicter le bilan en un seul message et attendre une minute avant une autre demande. <span class="st st-prop">Non vérifié</span> Les bilans des 12 et 13/11 sont à rattraper.

## 26. Une fonction écrite, testée et jamais appelée

Les réponses directes (météo, conversions, titres, compte à rebours, « mon PC est-il allumé ? ») étaient écrites, couvertes par 21 tests et déclarées validées par l'utilisateur le 10/11, mais **tout partait quand même au modèle, jetons compris**. **Cause**, trouvée à la revue finale du 14/11 : le module était importé dans `askLora` depuis le commit du 09/11, jamais appelé. Les tests visaient le module seul ; aucun ne passait par l'entrée réelle. **Correction** : l'appel est placé au tout début de `askLora` (avant le choix du modèle, sans clé requise), l'échange est écrit dans l'historique, et un test de régression garde le branchement. *Leçon : un test unitaire prouve qu'un module marche, pas qu'il est utilisé ; tester le chemin d'entrée, et ne pas confondre « validé à l'usage » avec « le chemin prévu a tourné » (un résultat correct peut venir du chemin de repli).* <span class="st st-prop">Non vérifié</span> Jamais essayé en réel.

## 27. Une caméra qui reste allumée

Si la fenêtre de la caméra était démontée pendant le chargement (autorisation, modèle), le nettoyage passait avant l'ouverture du flux : le flux n'était jamais fermé et le voyant restait allumé. **Correction** : le flux et le détecteur sont relâchés par une même fonction, aussi dans ce cas. *Leçon : un nettoyage qui s'exécute avant la ressource qu'il doit libérer ne libère rien ; pour une caméra, c'est aussi une question de confiance.* Trouvé à la relecture, pas à l'usage.

---

@@ page cloture-v1 | I | 16 | Clôture de la V1 | Ce qui est livré, ce qui a été vérifié le 09/10/2026, l'échec connu et ce qui ne l'est pas.

La V1 a été **déclarée terminée par l'utilisateur le 09/10/2026**, après trois semaines et 73 commits. La V2 a démarré le 12/10/2026 par des étapes de finition (voir « Depuis la clôture » en bas de ce chapitre). Les sections ci-dessous décrivent l'état **au 09/10/2026**, sauf mention contraire.

## Ce qui est livré

- **Backend en production** : webhook Telegram, worker, crons du matin et du soir (briefings rédigés par Løra), endpoint du dashboard ; 13 outils IA, une seule personnalité, pause de plusieurs jours.
- **Application de bureau** : intro en particules puis dashboard en verre, temps d'écran réel, démarrage avec Windows, zone de notification, installeur NSIS, barre de titre aux couleurs du fond.
- **Qualité** : 286 tests, deux passes d'optimisation mesurées, documentation à jour.

## Ce qui a été fait le 09/10/2026

Le ruban à 60 images/s et la barre de titre Windows 11 en `#2757D0` (PR 8), les surnoms variés (PR 9), l'installeur reconstruit et lancé par l'utilisateur (rien d'anormal constaté), et un déploiement de production après la dernière fusion (Vercel « Ready », constaté environ 3 minutes après la fusion).

## Vérifié à la clôture

| Vérification | Résultat |
| --- | --- |
| `npm test` (unitaires, intégration, régression) | <span class="st st-ok">Fait</span> 263 tests passent (22 fichiers, moins d'une seconde) |
| `tsc --noEmit` | <span class="st st-ok">Fait</span> sans erreur |
| `npm run test:acceptance` | <span class="st st-repl">Échec connu</span> 18 scénarios sur 20 passent |
| Tests Rust | <span class="st st-prop">Non vérifié</span> non rejoués ce jour-là |

## L'échec connu

Les deux tests de cadence de `tests/acceptance/performance.test.ts` figent l'ancienne cadence de 30 images/s alors que `REST_FPS` est passé à 60. « Au repos, le rendu est plafonné à environ 30 images par seconde » compte 125 images en 2 s pour moins de 75 attendues ; « pendant l'intro, pleine cadence… » dépasse son délai de 60 s. **Le code n'est pas en cause, ce sont les attentes qui sont périmées** : à corriger en lisant `REST_FPS` plutôt qu'une valeur en dur.

## Non vérifié à la clôture

Listé comme restant avant la déclaration ; sa vérification n'est pas consignée.

- <span class="st st-prop">Non vérifié</span> Démarrage automatique réel au lancement de session, menu de la zone de notification, reprise du rendu après un clic sur l'icône, suivi du temps d'écran et clics sur les blocs dans l'application installée.
- <span class="st st-prop">Non vérifié</span> `npm run test:live` : les écritures Notion avec le client 4, jamais vérifiées contre le vrai Notion.
- <span class="st st-prop">Non vérifié</span> Une passe sécurité (`/security-review`).
- <span class="st st-prop">Non vérifié</span> Quelques jours d'usage, et le premier texte réel du briefing du soir et du matin avec le nouveau code.
- <span class="st st-prop">Non vérifié</span> La cause du lag (le filtre de réfraction était suspecté, non mesuré). **Mesurée et traitée le 12/10/2026.**

## Optionnel, non fait

La catégorie Notion lue par le code (les logos restent devinés), le statut `Terminé` pour les idées (« Finis » compte aujourd'hui les idées archivées, ce qui n'est pas la même chose), un nom et un logo pour les applications inconnues du temps d'écran, le calcul des positions des particules sur le GPU.

## Limites assumées

- Les relecteurs expérimentés y regarderont en premier ; autant les dire : **à la clôture, la suite de tests n'était pas entièrement verte** (2 scénarios sur 20 en échec ; corrigé le 12/10/2026).
- **Un seul projet « En cours » à la fois** (à la clôture ; la limite n'existait déjà plus dans le code, et plusieurs projets en parallèle sont gérés depuis le 15/10/2026) : limite choisie pour la simplicité, documentée plutôt que contournée.
- **Dépendance à des services tiers gratuits** (Vercel, QStash, Upstash, Notion, Groq) avec leurs quotas.
- **Le texte produit par le modèle n'est pas testé automatiquement** : seul le contexte qui lui est envoyé l'est.
- **Pas de limitation de débit** au-delà du filtre `chat_id`.
- **Détection des icônes par mots-clés** : fragile par construction.

## Ce que la journée de clôture enseigne

- Un réglage validé sur une mesure (le CPU) peut être mauvais à l'usage (la fluidité) ; et un test qui recopie la valeur du réglage plutôt que de la lire échoue dès qu'on le change.
- Une valeur « par défaut » écrite dans un prompt tend à se retrouver partout dans les réponses.
- Déclarer « terminé » n'efface pas la liste de ce qu'on n'a pas vérifié : la clôture l'inscrit au lieu de l'omettre.

## Depuis la clôture (12/10 → 16/11/2026)

Une pull request par étape (PR #12 à #52 : étapes, correctifs et documentation ; une 53e est ouverte), faites du plus simple au plus dur. Le détail est dans [Avancement de la V2](#avancement-v2), [La voix de Løra](#voix), [Les commandes de Løra](#commandes-vocales) et [Les gestes de la main](#gestes-main) :

| Étape | Ce qui a changé | Statut |
| --- | --- | --- |
| 0 | Les 2 tests de cadence lisent `REST_FPS` ; l'ordre des étapes de la V2 est fixé | <span class="st st-ok">Fait</span> |
| 1 | Statut `Terminé` pour les idées ; catégorie Notion lue et écrite par le code ; noms et logos pour les applications inconnues du temps d'écran | <span class="st st-ok">Fait</span> ; écriture réelle vérifiée sur Notion le 13/10/2026 |
| 2 | `/api/dashboard` expose la pause en cours ou à venir ; le bloc briefing l'annonce (pas de persistance Notion, Redis suffit) | <span class="st st-ok">Fait</span> ; <span class="st st-prop">Non vérifié</span> sur le vrai Redis |
| 3 | Cause du lag mesurée, réfraction du verre coupée | <span class="st st-ok">Fait</span> |
| 4 | Ouvrir et fermer une application nommée depuis une barre `Ctrl+K` | <span class="st st-ok">Fait</span> validé par l'utilisateur |
| 5 | Recherche web de Løra sur Telegram | <span class="st st-ok">Fait</span> sur le vrai modèle ; <span class="st st-prop">Non vérifié</span> sur Telegram |
| 6 | Volume, musique, batterie de la souris | <span class="st st-ok">Fait</span> essayés sur la vraie machine |
| 7 | Plusieurs projets en parallèle | <span class="st st-ok">Fait</span> confirmé par l'utilisateur |
| 8 | Moteur de protocoles, premier protocole | <span class="st st-ok">Fait</span> essayé en réel par l'utilisateur le 05/11/2026 ; **en pause** (« bientôt fait », 13/11) ; deux autres protocoles reportés |
| 9 | La voix : synthèse entraînée sur des enregistrements consentis, reconnaissance locale, mot d'activation | <span class="st st-ok">Fait</span> essayés avec la vraie voix le 05/11/2026, validés par l'utilisateur le 08/11/2026 |
| 10 | Widget de bureau, ruban synchronisé sur la vraie voix | <span class="st st-ok">Fait</span> validés par l'utilisateur le 08/11/2026 |
| 11 | « Qu'est-ce que je regarde ? » | <span class="st st-repl">Retiré</span> abandonnée le 08/11/2026, code retiré |
| 12 | Gestes de la main : détecteur local, mode manuel Blender (rotate, zoom, scaling), caméra à la voix | <span class="st st-ok">Fait</span> cochée le 13/11/2026 sur décision de l'utilisateur, essayée par lui dans Blender (sa parole) ; seuils et gains à régler à l'usage |
| 13 | Élargir les commandes vocales : `api/ask`, commandes de contrôle, modes de travail, Claude Code, près de vingt « broutilles », réponses sans modèle, outils Notion | <span class="st st-ok">Fait</span> validées par l'utilisateur (08 et 10/11/2026, sur sa parole) ; essayées avant contre des simulations |
| Revue | Revue finale (14/11/2026) : réponses directes branchées, caméra libérée au démontage, avertissements Rust corrigés, paquet principal −18 % | <span class="st st-ok">Fait</span> 547 tests rapides, 155 Rust, 23 scénarios navigateur, 0 vulnérabilité |

Au 10/11/2026, l'utilisateur déclare la liste « non vérifié » de la V1 soldée (« la V1 est finie, tout est bon », pause comprise) et dit qu'il ne reste que l'étape 12 et l'étape 8 ; **au 13/11 il déclare** Spotify reconnecté, les « broutilles » et les deux Extras faits, la voix de Piper correcte, l'étape 8 en pause (« bientôt fait »), les tests d'acceptation à lancer « bientôt » et l'installeur à reconstruire avec le backend à redéployer **à la fin de la session, par lui** ; l'étape 12 est cochée le même jour. Je n'ai rien rejoué de ces déclarations. La passe sécurité a été faite le 08/11 (rien de bloquant), de vrais textes de briefings ont été lus le 08/11, et `npm run test:live` avait été relancé le 13/10/2026 (48/48 sur le vrai Notion, Redis et Groq). La revue finale du 14/11 a pourtant trouvé que les réponses directes n'étaient pas branchées (bug [26](#bugs)) : une validation sur parole ne remplace pas la lecture du chemin de code.
