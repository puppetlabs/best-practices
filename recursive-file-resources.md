# Recursive File Resources

## Summary

File resources with `recurse => true` should  be used only when narrow circumstances apply. Most use cases for managing large numbers of files should be achieved using different patterns. Avoid recursive file resources.

The narrow circumstances in which a recursive file resource should be used are:

* When the total number of recursive files to manage is small (such as ten or fewer) and the content of the files needs to be enforced
* When the total number of recursive files to manage is small (such as ten or fewer) and `purge => true` is being used

## Expectations

Users looking to manage large numbers of files using Puppet should avoid using `recurse => true` on a file resource.

As a rule of thumb, do not use File resources with `recurse => true` to manage more than ten files. It is non-performant on the agent, and can cause performance problems when recording 100s or 1000s of file resources in reports that must be handled by PuppetDB.

## Best Practice Details

### Preferred Option

* To manage permissions, owner, and/or group on all files in a directory, use the [recursive_file_permissions defined type](https://forge.puppet.com/npwalker/recursive_file_permissions). The defined type uses a single exec resource with `find` to idempotently and performantly manage file permissions, but not content,for files in a directory.

* To copy a large directory structure into place on an agent, use the [puppet/archive module](https://forge.puppet.com/puppet/archive). The module allows you to send a compressed tarball to the agent and extract it into place. You can combine this with recursive_file_permissions to ensure permissions are managed on the files after extraction.

* When file content is important and must be managed, versioned, or audited for many files, build a package for the files, and use a Package resource to manage it.

* When file content for a small number of files is being managed and using `purge => true`, such as in a directory.d pattern, it is good practice to use `recurse => true`.

### Alternate Options

For use cases where the preferred options are insufficient or do not apply, `recurse => true` should still be avoided. Other approaches besides those enumerated in the preferred options but which still avoid `recurse => true` should be sought.
