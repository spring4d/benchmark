# Benchmark

Delphi port of [Google Benchmark](https://github.com/google/benchmark) - v1.5.3

Hello everyone - you are amongst the few selected people to get a preview to help test things out and give some feedback, welcome!

The goal of this project is to provide a powerful benchmarking library for Delphi developers and stay as close as possible to its origin.
That means I will be reluctant to add a lot of features but aim at making work what works with its origin.

This readme is WIP - most features are implemented and working - currently missing:

- file output (csv done, json missing)

The content of the README is mostly copied from the Google Benchmark project with some first modifications to show the correct Syntax in Delphi.

I developed and tested with Delphi 10.4.2 but I am aiming to target XE7 and higher. The syntax for the benchmark code is using inline variables.
If on an older version the loop variable needs to be of type `TState.TValue`. I tested on Windows 10 but it should run on all Windows versions Delphi supports. At some point I might add Linux support. Not sure about other platforms though.

I added support for FPC - tested with whatever FPC version comes with Lazarus 2.0.12 on Windows though - support for other platforms missing. I guess Linux will be interesting, not sure about others.

Please feel free to open issues for questions or bugs you encounter. Happy benchmarking!



A library to benchmark code snippets, similar to unit tests. Example:

```delphi
uses
  Spring.Benchmark;

procedure BM_SomeFunction(const state: TState);
begin
  // Perform setup here
  for var _ in state do
  begin
    // This code gets timed
    SomeFunction();
  end;
end;

begin
  // Register the function as a benchmark
  Benchmark(BM_SomeFunction, 'SomeFunction');
  // Run the benchmark
  Benchmark_Main;
end.
```

To get started, simply add `Spring.Benchmark` to the uses.

## Requirements

The library can be used with Delphi XE7 or higher and FPC 3.2.0 or higher.

## Installation

Actually there is no installation. Simply add the unit to the uses clause and you are ready to go.

## User Guide

### Command Line

[Output Formats](#output-formats)

[Output Files](#output-files)

[Running Benchmarks](#running-benchmarks)

[Running a Subset of Benchmarks](#running-a-subset-of-benchmarks)

[Result Comparison](#result-comparison)

### Library

[Runtime and Reporting Considerations](#runtime-and-reporting-considerations)

[Passing Arguments](#passing-arguments)

[Calculating Asymptotic Complexity](#asymptotic-complexity)

[Custom Counters](#custom-counters)

[Multithreaded Benchmarks](#multithreaded-benchmarks)

[CPU Timers](#cpu-timers)

[Manual Timing](#manual-timing)

[Setting the Time Unit](#setting-the-time-unit)

[Preventing Optimization](#preventing-optimization)

[Reporting Statistics](#reporting-statistics)

[Custom Statistics](#custom-statistics)

[Using RegisterBenchmark](#using-register-benchmark)

[Exiting with an Error](#exiting-with-an-error)


<a name="output-formats" />

### Output Formats

The library supports multiple output formats. Use the
`--benchmark_format=<console|json|csv>` flag (or set the
`BENCHMARK_FORMAT=<console|json|csv>` environment variable) to set
the format type. `console` is the default format.

The Console format is intended to be a human readable format. By default
the format generates color output. Example tabular output looks like:

```
Benchmark                               Time(ns)    CPU(ns) Iterations
----------------------------------------------------------------------
BM_SetInsert/1024/1                        28928      29349      23853  133.097kB/s   33.2742k items/s
BM_SetInsert/1024/8                        32065      32913      21375  949.487kB/s   237.372k items/s
BM_SetInsert/1024/10                       33157      33648      21431  1.13369MB/s   290.225k items/s
```

The JSON format outputs human readable json split into two top level attributes.
The `context` attribute contains information about the run in general, including
information about the CPU and the date.
The `benchmarks` attribute contains a list of every benchmark run. Example json
output looks like:

```json
{
  "context": {
    "date": "2015/03/17-18:40:25",
    "num_cpus": 40,
    "mhz_per_cpu": 2801,
    "cpu_scaling_enabled": false,
    "build_type": "debug"
  },
  "benchmarks": [
    {
      "name": "BM_SetInsert/1024/1",
      "iterations": 94877,
      "real_time": 29275,
      "cpu_time": 29836,
      "bytes_per_second": 134066,
      "items_per_second": 33516
    },
    {
      "name": "BM_SetInsert/1024/8",
      "iterations": 21609,
      "real_time": 32317,
      "cpu_time": 32429,
      "bytes_per_second": 986770,
      "items_per_second": 246693
    },
    {
      "name": "BM_SetInsert/1024/10",
      "iterations": 21393,
      "real_time": 32724,
      "cpu_time": 33355,
      "bytes_per_second": 1199226,
      "items_per_second": 299807
    }
  ]
}
```

The CSV format outputs comma-separated values. The `context` is output on stderr
and the CSV itself on stdout. Example CSV output looks like:

```
name,iterations,real_time,cpu_time,bytes_per_second,items_per_second,label
"BM_SetInsert/1024/1",65465,17890.7,8407.45,475768,118942,
"BM_SetInsert/1024/8",116606,18810.1,9766.64,3.27646e+06,819115,
"BM_SetInsert/1024/10",106365,17238.4,8421.53,4.74973e+06,1.18743e+06,
```

<a name="output-files" />

### Output Files

Write benchmark results to a file with the `--benchmark_out=<filename>` option
(or set `benchmark_out`). Specify the output format with
`--benchmark_out_format={json|console|csv}` (or set
`benchmark_out_format={json|console|csv}`).

Specifying `--benchmark_out` does not suppress the console output.

<a name="running-benchmarks" />

### Running Benchmarks

Benchmarks are executed by running the produced binaries. Benchmarks binaries,
by default, accept options that may be specified either through their command
line interface or by setting environment variables before execution. For every
`--option_flag=<value>` CLI switch, a corresponding environment variable
`option_flag=<value>` exist and is used as default if set (CLI switches always
 prevails). A complete list of CLI options is available running benchmarks
 with the `--help` switch.

<a name="running-a-subset-of-benchmarks" />

### Running a Subset of Benchmarks

The `--benchmark_filter=<regex>` option (or `benchmark_filter=<regex>`
environment variable) can be used to only run the benchmarks that match
the specified `<regex>`. For example:

```bash
$ ./run_benchmarks.exe --benchmark_filter=QuickSort
Run on (4 X 3410,01 MHz CPU s)
CPU Caches:
  L1 Data 32 K (x4)
  L1 Instruction 32 K (x4)
  L2 Unified 256 K (x4)
  L3 Unified 6144 K (x1)
---------------------------------------------------------
Benchmark               Time             CPU   Iterations
---------------------------------------------------------
QuickSort/1024       1679 ns         1674 ns       448000
QuickSort/2048       3296 ns         3296 ns       213333
QuickSort/4096       6542 ns         6557 ns       112000
QuickSort/8192      13081 ns        12277 ns        56000
```

<a name="result-comparison" />

### Result comparison

It is possible to compare the benchmarking results.
See [Additional Tooling Documentation](https://github.com/google/benchmark/blob/main/docs/tools.md)

<a name="runtime-and-reporting-considerations" />

### Runtime and Reporting Considerations

When the benchmark binary is executed, each benchmark function is run serially.
The number of iterations to run is determined dynamically by running the
benchmark a few times and measuring the time taken and ensuring that the
ultimate result will be statistically stable. As such, faster benchmark
functions will be run for more iterations than slower benchmark functions, and
the number of iterations is thus reported.

In all cases, the number of iterations for which the benchmark is run is
governed by the amount of time the benchmark takes. Concretely, the number of
iterations is at least one, not more than 1e9, until CPU time is greater than
the minimum time, or the wallclock time is 5x minimum time. The minimum time is
set per benchmark by calling `MinTime` on the registered benchmark object.

Average timings are then reported over the iterations run. If multiple
repetitions are requested using the `--benchmark_repetitions` command-line
option, or at registration time, the benchmark function will be run several
times and statistical results across these repetitions will also be reported.

As well as the per-benchmark entries, a preamble in the report will include
information about the machine on which the benchmarks are run.

<a name="passing-arguments" />

### Passing Arguments

Sometimes a family of benchmarks can be implemented with just one routine that
takes an extra argument to specify which one of the family of benchmarks to
run. For example, the following code defines a family of benchmarks for
measuring the speed of `TArray.Sort` calls of different lengths:

```delphi
procedure BM_QuickSort(const state: TState);
var
  count: Integer;
  numbers: TArray<Integer>;
begin
  count := state[0];
  numbers := TEnumerable.Range(1, count).Shuffled.ToArray;
  for var _ in state do
    TArray.Sort<Integer>(numbers);
end;

Benchmark(BM_QuickSort, 'QuickSort').Arg(8).Arg(64).Arg(512).Arg(1024).Arg(8192);
```

The preceding code is quite repetitive, and can be replaced with the following
short-hand. The following invocation will pick a few appropriate arguments in
the specified range and will generate a benchmark for each such argument.

```delphi
Benchmark(BM_QuickSort, 'QuickSort').Range(8, 8192);
```

By default the arguments in the range are generated in multiples of eight and
the command above selects [ 8, 64, 512, 1024, 8192 ]. In the following code the
range multiplier is changed to multiples of two.

```delphi
Benchmark(BM_QuickSort, 'QuickSort').RangeMultiplier(2).Range(8, 8192);
```

Now arguments generated are [ 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192 ].

The preceding code shows a method of defining a sparse range. The following
example shows a method of defining a dense range. It is then used to benchmark
the performance of `FillChar` initialization for uniformly increasing sizes.

```delphi
procedure BM_DenseRange(const state: TState);
var
  n: Integer;
  p: Pointer;
begin
  n := state[0];
  GetMem(p, n * 4);
  for var _ in state do
    FillChar(p^, n * 4, 0);
  FreeMem(p);  
end;

Benchmark(BM_DenseRange).DenseRange(0, 1024, 128);
```

Now arguments generated are [ 0, 128, 256, 384, 512, 640, 768, 896, 1024 ].

You might have a benchmark that depends on two or more inputs. For example, the
following code defines a family of benchmarks for measuring the speed of set
insertion.

```delphi
procedure BM_SetInsert(const state: TState);
var
  data: ISet<Integer>;
begin
  for var _ in state do
  begin
    state.PauseTiming;
    data := ConstructRandomSet(state[0]);
    state.ResumeTiming;
    for var j := 0 to state[1]-1 do
      data.Add(RandomNumber());
  end;
end;

Benchmark(BM_SetInsert, 'SetInsert')
    .Args([1024, 128])
    .Args([2048, 128])
    .Args([4096, 128])
    .Args([8192, 128])
    .Args([1024, 512])
    .Args([2048, 512])
    .Args([4096, 512])
    .Args([8192, 512]);
```

The preceding code is quite repetitive, and can be replaced with the following
short-hand. The following method will pick a few appropriate arguments in the
product of the two specified ranges and will generate a benchmark for each such
pair.

```delphi
Benchmark(BM_SetInsert, 'SetInsert').Ranges([Range(1024, 8192), Range(128, 512)]);
```

Some benchmarks may require specific argument values that cannot be expressed
with `Ranges`. In this case, `ArgsProduct` offers the ability to generate a
benchmark input for each combination in the product of the supplied vectors.

```delphi
Benchmark(BM_SetInsert, 'SetInsert')
    .ArgsProduct([[1024, 3072, 8192], [20, 40, 60, 80]]);
// would generate the same benchmark arguments as
Benchmark(BM_SetInsert)
    .Args([1024, 20])
    .Args([3072, 20])
    .Args([8192, 20])
    .Args([1024, 40])
    .Args([3072, 40])
    .Args([8192, 40])
    .Args([1024, 60])
    .Args([3072, 60])
    .Args([8192, 60])
    .Args([1024, 80])
    .Args([3072, 80])
    .Args([8192, 80]);
```

For more complex patterns of inputs, passing a custom function to `Apply` allows
programmatic specification of an arbitrary set of arguments on which to run the
benchmark. The following example enumerates a dense range on one parameter,
and a sparse range on the second.

```delphi
procedure CustomArguments(const benchmark: TBenchmark);
begin
  for var i := 0 to 10 do
  begin
    var j := 32;
    repeat
      benchmark.Args([i, j]);
      j := j * 8;
    until j > 1024*1024;
  end;
end;    

Benchmark(BM_SetInsert, 'SetInsert').Apply(CustomArguments);
```

<a name="asymptotic-complexity" />

### Calculating Asymptotic Complexity (Big O)

Asymptotic complexity might be calculated for a family of benchmarks. The
following code will calculate the coefficient for the high-order term in the
running time and the normalized root-mean square error of string comparison.

```delphi
procedure BM_StringCompare(const state: TState);
var
  s1, s2: string;
begin
  s1 := StringOfChar('-', state[0]);
  s2 := StringOfChar('-', state[0]);
  for var _ in state do
    string.compare(s1, s2);
  state.SetComplexityN(state[0]);
end;

begin
  Benchmark(BM_StringCompare, 'StringCompare')
    .RangeMultiplier(2).Range(1024, 1 shl 18).Complexity(oN);
```

As shown in the following invocation, asymptotic complexity might also be
calculated automatically.

```delphi
Benchmark(BM_StringCompare, 'StringCompare')
    .RangeMultiplier(2).Range(1024, 1 shl 18).Complexity;
```

The following code will specify asymptotic complexity with an anonymous method,
that might be used to customize high-order term calculation.

```delphi
Benchmark(BM_StringCompare, 'StringCompare').RangeMultiplier(2)
    .Range(1024, 1 shl 18).Complexity(function(const n: TIterationCount): Double begin Result := n end);
```

<a name="custom-counters" />

### Custom Counters

You can add your own counters with user-defined names. The example below
will add columns "Foo", "Bar" and "Baz" in its output:

```delphi
procedure UserCountersExample1(const state: TState);
var
  numFoos, numBars, numBazs: Double;
begin
  numFoos := 0; numBars := 0; numBazs := 0;
  for var _ in state do
    numFoos := numFoos + 1;
    // ... count Foo,Bar,Baz events
  state.Counters['Foo'] := numFoos;
  state.Counters['Bar'] := numBars;
  state.Counters['Baz'] := numBazs;
end;
```

The `state.Counters` property provides write acces via `string` keys
to `TCounter` values. The latter is a `Double`-like record, via an implicit
conversion to `Double`.

In multithreaded benchmarks, each counter is set on the calling thread only.
When the benchmark finishes, the counters from each thread will be summed;
the resulting sum is the value which will be shown for the benchmark.

The `Counter` function accepts three parameters: the value as a `Double`
; a set of flags which allows you to show counters as rates, and/or as per-thread
iteration, and/or as per-thread averages, and/or iteration invariants,
and/or finally inverting the result; and a flag specifying the 'unit' - i.e.
is 1k a 1000 (default, `kIs1000`), or 1024 (`kIs1024`)?

```delphi
  // sets a simple counter
  state.Counters['Foo'] := numFoos;

  // Set the counter as a rate. It will be presented divided
  // by the duration of the benchmark.
  // Meaning: per one second, how many 'foo's are processed?
  state.Counters['FooRate'] := Counter(numFoos, [kIsRate]);

  // Set the counter as a rate. It will be presented divided
  // by the duration of the benchmark, and the result inverted.
  // Meaning: how many seconds it takes to process one 'foo'?
  state.Counters['FooInvRate'] := Counter(numFoos, [kIsRate, kInvert]);

  // Set the counter as a thread-average quantity. It will
  // be presented divided by the number of threads.
  state.Counters['FooAvg'] := Counter(numFoos, [kAvgThreads]);

  // There's also a combined flag:
  state.Counters['FooAvgRate'] := Counter(numFoos, kAvgThreadsRate);

  // This says that we process with the rate of state.Range[0] bytes every iteration:
  state.Counters['BytesProcessed'] := Counter(state.Range[0], kIsIterationInvariantRate, kIs1024);
```

#### Counter Reporting

When using the console reporter, by default, user counters are printed at
the end after the table, the same way as ``bytes_processed`` and
``items_processed``. This is best for cases in which there are few counters,
or where there are only a couple of lines per benchmark. Here's an example of
the default output:

```
------------------------------------------------------------------------------
Benchmark                        Time           CPU Iterations UserCounters...
------------------------------------------------------------------------------
BM_UserCounter/threads:8      2248 ns      10277 ns      68808 Bar=16 Bat=40 Baz=24 Foo=8
BM_UserCounter/threads:1      9797 ns       9788 ns      71523 Bar=2 Bat=5 Baz=3 Foo=1024m
BM_UserCounter/threads:2      4924 ns       9842 ns      71036 Bar=4 Bat=10 Baz=6 Foo=2
BM_UserCounter/threads:4      2589 ns      10284 ns      68012 Bar=8 Bat=20 Baz=12 Foo=4
BM_UserCounter/threads:8      2212 ns      10287 ns      68040 Bar=16 Bat=40 Baz=24 Foo=8
BM_UserCounter/threads:16     1782 ns      10278 ns      68144 Bar=32 Bat=80 Baz=48 Foo=16
BM_UserCounter/threads:32     1291 ns      10296 ns      68256 Bar=64 Bat=160 Baz=96 Foo=32
BM_UserCounter/threads:4      2615 ns      10307 ns      68040 Bar=8 Bat=20 Baz=12 Foo=4
BM_Factorial                    26 ns         26 ns   26608979 40320
BM_Factorial/real_time          26 ns         26 ns   26587936 40320
BM_CalculatePiRange/1           16 ns         16 ns   45704255 0
BM_CalculatePiRange/8           73 ns         73 ns    9520927 3.28374
BM_CalculatePiRange/64         609 ns        609 ns    1140647 3.15746
BM_CalculatePiRange/512       4900 ns       4901 ns     142696 3.14355
```

If this doesn't suit you, you can print each counter as a table column by
passing the flag `--benchmark_counters_tabular=true` to the benchmark
application. This is best for cases in which there are a lot of counters, or
a lot of lines per individual benchmark. Note that this will trigger a
reprinting of the table header any time the counter set changes between
individual benchmarks. Here's an example of corresponding output when
`--benchmark_counters_tabular=true` is passed:

```
---------------------------------------------------------------------------------------
Benchmark                        Time           CPU Iterations    Bar   Bat   Baz   Foo
---------------------------------------------------------------------------------------
BM_UserCounter/threads:8      2198 ns       9953 ns      70688     16    40    24     8
BM_UserCounter/threads:1      9504 ns       9504 ns      73787      2     5     3     1
BM_UserCounter/threads:2      4775 ns       9550 ns      72606      4    10     6     2
BM_UserCounter/threads:4      2508 ns       9951 ns      70332      8    20    12     4
BM_UserCounter/threads:8      2055 ns       9933 ns      70344     16    40    24     8
BM_UserCounter/threads:16     1610 ns       9946 ns      70720     32    80    48    16
BM_UserCounter/threads:32     1192 ns       9948 ns      70496     64   160    96    32
BM_UserCounter/threads:4      2506 ns       9949 ns      70332      8    20    12     4
--------------------------------------------------------------
Benchmark                        Time           CPU Iterations
--------------------------------------------------------------
BM_Factorial                    26 ns         26 ns   26392245 40320
BM_Factorial/real_time          26 ns         26 ns   26494107 40320
BM_CalculatePiRange/1           15 ns         15 ns   45571597 0
BM_CalculatePiRange/8           74 ns         74 ns    9450212 3.28374
BM_CalculatePiRange/64         595 ns        595 ns    1173901 3.15746
BM_CalculatePiRange/512       4752 ns       4752 ns     147380 3.14355
BM_CalculatePiRange/4k       37970 ns      37972 ns      18453 3.14184
BM_CalculatePiRange/32k     303733 ns     303744 ns       2305 3.14162
BM_CalculatePiRange/256k   2434095 ns    2434186 ns        288 3.1416
BM_CalculatePiRange/1024k  9721140 ns    9721413 ns         71 3.14159
BM_CalculatePi/threads:8      2255 ns       9943 ns      70936
```

Note above the additional header printed when the benchmark changes from
``BM_UserCounter`` to ``BM_Factorial``. This is because ``BM_Factorial`` does
not have the same counter set as ``BM_UserCounter``.

<a name="multithreaded-benchmarks"/>

# TBD / needs translation

### Multithreaded Benchmarks

In a multithreaded test (benchmark invoked by multiple threads simultaneously),
it is guaranteed that none of the threads will start until all have reached
the start of the benchmark loop, and all will have finished before any thread
exits the benchmark loop. (This behavior is also provided by the `KeepRunning()`
API) As such, any global setup or teardown can be wrapped in a check against the thread
index:

```delphi
procedure BM_MultiThreaded(const state: TState);
begin
  if state.ThreadIndex = 0 then
    // Setup code here.

  for var _ in state do
    // Run the test as normal.

  if state.ThreadIndex = 0 then
    // Teardown code here.
end;

Benchmark(BM_MultiThreaded, 'BM_MultiThreaded').Threads(2);
```

If the benchmarked code itself uses threads and you want to compare it to
single-threaded code, you may want to use real-time ("wallclock") measurements
for latency comparisons:

```delphi
Benchmark(BM_test, 'test').Range(8, 8 shl 10).UseRealTime;
```

Without `UseRealTime`, CPU time is used by default.

<a name="cpu-timers" />

### CPU Timers

By default, the CPU timer only measures the time spent by the main thread.
If the benchmark itself uses threads internally, this measurement may not
be what you are looking for. Instead, there is a way to measure the total
CPU usage of the process, by all the threads.

```delphi
// Measure the time spent by the main thread, use it to decide for how long to
// run the benchmark loop. Depending on the internal implementation detail may
// measure to anywhere from near-zero (the overhead spent before/after work
// handoff to worker thread[s]) to the whole single-thread time.
Benchmark(BM_ParallelFor, 'ParallelFor').Range(8, 8 shl 10);

// Measure the user-visible time, the wall clock (literally, the time that
// has passed on the clock on the wall), use it to decide for how long to
// run the benchmark loop. This will always be meaningful, and will match the
// time spent by the main thread in single-threaded case, in general decreasing
// with the number of internal threads doing the work.
Benchmark(BM_ParallelFor, 'ParallelFor').Range(8, 8 shl 10).UseRealTime;

// Measure the total CPU consumption, use it to decide for how long to
// run the benchmark loop. This will always measure to no less than the
// time spent by the main thread in single-threaded case.
Benchmark(BM_ParallelFor, 'ParallelFor').Range(8, 8 shl 10).MeasureProcessCPUTime;

// A mixture of the last two. Measure the total CPU consumption, but use the
// wall clock to decide for how long to run the benchmark loop.
Benchmark(BM_ParallelFor, 'ParallelFor').Range(8, 8 shl 10).MeasureProcessCPUTime.UseRealTime;
```

#### Controlling Timers

Normally, the entire duration of the work loop (`for var _ in state do`)
is measured. But sometimes, it is necessary to do some work inside of
that loop, every iteration, but without counting that time to the benchmark time.
That is possible, although it is not recommended, since it has high overhead.

```delphi
procedure BM_SetInsert_With_Timer_Control(const state: TState);
var
  data: ISet<Integer>;
begin
  for var _ in state do
  begin
    state.PauseTiming; // Stop timers. They will not count until they are resumed.
    data = ConstructRandomSet(state.Range[0]); // Do something that should not be measured
    state.ResumeTiming; // And resume timers. They are now counting again.
    // The rest will be measured.
    for var j := 0 to state.range[1] - 1 do
      data.insert(RandomNumber());
  }
}
Benchmark(BM_SetInsert_With_Timer_Control, 'SetInsert').Ranges([Range(1 shl 10, 8 shl 10), Range(128, 512)]);
```

<a name="manual-timing" />

### Manual Timing

For benchmarking something for which neither CPU time nor real-time are
correct or accurate enough, completely manual timing is supported using
the `UseManualTime` function.

When `UseManualTime` is used, the benchmarked code must call
`SetIterationTime` once per iteration of the benchmark loop to
report the manually measured time.

An example use case for this is benchmarking GPU execution (e.g. OpenCL
or CUDA kernels, OpenGL or Vulkan or Direct3D draw calls), which cannot
be accurately measured using CPU time or real-time. Instead, they can be
measured accurately using a dedicated API, and these measurement results
can be reported back with `SetIterationTime`.

<a name="setting-the-time-unit" />

### Setting the Time Unit

If a benchmark runs a few milliseconds it may be hard to visually compare the
measured times, since the output data is given in nanoseconds per default. In
order to manually set the time unit, you can specify it manually:

```delphi
Benchmark(BM_test, 'test').TimeUnit(kMillisecond);
```

<a name="preventing-optimization" />

### Preventing Optimization

To prevent a value or expression from being optimized away by the compiler
make sure to use it after the benchmark loop

```delphi
procedure DoNotOptimize(i: Integer); // make sure to not use INLINE AUTO
begin
end;

procedure BM_test(const state: TState);
begin
  for var _  in state do
  begin
    int x := 0;
    for var i := 0 to 63 do
      Inc(x, i);
    DoNotOptimize(x);
  end;
end;
```

<a name="reporting-statistics" />

### Statistics: Reporting the Mean, Median and Standard Deviation of Repeated Benchmarks

By default each benchmark is run once and that single result is reported.
However benchmarks are often noisy and a single result may not be representative
of the overall behavior. For this reason it's possible to repeatedly rerun the
benchmark.

The number of runs of each benchmark is specified globally by the
`--benchmark_repetitions` flag or on a per benchmark basis by calling
`Repetitions` on the registered benchmark object. When a benchmark is run more
than once the mean, median and standard deviation of the runs will be reported.

Additionally the `--benchmark_report_aggregates_only={True|False}`,
`--benchmark_display_aggregates_only={True|False}` flags or
`ReportAggregatesOnly(Boolean)`, `DisplayAggregatesOnly(Bool)` functions can be
used to change how repeated tests are reported. By default the result of each
repeated run is reported. When `report aggregates only` option is `True`,
only the aggregates (i.e. mean, median and standard deviation, maybe complexity
measurements if they were requested) of the runs is reported, to both the
reporters - standard output (console), and the file.
However when only the `display aggregates only` option is `True`,
only the aggregates are displayed in the standard output, while the file
output still contains everything.
Calling `ReportAggregatesOnly(Boolean)` / `DisplayAggregatesOnly(Boolean)` on a
registered benchmark object overrides the value of the appropriate flag for that
benchmark.

<a name="custom-statistics" />

### Custom Statistics

While having mean, median and standard deviation is nice, this may not be
enough for everyone. For example you may want to know what the largest
observation is, e.g. because you have some real-time constraints. This is easy.
The following code will specify a custom statistic to be calculated, defined
by a lambda function.

```delphi
procedure BM_spin_empty(const state: TState);
begin
  for var _ in state do
  begin
    for var x := 0 to state.Range[0] - 1 do
      DoNotOptimize(x);
  end;
end;

Benchmark(BM_spin_empty, 'spin_empty')
  .ComputeStatistics('max',
  function(const values: array of Double): Double
  begin
    Result := TEnumerable.From(values).Max;
  end).Arg(512).Repetitions(10);
```

<a name="using-register-benchmark" />

### Using RegisterBenchmark(name, fn, args...)

Currently not supported in Spring.Benchmark - use `Benchmark()`.

<a name="exiting-with-an-error" />

### Exiting with an Error

When errors caused by external influences, such as file I/O and network
communication, occur within a benchmark the
`TState.SkipWithError(const msg: string)` function can be used to skip that run
of benchmark and report the error. Note that only future iterations of the
`KeepRunning()` are skipped. For the for-in version of the benchmark loop
Users must explicitly `break` the loop, otherwise all iterations will be performed.
Users may explicitly return to exit the benchmark immediately.

The `SkipWithError(...)` function may be used at any point within the benchmark,
including before and after the benchmark loop. Moreover, if `SkipWithError(...)`
has been used, it is not required to reach the benchmark loop and one may return
from the benchmark function early.

For example:

```delphi
procedure void BM_test(const state: TState);
begin
  var resource := GetResource();
  if not resource.isvalid then
    state.SkipWithError('Resource is not valid!');
    // KeepRunning() loop will not be entered.
  
  while state.KeepRunning do
  begin
    var data = resource.ReadData;
    if not resource.isvalid then
    begin
      state.SkipWithError('Failed to read data!');
      Break; // Needed to skip the rest of the iteration.
    end
    doStuff(data);
  end;
end;

procedure BM_test_forin(const state: TState);
begin
  var resource = GetResource();
  if not resource.isvalid then
  begin
    state.SkipWithError('Resource is not good!');
    Exit; // Early return is allowed when SkipWithError() has been used.
  end;
  
  for var _ in state do
  begin
    var data := resource.ReadData;
    if not resource.isvalid then
    begin
      state.SkipWithError('Failed to read data!');
      Break; // REQUIRED to prevent all further iterations.
    end;
    doStuff(data);
  end;
end;
```
