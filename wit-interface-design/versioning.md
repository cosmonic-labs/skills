# Versioning and Stability

## Versioning and stability

- **`@since(version = X.Y.Z)` on every interface**, and on every type, function and world item added later. It records the package version that introduced the item.
- **Deprecate, don't delete.** Mark retired items `@deprecated(version = X.Y.Z)` (the version that deprecated them) and keep them working. Use `@unstable(feature = ...)` for experimental items.
- **Breaking changes are only allowed on a version bump that permits them:**
  - Major version `0` (`0.Y.Z`): bump the **minor** (`0.2.0` to `0.3.0`). Minor is the breaking boundary.
  - Major version `>= 1`: bump the **major** (`1.x` to `2.0.0`).
  - Otherwise (patch, or minor at `>= 1`): additive only. New functions, new interfaces, new `variant` cases (hence `other`).
- Removing, renaming, reordering, or changing a signature counts as breaking.
- Prefer adding a new interface over widening an existing one when the new power is a **new capability**.

```wit
package acme:widgets@0.3.0;

@since(version = 0.1.0)
interface store {
  @since(version = 0.1.0)
  @deprecated(version = 0.3.0)
  get-legacy: async func(key: string) -> result<widget, error-code>;

  @since(version = 0.3.0)
  get: async func(key: string) -> result<widget, error-code>;
}
```
