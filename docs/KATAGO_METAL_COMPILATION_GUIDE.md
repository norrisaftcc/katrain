# KataGo Metal Backend Compilation Guide

## Overview
This guide provides step-by-step instructions for compiling KataGo with Metal backend support on macOS.

## System Requirements

### Minimum Requirements
- **macOS Version**: 13.0 (Ventura) or later
- **Xcode**: Command Line Tools installed
- **Architecture**: Intel x86_64 or Apple Silicon (ARM64)
- **Build System**: Ninja (required for Swift/C++ interop)

### Software Dependencies
```bash
# Install Xcode Command Line Tools
xcode-select --install

# Install required dependencies via Homebrew
brew install cmake ninja libzip
```

## Compilation Steps

### 1. Clone KataGo Repository
```bash
git clone https://github.com/lightvector/KataGo.git
cd KataGo
```

### 2. Configure Build with Metal Backend
```bash
cd cpp
cmake -G Ninja -DUSE_BACKEND=METAL \
      -DCMAKE_OSX_DEPLOYMENT_TARGET=13.0 \
      -DCMAKE_BUILD_TYPE=Release
```

### 3. Build KataGo
```bash
ninja
```

### 4. Verify Build
```bash
./katago version
# Should show version with Metal backend enabled
```

## Build Configuration Options

### Basic Options
- `-DUSE_BACKEND=METAL`: Enable Metal backend (required)
- `-DCMAKE_BUILD_TYPE=Release`: Build optimized binary
- `-DCMAKE_OSX_DEPLOYMENT_TARGET=13.0`: Set minimum macOS version

### Advanced Options
- `-DUSE_FP16=ON/OFF`: Enable/disable half-precision computation
- `-DUSE_TCMALLOC=OFF`: Disable TCMalloc (not needed on macOS)

## Architecture-Specific Builds

### Apple Silicon (M1/M2/M3)
```bash
cmake -G Ninja -DUSE_BACKEND=METAL \
      -DCMAKE_OSX_ARCHITECTURES=arm64 \
      -DCMAKE_BUILD_TYPE=Release
```

### Intel x86_64
```bash
cmake -G Ninja -DUSE_BACKEND=METAL \
      -DCMAKE_OSX_ARCHITECTURES=x86_64 \
      -DCMAKE_BUILD_TYPE=Release
```

### Universal Binary
```bash
cmake -G Ninja -DUSE_BACKEND=METAL \
      -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64" \
      -DCMAKE_BUILD_TYPE=Release
```

## Troubleshooting

### Common Issues

1. **"Could not find generator: Ninja"**
   ```bash
   brew install ninja
   ```

2. **"No Metal device found"**
   - Ensure macOS 13.0+ is installed
   - Check GPU availability: `system_profiler SPDisplaysDataType`

3. **Swift compiler errors**
   - Verify Xcode installation: `xcode-select -p`
   - Accept Xcode license: `sudo xcodebuild -license accept`

4. **libzip not found**
   ```bash
   brew install libzip
   export PKG_CONFIG_PATH="/opt/homebrew/opt/libzip/lib/pkgconfig:$PKG_CONFIG_PATH"
   ```

## Performance Optimization

### Compilation Flags
```bash
cmake -G Ninja -DUSE_BACKEND=METAL \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_CXX_FLAGS="-O3 -march=native" \
      -DUSE_FP16=ON
```

### Runtime Configuration
Create a config file with Metal-specific settings:
```json
{
  "nnCacheSizePowerOfTwo": 23,
  "numSearchThreads": 6,
  "fp16": true
}
```

## Integration with KaTrain

### Binary Location
After successful compilation, copy the binary to KaTrain:
```bash
cp katago /path/to/katrain/katrain/KataGo/katago-osx
```

### Verification
```bash
./katago benchmark -model /path/to/model.bin.gz -config /path/to/config.cfg
```

## Metal Backend Features

### Supported Operations
- Convolutional layers with various activation functions
- Batch normalization
- Residual blocks
- Global pooling
- Matrix multiplication
- Custom Mish activation (optimized for Metal)

### FP16 Considerations
- Improved performance on compatible GPUs
- Some activation functions have reduced precision
- Mish activation uses special thresholds for FP16

### Device Selection
- Uses system default Metal device
- No multi-GPU support currently
- Automatic fallback if Metal unavailable

## Build Artifacts

After successful build:
- Binary: `cpp/katago`
- Size: ~2-5MB (depending on options)
- Dependencies: System frameworks only

## Next Steps

1. Test the compiled binary with benchmark
2. Copy to KaTrain directory structure
3. Update KaTrain configuration
4. Run integration tests