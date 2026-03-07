2026-03-07 13:40
Status: #baby 
Tags: [[performance]]
# How the cache effects performance

As you may already know going to main memory is really slow compared to the **ADD** circuitry. Meaning that when we did the work with [[Solving for serial dependencies with SIMD]] we would've got anywhere close to the speeds we would've got to if we had to go to main memory.

When you have something like this
```c
ADD A, input[0]
```

The **A** values isn't going anywhere in a rolling sum. So **A** in this case, belongs to something called the **register file**. The register file feeds data into the instructions extremely quickly, but it's very small only able to store about 100 values or so.

The **input[0]** needs to fetch and it will first look to the **register file**, then the L1 (level1) cache, then the L2 cache, L3 cache if you're CPU has it then, main memory, then your hard disk.

It's important to note that each CPU you have core and each level cache. So on a intel skylake, you have L1 and L2 for each core and the L3 is shared between cores.

You can lose so much speed if you fall out the L1 cache. If you take the code we did for [[Solving for serial dependencies with SIMD]] and we take the original buffer size from 4096 to 32768 which falls out the the L2 cache we get a huge slow down. 

```c
int array[4096] = { /*random values in here*/ };
void __attribute__((target("avx2"))) addAVXSIMD(uint32_t count, uint32_t *input)
{
	__m256i sumA = _mm_setzero_si256();
	__m256i sumB = _mm_setzero_si256();
	__m256i sumC = _mm_setzero_si256();
	__m256i sumD = _mm_setzero_si256();
	count /= 32;
	while(count--)
	{
		sumA = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i]);
		sumB = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i + 8]);
		sumC = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i + 16]);
		sumD = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i + 24]);
		input += 32;
	}
	
	__m256i sumAB = _mm256_hadd_epi32(sumA, sumB);
	__m256i sumDC = _mm256_hadd_epi32(sumD, sumC);
	__m256i sum = _mm256_hadd_epi32(sumAB, sumDC);
	
	sum = _mm256_hadd_epi32(sum, sum);
	sum = _mm256_hadd_epi32(sum, sum);
	//This is lane segregation meaning that you only have access to 4 lanes at a time, not 8. 
	__m256i sums = _m256_permute2x128_si256(sum, sum, 1 | (1 << 4));
	sum = _mm256_add_epi32(sum, sums);
	
	return _mm256_svtsi256_si32(sum);
}
```

1) Buffer size of 4096 which fits in the L1 cache, the adds per clock is 13.38
2) Buffer size of 32768 which fits in the L2 cache, the adds per clock is 7.708
3) Buffer size of 262144 which fits in the L3 cache, the adds per clock is 4.003
4) Buffer size of 33554432 which fits in the main memory, the adds per clock is 1.4462

As you can see going from 13.38 to 1.4462 is terrible considering the work you have to go through to get this point. This is actually one of the hardest ones to improve improve performance but it can give a huge speed up if you can nail it.
# References
##### Main Notes
[[What is the cache]]
#### Source Notes
[[Caching]]