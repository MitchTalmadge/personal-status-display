# ESPHome Dev Container

This dev container provides a complete ESPHome development environment.

## Features

- **ESPHome**: Latest version for building and deploying firmware
- **VS Code Extensions**: 
  - ESPHome extension for syntax highlighting and validation
  - YAML extension for better YAML editing
- **Dashboard**: ESPHome web dashboard on port 6052

## Usage

### Building Firmware
```bash
esphome compile your-device.yaml
```

### Uploading Firmware
```bash
# Over-the-air (OTA) update
esphome upload your-device.yaml

# Or via USB (requires USB passthrough)
esphome run your-device.yaml
```

### Running Dashboard
```bash
esphome dashboard /config
```

Then access it at http://localhost:6052

### Validating Configuration
```bash
esphome config your-device.yaml
```

## Notes

- USB device passthrough may require additional Docker configuration on your host
- For OTA updates, ensure your device is on the same network
- The workspace folder is mounted at `/config` inside the container
