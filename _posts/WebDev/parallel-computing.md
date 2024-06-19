---
layout: default
title: Parallel Computing
parent: WebDev
nav_order: 1
---

# 14. Reduction
## 1. Sum Reduction w/ Divergence
- Sum reduction이 threadIdx.x=0에 되기 때문에, SMEM -> global mem. 할 떄 threadIdx.x=0에 대해서만 하면 됨. 
```cpp
__global__ void kernel(unsigned* pData, unsigned* pAns){
    __shared__ float dataShared[2*BLOCKSIZE];   
    unsigned int i = blockDim.x*blockIdx.x + threadIdx.x;
    dataShared[threadIdx.x] = pData[i];
    dataShared[threadIdx.x+BLOCKSIZE] = pData[i+BLOCKSIZE]; 
    __syncthreads(); 

    for (register unsigned s=1; s<=BLOCKSIZE; s*=2){
        if (threadIdx.x%s == 0){ dataShared[2*threadIdx.x] += dataShared[2*threadIdx.x+s]; }
        __syncthreads(); 
    }
    if (threadIdx.x==0){ pAns[threadIdx.x] = dataShared[threadIdx.x]; }
}
```
## 2. Sum Reduction w/o Divergence
```cpp
__global__ void kernel (unsigned* pData, unsigned* pAns){
    __shared__ float dataShared[2*BLOCKSIZE]; 
    unsigned int i = blockDim.x*blockIdx.x + threadIdx.x;
    dataShared[threadIdx.x] = pData[i];
    dataShared[threadIdx.x+BLOCKSIZE] = pData[i+BLOCKSIZE]; 
    __syncthreads(); 

    for (register unsigned s=BLOCKSIZE; s>0; s/=2){
        if (threadIdx.x<s){ dataShared[threadIdx.x] += dataShared[threadIdx.x+s]; }
        __syncthreads(); 
    }
    if (threadIdx.x==0){ pAns[threadIdx.x] = dataShared[threadIdx.x]; }
}
```


# 15. Scan (Prefix Sum)
## 1. Kogge-Stone Parallel Scan
```cpp
for (int s=1; s<BLOCKSIZE; s*=2){
    float temp = 0.0f; 
    if (threadIdx.x>=s){ temp=T[threadIdx.x]+T[threadIdx.x-s]; }
    __syncthreads(); 
    if (threadIdx.x<s){ T[threadIdx.x]=temp; }
    __syncthreads(); 
}
if (gid<n) { output[gid] = T[threadIdx.x]; }
```
## 2. Kogge-Stone (Double Buffering ver.) 
```cpp
for (int s=1; s<BLOCKSIZE; s*=2){
    if (threadIdx.x>=s) { dst[threadIdx.x] = src[threadIdx.x]+src[threadIdx.x-s]; }
    else { dst[threadIdx.x] = src[threadIdx.x]; }
    __syncthreads(); 

    float* tmp=src; src=dst; dst=tmp;
}
if (gid<n){ output[gid] = src[threadIdx.x]; }
```
## 3. Brent-Kung Parallel Scan
- `int idx = (threadIdx.x+1) * 2 * s - 1;` 
```cpp
int s=1; 
while(s<BLOCKSIZE){
    int idx = (threadIdx.x+1) * 2 * s - 1; 
    if (idx<BLOCKSIZE){ T[idx] += T[idx-s]; }
    s *= 2; 
    __syncthreads(); 
}

int s=BLOCKSIZE/2; 
while(s>0){
    int idx = (threadIdx.x+1) * 2 * s - 1; 
    if (idx+s<BLOCKSIZE && idx<BLOCKSIZE){ T[idx+s] += T[idx]; }
    s /= 2; 
    __syncthreads(); 
}
```


# 13. Convolution 

## 1. Boundary Condition Handling & Constant Cache
1. 1D CONV w/ Boundary Condition Handling 
```cpp
__global__ void CONV_1D (float* N, float* M, float* P, int Mask_Width, int Width){
    int i = blockIdx.x*blockdDim.x + threadIdx.x;
    float Pvalue = 0; 
    int N_start = i - Mask_Width/2; 

    for (int j = 0; j<Mask_Width; j++){
        if (N_start+j>=0 && N_start+j<Width) { 
            Pvalue += N[N_start+j]*M[j]; 
        }
    }
    P[i] = Pvalue; 
}
```
2. Usage of Constant Memory 
```cpp
#define MASK_WIDTH 5
typedef struct{
    unsigned int weight;
    unsigned int height; 
    unsigned int pitch; 
    float* elements; 
} Matrix; 

// Matrix allocation func. 
Matrix AllocateMatrix(int height, int width, int init){...}

// global var. -> declare outside any kernel function 
__constant__ float Mc[MASK_WIDTH][MASK_WIDTH]; 
Matrix M = AllocateMatrix(MASK_WIDTH, MASK_WIDTH, 1); 
// initialize M elems, allocate N/P, initialize N/P, copy N/P to N_d/P_d

cudaMemcpyToSymbol(Mc, M.elements, MASK_WIDTH*MASK_WIDTH*sizeof(float)); 
// launch kernel 
```

## 2. Tiled 1D
### 1. Load halo + internal elem. into SMEM
```cpp
__global__ void convolution_1D_tiled_kernel(float* N, float* P, int Mask_Width, int Width){
    int i = blockIdx.x*blockDim.x+threadIdx.x; 
    __shared__ float N_ds[TILE_SIZE + Mask_Width - 1]; 
    int n = Mask_Width/2;

    // left halo 
    int halo_idx_left = (blockIdx.x-1)*blockDim.x+threadIdx.x; 
    if (threadIdx.x >= (blockDim.x-n)){
        N_ds[threadIdx.x + n - blockDim.x] = (halo_idx_left<0) ? 0 : N[halo_idx_left]; 
    }
    // internal 
    N_ds[n+threadIdx.x] = N[blockIdx.x*blockDim.x+threadIdx.x];

    // right halo 
    int halo_idx_right = (blockIdx.x+1)*blockDim.x+threadIdx.x; 
    if (threadIdx.x < n){
        N_ds[threadIdx.x + n + blockDim.x] = (halo_idx_right>Width) ? 0 : N[halo_idx_right]; 
    }
    __syncthreads(); 

    float Pvalue = 0; 
    for (int j=0; j<MASK_WIDTH; j++){ Pvalue += N_ds[threadIdx.x + j]*M[j]; }
    P[i] = Pvalue;
}
```
- Practice!
```cpp
__global__ void convolution_1D_tiled_kernel(float* N, float* P, int Mask_Width, int Width){
    int i = blockIdx.x*blockDim.x+threadIdx.x; 
    __shared__ float N_ds[??]; 
    int n = Mask_Width/2;

    // left halo 
    int halo_idx_left = ??
    if (??){
        N_ds[??] = (??) ? 0 : N[??]; 
    }
    // internal 
    N_ds[??] = N[??];

    // right halo 
    int halo_idx_right = ??
    if (??){
        N_ds[??] = (??) ? 0 : N[??]; 
    }
    __syncthreads(); 

    float Pvalue = 0; 
    for (int j=0; j<MASK_WIDTH; j++){ Pvalue += N_ds[??]*M[??]; }
    P[??] = Pvalue;
}
```


### 2. Data Reuse in SMEM & General Caching 
- 🔥 Everything is gid based!! 
```cpp
__global__ void convolution_1D_tiled_kernel (float* N, float* P, int Mask_Width, int Width){
    int i = blockIdx.x*blockDim.x+threadIdx.x; 
    int n = Mask_Width/2; 
    __shared__ float N_ds[TILE_SIZE]; 
    
    N_ds[threadIdx.x]=N[i]; __syncthreads(); 

    int This_tile_start_point = blockIdx.x*blockDim.x; 
    int Next_tile_start_point = (blockIdx.x+1)*blockDim.x; 
    int N_start_point = i - n;

    float Pvalue = 0; 
    for (int j=0; j<MASK_WIDTH; j++){
        int N_index = N_start_point + j; 
        if (N_index >= This_tile_start_point && N_index < Next_tile_start_point){
            Pvalue += N_ds[threadIdx.x+j-n]*M[j]; 
        } else {
            Pvalue += N[N_index]*M[j]; 
        }
    }
    P[i] = Pvalue; 
}
```
- Practice!
```cpp
__global__ void convolution_1D_tiled_kernel (float* N, float* P, int Mask_Width, int Width){
    int i = blockIdx.x*blockDim.x+threadIdx.x; 
    int n = Mask_Width/2; 
    __shared__ float N_ds[??]; 
    
    N_ds[??]=N[??]; __syncthreads(); 

    int This_tile_start_point = blockIdx.x*blockDim.x; 
    int Next_tile_start_point = (blockIdx.x+1)*blockDim.x; 
    int N_start_point = ??

    float Pvalue = 0; 
    for (int j=0; j<MASK_WIDTH; j++){
        int N_index = ??; 
        if (N_index >= ?? && N_index < ??){
            Pvalue += N_ds[??]*M[??]; 
        } else {
            Pvalue += N[??]*M[??]; 
        }
    }
    P[??] = Pvalue; 
}
```

## 3. Tiled 2D
- it's easier to think: `row_o`, `col_o` gid based.
- N_ds is tid based shared memory! 
1. Shifting from output coordinates to input coordinates 
```cpp
__global__ void convolution(Matrix N, Matrix P){
    __shared__ float N_ds[BLOCK_SIZE][BLOCK_SIZE]; 
    int ty = threadIdx.y; int tx = threadIdx.x; 
    int row_o = blockIdx.y*TILE_SIZE+ty; 
    int col_o = blockIdx.x*TILE_SIZE+tx;
    int row_i = row_o - KERNEL_SIZE/2; 
    int col_i = col_o - KERNEL_SIZE/2; 
    float output = 0.0f; 
}
```
2. Loading input matrix to the shared memory 
```cpp
if (0<=row_i<N.height && 0<=col_i<N.width){
    N_ds[ty][tx] = N.elements[row_i * N.width + col_i];
} else{
    N_ds[ty][tx] = 0;
}
__syncthreads();
```
3. Do convolution! 
```cpp
if (ty<TILE_SIZE && tx<TILE_SIZE){
    for (i=0; i<KERNEL_SIZE; i++){
        for (j=0; j<KERNEL_SIZE; j++){
            output += Mc[i][j]*N_ds[i+ty][j+tx]; 
        }
    }
    if (row_o<P.height && col_o<P.width){ P.elements[row_o*P.width+col_o] = output; }
}
```
4. Setting Block Size
```cpp
#define BLOCK_SIZE (TILE_SIZE + MASK_WIDTH -1)
dim3 dimBlock(BLOCK_SIZE, BLOCK_SIZE, 1); 
dim3 dimGrid(ceil(N.width/float(TILE_SIZE)), ceil(N.width/float(TILE_SIZE)), 1); 
convolution<<<dimGrid, dimBlock>>>(N_d, P_d)
```

- Practice!
```cpp
__global__ void convolution(Matrix N, Matrix P){
    // Shifting from  output coordinates to input coordinates
    __shared__ float N_ds[??][??]; 
    int ty = threadIdx.y; int tx = threadIdx.x; 
    int row_o = ??; 
    int col_o = ??;
    int row_i = ??; 
    int col_i = ??; 
    float output = 0.0f; 

    // Loading input matrix to the shared memory 
    if (??){
        N_ds[??][??] = N.elements[??];
    } else{
        N_ds[??][??] = 0;
    }
    __syncthreads();

    // Do convolution! 
    if (ty<?? && tx<??){
        for (i=0; i<KERNEL_SIZE; i++){
            for (j=0; j<KERNEL_SIZE; j++){
                output += Mc[i][j]*N_ds[??][??]; 
            }
        }
        if (??<P.height && ??<P.width){ P.elements[??] = output; }
    }
}
// Setting block size
#define BLOCK_SIZE (??)
dim3 dimBlock(BLOCK_SIZE, BLOCK_SIZE, 1); 
dim3 dimGrid(??, ??, 1); 
convolution<<<dimGrid, dimBlock>>>(N_d, P_d)
```




# 12. Histogram w/ Atomic Ops. 
## 1. Atomic Ops. 
1. atomicCAS
```cpp
int atomicCAS(int* addr, int expected, int newVal){
    oldVal = read(*addr);
    *addr = (oldVal==expected) ? newVal : oldVal;
    return oldVal
}
```
2. atomicCAS Usage
```cpp
oldVal = *addr; 
readback = atomicCAS(addr, oldVal, newVal); 
if (oldVal==readback) → success
else → fail, another T did atomicCAS 
```
3. atomicAdd
```cpp
__device__ inline void atomicAdd (float* addr, float value){
    int oldVal, newVal, readback; 
    oldVal = __float_as_int(*addr);
    newVal = __float_as_int(__int_as_float(oldVal)+value); 
    while ([readback=atomicCAS(*addr, oldVal, newVal)] != oldVal){
        oldVal = readback; 
        newVal = __float_as_int(__int_as_float(oldVal)+Value);
    }
}
```
## 2. Example: Count 
1. Non-atomic version
- total # of threads ≠ count, `t=4msec`
 ```cpp
 __global__ void kernel(unsigned long long int* pCount){
    (*pCount) = (*pCount) + 1; 
 }
 ```
2. Atomic version
```cpp
__gobal__ void kernel(~){
    atomicAdd(pCount, 1ULL); 
}
```
3. Shared memory version
```cpp
__global__ void kernel(int* pCount){
    __shared__ int nCountShared; 
    if (threadIdx.x==0){ nCountShared=0; } __sycthreads(); 
    atomicAdd(&nCountShared, 1); __syncthreads(); 
    if (threadIdx.x==0){ atomicAdd(pCount, nCountShared); }
}
```
## 3. Example: Histogram Count
1. CPU version 
- sum of each hist = image size, `t=4340usec`
```cpp
void kernel (unsigned int* hist, unsigned int* img, unsigned int size){
    while (size--){
        unsigned int pixelVal = *img++;
        hist[pixelVal] += 1;
    }
}
```
2. GPU version
- sum of each hist = image size, `t=1.61msec`
```cpp 
__global__ void kernel(unsigned int* hist, unsigned int* img, unsigned int size){
    unsigned int i = blockIdx.x*blockDim.x+threadIdx.x; 
    unsigned int pixelVal = img[i];
    atomicAdd(&hist[pixelVal], 1);
}
```
3. SMEM version 
- sum of each hist = image size
```cpp 
__global__ void kernel(unsigned int* hist, unsigned int* img, unsigned int size){
    __shared__ int histShared[NUMHIST]; 
    if (threadIdx.x<NUMHIST) { histShared[threadIdx.x]=0 } __syncthreads(); 

    unsigned int i = blockIdx.x*blockDim.x+threadIdx.x; 
    unsigned pixelVal = img[i];
    atomicAdd(&histShared[pixelVal], 1); __syncthreads(); 

    if (threadIdx.x<NUMHIST) { atomicAdd(&hist[threadIdx.x],histShared[threadIdx.x]); }
}
```
- Practice! 
```cpp 
__global__ void kernel(unsigned int* hist, unsigned int* img, unsigned int size){
    __shared___ int histShared[NUMHIST]; 
    if (threadIdx.x<NUMHIST) { histShared[threadIdx.x]=0; } __syncthreads();

    unsigned i= blockIdx.x*blockDim.x+threadIdx.x; 
    pixelVal=img[i]; 
    atomicAdd(&histShared[pixelVal], 1); __syncthreads(); 

    if (threadIdx.x<NUMHIST) { atomicAdd(&hist[threadIdx.x],histShared[threadIdx.x]); }
}
```

# 18. Sparse Matrix Multiplication 
## 1. Sequential SpMV
```cpp
for (int row=0; row<num_rows; row++){
    float dot = 0; 
    int row_start = row_ptr[row]; 
    int row_end = row_ptr[row+1]; 
    for (int elem=row_start; elem<row_end; elem++){
        dot += data[elem] * x[col_idx[elem]];
    }
    y[row] += dot;
}
```

## 2. Parallel SpMV
- Shortcoming: 1) No memory coalescing, 2) Control Flow Divergence
```cpp
__global__ void SpMV_CSR(int num_rows, float* data, int* col_idx, int* row_ptr, float* x, float* y){
    // each thread processes one row
    int row = blockIdx.x*blockDim.x+threadIdx.x; 
    int row_start = row_ptr[row]; int row_end = row_ptr[row+1]; 
    
    for (int elem=row_start; elem<row_end; elem++){
        float dot; 
        dot += data[elem]*x[col_idx[elem]]; 
    }
    y[row]=dot;

}
```

## 3. ELL (PACK) Format
- data[], col_idx[] both padded → Solves control flow divergence, row_ptr[] not needed!
- transposed → Solves coalescing
- Shortcoming: excessive # of padded elements
```cpp
__global__ void SpMV_ELL(int num_rows, float* data, int* col_idx, int num_elem, float* x, float* y){
    int row = blockIdx.x*blockDim.x+threadIdx.x; 
    
    if (row<num_rows){
        for (unsigned int i=0; i<num_elem; i++){
            dot += data[row+i*num_rows]*x[col_idx[row+i*num_rows]];
        }
        y[row]=dot
    }
        
}
```

## 4. Coordinate Format
- Elem. in COO format can be arbitrarily record
1. Sequential SpMV (COO)
``` cpp 
for (int i=0; i<num_elem; i++){
    y[row_idx[i]] = data[i]x[col_idx[i]]
}
```
2. Parallel SpMV (COO)
- Each thread takes care of one elem. 
```cpp
__global__ void SpMV_COO(float* data, int* col_idx, int* row_idx, int num_elem, float* x, float* y){
    int i = blockDim.x*blockIdx.x+threadIdx.x; 
    if (i<num_elem){
        float dot; 
        dot = data[i]*x[col_idx[i]]; 
        atomicAdd(&y[row_idx[i]], dot);
    }
}
```



# 17. Graph Structure 
## 1. Sequential BFS
```cpp
void BFS_sequential (int source, int* edges, int* dest, int *label){

    int frontier[2][MAX_FRONTIER_SIZE]; 
    int* c_frontier = frontier[0]; int* p_frontier = frontier[1];
    int c_frontier_tail = 0; int p_frontier_tail = 0; 

    // insert source to p_frontier array, and add 1 to p_frontier_tail
    insert_frontier(source, p_frontier, &p_frontier_tail); 
    label[source]=0

    int c_vertex // current vertex
    while (p_frontier_tail>0){
        for (int f=0; f<p_frontier_tail; p++){
            c_vertex = p_frontier[f]; // source of this loop 
            for (int i=edges[c_vertex]; i<edges[c_vertex+1]; i++){
                if (label[dest[i]]==-1){
                    insert_frontier(dest[i], c_frontier, &c_frontier_tail); 
                    label[dest[i]] += label[c_vertex] + 1
                }
            }
        }
        // swap c_frontier and p_frontier
        int* temp = c_frontier; c_frontier = p_frontier; p_frontier = temp; 
        p_frontier_tail = c_frontier_tail; c_frontier_tail = 0; 
    }
}
```

## 2. Parallel BFS 
```cpp
void BFS_host(unsigned int source, unsigned int* edges, unsigned int* dest, unsigned int* label){
    ... 
    // cudaMalloc edges_d, dest_d, label_d, and visited_d
    // cudaMemecpy edges, dest, and label to device global memory 
    // cudaMalloc frontier_d, c_frontier_tail_d, p_frontier_tail_d 
    unsigned int* c_frontier_d = &frontier_d[0]; 
    unsigned int* p_frontier_d = &frontier_d[MAX_FRONTIER_SIZE]; 

    // launch simple kernel to initialize source
    // *c_frontier_tail_d = 0 ; 
    // p_frontier_d = source; 
    // *p_frontier_tail_d = 1; 
    // label_d[source] = 0;

    p_frontier_tail = 1; 
    while(p_frontier_tail>0){
        int num_blocks = ceil(p_frontier_tail/float(BLOCK_SIZE)); 
        BFS_Bqueue_kernel<<<num_blocks, BLOCK_SIZE>>>(p_frontier_d, p_frontier_tail_d, c_frontier_d, c_frontier_tail_d, edges_d, dest_d, label_d, visited_d); 
        // cudaMemcpy to read the *c_frontier_tail value back to host 
        // assign int to p_frontier_tail for the while-loop condition test
        int* temp = c_frontier_d; c_frontier_d=p_frontier_d; p_frontier_d=temp; 
        // launch a simple kernel to set *p_frontier_tail_d = *c_frontier_tail_d; *c_frontier_tail_d=0; 
    }
}
__global__ void BFS_Bqueue_kernel(unsigned int* p_frontier, unsigned int*p_frontier_tail, unsigned int* c_frontier, unsigned int* c_frontier_tail, unsigned int* edges, unsigned int* dest, unsigned int* label, unsigned int* visited){
    __shared__ unsigned int c_frontier_s[BLOCK_QUEUE_SIZE]; 
    __shared__ unsigned int c_frontier_tail_s, our_c_frontier_tail; 

    if (threadIdx.x ==0){ c_frontier_tail_s=0; } __syncthreads(); 
    const unsigned gid = blockIdx.x*blockDim.x+threadIdx.x; 

    if (gid < *p_frontier_tail){
        const unsigned int c_vertex = p_frontier[gid]; 

        for (unsigned int i = edges[c_vertex]; i<edges[c_vertex+1]; i++){
            const unsigned int was_visited = atomicExch(&(visited[dest[i]]), 1); 

            if (!was_visited){
                label[dest[i]] = label[c_vertex]+1; 
                const unsigned int my_tail = atomicAdd(&c_frontier_tail_s, 1); 

                // check if # of neighboring vertex exceeds BLOCK_QUEUE_SIZE
                if (my_tail<BLOCK_QUEUE_SIZE){
                    c_frontier_s[my_tail] = dest[i]; 
                } else {
                    c_frontier_tail_s = BLOCK_QUEUE_SIZE; 
                    const unsigned int my_global_tail = atomicAdd(c_frontier_tail, 1);
                    c_frontier[my_global_tail] = dest[i]; 
                }
            }
        }
        // we need to reference global tail multiple times- make it to smem var. 
        __syncthreads(); 
        if (threadIdx.x==0){ 
            our_c_frontier_tail = atomicAdd(c_frontier_tail, c_frontier_tail_s); 
        }
    }
    __syncthreads(); 
    // vertex searching order doesn't matter- just append to global memory tail
    // for vertexes that exceed BLOCK_QUEUE_SIZE will already be in global mem. 
    for (unsigned int i=threadIdx.x; i<c_frontier_tail_s; i+=blockDim.x){
        c_frontier[our_c_frontier_tail+1]=c_frontier_s[i];
    }

}
```


# 16. Streaming and AsyncCopy

## 1. Streaming Operation 
1. Host version - `t=588533usec`
```cpp
void kernel (float* dst, float* src, float val, int size){
    while (size--){
        *dst = 0.0F; 
        for (register int j=0; j<REPEAT; ++j){ *dst += *src; }
        dst++; src; 
    }
}
```

2. Device version - `t=108413usec` (winner!)
```cpp
// size not needed - each thread operates on one elem. 
__global__ void kernel (float* dst, float* src) 
    unsigned int i = blockIdx.x*blockDim.x + threadIdx.x; 

    if (i<SIZE){
        dst[i] = 0.0F
        for (register int j=0; j<REPEAT; j++) { dst[i] += src[i]}
    }
```

## 2. Asynchronous Copy
1. Synchronous Streaming 
- `cudaMemcpy (void* dst, const void* src, size_t count, enum kind)`

2. Asynchronous Copy
- `cudaMemcpyAsync (void* dst, const void* src, size_t count, enum kind, cudaStream_t stream=0)`
- Pinned-memory
- `cudaMallocHost()`, `cudaFreeHost()`

## 3. CUDA Streams
### 1. CUDA Streams - Asynchronous Ver. 
- kernel launch 때 stream 이 마지막 인자로 들어가야 되기 때문에 세번째 차원 0 반드시 명시 `kernel<<<dimGrid, dimBlock, 0, stream>>>(pResultDev, pSourceDev)`
- kenrel launch 다음에 synchronize stream 까먹지 말기 
```cpp
int main(void){
    // allocate pinned-memory 
    float pSource = NULL; float pResult = NULL; 
    cudaMallocHost((void**)&pSource, TOTALSIZE*sizeof(float)); 
    cudaMallocHost((void**)&pResult, TOTALSIZE*sizeof(float)); 

    // create stream object
    cudaStream_t stream; 
    cudaStreamCreate(&stream);
    ... 
    // CUDA: copy from host to device
    cudaMemcpyAsync(pSourceDev, pSource, TOTALSIZE*sizeof(float), cudaMemcpyHostToDevice, stream);

    // CUDA: launch the kernel 
    ... 
    kernel<<<dimGrid, dimBlock, 0, stream>>>(pResultDev, pSourceDev)
    
    // sync to make sure operations complete
    cudaStreamSynchronize(stream);

    // deallocate pinned-memory 
    cudaFreeHost(pSource); cudaFreeHost(pResult); 
    
    // destroy stream 
    cudaStreamDestroy(stream); 
}
```

### 2. CUDA Streams - Multiple Streams
- cudaStreamCreate(***&stream***), cudaStreamSynchronize(stream), cudaStreamDestroy(stream)
```cpp 
int main(void){
    // allocate pinned-memory 
    cudaMallocHost((void**)&pSource, TOTALSIZE*sizeof(float));
    cudaMallocHost((void**)&pResult, TOTALSIZE*sizeof(float));

    // create stream object 
    cudaStream_t aStream[STREAMSIZE]; 
    for (i=0; i<STREAMSIZE; i++) { cudaStreamCreate(&aStream[i]); }
    
    // CUDA: copy from host to device
    for (i=0; i<STREAMSIZE; i++){
        int offset = TOTALSIZE/STREAMSIZE*i; 
        cudaMemcpyAsync(pSourceDev+offset, pSource+offset, TOTALSIZE/STREAMSIZE*sizeof(float), cudaMemcpyHostToDevice, aStream[i]);
    }
    
    // CUDA: launch the kernel
    dim3 dimGrid(GRIDSIZE/STREAMSIZE, 1, 1); 
    dim3 dimBlock(BLOCKSIZSE, 1, 1); 
    for (i=0; i<STREAMSIZE; i++){
        int offset = TOTALSIZE/STREAMSIZE*i;
        kernel<<<dimGrid, dimBlock, 0, aStream[i]>>>(pResultDev+offset, pSourceDev+offset);
    }
    
    // CUDA: copy from device to host 
    for (i=0; i<STREAMSIZE; i++){
        cudaStreamSynchronize(aStream[i]); 
        cudaStreamDestroy(aStream[i])
    }
}
```

## 4. MISC
### 1. Elapsed Time: CUDA Event API Usage
```cpp
cudaEvent_t start, stop; 
float time; 

cudaEventCreate(&start); cudaEventCreate(&stop); 
cudaEventRecord(start, 0); 
VectorAdd<<<gridDim, gridBlock>>>(dev_A, dev_B, dev_R, size); 
cudaEventRecord(stop, 0); 

cudaEventSynchronize(stop); 
cudaEvenElapsedTime(&time, start, stop); 
cudaEventDestroy(start); cudaEventDestroy(stop); 
printf("elapsed time = %f msec\n", time)
```