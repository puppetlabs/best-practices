# ⚠ WARNING ⚠

This repository is no longer mantained since ~ 2019. While some of these documentation might still be useful, much of it is obsolete or out of date.

For a maintained collection of similar information, please see [Patterns and Tactics](https://puppet.com/docs/patterns-and-tactics).
# Module data in Profiles

## Summary

Profile modules containing Hiera v5 data-in-modules are a very specific use case for a focused usage. This use case is to allow users to delegate development of profiles and its defaults to different teams. Allowing the use of data-in-modules in this case, it allows the team to specify profile defaults while maintaining complete control (data in modules has the least priority).

## Expectations

Advanced users looking to delegate profile management code to application development teams should be using this best practice. It allows restricting those teams to only the keys used by the module.

## Standard Details

### Preferred Option

For many users delegation of specific applications is desired and 100% security is not a concern. In those cases use a separate profile repo with different developer access than the standard control repo.  This will allow profile developers to modify code and add hiera data that can be over written by any other level of hiera.

### Alternate Option

If security of module code is a significant concern (e.g. concerns delegated teams may override parameters via puppet code using collector overrides, class includes, etc) use the preferred option, but split the data-in-modules hiera data out into a separate repo.  Set permissions to allow the team delegated to only modify the data and not the code. This adds further complexity by requiring this split, and the configuration of the Puppetfile to merge the module and data back together during code deployment.

### Discouraged Options

Use data in modules in the control repo.  This should be discouraged due to access restrictions not providing any additional capabilities and does add additional complexity that can confuse the location of where to add data for the profiles layer.

## Feedback / Ideas for Improvement
In the future when Onceover/Gatekeeper or other testing solutions make it easier to detect and restrict the keys that can be modified less manual review will be required and the solutions in this standard may no longer be useful/applicable.

## Other Information
