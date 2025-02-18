# JSON schemas

JSON schemas for the Kyverno Envoy Plugin are available:

- [AuthorizationPolicy (v1alpha1)](https://github.com/kyverno/kyverno-envoy-plugin/blob/main/.schemas/json/authorizationpolicy-envoy-v1alpha1.json)

They can be used to enable validation and autocompletion in your IDE.

## VS code

In VS code, simply add a comment on top of your YAML resources.

### AuthorizationPolicy

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/kyverno/kyverno-envoy-plugin/main/.schemas/json/authorizationpolicy-envoy-v1alpha1.json
apiVersion: envoy.kyverno.io/v1alpha1
kind: AuthorizationPolicy
metadata:
  name: demo-policy.example.com
spec:
  variables:
  - name: force_authorized
    expression: object.attributes.request.http.headers[?"x-force-authorized"].orValue("") in ["enabled", "true"]
  - name: force_unauthenticated
    expression: object.attributes.request.http.headers[?"x-force-unauthenticated"].orValue("") in ["enabled", "true"]
  - name: metadata
    expression: '{"my-new-metadata": "my-new-value"}'
  authorizations:
    # if force_unauthenticated -> 401
  - expression: >
      variables.force_unauthenticated
        ? envoy
            .Denied(401)
            .WithBody("Authentication Failed")
            .Response()
            .WithMetadata(variables.metadata)
        : null
    # if force_authorized -> 200
  - expression: >
      variables.force_authorized
        ? envoy
            .Allowed()
            .WithHeader("x-validated-by", "my-security-checkpoint")
            .WithoutHeader("x-force-authorized")
            .WithResponseHeader("x-add-custom-response-header", "added")
            .Response()
            .WithMetadata(variables.metadata)
        : null
    # else -> 403
  - expression: >
      envoy
        .Denied(403)
        .WithBody("Unauthorized Request")
        .Response()
```
