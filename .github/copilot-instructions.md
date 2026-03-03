# Copilot Instructions for vcpkg-registery

This is a **custom vcpkg registry** for C/C++ libraries. It follows the standard vcpkg registry layout.

## Repository Structure

```
ports/<port-name>/
    portfile.cmake    # Build instructions (download, configure, build, install)
    vcpkg.json        # Port manifest (name, version, dependencies)
    usage             # Usage instructions for consumers
versions/
    baseline.json     # Current baseline versions for all ports
    <first-letter>-/<port-name>.json   # Version history per port
```

## Creating a New Port

### portfile.cmake

- Use `vcpkg_from_github` to download sources with `REF "v${VERSION}"` and a `SHA512` hash
- Use `vcpkg_cmake_configure`, `vcpkg_cmake_install`, `vcpkg_cmake_config_fixup` for CMake projects
- For header-only libraries, set `set(VCPKG_BUILD_TYPE release)` and remove the debug directory
- Install the license with `vcpkg_install_copyright(FILE_LIST "${SOURCE_PATH}/LICENSE")`
- Install usage and README files to `${CURRENT_PACKAGES_DIR}/share/${PORT}`

### vcpkg.json

- `name`: lowercase, hyphen-separated
- `version`: semver without `v` prefix (e.g., `"1.2.3"`)
- `dependencies`: always include `vcpkg-cmake` and `vcpkg-cmake-config` as host dependencies

### usage

- Show the `find_package` and `target_link_libraries` commands needed by consumers

## Updating the Version Database

After creating or modifying port files, the version database must be updated:

```bash
# 1. Commit the port changes first (vcpkg needs the git-tree hash)
git add ports/<port-name>
git commit -m "<message>"

# 2. Run vcpkg x-add-version to update baseline.json and version history
git clone https://github.com/microsoft/vcpkg.git /tmp/vcpkg --depth 1
/tmp/vcpkg/bootstrap-vcpkg.sh -disableMetrics
/tmp/vcpkg/vcpkg --x-builtin-ports-root=./ports --x-builtin-registry-versions-dir=./versions x-add-version --all --verbose

# 3. Amend the commit to include the version database changes
git add versions/
git commit --amend -m "<message>"
```

## Reference

Use the existing port `reactivelitepp` in `ports/reactivelitepp/` as a reference for style and conventions.
