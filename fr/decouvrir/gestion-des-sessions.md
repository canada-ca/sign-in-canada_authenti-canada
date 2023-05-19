---
layout: default
ref: discover-session
title: Gestion des séances
lang: fr
---
# Gestion des séances

La plateforme Authenti-Canada fonctionne comme une autorité de gestion de
session qui coordonne de façon centralisée les sessions des utilisateurs dans
plusieurs applications de parties de confiance (PC). La plateforme comporte deux
caractéristiques clés qui la soutiennent : authentification unique et
déconnexion unique.

## Authentification unique

Une fois qu’un utilisateur a utilisé avec succès un justificatif quelconque ou
un service d’identité numérique de confiance pour s’authentifier auprès d’une
PC, Authenti-Canada peut lui permettre de s’authentifier en silence auprès
d’autres PC, sous réserve d’une période d’attente d’authentification unique. La
plateforme y parvient en créant sa propre session avec l’utilisateur lors de sa
première connexion, puis en ne les invitant pas à se réauthentifier lorsqu’ils
se connectent à d’autres PC pendant un délai d’inactivité spécifiée.

La fenêtre d’attente de l’authentification unique démarre chaque fois qu’un
utilisateur saisit son mot de passe correct à un fournisseur de justificatif
d’identité (FJI) et se termine après une période spécifiée. Par défaut, il
s’agit de 20 minutes, mais les parties qui se connectent au Canada peuvent
modifier ce délai si elles le souhaitent. Il est toutefois important de noter
que la fenêtre d’authentification unique des services d’authentifiant CléGC et
le service de courtier de justificatifs d’identité, qui est également de 20
minutes, ne peut pas être modifié, elle ne peut être désactivée (appelée
authentification forcée). Dans toute situation où le délai d’attente « codé en
dur » pourrait entrer en conflit avec le délai d’attente préféré d’une partie de
confiance, la plateforme Authenti-Canada synchronisera automatiquement les deux
et résoudra le conflit.

_Notez que pour que l’authentification unique fonctionne, l’utilisateur doit
utiliser les mêmes informations d’identification ou la même identité numérique
de confiance pour accéder à tous les services gouvernementaux numériques. À
l’heure actuelle, cela n’est pas possible puisque tous les services du
gouvernement du Canada n’acceptent pas les mêmes justificatifs. Par exemple,
l’accès aux services fiscaux en ligne fournis par l’Agenge du revenu du Canada
n’est actuellement pas possible à l’aide d’un justificatif d’identité de CléGC.
Par conséquent, les utilisateurs qui ont choisi d’utiliser un justificatif de
CléGC ne peuvent pas s’inscrire à [Mon dossier pour les particuliers de
l’ARC](https://www.canada.ca/fr/agence-revenu/services/services-electroniques/services-electroniques-particuliers/dossier-particuliers.html)._

## Déconnexion unique

Lorsqu’un utilisateur a visité plusieurs sites Web (applications de partie de
confiance) au cours de la même session, la plateforme Authenti-Canada a la
capacité de coordonner de manière centralisée une déconnexion unique. Avec une
déconnexion unique, l’utilisateur n’a qu’à cliquer sur un bouton de déconnexion
sur le site qu’il visite actuellement, et Authenti-Canada les déconnectera
ensuite de tous les autres sites qu’il a visités pendant sa session.

La propagation de la déconnexion sur tous ces sites peut utiliser le mécanisme
de déconnexion [OpenID Connect
Front-Channel](https://openid.net/specs/openid-connect-frontchannel-1_0.html),
le mécanisme de déconnexion [OpenID Connect
Back-Channel](https://openid.net/specs/openid-connect-backchannel-1_0.html) ou
le profil de déconnexion SAML à l’aide de
[liaisons](https://www.oasis-open.org/committees/download.php/56779/sstc-saml-bindings-errata-2.0-wd-06.pdf)
de canal frontal (HTTP-Redirect) ou de canal secondaire (SOAP). La plateforme
Authenti-Canada appuie tous ces éléments.

Les protocoles OpenID Connect (OIDC) et SAML suivent la même séquence d’événements de base :

1. Comme un utilisateur visite plusieurs sites Web (applications) au cours de la
   même session, la plateforme Authenti-Canada permet de suivre les sites qu’il
   a visités.
2. Lorsque l’utilisateur clique sur un bouton de déconnexion sur l’un des sites
   qu’il visite, l’application Web de ce site envoie une demande de déconnexion
   à la plateforme Authenti-Canada.
3. Authenti-Canada envoie ensuite une demande de déconnexion à tous les autres
   sites que l’utilisateur a visités, afin qu’il puisse déconnecter
   l’utilisateur.

## Transition et coexistence

Il y aura une période de transition prolongée à mesure que les applications PC
se connecteront à Authenti-Canada et se déconnecteront progressivement un à un
de la fédération des justificatifs du gouvernement du Canada (FJGC) CléGC et
Services de courtier de justificatifs d’identité. Pendant cette période de
transition, la plateforme Authenti-Canada servira d’autorité de gestion de
session pour les applications qui y sont connectées, tandis que les services
CléGC et services de courtier de justificatifs d’identité serviront d’autorité
de gestion de session pour les applications qui y sont connectées.

Cela crée une obligation pour Authenti-Canada de coordonner une déconnexion
unique avec CléGC et les services de courtier de justificatifs d’identité
(SCJI), ainsi qu’avec ses propres PC. La plateforme y parvient en appuyant le
profil de déconnexion unique SAML dans le cadre de son intégration, en tant que
fournisseur de justificatifs d’identité proxy SAML, avec CléGC et services de
courtier de justificatif d’identité. La plateforme Authenti-Canada est en mesure
de coordonner une déconnexion unique avec la fédération des justificatifs du
gouvernement du Canada (FJGC) dans les deux sens. C’est-à-dire, peu importe si
l’utilisateur se déconnecte d’abord d’Authenti-Canada ou s’il se déconnecte
d’abord de la FJGC.

### Scénario 1 : Un utilisateur se déconnecte d’abord d’Authenti-Canada

L’exemple suivant illustre comment une déconnexion unique est réalisée dans le
cas où l’utilisateur clique sur un bouton de déconnexion sur un site de la
partie de confiance qui est connecté à Authenti-Canada.

```plantuml!
skinparam sequenceMessageAlign direction
skinparam responseMessageBelowArrow true
skinparam titleBorderRoundCorner 15
skinparam titleBorderThickness 2
skinparam titleBorderColor red
skinparam titleBackgroundColor Aqua-CadetBlue
title Déconnexion initiée par une partie utilisatrice de Authenti-Canada
Actor Navigateur as Browser
Participant "Partie de confiance" as RP
box "Authenti-Canada"
Participant "Fournisseur OpenID" as OP
Participant "Cadre d'acceptation" as Passport
end box
Participant "SAML Fournisseur\nd'identité SAML (FJI)" as IDP
Participant "Autre(s) PR\nd'Authenti-Canada" as Other_SIC
Participant "Autre(s) PR\ndu FJGC" as Other_GCCF
Browser -> RP : Se déconnecter
RP -> RP : local Se déconnecter
RP --> Browser : Redirection vers le point de terminaison end_session
Browser -> OP : Demande de déconnexion OIDC au point de terminaison end_session
group Par
   OP ->> Other_SIC : Demande(s) de déconnexion du canal de retour OIDC
   Other_SIC -> Other_SIC : local Se déconnecter
   OP <<-- Other_SIC : réponses de déconnexion
end
Browser <-- OP : Page de propagation de déconnexion avec iframes
group Par
   Browser ->> Other_SIC : Demande(s) de déconnexion du canal frontal OIDC
   Other_SIC -> Other_SIC : local Se déconnecter
   Browser <<-- Other_SIC : Réponse(s) HTTP
   Browser ->> Passport : Demande de déconnexion
   Browser <<-- Passport : Demande de déconnexion SAML
   Browser ->> IDP : Demande de déconnexion SAMLt
   loop
      IDP -> Other_GCCF : Demande de déconnexion SAML
      Other_GCCF -> Other_GCCF : local Se déconnecter
      Other_GCCF --> IDP : Réponse de déconnexion SAML
   end
   note right of IDP : Déconnexion du FJGC terminée
   Browser <<-- IDP : Réponse de déconnexion SAML
end
alt une erreur s'est produite
   Browser -> OP : Charger la page d'avertissement de déconnexion
   Browser <-- RP : Page d'avertissement de déconnexion
else aucune erreur n'est survenue
   Browser -> RP : Charger la page de déconnexion terminée
   Browser <-- RP : Page de déconnexion terminée
end
note right of Browser
   Authenti-Canada
   déconnexion terminée
end note
```

Le scénario commence lorsqu’un utilisateur clique sur un bouton de déconnexion
sur le site de la partie de confiance qu’il utilise. Lorsque cela se produit :

1. L’application de la partie de confiance déconnecte l’utilisateur localement
   d’abord, puis redirige le navigateur vers le point de terminaison OpenID
   Connect de session des plateformes d’acceptation.
2. Si des parties de confiance qui participent à la session appuient la
   déconnexion en mode canal arrière de Connexion OpenID, la plateforme
   d’acceptation envoie d’abord un jeton de déconnexion à tous, à travers un
   POST HTTP à leurs URI de déconnexion en mode canal arrière enregistré pour
   déclencher les actions de déconnexion de ces PC. S’il y a plusieurs parties
   de confiance, ces demandes sont toutes envoyées simultanément, en parallèle.
3. Une fois la déconnexion canal arrière terminée, la plateforme d’acceptation
   retourne une page de propagation de déconnexion
   [HTML](https://html.spec.whatwg.org/multipage/) au navigateur. Cette page
   contient un certain nombre d’éléments
   [iframe](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element).
   L’attribut src (source) de tous les éléments, sauf un, pointe vers l’URI de
   déconnexion du canal avant de l’un des sites de partie de confiance
   d’Authenti-Canada où l’utilisateur s’est connecté pendant sa session en
   cours. Dans le diagramme, ces autres sites sont désignés comme « autres PC de
   l’OIDC ». L’attribut `src` de l’_iframe_ final pointe vers un point terminal URI
   du Cadre d’acceptation de la plateforme d’acceptation.
4. Dès réception de la page de propagation, le navigateur commence à charger en
   parallèle l’intégralité de la valeur des __iframe_s, en envoyant de manière
   asynchrone une requête HTTP GET à l’URI `src` de chaque _iframe_.
5. Lorsque les autres parties de confiance reçoivent ces demandes, elles
   déconnectent l’utilisateur localement, puis retournent une réponse HTTP au
   navigateur.
6. Lorsque le cadre d’acceptation de la plateforme d’acceptation reçoit sa
   demande, il crée un message SAML de demande de déconnexion, puis redirige le
   navigateur avec ce message (dans le cadre _iframe_) vers l’ancien PSC (CléGC ou
   FJI) avec lequel l’utilisateur a choisi de se connecter.
7. Le FSI déconnecte ensuite l’utilisateur de ses informations d’identification.
   Si l’utilisateur a visité des sites qui sont encore reliés directement au FSI
   (au lieu d’Authenti-Canada), il propagera également la déconnexion à tous ces
   sites, progressivement, une à une.
8. Le FSI redirige ensuite le navigateur (dans l’_iframe_) vers le Cadre
   d’acceptation de la plateforme d’acceptation avec un message SAML
   `LogoutResponse`. Le Cadre d’acceptation traite ce message, puis envoie une
   réponse HTTP au navigateur.
9. La page de propagation de déconnexion comporte un gestionnaire d’événements
   d’erreur global qui sera déclenché si un type d’erreur se produit alors que
   tous les _iframes_ propagent la déconnexion vers divers sites de manière
   asynchrone.
10. Si l’une des _iframes_ échoue ou s’interrompt, si la réponse de déconnexion
    SAML du FSI indique une erreur, alors le gestionnaire onerrorevent redirige
    le navigateur vers une page d’erreur, avertissant l’utilisateur qu’il ne
    peut pas être complètement déconnecté et lui recommandant de fermer son
    navigateur.
11. Si toutes les _iframes_ propagent avec succès la déconnexion vers leur site
    cible, le navigateur est redirigé vers la page d’accueil de déconnexion du
    site où l’utilisateur a initialement cliqué sur « Déconnexion » (l’URI
    post_déconnexion_redirect_uri de la partie de confiance).

### Scénario 2 : Un utilisateur se déconnecte d’abord du FJGC

L’exemple suivant illustre comment une déconnexion unique est réalisée dans le
cas où l’utilisateur clique sur un bouton de déconnexion sur un site de la
partie de confiance qui est toujours connecté à l’un des services de
justificatif d’identification de la FJGC (CléGC ou service de courtier de
justificatif d’identité).

```plantuml!
skinparam sequenceMessageAlign direction
skinparam responseMessageBelowArrow true
skinparam titleBorderRoundCorner 15
skinparam titleBorderThickness 2
skinparam titleBorderColor red
skinparam titleBackgroundColor Aqua-CadetBlue
title Déconnexion initiée par une partie de confiance FJGC
Actor Navigateur as Browser
Participant "Partie de confiance SAML" as RP
Participant "SAML Fournisseur\nd'identité SAML (FJI)" as IDP
Participant "Autre(s) PR\ndu FJGC" as Other_GCCF
box "Authenti-Canada"
Participant "Cadre d'acceptation" as Passport
Participant "Fournisseur OpenID" as OP
end box
Participant "Autre(s) PR\nd'Authenti-Canada" as Other_SIC
Browser -> RP : Se déconnecter
RP -> RP : local Se déconnecter
Browser <-- RP : Redirection HTTP\navec demande de déconnexion SAML
Browser -> IDP : Demande de déconnexion SAML
loop
   IDP -> Other_GCCF : Demande de déconnexion SAML
   Other_GCCF -> Other_GCCF : local Se déconnecter
   Other_GCCF --> IDP : SAML Réponse de déconnexion
end
note right of IDP : Déconnexion FJGC terminée
Browser <-- IDP  : Redirection HTTP vers Authenti-Canada
Browser -> Passport : Demande de déconnexion SAML
Browser <-- Passport : Redirection HTTP vers le point de terminaison end_session
Browser -> OP : Demande de déconnexion OIDC au point de terminaison end_session
group Par
   OP ->> Other_SIC : demande(s) de déconnexion
   Other_SIC -> Other_SIC : local Se déconnecter
   OP <<-- Other_SIC : réponse(s) de déconnexion
end
Browser <-- OP : Page de propagation de déconnexion avec iframes
group Par
   Browser ->> Other_SIC : Demande(s) de déconnexion du canal frontal OIDC
   Other_SIC -> Other_SIC : local Se déconnecter
   Browser <<-- Other_SIC : Réponse(s) HTTP
end
Browser -> Passport : Résultat de déconnexion (Succès / Échec)
Browser <-- Passport : Réponse de déconnexion SAML
note right of Browser
   Authenti-Canada
   Déconnexion terminée
end note
Browser -> IDP : Réponse de déconnexion SAML
alt échec de déconnexion
   Browser <-- IDP : Réponse de déconnexion SAML avec code "Déconnexion partielle"
   Browser -> RP : Réponse de déconnexion SAML
   Browser <-- RP : Page d'avertissement de déconnexion
else déconnexion réussie
   Browser <-- IDP : Réponse de déconnexion SAML avec code "succès"
   Browser -> RP : Réponse de déconnexion SAML
   Browser <-- RP : Page de déconnexion terminée
end
```

Le scénario commence lorsqu’un utilisateur clique sur un bouton de déconnexion
sur le site de la partie de confiance qu’il utilise. Lorsque cela se produit :

1. L’application de la partie de confiance déconnecte l’utilisateur localement
   d’abord, puis redirige le navigateur vers le point terminal de déconnexion
   unique SAML du FSI avec un message SAML `LogoutRequest`.
2. Si l’utilisateur a visité d’autres sites qui sont toujours connectés
   directement au FSI (au lieu de l’Authenti-Canada), il propagera d’abord la
   déconnexion de tous ces sites à l’aide de la déconnexion SAML dans le canal
   arrière (SOAP). Cela se fait un à un, pendant que l’utilisateur attend,
   ensuite il sera alors *complètement déconnecté de la FJGC*.
3. Pour déconnecter l’utilisateur de l’Authenti-Canada, le FSI de la FJGC
   redirige ensuite le navigateur vers le Cadre d’acceptation de
   l’Authenti-Canada avec un message SAML `LogoutRequest`.
4. Le cadre d’acceptation crée ensuite une demande de déconnexion OpenID Connect
   et redirige le navigateur vers le point terminal de session du fournisseur
   d’OpenID de la plateforme d’acceptation.
5. Si des parties de confiance qui participer à la session de la plateforme
   d’acceptation prennent en charge la déconnexion, la plateforme d’acceptation
   envoie d’abord un jeton de déconnexion à tous, via un POST HTTP à son URI de
   déconnexion en mode canal arrière enregistré. Cela déclenche les actions de
   déconnexion de chacun de ces PC. S’il y a plusieurs parties de confiance, ces
   demandes sont toutes envoyées simultanément, en parallèle.
6. Une fois l’ouverture de session dans le canal arrière terminée, la plateforme
   d’acceptation retourne une page de propagation de déconnexion
   [HTML](https://html.spec.whatwg.org/multipage/). Cette page contient un
   certain nombre d’éléments
   [iframe](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element).
   L’attribut `src` (source) de chaque élément iframe pointe vers l’URI de
   déconnexion du canal avant de l’un des sites de la partie de confiance où
   l’utilisateur s’est connecté pendant sa session en cours. Dans le diagramme,
   ces autres sites sont désignés comme « autres PC de l’OIDC ». L’attribut src
   de l’iframe final pointe vers un point terminal URI du Cadre d’acceptation de
   la plateforme d’acceptation.
7. Dès réception de la page de propagation, le navigateur commence à charger en
   parallèle l’intégralité de la valeur de _iframe_s, en envoyant de manière
   asynchrone une requête HTTP GET à l’URI src de chaque iframe.
8. Lorsque les autres parties de confiance reçoivent ces demandes, elles
   déconnectent l’utilisateur localement, puis retournent une réponse HTTP au
   navigateur.
9. La page de propagation de déconnexion comporte un gestionnaire d’événements
   d’erreur global qui sera déclenché si un type d’erreur se produit alors que
   tous les _iframe_s propagent la déconnexion vers divers sites de manière
   asynchrone.
10. Si l’une des iframes échoue ou s’interrompt, le gestionnaire onerrorevent
    redirige le navigateur vers le Cadre d’acceptation avec une requête HTTP
    indiquant un problème de déconnexion.
    1. Le Cadre d’acceptation redirigera ensuite le navigateur vers le FSI avec
      un message SAML LogoutResponse portant un code de statut urn : oasis :
      names : tc : SAML : 2,0 : status : Responder.
    2. La FSI redirigera ensuite le navigateur vers sa propre PC avec un message
      LogoutResponse portant un code de statut de haut niveau
      `urn:oasis:names:tc:SAML:2.0:status:Success` et un code de statut de
      second niveau `urn:oasis:names:tc:SAML:2.0:status:PartialLogout`.
    3. Cela entraînera l’affichage de la page d’erreur PC du SAML, avertissant
      l’utilisateur qu’il n’est peut-être pas complètement déconnecté et lui
      recommandant de fermer son navigateur.
11. Si toutes les _iframes_ propagent correctement la déconnexion vers leur site
    cible, le code JavaScript redirige le navigateur vers le Cadre d’acceptation
    avec une demande HTTP indiquant une déconnexion réussie.
    1. Le Cadre d’acceptation redirigera ensuite le navigateur vers le FSI
       avec un message SAML `LogoutResponse` portant un code de statut
       `urn:oasis:names:tc:SAML:2.0:status:Success`.
    2. Le FJI redirigera ensuite le navigateur vers sa propre PC avec un message
       `LogoutResponse` portant un code de statut de haut niveau
       `urn:oasis:names:tc:SAML:2.0:status:Success` et aucun code de statut de
       niveau supérieur.
    3. Cela entraînera l’affichage de la page de déconnexion ou d’accueil
       habituelle de SAML PC.
