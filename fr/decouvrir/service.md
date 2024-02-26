---
layout: default
ref: discover-service
title: 
lang: fr
---
# Gestion des services et soutien

## Intervention en cas d’incidents

La vitesse à laquelle l’équipe de soutien d’Authenti-Canada réagit aux incidents 
de service de production dépend du moment où l’incident est détecté ou signalé 
(toutes les heures indiquées sont en heure de l’Est):

|Fenêtre d’occurrence de l’incident            |Cible pour la réponse                    |
|----------------------------------------------|-----------------------------------------|
|Jours ouvrables, de 8 h à 17 h                |Immédiatement                            |
|Jours ouvrables, de 17 h à 22 h               |Faire au mieux                           |
|Fin de semaine et jours fériés, de 8 h à 22 h |Faire au mieux                           |
|Nuits, de 22 h à 8 h                          |Le jour suivant, comme indiqué ci-dessus |
{: .table .table-bordered .table-condensed}

## Entretien planifié

### Maintenance des logiciels

Le logiciel Authenti-Canada exige une maintenance régulière afin de satisfaire aux niveaux 
de service établis et d’introduire de nouvelles fonctionnalités. Afin de réduire au minimum 
l’incidence sur les parties utilisatrices, les activités de maintenance qui nécessitent la 
mise hors ligne du service sont normalement prévues au cours d’une fenêtre de maintenance des 
logiciels définie, de 4 h à 5 h, heure de l’Est, les dimanches matin. Normalement, les parties 
utilisatrices ne sont pas informées des pannes prévues qui se produisent pendant cette période, 
qui durent normalement de 10 à 25 minutes. Dans des cas exceptionnels, lorsqu’une panne d’entretien 
doit être prévue à l’extérieur de cette fenêtre, l’équipe d’Authenti-Canada fournira aux parties 
utilisatrices un préavis d’au moins 7 jours.

### Maintenance du réseau

L’environnement de production Authenti-Canada tire parti de l’infrastructure de sécurité du réseau 
qui est gérée par l’équipe de l’infrastructure et d’architecture d’entreprise du SCT. Cette infrastructure 
nécessite également un entretien périodique, qui est normalement prévu pour le jeudi suivant le deuxième 
mardi du mois, de 20 h à minuit. La maintenance prévue de l’infrastructure du réseau n’a pas toujours 
d’incidence sur la disponibilité du service, mais dans les cas où une panne est possible, l’équipe 
d’Authenti-Canada fera tout son possible pour informer les parties utilisatrices à l’avance.

## Entretien imprévu

L’environnement de production Authenti-Canada tire parti du soutien en nuage, en plateforme et 
en infrastructure du SCT et de SPC. Ils se réservent le droit de prévoir des pannes d’entretien 
d’urgence pour les mises à jour critiques, dans le but de fournir aux parties utilisatrices un 
préavis d’au moins vingt quatre (24) heures. Les deux parties feront de leur mieux pour planifier 
ces pannes pendant les heures creuses et pour limiter leur survenue aux mises à niveau strictement 
nécessaires et à l’entretien requis. (Il convient de noter qu’il est question d’un entretien d’urgence 
et non d’un correctif d’urgence à la suite d’un incident.)

## Dépendances des services

Le service Authenti-Canada tire parti de trois services de justificatifs d’identité fournis par 
Services partagés Canada, à savoir le service de justificatif portant la marque du gouvernement 
du Canada « CléGC », le service de courtier de justificatifs d’identité « Gouvernement Connecté 
par Vérifiez.Moi » et le service de deuxième facteur d’authentification. Les niveaux de service 
pour ces trois services sont distincts et séparés de ceux de l’environnement de production Authenti-Canada.

## Soutien aux parties utilisatrices

Les parties utilisatrices peuvent signaler des incidents ou demander de l’aide au dépannage en envoyant 
un courriel à <SignIn-AuthentiCanada@tbs-sct.gc.ca>. L’équipe de soutien d’Authenti-Canada surveille cette 
boîte aux lettres de 8 h à 17 h les jours ouvrables.
