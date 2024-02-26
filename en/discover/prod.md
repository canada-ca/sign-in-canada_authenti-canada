---
layout: default
ref: discover-prod
title: Production Environment
lang: en
---
# Production Environment

The Sign In Canada production environment has been live since March 2021.

## Security Authorization

The security of the Sign In Canada production environment has been assessed and authorized to authenticate users on behalf of relying party applications:

* that require a credential assurance level of 2 (LoA 2) or lower, in compliance with the [Directive on Identity Management](https://www.tbs-sct.gc.ca/pol/doc-eng.aspx?id=16577) ; and
* for which the sensitivity (confidentiality, integrity, and availability) of any data to be processed, stored, transmitted on or by the Relying Parties does not exceed Protected B, Medium Integrity, Medium Availability (PM/M/M).

## Availability

The production environment operates 24 hours a day, 365 days a year, with a target availability of 99.8% calculated monthly and inclusive of scheduled maintenance activities.

## Performance

The performance of the production environment is calculated by measuring the response time for every HTTP request received by the service. These HTTP requests include all those received from users’ browsers for web pages and related web content, and all requests to the Sign In Canada application programming interfaces (APIs) from relying party applications.

The performance target for the production environment is to respond to 95% of HTTP requests within 200 milliseconds or less, and to respond to 99 % of HTTP requests within 500 milliseconds or less.

## Capacity

The production environment currently has sufficient capacity to authenticate up to 12 users per second (43,200 per hour).

## Data Backup and Recovery

All user data and all relying party configuration data stored in the production environment is backed up once per hour, every hour to geographically redundant storage, with a durability of at least 99.99999999999999% over a given year.

## Monitoring

The availability and performance of the production environment is monitored 24 hours a day, 365 days a year. This monitoring is fully automated.
Whenever an availability interruption or performance degradation is detected, members of the Sign In Canada support team are automatically alerted by email and by text message.
