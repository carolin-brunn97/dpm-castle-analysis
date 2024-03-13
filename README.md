# Analyzing Continuous ks-Anonymization of Smart Meter Data
In this project, we analyzed the continuous ks-anonymization with CASTLE of data streams generated by smart meters. The results were published as a short paper at the DPM Workshop 2023. In this repo, we include the preprint that has not undergone peer review. The Version of Record of this contribution is published in "Computer Security, ESORICS 2023 International Workshops: CyberICPS, DPM, CBT, SECPRE, CPS4CIP, ADIoT, SecAssure, WASP, TAURIN, PriST-AI, SECAI", and is available online at https://link.springer.com/chapter/10.1007/978-3-031-54204-6_16.

Data anonymization is crucial to allow the widespread adoption of some technologies, such as smart meters. However, anonymization techniques should be evaluated in the context of a dataset to make meaningful statements about their eligibility for a particular use case. In this paper, we therefore analyze the suitability of continuous ks-anonymization with CASTLE for data streams generated by smart meters. We compare CASTLE’s continuous, piecewise ks -anonymization with a global process in which all data is known at once, based on metrics like information loss and properties of the sensitive attribute. Our results suggest that continuous ks-anonymization of smart meter data is reasonable and ensures privacy while having comparably low utility loss.

## Ressources and Prerequisites

The CASTLE implementation used for our simulations is mainly based on the publicly available CASTLEGUARD implementation ([CASTLEGUARD](https://github.com/hallnath1/CASTLEGUARD), corresponding [paper](https://doi.org/10.1109/DASC-PICom-CBDCom-CyberSciTech49142.2020.00102)) with some adaptations.

The code is written and run with Python (3.11.4) and was executed with the following packages: 
numpy (1.24.3), pandas (1.5.3), matplotlib (3.7.1), seaborn (0.12.2).

### CASTLEGUARD Adaptations
All essential adaptations that we did on the code are marked with comments. During our work, we found potential bugs or exceptions that we briefly describe here. 

**1:** Under certain conditions of the dataset, the function `castle.split_l()` raises an exception, because only k *items* instead of k *individuals* are included in the newly generated clusters. Thislater leads to an exception when this condition is checked right before publication. During our simulations we used the castle.split() function, since we did not require l-diverse clusters. 

To reproduce this error, please run the simulation with the provided test dataset that can be found in `./consumption_data/example_data_raise_split_l_excep.csv`.
Additionally, uncomment 2 lines of code in `./src/castle.py`:

	`splittable = len(c.contents) >= 2 * self.k and len(c.diversity) >= self.l` (line 294)
 
	`sc = self.split_l(c) if splittable else [c]` (line 305)

**2:** We intriduced a new parameter to record the number of distinct PID values within a cluster, `cluster.diverse_pids`. 

**3:** We introduced a new `break`condition in the function `castle.merge_clusters()`. So far only the number of tuples contained in the clusters was checked and not the number of distinct individuals. To implement this, we used the new parameter `cluster.diverse_pids`.

**4:** We also used `cluster.diverse_pids` to check whether a cluster is splittable since this was previously based on the number of elements in a cluster.

**5:** Additionally, we added the parameter T_kc to truncate the size of the ks-anonymous clusters as described in [FADS](https://doi.org/10.1016/j.knosys.2013.03.007).

**6:** Last, we added the functionality that also the last data tuples of the input dataset are processed even though delta is not exceeded anymore. Note, that this has to be eliminated if a realistic network with latency effects shall be simulated.

### Dataset

The dataset used for our simulations is available at the UC Irvine Machine Learning Repository (https://doi.org/10.24432/C58C86).

According to the description of the dataset, the consumption values are given in kW. However, based on the magnitude of the values, this does not seem realistic since the values are too high for that.
To further analyze the consumption values and course of values in the dataset, we provide the functionality in `./dataAnalysis_plotting/generate_SLA_fromConsData.py`.
With this analysis, we see a typical course that reflects an SLA. When comparing the measurement values to a representative consumption of a 2-person household, they are higher then the representative consumption by a factor of approx. 1000.
This indicates that the values are actually given in W and not in kW.

## Usage
There are several steps to reproduce our results or run your own simulation.

### Data Preparation
First, we adapted the raw dataset taken from the UCI Repository. The programs used for this procedure can be found in the folder ./dataPreparation_preprocessing. They were executed in the following order:
1) `convert_extend_orig_data.py`
2) `merge_singleClients2OneSet.py`
3) `split_csv_year.py`
4) `split_csv_month.py`
5) optional: `transform_date_time.py` (Whereas in theory this step is optional, we always applied it in our simulation. Therefore, we do not guarantee the full funtionality for the original date format.)

### Command Line Simulation (Anonymization Process)
Next, we run the simulation of the piecewise anonymization of a continuous data stream.
To run the simulation from the command line, access the `src/` folder and execute: 

`python3 main.py --k 10 --sample-size 10000 --delta 50 --beta 200000 --mu 100000 --l 1 --tkc 50 --disable-dp --year "2014" --month "11" --n_users 370 -f "transformed_date_2014-11-eletricity_consumption_sample_164102.csv"`
Change the values for k and delta to simulate different settings.

Alternatively, start the simulations for different settings with the bash scripts provided in the `src/` folder.

The datasets generated in the anonymizatino process can be found in the folder `./anonymized_data`. For reproduction purposes, we include the results we obtained in a `simulationResults.zip` folder.


### Data Analysis
Again, several steps are required for the data analysis. First, we prepare the dataset by analyzing the clusters and applying the metrics Information Loss, Cluster Size vs. PID Diversity, PID Diversity vs. Consumption (l-)Diversity, Consumption Range vs. PID Diversity, and Consumption Proximity. Next, the results can be visualized. The programs used for this procedure can be found in the folder `./dataAnalysis_plotting`.
Please first apply `analyze_{ARX,CASTLE}_generalizations.py`.
Afterwards, the results of the metrics can be plotted with the respectively named programs.
