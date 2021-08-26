---
layout: post
title:  "[GSoC] Accurate Simulator for Incubator-nemo"
date:   2021-08-24 14:13:51 +9000
categories: GSoC
---
## 1. Short Description

The Accurate Simulator predicts throughput and latency of a cluster in a stream processing environment. It is consisted of functional models, and emulate actual execution with internal timestamp. It is much different from the initial idea based on statistical models, but it is the result of the discussion for the better approach through communication with the mentor.  

There are two main part.
1. Nemo did not have enough metric to conduct research and verify the models in a stream processing environment, So I improved the metric-related parts.
2. Implements Simulator based on functional models and check the fidelity.

## 1. What is Done

### Metric
- Periodically, the number of tuples and the size of the data received from upstream tasks can be recorded for each task. 
- With reference to "flink", I have implemented that source vertex can periodically generate latencymarks and record traverse time up to each task. 

### Simulator
![overview](/assets/overview.png)

**Simulator Launcher**
 - For the purpose of developing simulators that it can be utilized for optimization policy, simulation results should be known before the job is executed. To do this, I implemented SimulationLauncher, which runs independently of JobLauncher and can output simulation results.
 - Compiler: It makes a physical plan, which is a group of tasks that can be allocated in the executor, So it received query from Beam pipeline or Nexmark, and it transfer the physical plan to Plan Simulator. 
 
**Plan Simulator**
- before the start of the simulation, Container Manger, Schedule Simulator and Task Dispatcher prepare nodes and tasks.  
	- **Container Manger**: It creates Executor Simulator according to the Node Configuration. It manages allocated resources for each tasks.  
	- **Schedule Simulator** and **Task Dispatcher** operate the same as the Original Version of Scheduler and Task Dispatcher. they schedule the tasks and distibute the tasks to the executors.  
- When the simulate method is called
	- It triggers simulation for each Executor simulator.
	- It transfer the size of data processed by each task in the Executor Simulator to the downstream tasks. 
	- iterate until the amount of the data processed by each tasks reached balance. 

**Executor Simulator**
- It is virtual node to simulate the execution of task
- It Calculates the time it takes to process one tuple and returns the number of tuples processed when duration is inputted from the Plan Simulator. 

## 2. What code got merged

There are no merged code, yet.

## 3. What code didn't get merged

### Metric

Pull request is created, and I'm trying to address feedback. I think it can be merged soon. 

[[NEMO-483]](https://github.com/apache/incubator-nemo/pull/317)

### Simulator

Although many parts of the simulator was implemented, It was difficult divide it into units that could operate independently. So I couldn't merge it into master. As a result, full request is not be created because it was not completed.

## 4. what's left to do

### Simulator
- It is necessary that Decoupling CPU benchmark as it is dependent on operator + node than node only.
- Check a fidelity under the various environment is needed. 

