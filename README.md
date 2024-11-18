# Sunfish-fix-ramdump-issue-on-evox-10.0

MAIN ISSUES:
1. Initial error: `unknown type vendor_ramdump_prop`
2. Second error: `unknown type ramdump_vendor_data_file`
3. Problematic path: `device/google/sunfish/sepolicy/vendor/google/`

COMPLETE SOLUTION:

1. Property Definitions (Path: `device/google/sunfish/sepolicy/vendor/google/property.te`):
```bash
# Add in vendor_internal_prop section
vendor_internal_prop(vendor_ramdump_prop)
```

2. Property Contexts (Path: `device/google/sunfish/sepolicy/vendor/google/property_contexts`):
```bash
# Add declaration
vendor.ramdump.                                 u:object_r:vendor_ramdump_prop:s0
```

3. File Type Definitions (Path: `device/google/sunfish/sepolicy/vendor/google/file.te`):
```bash
# Ramdump file types
type ramdump_vendor_data_file, file_type, data_file_type, vendor_file_type;
type ramdump_vendor_mnt_file, file_type, vendor_file_type;
```

4. File Contexts (Path: `device/google/sunfish/sepolicy/vendor/google/file_contexts`):
```bash
# Ramdump paths
/data/vendor/ramdump(/.*)?     u:object_r:ramdump_vendor_data_file:s0
/mnt/vendor/ramdump(/.*)?      u:object_r:ramdump_vendor_mnt_file:s0
```

5. Ramdump App Policy (Path: `device/google/sunfish/sepolicy/vendor/google/ramdump_app.te`):
```bash
type ramdump_app, domain;

userdebug_or_eng(`
  app_domain(ramdump_app)

  # Allow access to app API service
  allow ramdump_app app_api_service:service_manager find;

  # File permissions for ramdump data
  allow ramdump_app ramdump_vendor_data_file:file create_file_perms;
  allow ramdump_app ramdump_vendor_data_file:dir create_dir_perms;

  # Property permissions
  set_prop(ramdump_app, vendor_ramdump_prop)
  get_prop(ramdump_app, system_boot_reason_prop)

  # Mount point access
  allow ramdump_app mnt_vendor_file:dir search;
  allow ramdump_app ramdump_vendor_mnt_file:dir create_dir_perms;
  allow ramdump_app ramdump_vendor_mnt_file:file create_file_perms;
')
```

REBUILD STEPS:
```bash
make clean
make clobber
source build/envsetup.sh
lunch evolution_sunfish-userdebug
mka evolution
```

ISSUE CONTEXT:
1. Ramdump is a debugging feature that stores memory contents when a crash occurs
2. Requires several permissions:
   - Property permissions to manage vendor properties
   - File permissions to store ramdump data
   - Mount point access for storage location access
3. SELinux policy is required for:
   - Allowing ramdump application to access filesystem
   - Managing vendor properties
   - Creating and managing ramdump files
   - Accessing mount points

All modified files are located under the path:
`device/google/sunfish/sepolicy/vendor/google/`

This is a comprehensive solution for handling ramdump-related SELinux policies in the Android system, specifically for the Google Pixel 4a (sunfish) device running Evolution X 10.0 VIC custom ROM.
