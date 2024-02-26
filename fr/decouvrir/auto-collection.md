---
layout: default
ref: discover-auto-collection
title: Collection automatique de l’identificateur par paire
lang: fr
---
# Collection automatique de l’identificateur par paire

L’un des principaux objectifs de la plateforme d’acceptation d’Authenti-Canada
est de faciliter le remplacement des anciens systèmes de fournisseurs de
services de justificatifs d’identité, en particulier les anciens systèmes CléGC
et Service de courtier de justificatifs d’identité qui sont utilisés depuis
2011.

L’une des principales conditions préalables à la mise hors service ou au
remplacement de ces anciens systèmes est de déplacer les mappages
d’identificateur par paire actuellement détenus par ces services plus anciens
vers la plateforme d’acceptation, de sorte que les inscriptions des utilisateurs
avec les parties de confiance ne soient pas touchées lorsque les parties de
confiance passent à la plateforme Authenti-Canada.

La plateforme d’acceptation Authenti-Canada met en œuvre une fonctionnalité qui
permet de copier automatiquement les mappages des identifiants par l’utilisateur
au moyen d’une paire à partir d’un service de justificatif d’identité vers la
plateforme d’acceptation la première fois qu’ils sont utilisés. Ils sont ensuite
stockés par la plateforme d’acceptation pour une utilisation future, de sorte
que le mappage stocké par le service de justificatif d’identité n’est plus
nécessaire. Cela réduit les risques et les efforts requis pour déplacer ces
données. Au fil du temps, les données cartographiques des utilisateurs actifs
sont progressivement « recueillies » par la plateforme d’acceptation, ce qui
réduit la nécessité de copier régulièrement les données manuellement au moyen
d’une sorte de « transfert en vrac ».

## Fonctionnement

```plantuml!
skinparam sequenceMessageAlign direction
skinparam responseMessageBelowArrow true
skinparam titleBorderRoundCorner 15
skinparam titleBorderThickness 2
skinparam titleBorderColor red
skinparam titleBackgroundColor Aqua-CadetBlue
title Collecte automatique d'un identifiant par paire
Participant "Partie de Confiance" as RP
Participant "Authenti-Canada" as SIC
Participant "SAML IDP (FJGC FSI)" as IDP
RP -> SIC : demande d’authentification
SIC -> IDP : demande d’authentification SAML
SIC <-- IDP : réponse d'authentification avec assertion SAML
SIC -> SIC : recherche l’utilisateur
alt identifiant par paire demandé introuvable
   SIC -> IDP : demande d’authentification SAML
   note right : pour l'identifiant demandé
   SIC <-- IDP : réponse d'authentification
   alt la réponse contient une assertion SAML
      SIC -> SIC : stocker l'identifiant par paire collecté
   else la réponse ne contient pas d'assertion SAML
      SIC -> SIC : créer un nouvel identifiant par paire
   end
end
RP <-- SIC : réponse d'authentification
note right : avec identifiant par paire
```

La plateforme d’acceptation effectue la collecte automatique des mappages
d’identificateurs par paire dans le cadre du processus normal de connexion
d’utilisateur, en envoyant une deuxième demande d’authentification SAML aux
fournisseurs de services de justificatifs, le cas échéant.

1.	Le processus commence lorsqu’une partie de confiance envoie une demande
d’authentification à la plateforme d’acceptation, à l’aide d’OpenID Connect ou
de SAML.

2.	La plateforme d’acceptation envoie ensuite une demande d’authentification au
FSI en son propre nom. Elle demande l’identificateur par paire que le FSI a
créée pour Authenti-Canada.

3.	À la réception de l’assertion SAML du FSI, la plateforme d’acceptation
recherche l’utilisateur dans son propre référentiel de profils d’utilisateur.

4.	La plateforme d’acceptation vérifie ensuite si elle dispose déjà d’un mappage
d’identificateur par paire pour l’utilisateur authentifié avec la partie de
confiance requérante. Dans l’affirmative, la collecte n’est pas requise, le flux
de connexion se termine normalement et la plateforme d’acceptation renvoie
l’identificateur par paire approprié à la partie de confiance (PC).

5.	Toutefois, si la plateforme d’acceptation ne possède pas de mappage
d’identificateur par paire pour l’utilisateur authentifié avec la partie de
confiance requérante, elle envoie une deuxième demande d’authentification au FSI
au nom de la partie de confiance.

6.	Si le FSI a un identificateur par pair existant pour l’utilisateur avec la PC
requérante, il retourne cet identificateur dans une assertion et la plateforme
Acceptantion l’ajoute à son propre référentiel utilisateur. À ce stade,
l’identificateur a été « collecté » par la plateforme d’acceptation et sera
utilisé chaque fois que l’utilisateur se connectera à cette PC à l’avenir
(conformément à l’étape 4 ci-dessus).

7.	Si le FSI ne possède pas d’identificateur par pair existant pour
l’utilisateur avec la PC requérante, il n’y a donc rien à « collecter », de
sorte que la plateforme d’acceptation crée un nouvel identificateur pour
l’utilisateur avec la PC requérante, et cet identificateur est utilisé pour
toutes les connexions futures.

## Détails techniques

Lors de l’envoi d’une deuxième demande d’authentification au FSI pour le compte
ou la partie de confiance, la plateforme d’acceptation utilise l’élément
`<NameIDPolicy>` du message SAML `<AuthnRequest>`. Plus précisément, il utilise les
deux attributs suivants de l’élément `<NameIDPolicy>` :

* La valeur de l’attribut `SPNameQualifier` est renseignée avec l’entité de
  l’ancien ID SAML de la partie de confiance requérante, pour indiquer que la
  plateforme d’acceptation demande l’identificateur par paire de PC au lieu du
  sien.

* La valeur de l’attribut `AllowCreate` est définie sur `"false"`, pour indiquer
  au FSI qu’il ne doit pas créer un nouvel identificateur par paire pour la PC
  si celui-ci n’en a pas déjà un.

Voici un exemple d’une demande d’authentification émise par la plateforme
d’acceptation en son nom propre :

```xml
<samlp:AuthnRequest xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="_f8e30829f6f60061c46e"
                    Version="2.0" IssueInstant="2021-05-05T17:17:02.583Z"
                    ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
                    Destination="https://clegc-gckey.gc.ca/j/SSORedirect/metaAlias/GCKey/idp"
                    AssertionConsumerServiceURL="https://auth.id.canada.ca/passport/auth/saml/gckey/callback">
   <saml:Issuer xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
      https://auth.id.canada.ca
   </saml:Issuer>
   <samlp:NameIDPolicy xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
                       Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"
                       AllowCreate="true"
                       SPNameQualifier="https://auth.id.canada.ca"/>
   <samlp:RequestedAuthnContext xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Comparison="exact">
      <saml:AuthnContextClassRef xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
         urn:gc-ca:cyber-auth:assurance:loa2
      </saml:AuthnContextClassRef>
   </samlp:RequestedAuthnContext>
</samlp:AuthnRequest>
```

Voici un exemple d’une demande d’authentification émise par la plateforme
d’acceptation en nom de la partie de confiance :

```xml
<samlp:AuthnRequest xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="_f8e30829f6f60061c46f"
                    Version="2.0" IssueInstant="2021-05-05T17:20:02.583Z"
                    ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
                    Destination="https://clegc-gckey.gc.ca/j/SSORedirect/metaAlias/GCKey/idp"
                    AssertionConsumerServiceURL="https://auth.id.canada.ca/passport/auth/saml/gckey/callback">
   <saml:Issuer xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
      https://auth.id.canada.ca
   </saml:Issuer>
   <samlp:NameIDPolicy xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
                       Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"
                       AllowCreate="false"
                       SPNameQualifier="https://relyingparty.department.gc.ca"/>
   <samlp:RequestedAuthnContext xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Comparison="exact">
      <saml:AuthnContextClassRef xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
         urn:gc-ca:cyber-auth:assurance:loa2
      </saml:AuthnContextClassRef>
   </samlp:RequestedAuthnContext>
</samlp:AuthnRequest>
```

## Foire aux questions

### QÀ quoi ressemble l’expérience de l’utilisateur? Doit-il saisir son mot de passe deux fois?

Ce processus tire parti des fonctionnalités d’authentification unique inhérentes
à SAML, de sorte que l’utilisateur n’aura pas à s’authentifier plus d’une fois,
si ce n’est du tout :

* Si l’utilisateur n’est pas encore connecté au FSI, ou s’il est connecté,
  mais que la fenêtre connexion unique (20 minutes avec le FSI de la FJGC) a
  expiré, il sera invité à s’authentifier après la première demande
  d’authentification SAML. Une fois qu’ils se sont connectés avec succès au FSI,
  la plateforme d’acceptation envoie ensuite la deuxième demande
  d’authentification, dont une partie plus tard, dans la fenêtre
  d’authentification unique, de sorte qu’ils ne seront pas invités à se
  connecter une deuxième fois.
* Dans le cas improbable où l’utilisateur est déjà connecté au FSI, mais que
  la fenêtre d’authentification unique expire dans quelques secondes entre les
  deux demandes d’authentification, alors la connexion unique s’appliquera à la
  première demande, mais pas à la seconde. Encore une fois, l’utilisateur n’a
  besoin de fournir ses informations d’identification qu’une seule fois.
* Si l’utilisateur est déjà connecté au FSI et que les deux demandes
  d’authentification sont traitées dans la fenêtre de connexion unique, le FSI
  n’invite pas l’utilisateur à saisir à nouveau ses informations
  d’identification. La partie de confiance peut remplacer ce comportement en
  spécifiant `prompt="login"` (pour OpenID Connect) ou `forceAuthn="true"` (dans
  SAML) dans sa demande d’authentification pour se connecter au Canada, où la
  plateforme d’acceptation spécifiera `forceAuthn="true"` sur sa première
  demande au FSI, mais pas la seconde.

### Qu’advient-il si l’utilisateur du clavier change? Y a-t-il un risque que la plateforme d’acceptation recueille le mauvais identificateur appartenant à la mauvaise personne?

Ce scénario possible peut survenir dans les cas où plusieurs personnes partagent
le même appareil, et l’une d’elles a laissé l’appareil sans surveillance pendant
qu’elles étaient encore connectées au FSI. À titre d’exemple :

Prenons l’exemple de deux utilisateurs, Alice et Bob, qui ont tous deux accès au
même ordinateur partagé :

1. Alice se connecte à la partie de confiance de la FJGC en utilisant sa CléGC,
  mais s’éloigne ensuite de l’ordinateur sans se déconnecter.
2. Bob s’assoit ensuite sur le même ordinateur et tente de se connecter à la
  partie de confiance Auntheti-Canada, quelques secondes seulement avant
  l’expiration de la fenêtre de 20 minutes de connexion unique d’Alice.
3. La plateforme d’acceptation envoie la première demande d’authentification à
  CléGC et reçoit en retour l’identificateur par paire d’Alice.
4. La fenêtre connexion unique d’Alice expire ensuite en quelques secondes avant
  que la plateforme d’acceptation envoie la deuxième demande d’authentification.
  CléGC demande à Bob d’entrer ses informations d’identification de CléGC, puis
  retourne l’identificateur par paire de Bob à la plateforme d’acceptation.
5. La plateforme d’acceptation associe ensuite l’identificateur de la CléGC de
  Bob au profil d’utilisateur d’Alice.


Afin d’éviter cela, la plateforme d’acceptation vérifie l’Index de session
(`SessionIndex`)des deux assertions SAML et s’assure qu’elles correspondent
avant d’accepter l’identificateur collecté.

### Les identificateurs par paires SAML émis par un fournisseur d’identité sont censés être propres au fournisseur de services SAML (PC). Comment Authenti-Canada est-il autorisé à obtenir les identificateurs générés pour un autre fournisseur de services SAML?

Les spécifications SAML permettent l’existence *d’affiliations* : groupes d’un ou
de plusieurs fournisseurs de services qui partagent le même identificateur par
paire pour un utilisateur donné. Lorsqu’une partie de confiance se déconnecte de
la FJGC pour se connecter à Authenti-Canada, une nouvelle affiliation SAML est
définie, au moyen des métadonnées SAML, qui permettent à la plateforme
d’acceptation d’obtenir les identificateurs par paire qui ont été créés à
l’origine pour la PC.
