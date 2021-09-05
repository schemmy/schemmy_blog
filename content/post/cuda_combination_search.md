---
title: "Cuda_combination_search"
date: 2021-09-03T21:56:33-07:00
# draft: true
---



```c
#include <stdio.h>
#include <thrust/host_vector.h>
#include <sys/time.h>
#include <time.h>

#define MAX_N 12
#define nTPB 256
#define GRIDSIZE (32*nTPB)


#define cudaCheckErrors(msg) \
    do { \
        cudaError_t __err = cudaGetLastError(); \
        if (__err != cudaSuccess) { \
            fprintf(stderr, "Fatal error: %s (%s at %s:%d)\n", \
                msg, cudaGetErrorString(__err), \
                __FILE__, __LINE__); \
            fprintf(stderr, "*** FAILED - ABORTING\n"); \
            exit(1); \
        } \
    } while (0)


// thrust code is to quickly prototype a CPU based
// method for verification
int increment(thrust::host_vector<unsigned> &data, unsigned max){
  int pos = 0;
  int done = 0;
  int finished = 0;

  while(!done){
    data[pos]++;
    if (data[pos] >= max) {
      data[pos] = 0;
      pos++;
      if (pos >= data.size()){
        done = 1;
        finished = 1;
        }
      }
    else done = 1;
  }
  return finished;
}

__constant__ unsigned long powers[MAX_N];

__device__ unsigned vec_sum(unsigned *vector, int size){
  unsigned sum = 0;
  for (int i=0; i<size; i++) sum += vector[(i*nTPB)];
  return sum;
}

__device__ void create_vector(unsigned long index, unsigned *vector, int size){
  unsigned long residual = index;
  unsigned pos = size;
  while ((residual > 0) && (pos > 0)){
    unsigned long temp = residual/powers[pos-1];
    vector[(pos-1)*nTPB] = temp;
    residual -= temp*powers[pos-1];
    pos--;
    }
  while (pos>0) {
   vector[(pos-1)*nTPB] = 0;
   pos--;
   }
}
__device__ void increment_vector(unsigned *vector, int size, int k){
  int pos = 0;
  int done = 0;

  while(!done){
    vector[(pos*nTPB)]++;
    if (vector[pos*nTPB] >= k) {
      vector[pos*nTPB] = 0;
      pos++;
      if (pos >= size){
        done = 1;
        }
      }
    else done = 1;
  }
}

__global__ void find_vector_match(unsigned long long int *count, int k, int n, unsigned sum){
  __shared__ unsigned vecs[MAX_N *nTPB];
  unsigned *vec = &(vecs[threadIdx.x]);
  unsigned long idx = threadIdx.x+blockDim.x*blockIdx.x;
  if (idx < (k*powers[n-1])){
    unsigned long vec_count = 0;
    unsigned long vecs_per_thread = (k*powers[n-1])/(gridDim.x*blockDim.x);
    vecs_per_thread++;
    unsigned long vec_num = idx*vecs_per_thread;
    create_vector((vec_num), vec, n);
    while ((vec_count < vecs_per_thread) && (vec_num < (k*powers[n-1]))){
      if (vec_sum(vec, n) == sum) atomicAdd(count, 1UL);
      increment_vector(vec, n, k);
      vec_count++;
      vec_num++;
      }
   }
}

int main(){

// calculate on CPU first for verification
  struct timeval t1, t2, t3;
  int n, k, sum;
  printf("Enter the length of vector (maximum: %d) n=", MAX_N);
  scanf("%d",&n);
  printf("Enter the max value of vector elements k=");
  scanf("%d",&k);
  printf("Enter the sum of vector elements sum=");
  scanf("%d",&sum);
  int count = 0;
  gettimeofday(&t1, NULL);
  k++;

  thrust::host_vector<unsigned> test(n);
  thrust::fill(test.begin(), test.end(), 0);
  int finished = 0;
  do{
    if (thrust::reduce(test.begin(), test.end()) == sum) count++;
    finished = increment(test, k);
    }
    while (!finished);
  gettimeofday(&t2, NULL);
  printf("CPU count = %d, in %d seconds\n", count, t2.tv_sec - t1.tv_sec);
  unsigned long h_powers[MAX_N];
  h_powers[0] = 1;
  if (n < MAX_N)
    for (int i = 1; i<n; i++) h_powers[i] = h_powers[i-1]*k;
  cudaMemcpyToSymbol(powers, h_powers, MAX_N*sizeof(unsigned long));
  cudaCheckErrors("cudaMemcpyToSymbolfail");
  unsigned long long int *h_count, *d_count;
  h_count = (unsigned long long int *)malloc(sizeof(unsigned long long int));
  cudaMalloc((void **)&d_count, sizeof(unsigned long long int));
  cudaCheckErrors("cudaMalloc fail");
  *h_count = 0;
  cudaMemcpy(d_count, h_count, sizeof(unsigned long long int), cudaMemcpyHostToDevice);
  cudaCheckErrors("cudaMemcpy H2D fail");
  find_vector_match<<<(GRIDSIZE + nTPB -1)/nTPB, nTPB>>>(d_count, k, n, sum);
  cudaMemcpy(h_count, d_count, sizeof(unsigned long long int), cudaMemcpyDeviceToHost);
  cudaCheckErrors("cudaMemcpy D2H fail");
  gettimeofday(&t3, NULL);
  printf("GPU count = %d, in %d seconds\n", *h_count, t3.tv_sec - t2.tv_sec);

  return 0;
}
```