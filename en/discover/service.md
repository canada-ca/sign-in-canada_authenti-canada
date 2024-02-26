---
layout: default
ref: discover-service
title: Service Management and Support
lang: en
---
# Service Management and Support

## Incident Response

The speed at which the Sign In Canada support team responds to production
service incidents depends on when the incident is detected or reported (all
times listed are eastern time):

|Incident Occurrence Window         |Response Target       |
|-----------------------------------|----------------------|
|Working days, 8am to 5pm           |Immediate             |
|Working days, 5pm to 10pm          |Best Effort           |
|Weekends and Holidays, 8am to 10pm |Best Effort           |
|Nights, 10pm to 8am                |Next day, as per above|
{: .table .table-bordered .table-condensed}

## Planned Maintenance

### Software Maintenance

The Sign in Canada software requires regularly scheduled maintenance to meet the
established service levels and to introduce new features. To minimize the effect
on relying parties, maintenance activities that require taking the service
off-line are normally scheduled during a defined software maintenance window,
from 4am to 5am eastern time on Sunday mornings. Relying parties are not
normally notified of planned outages that take place during this window, which
are normally 10 to 25 minutes in duration. In exceptional cases, where a
maintenance outage needs to be scheduled outside of this window, the Sign In
Canada team will provide relying parties with at least 7 days notice.

### Network Maintenance

The Sign in Canada production environment leverages network security
infrastructure that is managed by the TBS Enterprise Architecture and
Infrastructure team. This infrastructure also requires periodic maintenance,
which is normally scheduled for the Thursday following the 2nd Tuesday of the
month, from 8pm to midnight. Planned network infrastructure maintenance does not
always affect service availability, but in cases where an outage is possible the
Sign In Canada team will make every effort to inform relying parties in advance.

## Unscheduled maintenance

The Sign In Canada production environment leverages cloud, platform and
infrastructure support from both TBS and SSC. They reserve the right to schedule
emergency maintenance outages for critical updates, with the goal of providing
relying parties at least twenty-four (24) hours advance notice. Both parties
will work on a best-efforts basis, to schedule these outages during non-peak
hours and limit their occurrence to strictly necessary upgrades and required
maintenance. (It should be noted that this is for emergency maintenance and not
emergency fixes following an incident).

## Service dependencies

The Sign In Canada service leverages three credential services provided by
Shared Services Canada, namely the Government of Canada-branded credential
service “GCKey”, the credential broker service “Government Sign-In by
Verified.Me”, and the 2nd-factor authentication service. The service levels for
these three services are separate and distinct from those of the Sign In Canada
production environment.

## Relying Party Support

Relying parties can report incidents or request troubleshooting assistance by
sending an email to <SignIn-AuthentiCanada@tbs-sct.gc.ca>. The Sign In Canada
support team monitors this mailbox from 8am to 5pm on working days.
