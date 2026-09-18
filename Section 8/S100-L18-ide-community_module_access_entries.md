# Lab: Declare a Grant Through the Cluster Module

## Goal

Grant a third identity cluster access without writing a single access entry resource. You teach the cluster module to accept access entries as an input, then declare the grant inside the `module "cluster"` block. By the end two mechanisms are live on one cluster, producing the same kind of AWS objects, and you have seen why a principal can only be declared through one of them.

## Starting point

The applied project with the team fully mapped: the platform administrator holds cluster administration, the developer holds cluster-wide read plus edit in `team-web`. All of those grants are separate resources sitting next to the cluster definition. You now add a second route to the same result.

## Configuration to target

- **A new input on the cluster module,** named `access_entries`, defaulting to an empty map so existing behaviour is unchanged. Key each entry by a name you choose. Each value carries the principal ARN, an optional type defaulting to `STANDARD`, and a map of policy associations, each holding a policy ARN and an access scope.
- **Trim the input to what this course uses.** The community module underneath accepts more fields than these.
- **Pass it straight through** to the community module. No transformation.
- **A CI deploy role:** the cluster name followed by `-ci-deployer`, reusing the trust policy and the cluster-describe permissions the team roles already share.
- **Its grant:** `AmazonEKSEditPolicy` scoped to the `team-web` namespace, declared in the `module "cluster"` block rather than as separate resources. This is deliberately the same grant the developer holds there, so the two routes can be compared directly.
- **The CI role goes in `infra/iam_ci.tf`, with an output holding its ARN.**

## Tasks

1. **Add the input to the cluster module.** Declare the variable and pass it to the community module. Nothing changes yet, because the default is empty.
2. **Write the CI deploy role.** One IAM role, one attachment of the cluster-describe policy the team roles already carry, and an output for the role ARN. It needs no access entry of its own.
3. **Declare the grant in the module block.** Add the `access_entries` argument to the `module "cluster"` call with a single entry for the CI role.
4. **Read the plan, then read the state.** Look at the addresses of the two access-entry objects the plan creates and note where they live. After applying, ask Terraform for every access-entry object in state. That list holds both routes at once, and it contains an entry you never declared anywhere. Work out where it came from.
5. **Apply, then compare the two routes from the AWS side.** List the cluster's access entries and their policies. Decide whether an entry created through the module is distinguishable from one created as a separate resource.
6. **Try to declare the same principal twice.** Add a separate access entry resource for the CI role on top of the module-declared one, and apply. Read the error, then remove the resource.

## Done when

- `terraform validate` reports the configuration is valid and the plan adds the CI role, its policy attachment, one access entry, and one policy association.
- Terraform state lists access entry objects at two addresses, one at the project root and one inside the cluster module, using the same two resource types.
- The module's set contains an entry for the cluster creator that you never wrote.
- The cluster reports six access entries, and the CI role's entry returns the same shape of result as a team role's.
- Declaring the CI role a second time plans cleanly and then fails during apply, with a `409` naming the conflict.

## Note

Neither route is better. They suit different situations, and the difference is about what you own rather than about the resources produced.

Separate resources need nothing but the cluster name, so they work against a cluster this project did not create and can live in their own state and repository. That suits a platform team owning the cluster and another team requesting access. Declaring through the module keeps the grant beside the cluster definition and puts everyone who can reach the cluster in one reviewable place, which suits a cluster you own outright. The cost is that the principals have to be reachable from where the module is called.

One rule holds across both: a principal has exactly one access entry, and declaring it twice fails no matter which routes you mix.

The CI role and its access objects are free. The cluster keeps billing while it runs.
