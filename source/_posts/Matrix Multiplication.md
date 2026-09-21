---
title: Matrix Multiplication
date: 2026-09-21
categories:
  - 线性代数
tags:
  - Matrix Multiplication
  - GEMM
toc: true
---

Matrix multiplication is built on a hierarchy of linear algebra operations that can be organized in several ways. There are four perspectives to compute matrix multiplication.

$$
C=AB, \qquad C\in \mathbb{R}^{M\times N}, A\in \mathbb{R}^{M \times K}, B\in \mathbb{R}^{K \times N}
$$
## Dot Products
Each entry of matrix $C$ is computed as a dot product of the $i$-th row of matrix $A$ and the $j$-th column of matrix $B$.

$$
C_{i,j} = A_{i,:}B_{:,j} = \sum_{k=0}^{K-1}A_{ik}B_{kj}
$$
![](/images/Pasted%20image%2020260921001423.png)
The dot-product view fixes $i$ and $j$ and accumulates over $k$. Visiting the entries of $C$ row by row gives the `ijk` loop ordering.
```text
for i = 0:M-1
	for j = 0:N-1
		for k = 0:K-1
			C[i,j] += A[i,k]*B[k,j]
		end
	end
end
```
Swapping the two outer loops visits $C$ column by column, giving the `jik` ordering.
```text
for j = 0:N-1
	for i = 0:M-1
		for k = 0:K-1
			C[i,j] += A[i,k]*B[k,j]
		end
	end
end
```


## Linear Combination of left-matrix columns
The $j$-th column of matrix $C$ is produced by multiplying the entire matrix $A$ by the $j$-th column of matrix $B$.

$$
C_{:,j}=AB_{:,j}=\sum_{k=0}^{K-1}B_{kj}A_{:,k}
$$
![](/images/Pasted%20image%2020260921002957.png)
For each output column `j`, accumulating scaled columns of A over k. Updating each entry of the column gives the `jki` ordering.
```text
for j = 0:N-1
	for k = 0:K-1
		for i = 0:M-1
			C[i,j] += A[i,k]*B[k,j]
		end
	end
end
```
## Linear Combination of right-matrix rows
This is the row-wise counterpart of the column-combination, the $i$-th row of $C$ is obtained by multiplying the $i$-th row of $A$ by the entire matrix $B$.

$$
C_{i,:}=A_{i,:}B=\sum_{k=0}^{K-1}A_{ik}B_{k,:}
$$
![](/images/Pasted%20image%2020260921004440.png)
Similarly, accumulating scaled rows of B into each output row gives the ikj ordering.
```text
for i = 0:M-1
	for k = 0:K-1
		for j = 0:N-1
			C[i,j] += A[i,k]*B[k,j]
		end
	end
end
```
## Outer Products
Matrix $C$ can be expressed as a sum of $K$ separate outer products. Take the $k$-th column of matrix $A$ and multiply it by the $k$-th row of matrix B, then sum these matrices together.

$$
C = \sum_{k = 0}^{K - 1}A_{:,k}B_{k,:}
$$
![](/images/Pasted%20image%2020260921195551.png)
Placing $k$ outermost processes one outer product at a time. Traversing each contribution row by row gives the `kij` ordering.
```text
for k = 0:K-1
	for i = 0:M-1
		for j = 0:N-1
			C[i,j] += A[i,k]*B[k,j]
		end
	end
end
```
Traversing each contribution column by column instead gives the `kji` ordering.
```text
for k = 0:K-1
	for j = 0:N-1
		for i = 0:M-1
			C[i,j] += A[i,k]*B[k,j]
		end
	end
end
```

Although these four perspectives are mathematically equivalent and conventionally counted as $2MNK$ FLOPs, they can lead to implementations with very different performance because they access and reuse data differently.

## Block Matrix Multiplication
Block matrix multiplication follows the same sum-of-products structure, with scalar entries replaced by submatrices. Partition $A$, $B$, $C$ into blocks of sizes $BM \times BK$, $BK \times BN$ and $BM \times BN$. For simplicity, assume that $M$, $N$, and $K$ are divisible by $BM$, $BN$ and $BK$.

$$
\mathcal C_{p,q} = \sum_{r=0}^{K/BK-1} \mathcal A_{p,r}\mathcal B_{r,q}
$$
![](/images/Pasted%20image%2020260921194226.png)
For a fixed output block, `p` and `q` remain unchanged while `r` traverses the shared dimension. Each pair of input blocks contributes to the same output block.
```text
for p = 0:M/BM-1
	for q = 0:N/BN-1
		for r = 0:K/BK-1
			C_tile[p,q] += A_tile[p,r] @ B_tile[r,q]
		end
	end
end
```
All four views extend to block matrix multiplication. As the computation is broken down into smaller blocks, these views can be combined at different levels of the hierarchy.
![](/images/Pasted%20image%2020260921200407.png)