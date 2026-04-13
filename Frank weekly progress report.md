### 2/20

**Worked on:**

- LMCache system bug fixes & improvements:   
* Fix 1 (hash speed) \- When identifying cached data, token sequences are converted to a hashable format. Previously this created one Python object per token; now it copies the raw bytes in bulk, which is significantly faster for long sequences.   
* Fix 2 (memory allocation) \- When saving a large request, each chunk used to request memory independently. If memory was full, each chunk would evict one old entry, wait 100ms, and retry \- so 16 chunks could mean 16 separate eviction+sleep cycles. Now all chunks declare their memory needs upfront and free space in bulk.  
* Fix 3 (thread locking) \- Inserting cached chunks into the in-memory store required grabbing a thread lock per chunk. Now the lock is grabbed once for the whole batch \- same work, fewer synchronization round-trips.  
* Fix 4 (GPU synchronization) \- Copying KV cache data from GPU to CPU used to wait for each chunk's transfer to finish before starting the next. Now all transfers are queued back-to-back on the GPU and we wait once at the end.

Above fixes should significantly improve LMCache’s performance when it comes to handling large chunks close to cache reaching full capacity. Before LMCache with smaller lower bound chunk size (128) outperforms larger (512) by \>2x, but this causes the model to hallucinate and sacrifices accuracy. With the fix larger chunk sizes should also be fast. 

- Line profiler (WIP):   
* Custom tool to measure wall-clock times of entry point, orchestrator, and GPU transfer layer each for identifying bottleneck with explicit CUDA synchronization.  
* Visualizer script for parsing log and printing summary \+ stats chart

**TODOs:**

- Complete line profiler implementation  
- Debug LMCache fixes  
- Run experiments to compare before and after of the fixes, verify fixes are working

### 2/27

**Worked on:**

- Debugged and visualized LMCache system fixes & improvements  
  - Large requests are handled much more efficiently: 

![][image1]

- Continued working on profiler and its integration  
  - Experiments still being run, job in queue

**TODOs:**

- Visualize & compare aggregated speedup for:   
  - no blend (prefix caching) before/after LMCache optimizations   
  - Cache blended before/after optimizations   
- Investigate why 70b won’t fit on 4 L40s  
- Investigate if randomly good speedup outliers near \~0 cache hit ratio are sometimes due to ipc/communication overhead, do this via running a 13b on 1 GPU  
- Performance benefits mostly are from system and implementation now, need to think about how to enhance caching benefit, or show when caching benefit \> system benefit  
  - Do a mock experiment with artificial data like the one for vLLM, but with larger requests, and test the entire spectrum of cache hit ratios 0 \-\> 100%  

### 2/27

**Worked on:**

- More profiler debugging and integration, now works with generic functions  
- Profiled save/load, storage get/put, etc.

![][image2]

- Per phase/layer wall-clock time for identifying bottlenecks. 

- \*\*\* LMCache/CacheBlend has major issue where it stops checking for cache hits immediately after seeing the first cache miss (e.g. ABC then ACB is all hits, but ABC then DBC is all miss). This is problematic for our agentic use case where we want caching to be position independent. Working on a fix right now

**TODOs:**

- Continue running verification & aggregated results experiment once fix is in place / we find that the issue is fundamentally tied to blending algo and need to workaround somehow

### 2/27

Meeting notes: 

1. Application side: comparing the coding performance of locally-hosted LLMs (with and without different caching algorithms) and APIs.  
2. System side: comparing the cache hit ratio / latency of different caching algorithms (both in the original CacheBlend/LMCache system and in his optimized version).

### 3/23

**Worked on:**

- Debugging:  
  - Low/no cache hit on simple request patterns  
    - Root cause: 5GB cache size is too small and disk offload was disabled  
  - Backwards compatibility changes for truly position-independent cache blending  
    - Redefined recomp\_ratio to handle diverging cached tokens, and missed tokens are always recomputed by default  
  - LMCache backend issues (FIXED as of 3/26):  
    - retrieve\_layer  does not pass correct disk backend location to layerwise\_batched\_get so disk lookup fails with KeyError  
    - Certain bugs exist for mixed CPU memory \+ disk usage for cache, fixing these WIP  
      - \*Likely this was never a proper feature nor was it tested  
      - Key lookup path from CPU \-\> disk not properly set  
      - When cache hit in disk, prefetch busy loops for CPU to evict until there is enough space instead of actively evicting to make room

**TODOs:**

- Now cache hits as expected, but TTFT does not scale with hit ratio  
  - Investigate root cause via profiler and optimize  
    - Hypothesis: CPU \-\> GPU PCIe bandwidth bottleneck? L40S uses gen 4 

### 4/3  
**Worked on:**

- Updated vllm adapter and lookup client to correctly support accurate cache hit statistics  
- Compared updated LMCache blend enabled vs no blend speedup  
  - Some outlier with \~0 cache hit rate still outperform/underperform no blend

Results:  
![][image3]

- Run single request profiler trace for near 0 outlier no-blend to see what bottlenecked  
- Think about how to adjust the profiler so:  
  - It also attaches to no blend runs  
  - It saves per-request trace and logs the outliers’ traces once run finishes

### 6/7
**Meeting notes:**

- Positional encoding worry given same old_position needed for cache hit reuse (e.g. ABC and ADC, C reuse requires len(B)=len(D))
  - Explained the Cacheblend algorithm does support positional encoding remap via high-variance KV token approximation
  - The issue blocking more flexible reuse is from system design with MemObj and GPU connector path (not passing old_position)

**TODOs:**

- 

### 7/24

**TODOs:**
- (X) in recommendation: explore breakeven point for token length minimum for caching benefit to supercede the constant overhead
  - no actual breakeven point unless with decent amount of cache hits
  - currently with the higher hit cluster, extrapolated breakeven length is ~2k
  - using this gate experimentally does reduce underperformers
- (X need verify) after fig 4.2 add this rec, say breakeven point, replot fig 4.3 given hypothesized, cut all requests below
- (X) locking investigation why there is lock thoroughly explain
  - Is because the cache put function holds lock per call. When updating a batch of memory objects to cache, this is done per object. Now batch helper actually only hold lock once
- (X) Cache hit rate improve before and after non-contig cache hit
- Fig 4.8 run multi-pass (like 8 or 16) for every instance to eliminate randomness induced (ISSUE: devtral small 2507 API is retired)
- (X) change 4.2 section wording performance -> accuracy
- (X need verify) add conclusion 