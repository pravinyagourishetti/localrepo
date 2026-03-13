Problem Statement (Point-wise)

An integration is required to synchronize Agent–Manager hierarchy data between AWS and Salesforce.

AWS will send agent-related data containing Agent BRID, Manager BRID, and TSYS ID.

The Contact object in Salesforce represents both agents and managers and maintains the reporting hierarchy through the Reports To field, which references another Contact.

The User object represents system users and maintains the user hierarchy through the Manager field.

The system must ensure that the Agent Contact is mapped to the correct Manager Contact based on the data received from AWS.

If the expected Manager Contact does not exist, the system should check whether a corresponding Manager User exists.

If a Manager User exists but the Manager Contact does not, a new Manager Contact should be created using the available user information.

If neither a Manager Contact nor a Manager User exists, the Agent Contact should be assigned the Default Contact Owner in the Reports To field to maintain a valid hierarchy.

When a new Manager Contact is created, its Reports To field should point to the Default Contact Owner.

The User object must also reflect the correct reporting hierarchy, ensuring that the Agent User’s Manager field points to the correct Manager User.

The TSYS ID in the Agent User record must be validated and updated if the value received from AWS differs from the value stored in Salesforce.

The objective of this integration is to ensure consistent and accurate reporting hierarchy and user information in Salesforce based on the data received from AWS.
