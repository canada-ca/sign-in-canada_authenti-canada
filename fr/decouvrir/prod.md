---
layout: default
ref: discover-prod
title: Environnement de production
lang: fr
---
# Environnement de production

L’environnement de production Authenti-Canada est en service depuis mars 2021.

## Autorisation de sécurité

La sécurité de l’environnement de production Authenti-Canada a été évaluée et autorisée à authentifier les utilisateurs au nom des applications des parties utilisatrices:

* qui exigent un niveau d’assurance des justificatifs d’identité de deux ou moins, conformément à la [Directive sur la gestion de l’identité](https://www.tbs-sct.gc.ca/pol/doc-fra.aspx?id=16577);
* pour laquelle la sensibilité (confidentialité, intégrité et disponibilité) de toute donnée à traiter, à stocker, à transmettre sur ou par les parties utilisatrices ne dépasse pas le niveau Protégé B, Intégrité moyenne, Disponibilité moyenne.

## Disponibilité

L’environnement de production fonctionne 24 heures sur 24, 365 jours par année. Il a une disponibilité cible de 99,8 % calculée mensuellement et incluant les activités d’entretien prévues.

## Rendement

Le rendement de l’environnement de production est calculé en mesurant le temps de réponse pour chaque demande HTTP reçue par le service. Ces demandes HTTP comprennent toutes les demandes reçues des navigateurs des utilisateurs pour les pages Web et le contenu Web connexe, ainsi que toutes les demandes aux interfaces de programmation d’applications (API) d’Authenti Canada provenant d’applications de parties utilisatrices.

La cible de rendement pour l’environnement de production est de répondre à 95 % des demandes HTTP dans un délai de 200 millisecondes ou moins, et de répondre à 99 % des demandes HTTP dans un délai de 500 millisecondes ou moins.


## Capacité

L’environnement de production dispose actuellement d’une capacité suffisante pour authentifier jusqu’à 12 utilisateurs par seconde (43 200 par heure).

## Sauvegarde et récupération des données

Toutes les données des utilisateurs et toutes les données de configuration des parties utilisatrices stockées dans l’environnement de production sont sauvegardées une fois par heure, toutes les heures, dans un stockage géographiquement redondant, ayant une durabilité d’au moins 99,99999999999999 % sur une année donnée.

## Surveillance

La disponibilité et le rendement de l’environnement de production sont surveillés 24 heures sur 24, 365 jours par année. Cette surveillance est entièrement automatisée. Chaque fois qu’une interruption de disponibilité ou une dégradation de la performance est détectée, les membres de l’équipe de soutien d’Authenti-Canada sont automatiquement alertés par courriel et par texto.
