# AGENTS.md

Instructions for AI coding agents working on this ESPHome configuration repository.

## Project Overview

This repository contains ESPHome YAML configuration files for IoT devices including Zigbee routers, sprinkler controllers, and ESP32/ESP8266-based devices. Configurations use a modular package system with shared common components.

## Key Documentation

### ESPHome Documentation

- **Main Docs**: <https://esphome.io/>
- **Component Reference**: <https://esphome.io/components/index.html>
- **YAML Configuration Guide**: <https://esphome.io/guides/configuration-types.html>
- **Lambdas (C++ Code)**: <https://esphome.io/guides/automations.html#lambda-expressions>
- **Packages**: <https://esphome.io/guides/packages.html>
- **Secrets**: <https://esphome.io/guides/configuration-types.html#config-secrets>

### Zigbee Component Documentation

- **Repository**: <https://github.com/luar123/zigbee_esphome>
- **Documentation**: Check the repository README for component-specific details
- **Configuration Pattern**: See `external/zigbee.yaml` for the standard setup

## Project Structure

```text
common/          # Shared configuration packages (included in git)
devices/         # Device-specific base configs (included in git)
external/        # External component configs (included in git)
templates/       # Reusable template configs (included in git)
archive/         # Old configs (gitignored)
secrets.yaml     # Sensitive credentials (gitignored)
*.yaml (root)    # Device configs
```

## Configuration Patterns

### Using Packages

Always use the package system to include common functionality:

```yaml
packages:
  - !include common/common.yaml
  - !include devices/esp32-c6.yaml
  - !include external/zigbee.yaml
```

### Using Secrets

**NEVER hardcode passwords, API keys, or tokens.** Always use `!secret`:

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

### Template Variables

Templates use substitution variables with `${variable_name}` syntax:

```yaml
pin: "${zone_gpio}"
id: z${zone_number}
```

### Internal Components

Mark diagnostic/internal components as `internal: true` to hide from Home Assistant:

```yaml
text_sensor:
  - platform: wifi_info
    ip_address:
      name: IP Address
      internal: true
```

## Security Requirements

1. **Always check for hardcoded credentials** before suggesting or making changes
2. **Never commit `secrets.yaml`** - it's gitignored for a reason
3. **Scan for patterns**: `password:`, `key:`, `token:`, `secret:` followed by quoted strings
4. **Root-level YAML files are gitignored** - they may contain device-specific secrets
5. **If you find hardcoded credentials**, replace with `!secret` references

## Code Style

- Use 2-space indentation (standard YAML)
- Use single quotes for strings when possible
- Group related components together
- Use descriptive IDs and names
- Add comments for complex logic or non-obvious configurations

## Common Tasks

### Adding a New Device Configuration

1. Create YAML file in root (will be gitignored automatically)
2. Include appropriate packages from `common/` and `devices/`
3. Add device-specific components
4. Use `!secret` for all sensitive values

### Modifying Common Packages

- Files in `common/` are shared across devices
- Changes affect all devices using that package
- Test impact on existing device configs
- Keep backward compatibility when possible

### Working with Zigbee

- Zigbee config is in `external/zigbee.yaml`
- Include it in device configs that need Zigbee: `- !include external/zigbee.yaml`
- See `devices/esp32-c6-zigbee.yaml` for example
- Zigbee devices may need custom partition tables (see `devices/esp32-c6-zigbee-partitions.csv`)

### Creating Templates

- Place in `templates/` directory
- Use substitution variables: `${var_name}`
- Document required variables in comments
- See `templates/sprinkler-remote/switch.yaml` for example

## Validation

Before suggesting changes:

1. Validate YAML syntax
2. Check all `!secret` references exist (mention if user needs to add to secrets.yaml)
3. Verify component syntax against ESPHome docs
4. For Zigbee changes, verify against zigbee_esphome documentation

## Git Rules

### Files Included in Git

- `.gitignore`
- All files in `common/`, `devices/`, `external/`, `templates/`
- `README.md`, `AGENTS.md`

### Files Excluded (per .gitignore)

- `secrets.yaml` - Never commit this
- `/*.yaml` in root - Device configs may contain secrets
- `/archive/` - Old configurations
- `/.esphome/` - Build artifacts

## Testing Instructions

1. **Validate YAML**: Use ESPHome's validation: `esphome config <file>.yaml`
2. **Check secrets**: Ensure all `!secret` references are documented
3. **Verify syntax**: Check component names and properties against ESPHome docs
4. **Test packages**: If modifying common packages, verify they work in existing device configs

## When Unsure

1. **Check ESPHome docs first**: <https://esphome.io/components/index.html>
2. **For Zigbee questions**: Check <https://github.com/luar123/zigbee_esphome>
3. **Follow existing patterns**: Match the style and structure of similar files
4. **Ask for clarification**: If component behavior is unclear, suggest checking docs or testing

## Important Notes

- ESPHome uses YAML with specific component schemas - always reference official docs
- Lambda expressions use C++ syntax, not Python
- Substitutions use `${var}` syntax, not `$var` or `{{var}}`
- Packages can be included with `!include` or `!include_dir_list` or `!include_dir_named`
- Secrets are loaded from `secrets.yaml` in the same directory or parent directories
