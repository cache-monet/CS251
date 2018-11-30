### OVERVIEW
---
Fast memory - data/instruction cache

Faster memories are more expensive and larger memories can be slower

Smaller faster memories are not unlimited
* solution: move items to smaller, faster memories when they are needed

Rationale: **Locality of reference**
* **Temporal**: Once accessed, likely to be access again and soon
  * **Temporal locality** if a data location is referenced once ...
* **Spatial**: items *nearby* are also likely to be accessed
  * **Spation locality** if a data location is referenced, data locations with nearby addresses...

**Key idea** YOU WILL SPEND 80% OF YOUR TIME IN 20% OF THE CODE

cache is fast for 2 reasons
1. it is *close*
2. it's *small enough* such that accessing any item in it will be fast
  
**Clock rate**: $\frac{1}{time of one clock cycle}$ 

[Memory Hierarchy](./pictures/mem_hierarchy.png)
* As we move away from the CPU, memories get slower (longer to access) and cheaper($)
  * 8GB SSD vs 8GB of RAM
* **DRAM** (dynamic RAM) main memory is implemented from this, substantially slower than SRAM
* **SRAM** (static RAM) for levels closer to processor (caches)
* The magnetic disk is SSD (largest and slowest in the hierarchy)

[Memory Layout](./pictures/mem_layout.png)
  * L1 cache is broken down to
    1. **instr cache** Instruction Mem
    2. **data cache** Data memory
  * L2 contains unified cache

---
### Accessing Data
---
[Memory Pyramid](./pictures/mem_levels)
* Items not in the top level are *brought up once they are requested*
* Data is only copied between **adjacent levels**
  * upper level (closer to processor) is smaller and faster than the lower level
  * if the data requested by the processor is in the upper level then it's a **hit**
  * if the data requested by the processor is not in upper level then it's a **miss**
* **block** the smallest unit of information
* **hit** information is present when requested
* **miss** information is not present

**Cache** level between CPU and main memory
* How do we know if data item is in the cache?
* How do we find it?
  
**Direct mapped** one-to-one mapping between memory location and cache location

Typical mapping: **(block address) modulo (number of cache blocks)**
ie: block is 1 word, cache size = 8, just take the **lower 3 bits** of word address


#### Mappings
---

**Direct mapped** there is only **one place** where blocks in Main mem can be mapped to cache
* if a block needs to go in cache, it can only go in this location, if something else is there, **it will need to be moved out**

[Direct mapped cache example](./pictures/direct_mapped.png)
* left table is CPU accessing memory
* right table is contents of cache

#### Testing for Cache hit miss
---

32-bit example
* 14 bits (*addr[16-29]*) are used as index 
* Compare the tag associated with the index with the last 16 bits (*addr[0-15]*
* HIT $\Leftrightarrow$ table[index].valid $\wedge$ addr[0-16] $=$ table[index].tag
* [Visual example](./pictures/test_hit_miss.png)

Issues with direct (one way set associative)
* multiple address maps to the same cache location (lots of kicking out and misses)
* **IDEA** instead of one location, allow in index to map to a **set of locations** (this will increase hit rate)
  
[Two way associative cache](./pictures/two_way.png)

[Miss rate benchmarks](./pictures/miss_rate_benchmark.png)

**Replacement Scheme** What to replace in cache when there's no more room?
* Overwrite the **LRU** (Least recently used) location


**AMAT** average memory access time
* $=$ time for a hit + ( miss rate  $\times$ miss penalty )
  * Miss penality clockcycles it takes to go into RAM and fetch the data

In direct mapping, upon a cache miss we bring one word of data from RAM while CPU waits
* this is inefficient
* **Better**  upon cache miss bring in **multiple words** from RAM
  * it takes a little longer to bring in more words (RAM access is 100cc, 4 words is 104cc, 8 words 108cc)
  * these words should be near the address (b/c spatial locality)
* [increase block size from 1 word to 4](./pictures/increase_block.png)
  * we use **block offset bits** to decide which of the 4 words in the block we want
  * [Example](./picture/block_example.png)
    * first address is a miss, but we retrieve 3 more addresses which turns out to be HITS.

**Handling cache misses**
* one miss: **stall entire processor** until item is fetched
* on write: written item goes into cache (usually)
* write-through: **write item back into RAM**
* write-back: write item **only** into **cache**, RAM is tempoarily inconsistent
  * Data block will have a **dirty bit** indicating that data was updated inconsistent with RAM
  * wait until word is kicked out of cache, then **update ram with new value of data**
  * This is good because **the entire block is written to RAM** (good when there are many sw)
  * Less RAM accesses than write-through
* Could have separate instr and data caches (incr bandwith)

[Block read example](./pictures/block_read.png)