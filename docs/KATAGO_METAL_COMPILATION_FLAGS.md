# KataGo Metal Backend Compilation Flags Reference

## Essential Flags

### Backend Selection
```bash
-DUSE_BACKEND=METAL
```
**Required**: Selects the Metal backend for GPU acceleration on macOS.

### Build System
```bash
-G Ninja
```
**Required**: Ninja generator is mandatory for Swift/C++ bidirectional interop.

### macOS Deployment Target
```bash
-DCMAKE_OSX_DEPLOYMENT_TARGET=13.0
```
**Required**: Sets minimum macOS version to Ventura (13.0) for Metal 3 API support.

## Architecture Flags

### Apple Silicon (ARM64)
```bash
-DCMAKE_OSX_ARCHITECTURES=arm64
```
Builds specifically for Apple Silicon Macs (M1/M2/M3).

### Intel (x86_64)
```bash
-DCMAKE_OSX_ARCHITECTURES=x86_64
```
Builds specifically for Intel-based Macs.

### Universal Binary
```bash
-DCMAKE_OSX_ARCHITECTURES="arm64;x86_64"
```
Creates a universal binary that runs on both architectures.

## Optimization Flags

### Release Build
```bash
-DCMAKE_BUILD_TYPE=Release
```
Enables compiler optimizations for production use.

### Debug Build
```bash
-DCMAKE_BUILD_TYPE=Debug
```
Includes debug symbols and disables optimizations.

### Custom Optimization
```bash
-DCMAKE_CXX_FLAGS="-O3 -march=native -mtune=native"
```
Maximum optimization for the current CPU architecture.

### Link-Time Optimization
```bash
-DCMAKE_INTERPROCEDURAL_OPTIMIZATION=ON
```
Enables LTO for potential performance improvements.

## Precision Flags

### FP16 Support
```bash
-DUSE_FP16=ON
```
Enables half-precision floating-point computation for better performance.

```bash
-DUSE_FP16=OFF
```
Forces FP32 precision for maximum accuracy.

## Feature Flags

### TCMalloc (Not recommended on macOS)
```bash
-DUSE_TCMALLOC=OFF
```
Disables TCMalloc as it's not beneficial on macOS.

### Assertions
```bash
-DNO_ASSERTIONS=ON
```
Disables runtime assertions for slight performance gain.

## Complete Build Examples

### Optimal Apple Silicon Build
```bash
cmake -G Ninja \
  -DUSE_BACKEND=METAL \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=13.0 \
  -DCMAKE_OSX_ARCHITECTURES=arm64 \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_FLAGS="-O3 -march=native" \
  -DUSE_FP16=ON \
  -DUSE_TCMALLOC=OFF \
  -DNO_ASSERTIONS=ON
```

### Optimal Intel Mac Build
```bash
cmake -G Ninja \
  -DUSE_BACKEND=METAL \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=13.0 \
  -DCMAKE_OSX_ARCHITECTURES=x86_64 \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_FLAGS="-O3 -march=native" \
  -DUSE_FP16=ON \
  -DUSE_TCMALLOC=OFF
```

### Universal Binary Build
```bash
cmake -G Ninja \
  -DUSE_BACKEND=METAL \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=13.0 \
  -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64" \
  -DCMAKE_BUILD_TYPE=Release \
  -DUSE_FP16=ON \
  -DUSE_TCMALLOC=OFF
```

### Debug Build for Development
```bash
cmake -G Ninja \
  -DUSE_BACKEND=METAL \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=13.0 \
  -DCMAKE_BUILD_TYPE=Debug \
  -DUSE_FP16=OFF
```

## Environment Variables

### Compiler Selection
```bash
export CC=/usr/bin/clang
export CXX=/usr/bin/clang++
```

### PKG_CONFIG Path (for libzip)
```bash
export PKG_CONFIG_PATH="/opt/homebrew/opt/libzip/lib/pkgconfig:$PKG_CONFIG_PATH"
```

### Swift Compiler
The Swift compiler is automatically detected from Xcode installation.

## Validation Commands

### Check CMake Configuration
```bash
cmake -L . | grep -E "(BACKEND|FP16|OSX|BUILD_TYPE)"
```

### Verify Binary Architecture
```bash
lipo -info katago
```

### Check Metal Support
```bash
./katago version
# Should show "Metal" in the backend information
```

## Performance Impact of Flags

| Flag | Performance Impact | Notes |
|------|-------------------|--------|
| `-O3` | +10-15% | Aggressive optimizations |
| `-march=native` | +5-10% | CPU-specific optimizations |
| `FP16=ON` | +20-40% | Hardware dependent |
| `NO_ASSERTIONS=ON` | +1-2% | Minor improvement |
| `Release` vs `Debug` | +50-100% | Significant difference |

## Troubleshooting Flag Issues

### Swift/C++ Interop Errors
Ensure using Ninja generator:
```bash
cmake -G Ninja ...  # Required, not optional
```

### Architecture Mismatch
Check system architecture:
```bash
uname -m  # Shows arm64 or x86_64
```

### Metal API Errors
Verify macOS version:
```bash
sw_vers -productVersion  # Must be 13.0+
```

## Flag Combinations to Avoid

1. **Don't mix architectures in CXXFLAGS**
   ```bash
   # Bad: -march=native with universal binary
   -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64" -DCMAKE_CXX_FLAGS="-march=native"
   ```

2. **Don't use CUDA/OpenCL flags**
   ```bash
   # Bad: These are for other backends
   -DUSE_BACKEND=CUDA
   -DUSE_BACKEND=OPENCL
   ```

3. **Don't lower deployment target**
   ```bash
   # Bad: Metal backend requires 13.0+
   -DCMAKE_OSX_DEPLOYMENT_TARGET=12.0
   ```

## Recommended Production Flags

For distribution in KaTrain:
```bash
cmake -G Ninja \
  -DUSE_BACKEND=METAL \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=13.0 \
  -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64" \
  -DCMAKE_BUILD_TYPE=Release \
  -DUSE_FP16=ON \
  -DUSE_TCMALLOC=OFF \
  -DNO_ASSERTIONS=ON
```

This creates an optimized universal binary with broad compatibility.