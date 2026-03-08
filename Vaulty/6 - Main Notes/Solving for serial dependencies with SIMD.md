2026-03-07 07:37
Status: #baby 
Tags: [[performance]]
# Solving for serial dependencies with SIMD

When you do things like [[Solving the serial dependency problem]] for scalar operations we basically have the same issue and type of solution for SIMD the only difference here is that we have a VPADDD or a PADDD for the addition. They can still be serially dependent. 

So lets unroll these loop
```c
int array[4096] = { /*random values in here*/ };
void __attribute__((target("avx2"))) addAVXSIMD(uint32_t count, uint32_t *input)
{
	__m256i sumA = _mm_setzero_si256();
	__m256i sumB = _mm_setzero_si256();
	for(uint32_t i = 0; i < count; i += 16)
	{
		sumA = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i]);
		sumB = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i + 8]);
	}
	
	__m256i sum = _mm256_hadd_epi32(sumA, sumB);
	
	sum = _mm256_hadd_epi32(sum, sum);
	sum = _mm256_hadd_epi32(sum, sum);
	//This is lane segregation meaning that you only have access to 4 lanes at a time, not 8. 
	__m256i sums = _m256_permute2x128_si256(sum, sum, 1 | (1 << 4));
	sum = _mm256_add_epi32(sum, sums);
	
	return _mm256_svtsi256_si32(sum);
}
```

```c
int array[4096] = { /*random values in here*/ };
void __attribute__((target("avx2"))) addAVXSIMD(uint32_t count, uint32_t *input)
{
	__m256i sumA = _mm_setzero_si256();
	__m256i sumB = _mm_setzero_si256();
	__m256i sumC = _mm_setzero_si256();
	__m256i sumD = _mm_setzero_si256();
	for(uint32_t i = 0; i < count; i += 32)
	{
		sumA = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i]);
		sumB = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i + 8]);
		sumC = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i + 16]);
		sumD = _mm256_add_epi32(sum, _mm256_loadu_si256((__m256*)&input[i + 24]);
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

These loops go for add per clock.
1) 9.43 for dual AVX
2) 11.01 for quad AVX
3) 13.38 for dual AVX base pointer

The point here is that IPC and SIMD multiply the effects on top of each other. This should be the main take away from this. Why? Well it's important to know how fast something can go to be aware of much improvement you can make to your code.
# References
##### Main Notes
[[How to use SIMD with a summation loop]]
[[Solving the serial dependency problem]]
#### Source Notes
[[Single Instruction, Multiple Data]]