# GCP Professional Security engineer notes

## Resource Hierarchy

- Organization (root node) - represents the organisation (ie company). Policies created at this level are inherited by all entities underneath. An Organization Admin role is created which has full privilges over GCP resources.
- Folders (optional) - allows admins to create groups of folders and projects as a grouping mechanism and isalation boundry between projects. IAM roles applied to a folder are applied to all projects.
- Projects - Core component of GCP, required to manage resources - any service that is created must be held within a project. Identifiers include Project ID (globally unique) Project Number (automatic) Project Name (friendly name)
- Resources - instances, services, APIs storage etc.

Note - when using a personal email to setup GCP you will not have access to organization or folders layers.

### Hierarchy Inheritence

Organisation of resources/entities with GCP - GCP has a top down policy inheritance flowing from each parent to child object, policies are controlled by IAM. Note each child can only have one parent.

Policies can be set on all layers of the hierarchy. The policy for a resource is the union of the policy set on the resource and the policy inherited from parents. Inheritance is transative at any larer. Note - Permissive parent policies overrule restrictive child policies.

Contraints - a type of restriction against a GCP service, or list of services, usually applied at the organisational layer to be inherited by all child projects and folders. They are useful to restrict sensitive actions.

## IAM
Cloud IAM facilitates the management of access controls to specify who can do what on which resources. IAM should be used to grant granular access to specific GCP resources following the principal of least privilege (assigning only the necessary access required to perform the intended actions)

Who:
- Google Account / Cloud Identity User (user@gmail.com, user@example.com configured at google.admin.com)
- Service Account - used by applications and automation, not linked to a user
- Google Groups - named collection of google accounts and can include service accounts. Recommended to use groups to manage privilege assignments based on roles.
- Cloud Identity Gsuite domain uses a gsuite domain name (similar to google acccount)
- allAuthenticatedUsers - a special identifier that represents all authed accounts (not anonymous)
- allUsers - any users included unauthenticated users.

What:
Instead of directly assigning individual permissions to users, users are assigned to a role which contains a collection of permissions which are grouped together to form a role. There are 3 types of roles within GCP:
- primitive roles - legacy broad roles (existed before IAM). Consists of Owner, Editor and Viewer roles, as the permissions assignments are broad they should not be used.
- predefined roles - created and maintained by google to provide finer-grained access controls and are automatically updated for new services as they are rolled out.
- custom roles - User defined roles which can be used to enforce the most fine-grained control over privilege assignments. Requires more overhead as they are not maintained by google.

How:
policies are required to grant roles through a collection of statements which defines who has what access. Policies are attached to resources to enforce access control. 
Policies are represented by IAM policy objects which consist of bindings which bind members to roles.

### Super Admin Best Practice
A super admin has admin access to the organization - It should not be used for day-to-day administration of GCP. The super admin role is created automatically when creating an organization and cannot be removed. 

To secure Super Admin accounts the following best practices should be followed:
- Create a super admin email address - which shouldn't be specific to a GSuite or Cloud Identity user
- Enable MFA - consider physical tokens
- Use other administrative accounts with principle of least privileges
- Enable stackdriver alerts for super admin useage.
