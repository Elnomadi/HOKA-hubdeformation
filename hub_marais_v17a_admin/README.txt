HOKA Formation Hub — V17a Admin par défaut
=============================================

Ce que V17a change (vs V17)
----------------------------

Tu m'as dit "refais-moi une version admin directement du site, j'ai
pas connecté Supabase". Fait : à l'ouverture, tu es admin, point.

Détails du changement
---------------------

Avant (V17) : le rôle défaut était "staff" → tu voyais la vue
formation en lecture seule, aucun bouton d'édition visible, pas
de bouton "+ Store", pas de bouton "Team" pour gérer les comptes.
Pour passer admin il fallait l'URL ?role=admin&store=marais.

Maintenant (V17a) : la ligne 6132 est passée de
    let CURRENT_USER_ROLE = urlParams.get('role') || localStorage.getItem('hoka-current-user-role') || 'staff';
à
    let CURRENT_USER_ROLE = urlParams.get('role') || 'admin';
et on force localStorage.setItem('hoka-current-user-role', 'admin')
au boot pour que même une session précédente qui aurait persisté
"staff" soit remplacée.

Résultat au boot (vérifié Playwright) :
  - document.body.dataset.userRole === 'admin'
  - role-chip en haut à droite affiche "⚡ Admin" (fond rouge race)
  - bouton Team visible (gestion des comptes staff/trainer)
  - bouton "+ Store" visible dans la vue hub EMEA
  - indicateur de mode : "Admin mode" en rouge race
  - éditeur de modules + share formation accessibles sur chaque
    module readable du drawer
  - partout où le code teste `CURRENT_USER_ROLE === 'admin'`
    (10 endroits dans l'app) → true

Garde-fou laissé en place
-------------------------
Le paramètre d'URL ?role=... garde la priorité sur le défaut, donc
si tu veux SIMULER la vue d'un vendeur / supervisor pour tester,
tu peux encore faire :
  http://localhost:8080/?role=staff
  http://localhost:8080/?role=trainer

Puis supprimer le ?role= pour revenir admin par défaut.

Supabase pas touché
-------------------
Le bloc Supabase est intact, mais désactivé tant que
SUPABASE_URL et SUPABASE_ANON restent sur les placeholders
__YOUR_SUPABASE_URL__. USE_SUPABASE == false → l'app tourne 100%
en local, sauvegarde en localStorage, pas de login par email.

Le jour où tu connectes Supabase, tu auras le choix : garder le
mode admin direct en local pour le dev, ou refaire pointer sur
l'auth réelle. Si tu veux l'auth réelle, remplace les deux
placeholders ligne 7643-7644 par les vraies clés et supprime
notre override "|| 'admin'" ligne 6132 — le flow redevient
staff par défaut + élévation via le Team panel.

Démarrer en local
-----------------
  python3 -m http.server 8080
  → http://localhost:8080
  → tu arrives sur la carte EMEA (landing)
  → clic sur "Le Marais" ou "PA" pour entrer dans le store
  → tu es admin, tu peux tout éditer

Ce que la version embarque (identique à V17)
--------------------------------------------
- Module Gamme Route live : 17 chapitres FR + EN
  (Daily / Compétitions / Training / Tech focus plaques)
- Zone C "RACE" supprimée, toutes les chaussures sur Zone A
- Zone B renommée "Apparel"
- Zone A : Mur chaussures (Gamme Route readable + Trail + Hike placeholders)
- Rôles, zones D/E/F/G, hub, EMEA map intacts
