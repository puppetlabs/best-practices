# Facts and Classification

## Summary

Facts provide information that can be used in the PE console to allocate a node to a Node Group within the Node Classifier. Facts are generated on the node and therefore can be manipulated.

Trusted facts which are embedded into the certificate and cannot be manipulated without creating a new certificate are the preferred information variables for classification rules. This provides a higher level of security to classification.

## Expectations

This standard applies to any customer that uses rules within Node Groups of the Node Classifier.

# Standard Details

## Preferred Option

All customers using the Node Classifier and using rules should use variables from the trusted facts array (`$trusted`). These facts are included in the node's certificate or generated on the master and, as such, cannot be changed on the node.

Inserted information into the node's certificate is done at agent installation time. The Certificate Extensions are part of the PKI Certificate standard (X.509) and are well understood. Puppet has a range of extensions allocated, with names, that can be easily used by customers, such as `pp_role`, `pp_region`, `pp_datacenter`, and `pp_apptier`. Further information on certificate extensions can be found at https://puppet.com/docs/puppet/latest/ssl_attributes_extensions.html

## Discouraged Options

Using facts from the standard facts array ($facts) is discouraged as these variables are generated on the node and therefore can be changed leading to a possible change in classification. From a security and operational perspective this is not desirable as it could be used to get configuration the node should not have, potentially including sensitive data(passwords, etc).

# Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the like: https://www.ietf.org/rfc/rfc2119.txt
* Feedback can be provided to Puppet CS.

# Other Information

* https://puppet.com/docs/puppet/latest/ssl_attributes_extensions.html
* https://puppet.com/docs/pe/latest/managing_nodes/grouping_and_classifying_nodes.html#writing-node-group-rules
