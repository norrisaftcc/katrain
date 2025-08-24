# KataGo Metal Performance Benchmark Plan

## Overview
This document outlines a comprehensive benchmarking strategy to evaluate KataGo Metal backend performance across different hardware configurations and compare with other backends.

## Benchmark Objectives

1. Measure absolute performance (nodes/second, visits/second)
2. Compare Metal vs CPU performance on same hardware
3. Evaluate FP16 vs FP32 performance impact
4. Test scaling with different board sizes
5. Measure power efficiency (performance per watt)
6. Compare with bundled binary performance

## Test Hardware Matrix

### Primary Test Systems
1. **Apple Silicon Mac**
   - Model: MacBook Pro 14"/16" with M3 Pro/Max
   - Memory: 32GB+ unified memory
   - OS: macOS 14.0+

2. **Intel Mac with AMD GPU**
   - Model: MacBook Pro 16" (2019)
   - GPU: AMD Radeon Pro 5500M
   - Memory: 16GB+
   - OS: macOS 13.0+

3. **Intel Mac with Intel GPU**
   - Model: MacBook Air (2020)
   - GPU: Intel Iris Plus
   - Memory: 8GB+
   - OS: macOS 13.0+

## Benchmark Suite

### 1. Basic Performance Test
```bash
# Standard benchmark
./katago benchmark \
  -model kata1-b18c384nbt-s9131461376-d4087399203.bin.gz \
  -config default_gtp.cfg \
  -time 60
```

### 2. Board Size Scaling
Test performance across different board sizes:
- 9x9 boards
- 13x13 boards
- 19x19 boards (standard)

```bash
# Board size specific benchmarks
for size in 9 13 19; do
  echo "Testing ${size}x${size} board..."
  ./katago benchmark \
    -model kata1-b18c384nbt-s9131461376-d4087399203.bin.gz \
    -config default_gtp.cfg \
    -boardsize $size \
    -time 30
done
```

### 3. FP16 vs FP32 Comparison
```bash
# FP32 benchmark
./katago benchmark \
  -model kata1-b18c384nbt-s9131461376-d4087399203.bin.gz \
  -config configs/gtp_fp32.cfg \
  -time 60

# FP16 benchmark
./katago benchmark \
  -model kata1-b18c384nbt-s9131461376-d4087399203.bin.gz \
  -config configs/gtp_fp16.cfg \
  -time 60
```

### 4. Multi-threading Efficiency
```bash
# Test with different thread counts
for threads in 1 2 4 6 8; do
  echo "Testing with $threads threads..."
  ./katago benchmark \
    -model kata1-b18c384nbt-s9131461376-d4087399203.bin.gz \
    -config default_gtp.cfg \
    -threads $threads \
    -time 30
done
```

### 5. Memory Usage Analysis
```bash
# Monitor memory usage during benchmark
./katago benchmark \
  -model kata1-b18c384nbt-s9131461376-d4087399203.bin.gz \
  -config default_gtp.cfg \
  -time 60 &

# Monitor in separate terminal
while true; do
  ps aux | grep katago | grep -v grep
  sleep 1
done
```

## Configuration Files

### FP32 Configuration (configs/gtp_fp32.cfg)
```
# Force FP32 precision
fp16 = false
nnCacheSizePowerOfTwo = 23
numSearchThreads = 6
```

### FP16 Configuration (configs/gtp_fp16.cfg)
```
# Enable FP16 precision
fp16 = true
nnCacheSizePowerOfTwo = 24
numSearchThreads = 6
```

### Power Efficiency Configuration
```
# Reduced power consumption
numSearchThreads = 2
nnCacheSizePowerOfTwo = 20
```

## Metrics to Collect

### Performance Metrics
1. **Visits/second**: Primary performance metric
2. **Nodes/second**: Neural network evaluations
3. **Latency**: Time to first move
4. **Throughput**: Moves per minute in game play

### System Metrics
1. **GPU Utilization**: Via Activity Monitor
2. **Memory Usage**: Peak and average
3. **Power Consumption**: Via powermetrics
4. **Temperature**: Via thermal monitoring

### Quality Metrics
1. **Move Quality**: Compare top moves between FP16/FP32
2. **Win Rate Stability**: Variation in evaluations
3. **Search Efficiency**: Nodes per visit ratio

## Benchmark Execution Protocol

### 1. System Preparation
```bash
# Close unnecessary applications
# Disable automatic updates
# Set performance power mode
sudo pmset -a gpuswitch 0  # Disable GPU switching on dual-GPU systems
```

### 2. Baseline Measurements
```bash
# CPU baseline
./katago benchmark \
  -model kata1-b18c384nbt-s9131461376-d4087399203.bin.gz \
  -config default_gtp.cfg \
  -backend cpu \
  -time 30
```

### 3. Metal Backend Tests
```bash
# Run full benchmark suite
./run_benchmarks.sh
```

### 4. Data Collection Script
```bash
#!/bin/bash
# run_benchmarks.sh

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
RESULTS_DIR="benchmark_results_${TIMESTAMP}"
mkdir -p $RESULTS_DIR

# System info
system_profiler SPHardwareDataType > $RESULTS_DIR/system_info.txt
system_profiler SPDisplaysDataType >> $RESULTS_DIR/system_info.txt

# Run benchmarks
for test in "standard" "fp16" "fp32" "small_board" "large_board"; do
  echo "Running $test benchmark..."
  ./katago benchmark \
    -model kata1-b18c384nbt-s9131461376-d4087399203.bin.gz \
    -config configs/${test}.cfg \
    -time 60 \
    > $RESULTS_DIR/${test}_results.txt 2>&1
done
```

## Results Analysis

### Performance Report Template
```markdown
## KataGo Metal Performance Report

### Test System
- Hardware: [Model]
- CPU: [CPU Model]
- GPU: [GPU Model]
- Memory: [RAM Amount]
- macOS Version: [Version]

### Performance Summary
| Metric | Metal FP32 | Metal FP16 | CPU | Metal vs CPU |
|--------|------------|------------|-----|--------------|
| Visits/sec | X | X | X | X% |
| Nodes/sec | X | X | X | X% |
| Memory Usage | X MB | X MB | X MB | - |

### Board Size Scaling
| Board Size | Visits/sec | Relative Performance |
|------------|------------|---------------------|
| 9x9 | X | 100% |
| 13x13 | X | X% |
| 19x19 | X | X% |

### Recommendations
- Best configuration: [Details]
- Optimal thread count: [Number]
- FP16 vs FP32 recommendation: [Choice]
```

## Comparative Analysis

### Compare with Other Backends
1. Download pre-built binaries for comparison
2. Run identical benchmarks
3. Create performance ratio charts

### Expected Performance Ranges
- Apple Silicon: 70-90% of CUDA performance
- Intel + AMD GPU: 40-60% of CUDA performance
- Intel + Intel GPU: 10-20% of CUDA performance

## Automation and CI Integration

### Automated Benchmark Script
```python
#!/usr/bin/env python3
import subprocess
import json
import time

def run_benchmark(config, duration=60):
    cmd = [
        './katago', 'benchmark',
        '-model', 'kata1-b18c384nbt-s9131461376-d4087399203.bin.gz',
        '-config', config,
        '-time', str(duration)
    ]
    result = subprocess.run(cmd, capture_output=True, text=True)
    # Parse output and return metrics
    return parse_benchmark_output(result.stdout)

def generate_report(results):
    # Generate markdown report
    pass

if __name__ == '__main__':
    configs = ['default', 'fp16', 'fp32', 'performance', 'efficiency']
    results = {}
    for config in configs:
        results[config] = run_benchmark(f'configs/{config}.cfg')
    generate_report(results)
```

## Next Steps

1. Execute benchmark plan on target hardware
2. Collect and analyze results
3. Create optimization recommendations
4. Document best practices for each hardware type
5. Integrate findings into KaTrain build process