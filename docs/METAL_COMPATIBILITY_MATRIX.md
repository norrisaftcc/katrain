# KataGo Metal Compatibility Matrix

## macOS Version Support

| macOS Version | Version Number | Metal Support | KataGo Compatibility | Notes |
|--------------|----------------|---------------|---------------------|--------|
| Ventura | 13.0+ | Metal 3 | ✅ Full Support | Minimum required version |
| Monterey | 12.0 | Metal 3 | ❌ Not Supported | Below minimum requirement |
| Big Sur | 11.0 | Metal 2.3 | ❌ Not Supported | Lacks required APIs |
| Catalina | 10.15 | Metal 2.2 | ❌ Not Supported | Lacks required APIs |

## Hardware Compatibility

### Apple Silicon Macs

| Chip | Architecture | Metal Feature Set | Performance | Notes |
|------|--------------|-------------------|-------------|--------|
| M3 Pro/Max | ARMv9 | Metal 3 | ⭐⭐⭐⭐⭐ | Latest, best performance |
| M3 | ARMv9 | Metal 3 | ⭐⭐⭐⭐ | Excellent performance |
| M2 Pro/Max/Ultra | ARMv8.5 | Metal 3 | ⭐⭐⭐⭐⭐ | Excellent performance |
| M2 | ARMv8.5 | Metal 3 | ⭐⭐⭐⭐ | Very good performance |
| M1 Pro/Max/Ultra | ARMv8.5 | Metal 3 | ⭐⭐⭐⭐ | Very good performance |
| M1 | ARMv8.5 | Metal 3 | ⭐⭐⭐ | Good performance |

### Intel Macs with AMD GPUs

| GPU | Metal Feature Set | Performance | Notes |
|-----|-------------------|-------------|--------|
| AMD Radeon Pro 5600M | Metal 3 | ⭐⭐⭐ | 2019-2020 16" MacBook Pro |
| AMD Radeon Pro 5500M | Metal 3 | ⭐⭐⭐ | 2019 16" MacBook Pro |
| AMD Radeon Pro 580X | Metal 2 | ⭐⭐ | 2017-2020 iMac |
| AMD Radeon Pro 570X | Metal 2 | ⭐⭐ | 2017-2020 iMac |

### Intel Macs with Intel Graphics

| GPU | Metal Feature Set | Performance | Notes |
|-----|-------------------|-------------|--------|
| Intel Iris Plus | Metal 2 | ⭐ | Limited performance |
| Intel UHD Graphics | Metal 2 | ⭐ | Minimal performance |

## Feature Support Matrix

| Feature | Apple Silicon | AMD GPU | Intel GPU | Notes |
|---------|---------------|---------|-----------|--------|
| FP32 Computation | ✅ | ✅ | ✅ | Full precision |
| FP16 Computation | ✅ | ✅ | ⚠️ | Limited on Intel |
| Metal Performance Shaders | ✅ | ✅ | ✅ | Core requirement |
| MPS Graph | ✅ | ✅ | ✅ | Required for NN ops |
| Unified Memory | ✅ | ❌ | ❌ | Apple Silicon only |
| Hardware Ray Tracing | ✅ | ❌ | ❌ | Not used by KataGo |

## Performance Characteristics

### Expected Performance (relative to CUDA)

| Platform | Relative Performance | Typical NPS* | Notes |
|----------|---------------------|--------------|--------|
| M3 Max | 80-90% | 2000-3000 | Excellent efficiency |
| M2 Max | 75-85% | 1800-2500 | Very good |
| M1 Max | 70-80% | 1500-2000 | Good |
| M1 | 50-60% | 800-1200 | Adequate |
| AMD Radeon Pro | 40-60% | 600-1000 | Variable |
| Intel Graphics | 10-20% | 100-300 | Not recommended |

*NPS = Nodes per second (approximate, depends on network size and board position)

## Build Configuration Recommendations

### Apple Silicon (M1/M2/M3)
```bash
cmake -G Ninja -DUSE_BACKEND=METAL \
      -DCMAKE_OSX_ARCHITECTURES=arm64 \
      -DUSE_FP16=ON \
      -DCMAKE_BUILD_TYPE=Release
```

### Intel with AMD GPU
```bash
cmake -G Ninja -DUSE_BACKEND=METAL \
      -DCMAKE_OSX_ARCHITECTURES=x86_64 \
      -DUSE_FP16=ON \
      -DCMAKE_BUILD_TYPE=Release
```

### Intel with Intel GPU
```bash
cmake -G Ninja -DUSE_BACKEND=METAL \
      -DCMAKE_OSX_ARCHITECTURES=x86_64 \
      -DUSE_FP16=OFF \
      -DCMAKE_BUILD_TYPE=Release
```

## Known Limitations

### Metal Backend Specific
1. Single GPU only (no multi-GPU support)
2. Requires macOS 13.0+ (no backwards compatibility)
3. Ninja build system mandatory
4. No dynamic GPU switching on dual-GPU systems

### Performance Limitations
1. FP16 may have reduced accuracy for some operations
2. Intel integrated GPUs have limited memory bandwidth
3. External GPU (eGPU) support is untested

## Testing Recommendations

### Quick Compatibility Test
```bash
# Check Metal support
system_profiler SPDisplaysDataType | grep Metal

# Run KataGo benchmark
./katago benchmark -model <model.bin.gz> -config <config.cfg>
```

### Performance Benchmarks
1. Use same model file for all tests
2. Test with various board sizes (9x9, 13x13, 19x19)
3. Compare visits/second across configurations
4. Monitor GPU usage with Activity Monitor

## Future Compatibility

### macOS 14 (Sonoma) and Beyond
- Expected to maintain compatibility
- May offer performance improvements
- New Metal features could be leveraged

### Hardware Trends
- Apple Silicon performance improving ~20% per generation
- Intel Macs being phased out
- Focus on Apple Silicon optimization recommended