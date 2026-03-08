# Reference https://www.computerenhance.com/p/single-instruction-multiple-data

##### Introduction to SIMD
Single instruction, multiple data is all about trying to reduce the amount of instructions you are trying to get the CPU to do all while doing the same amount of work.

The idea being is that if we have this ASM
```c
ADD A, input[0]
ADD B, input[1]
ADD C, input[2]
ADD D, input[3]
```

CPU designer have had this idea because typically most people do a tonne of the same operations. Maybe there is a way to do 1 operation for all these **ADD's** in one go with the data and writing into registers at the same time.
```c
SUPER_ADD A,B,C,D input[0], input[1], input[0], input[1]
```

So x64 ASM you have the SSE, or Streaming SIMD Extensions which came out first to do this. So the most basic SSE instruction is **PADDD** said P-ADD-D The extra **D** is because it's a D word meaning it's a 32-bit adding operation and **P** means packed, because we are packing things together. 

This takes 4 slots instead of 1 to add things together. So we do 4 loads and add them into 4 accumulators. Meaning when we do a PADDD it's like we do 4 ADD's This is were the idea of lanes being aligned with registers. 

You can think of these instructions like vector operations, which might be useful for when you're trying to use them. For example with PADDD it would be like this.

| A | + | input[0] |    | A + input[0] |
| A | + | input[1] |    | A + input[1] |
| A | + | input[2] | = | A + input[2] |
| A | + | input[3] |    | A + input[3] |

Even when you are [[Solving the serial dependency problem]] the CPU still has to do a bunch of work to work to check if things are dependent or not and then send that to the right place on the CPU. When you use SIMD you can save a bunch of work because you do that work for every 4 adds rather than for every **ADD**.
###### SIMD bit-width
You have family of SIMD, SSE which is supported every were, AVX which is basically supported on everything and AVX-512 which isn't well supported. With each knew family the instructions get wider.
1) SSE is 4 instructions wide
2) AVX is 8 instructions wide
3) AVX-512 is 16 instructions wide

This is only true if you're using a 32-bit line size. The amount of instructions you can do per operation changes depending on the bits you're using so if you go down to 8 bits for example for SSE you'll get 16 operations per instruction.

The reason you can do this is because these families are just set by a bit-width.
1) SSE being 128
2) AVX being 256
3) AVX-512 being 512

You can use the bit-width in anyway you like.
##### How to use SIMD for summation loop.
The code below shows you how to do these SIMD operations, it's important to note that data organisation is makes SIMD difficult. Meaning that if you care about how the data is organised you have to think carefully about how to use these operations. You might even sometimes have to rewrite algorithms to be able to use SIMD. We are going to use an summation loop which doesn't have this problem.

Lets use SIMD on this
```c
int array[4096] = { /*random values in here*/ };
void addNoSIMD(uint32_t count, uint32_t *input)
int sum = 0;
for(int i = 0; i < count; ++i)
{
	sum += input[i];
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
##### Solving for serial dependencies with SIMD
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
