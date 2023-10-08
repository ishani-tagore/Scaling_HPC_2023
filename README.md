## Question 1: 1x the Dataset, with single dimensional array inputs

With a serial operation on a Colab CPU, the code takes 0.9360079209999981 seconds on the first run. Averaged across 100 runs, the code takes 0.027999482270010957 secs to execute. Running the code using GPU resources actually takes slightly more time for a total of .039 seconds. This time is distributed across the GPU's CPU kernel (via cupy) at 0.019512575 seconds, and 0.019544274 seconds of GPU time.

The image below shows runtime across 100 runs on Colab's CPU. The table below shows the same operation on GPU resources. 

<img title="Serial CPU runtimes with 1x data" img src="https://github.com/macs30113-s23/a3-ishani-tagore/blob/a607eb4e8e1e0bf6d25b6e43fcac8f7981c4b4e9/CPU_data1x_runtimes.png" width=40% height=20%>

![image](https://user-images.githubusercontent.com/94510833/232163443-e63b7127-56e2-4a5e-bbb8-404cff3b12ba.png)


## Question 2: 
#### How does your parallel GPU implementation compare to the original serial CPU implementation in terms of computation time? Why? What bottlenecks might you be facing?
The vectorized GPU implementation offers no improvement over the serial CPU implementation. This is likely because of communication times between CPU and GPU: 1) The inputs are being sent from the CPU to the GPU during the vectorized nvdi_calc function. Next the nvdi_calc_py function, which is executed in CPU, is transfering the GPU output value to CPU. We had to write this separate nvdi_calc_py function because cupy's benchmark() function cannot directly take non-python objects like the GPU vectorized function's CUDA UFunc object. 

The benchmark() function is batch processing the ndvi_calc_py function 100 times, so I do not anticipate the majority of CPU time being spent here. It is likely in the data transfer between GPU and CPU. 

## Question 3
#### Does your parallel solution perform progressively better as you increase the size of the data?
Running the operation serially on a CPU increases the run time by approximately ~1sec for every additional 50 arrays processed. As you see in the diagrams below, mean runtimes increase from 1.14, 2.27 and 3.01 seconds for 50, 100 and 150x the data respectively.

<img title="Serial CPU runtimes with 50x data" img src="https://github.com/macs30113-s23/a3-ishani-tagore/blob/a607eb4e8e1e0bf6d25b6e43fcac8f7981c4b4e9/CPU_data50x_runtimes.png" width=30% height=15%> <img title="Serial CPU runtimes with 100x data" img src="https://github.com/macs30113-s23/a3-ishani-tagore/blob/a607eb4e8e1e0bf6d25b6e43fcac8f7981c4b4e9/CPU_data100x_runtimes.png" width=30% height=15%> <img title="Serial CPU runtimes with 150x data" img src="https://github.com/macs30113-s23/a3-ishani-tagore/blob/a607eb4e8e1e0bf6d25b6e43fcac8f7981c4b4e9/CPU_data150x_runtimes.png" width=30% height=15%>

GPUs and vectorization offer a considerable improvement over serial runs on a CPU when the size of the dataset involved increases. 
For example, for 50x the data, the GPU takes ~.139 seconds, compared to the 1.14 seconds the same code takes serially on CPU. See the table below. This is still a surprise because my GPU code had limitations: 1) Since "@vectorize" could not take in a 50 column array as an input, I had to serially feed in the tiled red input to process 50x the data 2) I sent the final ndvi array to the CPU by writing a new function in the CPU that called the vectorized ndvi (owing to benchmark () input restrictions; see why I did this in Question 1). Therefore, even with a substantial serial component and communication between CPU and GPU - the GPU offered a 8.2x speedup. 

NOTE: Despite trying to troubleshoot with both TAs, I ran into memory errors while running the vectorized ndvi calculation on a the tiled red and nir vectors. You will see the code I ran in the file Q1_Q3_GPU_Runs_Data100x, where intermediate objects were removed, and vectorize applied to the 100 column array, but still produced issues. I was able to get results for the 1x and 50x dataset as shown below. However, with the 50x dataset code, I serially ran the vectorize operation 50 times, defeating the purpose of parallelization. This was not for lack of trying other approaches. 

Code is referenced as follows: 
GPU code for 1x data: Q1_Q3_GPU_Runs_Data1x
GPU code for 50x data: Q1_Q3_GPU_Runs_Data50x
GPU code for 100x data: Q1_Q3_GPU_Runs_Data100x
GPU code for 150x data: Q1_Q3_GPU_Runs_Data150x

SBATCH commands are compiled into one text file as follows
Q3_SBATCH.sbatch commands
![image](https://user-images.githubusercontent.com/94510833/232163443-e63b7127-56e2-4a5e-bbb8-404cff3b12ba.png)

