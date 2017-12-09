# Classification and Data Pattern Standards

## Summary

This standard defines the preferred path for how to classify in the PE GUI, and
the preferred place to store data. In summary the console classification should
use 1 role per group, and data should be stored in Hiera to avoid split sources
of truth. PE configuration is the exception which is currently done in the
console (but likely to change in the future).

## Expectations

All new customers working with someone in CS should be directed to use the
below node classification and data standards.

This standard may not fit with high complexity environments such as
self-service provisioning systems integrated with PE(vRA, Openstack) that
require node specific classification.


## Standard Details

### Preferred Option


**Classification in the console - 1 Role / Node Group vs Profiles in Node Groups (node group functions as a role)**
**Decision: 1 Role / Node Group**

We came to this decision for 2 reasons:

1. We don’t want people using parameters in the console since it’s not versionable
2. If you go the profiles in Node Groups route, there’s no good way to test adding new profiles to a role

**Data - params in console vs params in Hiera**
**Decision: params in Hiera - except for PE infrastructure, those param values can go in console**

We came to this decision because, in general, we don’t want people editing
parameters in the console. However, our docs tell people to edit parameter
values for PE Infrastructure in the console. So, for customer modules, we’ll
advise that all data should be in Hiera. For PE Infra, they can use params in
console.

The console and Hiera have no visibility of each other, so err on Hiera for
most things that need a data override

We also want one source of truth for data, since PE itself puts some parameters
for PE infra in the console all others should go with it instead of splitting
some of it in Hiera and some in the console.

**Per Node classification**

Treat per node classification just like all other classification - one role in
a node group. We did talk about the possibility of using profiles directly in
the console for these snowflakes, but we don’t want to confuse people with one
off rules, so we’re not going to mention it.


### Alternate Options

These options may be used if the situation requires.

**Classification - 1 Role per node group with parameters from profiles surfaced at the role level**

In situations where a provisioning system is used, or where there are teams
that are solely given access to the console to change params it may be
necessary to expose those params in the console. The customer may require this
if end users have limited Puppet experience or limited experience with Git etc.

A company exposes the parameters they believe an end user (whoever that may be)
can modify for whatever reason: this may be different in prod to non-prod.

This should be avoided unless required as the console cannot be versioned and
this creates a second source of truth which may make debugging more difficult.


### Discouraged Options

**Classification - Multiple Profiles per Node Group when used with “have it
your way” provisioning systems**

In very rare situations a customer may have a provisioning system where the
users are able to compose the configuration of a node by picking which profiles
that node will be classified with. Essentially making them able to provision
bespoke server configurations.

This should be avoided at all costs unless absolutely required as it can lead
to a proliferation of node groups. Additionally it makes doing RSpec or
acceptance testing at the role level very difficult (how to test if someone is
overriding the standard operating environment(SOE) as an example)


## Feedback / Ideas for Improvement

Elizabeth Plumb drafted the initial proposal, Brett Gray provided feedback
around the alternate options, Owen Rodabaugh documented this standard.

## Other Information


