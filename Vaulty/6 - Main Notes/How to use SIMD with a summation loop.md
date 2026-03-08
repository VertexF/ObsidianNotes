2026-03-07 07:35
Status: #baby 
Tags: [[performance]]
# How to use SIMD with a summation loop

The code below shows you how to do these SIMD operations, it's important to note that data organisation is makes SIMD difficult. Meaning that if you care about how the data is organised you have to think carefully about how to use these operations. You might even sometimes have to rewrite algorithms to be able to use SIMD. We are going to use an summation loop which doesn't have this problem.

Lets use SIMD on this
```c
int array[4096] = { /*random values in here*/ };
void addNoSIMD(uint32_t count, uint32_t *input)
{
	int sum = 0;
	for(int i = 0; i < count; ++i)
	{
		sum += input[i];
	}
}
```

Here we have an SSE loop version
```c
int array[4096] = { /*random values in here*/ };
void __attribute__((target("ssse3"))) addSEESIMD(uint32_t count, uint32_t *input)
{
	__m128i sum = _mm_setzero_si128();
	for(uint32_t i = 0; i < count; i += 4)
	{
		sum = _mm_add_epi32(sum, _mm_load_si128((__m128*)&input[i]);
	}
	
	//These are horizontal adds.
	sum = _mm_hadd_epi32(sum, sum);
	sum = _mm_hadd_epi32(sum, sum);
	
	return _mm_svtsi128_si32(sum);
}
```

Here we have the AVX loop version
```c
int array[4096] = { /*random values in here*/ };
void __attribute__((target("avx2"))) addAVXSIMD(uint32_t count, uint32_t *input)
{
	__m256i sum = _mm_setzero_si256();
	for(uint32_t i = 0; i < count; i += 8)
	{
		sum = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i]);
	}
	
	sum = _mm256_hadd_epi32(sum, sum);
	sum = _mm256_hadd_epi32(sum, sum);
	//This is lane segregation meaning that you only have access to 4 lanes at a time, not 8. 
	__m256i sums = _m256_permute2x128_si256(sum, sum, 1 | (1 << 4));
	sum = _mm256_add_epi32(sum, sums);
	
	return _mm256_svtsi256_si32(sum);
}
```

Note there are ways to make this look less terrible. Thank intel for this. Also I'm skipping explaining this because we are not learning what the SIMD operations actually are.

For the SSE you get up to 3.12 adds per cycle and for AVX you get 7.04 adds per cycle.
# References
##### Main Notes
[[Introduction to SIMD]]
[[SIMD bit-width]]
[[Solving for serial dependencies with SIMD]]
#### Source Notes
[[Single Instruction, Multiple Data]]