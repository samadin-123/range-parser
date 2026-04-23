# Evaluation Setup

This file is outside the editable surface. It defines how results are judged. Agents cannot modify the evaluator or the scoring logic — the evaluation is the trust boundary.

Consider defining more than one evaluation criterion. Optimizing for a single number makes it easy to overfit and silently break other things. A secondary metric or sanity check helps keep the process honest.

eval_cores: 1
eval_memory_gb: 1.0
prereq_command: npm install

## Setup

Install dependencies with `npm install`.

The project is written in plain JavaScript (no build step required). The `prereq_command` ensures dependencies are installed before running the benchmark.

## Run command

```bash
node -e "
const rangeParser = require('./index.js');

// Benchmark test cases covering various range formats
const testCases = [
  [1000, 'bytes=0-499'],
  [1000, 'bytes=40-80'],
  [1000, 'bytes=-400'],
  [1000, 'bytes=400-'],
  [1000, 'bytes=0-0'],
  [1000, 'bytes=40-80,81-90,-1'],
  [200, 'bytes=0-499,1000-,500-999'],
  [1000, 'bytes=   40-80 , 81-90 , -1 '],
  [150, 'bytes=0-4,90-99,5-75,100-199,101-102'],
  [200, 'bytes=500-600'],
  [200, 'bytes=x-100'],
  [200, 'bytes=100-x'],
  [1000, 'items=0-5']
];

const iterations = 100000;
const start = Date.now();

for (let i = 0; i < iterations; i++) {
  for (const [size, header] of testCases) {
    rangeParser(size, header);
  }
}

const elapsed = (Date.now() - start) / 1000;
const opsPerSec = (iterations * testCases.length) / elapsed;

console.log('ops_per_sec=' + Math.round(opsPerSec));
"
```

## Output format

The benchmark must print `METRIC=<number>` to stdout.

## Metric parsing

The CLI looks for `METRIC=<number>` or `ops_per_sec=<number>` in the output.

## Ground truth

The baseline metric measures operations per second (ops/sec) for parsing HTTP Range headers. The benchmark runs 100,000 iterations over 13 different test cases covering:
- Simple byte ranges (start-end)
- Suffix byte ranges (-n)
- Open-ended ranges (start-)
- Single byte ranges
- Multiple ranges in one header
- Invalid/unsatisfiable ranges
- Non-byte range units (items=)
- Headers with whitespace

Higher ops/sec indicates better performance.
