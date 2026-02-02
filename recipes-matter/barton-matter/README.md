# Barton Matter

This recipe builds the Matter SDK for use by Barton, providing the base
implementation for integrating Matter with the Barton IoT Platform in Yocto-based
RDK environments. It is designed to be extended by client layers using bbappend
files, allowing each product to supply their own ZAP file and device-specific
configurations. The base recipe manages integration and build logic, while client
customizations define device-specific Matter configurations.

## Barton Matter Example

The `barton-matter-example` recipe serves as a template for clients to extend
the base Barton Matter integration. It demonstrates how to provide custom ZAP
files and other product-specific files needed to enable Matter device support
through the Barton IoT Platform.

### Building Matter in RDK Yocto

Matter is successfully integrated into RDK Yocto builds using the standard Matter
SDK build process with Yocto-specific patches for compatibility. The implementation:

- Uses Matter's native `bootstrap.sh` and build system within the Yocto environment
- Applies targeted patches to resolve incompatibilities between Matter/Pigweed's
  assumptions and Yocto's controlled build environment
- Enables Python virtual environment creation and package management within
  Yocto's constraints through careful environment setup

The patches handle specific integration challenges such as Python package management,
shell compatibility, and dependency resolution, allowing the standard Matter SDK
to build reliably in Yocto without requiring external pregeneration steps. To
achieve this, patches surgically break the isolation element of Matter's python
venvs and blends necessary packages with those existing in the Yocto native sysroot.
This is necessary due to shortcomings of the Python compiled and provided by Yocto
that prevent true package installation into a venv.

## Key Features

- Pre-configured CMake build environment for Matter integration
- Example Matter device implementations
- Reference ZAP file configuration
- Yocto-specific patches for build compatibility

---

## Usage Guidelines

### ZAP File Configuration

Every Matter-enabled Barton application must provide its own ZAP file that
defines the Matter device characteristics:

The ZAP file defines the complete Matter device data model including:

- Device type identifiers
- Supported clusters and endpoints
- Attributes and commands
- Event declarations

Use the ZAP Tool to create or modify your ZAP file based on your device
requirements, or use the provided example `barton.zap`.

See Matter's [ZAP tool guide](https://github.com/project-chip/connectedhomeip/blob/master/docs/zap_and_codegen/zap_intro.md)
for more details on generating a ZAP file.

### Recipe Usage

The Barton recipe explicitly depends on `barton-matter`, which enables Matter
device connectivity for the Barton IoT Platform. When creating your own
component that utilizes Barton with Matter capabilities, ensure this dependency
chain is maintained in your recipes.

Below is an example file structure for your Matter-enabled component:

```
example-layer/
└── example-component/
    ├── example-component_x.y.z.bb
    └── barton-matter/
        ├── barton-matter_x.y.z.bbappend
        └── files/
            └── your-zap-file.zap
```

---

## Matter 1.4 Legacy Requirements

**Note**: The following section applies only to Matter 1.4 builds. Matter 1.5+
builds have improved integration that eliminates these manual steps.

### Pregenerated Code (Matter 1.4 Only)

For Matter 1.4, the Yocto build environment cannot execute the Matter SDK's
`activation.sh` script directly. Therefore, code generation from your ZAP file
must happen before the build process begins. This "pregeneration" step creates
the required `zzz_generated` directory containing all Matter-generated code
needed for successful compilation. Helper scripts are included in the
`files/matter_1.4/scripts` directory.

After creating or updating your ZAP file to define your Matter configuration,
ensure your Barton Matter recipe has the following structure:

```
your-layer/
└── your-barton-matter-recipe/
    ├── barton-matter_*.bbappend
    └── files/
        └── your-zap-file.zap
```

Then run the `generate_zzz.sh` script and pass the path to your zap file as the
first argument:

```bash
files/matter_1.4/scripts/generate_zzz.sh /path/to/your-layer/your-barton-matter-recipe/files/your-zap-file.zap
```

This will generate the zzz_generated.tar.gz file in the files/ directory containing
the zap file, ready for use by the Yocto build system.

**Note**: Docker is required to run this script.

The complete file structure for Matter 1.4 should include the pregenerated files:

```
example-layer/
└── example-component/
    ├── example-component_x.y.z.bb
    └── barton-matter/
        ├── barton-matter_x.y.z.bbappend
        └── files/
            ├── your-zap-file.zap
            └── zzz_generated.tar.gz
```

## Developer Iteration

For iterating on barton-matter recipes and files, there is an important detail
to be aware of: When running Matter bootstrap process, the native sysroot is
written to. This means that if you were to remove the Matter .environment and
run bootstrap again, you would be starting from a different state than the first
time. This can manifest in errors. (Note: this is not the same as running
bootstrap back to back as that will work because bootstrap is idempotent).

What this means is that if you edit the recipe in a way that would invalidate
do_configure (for example: modifying an earlier section, editing the DEPENDS list,
modifying a patch), you should run a `bitbake barton-matter -c cleansstate` operation first before
building the recipe again. Similarly, if you edit anything under files/, you
should run `bitbake barton-matter -c cleansstate` to ensure your updated file is properly installed
into our build site within the Matter repository.

## Further Documentation

For more details on Matter implementation with Barton, refer to:

- [Barton documentation](https://github.com/rdkcentral/BartonCore/tree/main/docs)
- [Matter SDK documentation](https://github.com/project-chip/connectedhomeip/tree/master/docs)
