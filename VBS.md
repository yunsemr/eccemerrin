VSB Power Line Fault Detection :

# Background :

Medium voltage overhead power lines run for hundreds of miles to supply power to cities. These great distances make it expensive to manually inspect the lines for damage that doesn't immediately lead to a power outage, such as a tree branch hitting the line. These modes of damage lead to a phenomenon known as partial discharge. Partial discharges slowly damage the power line, so left unrepaired they will eventually lead to a power outage or start a fire.

The challenge in here is to detect partial discharge patterns in signals acquired from these power lines. Effective classifiers using this data will make it possible to continuously monitor power lines for faults.

# Let's get familiar with the data :

Power lines are measured using voltage, but you don't just measure voltage like you measure height. It's not just a single number. It goes up and down making waves. Each wave is a cycle and you can make judgements of each cycle by taking a multitude of measurements over time. 

* For this experiment, each signal contains 800,000 measurements of a power line voltage, taken over 20 milliseconds. Underlying electric grid operates at 50 Hz. In other words, if we want to measure 50 cycles of voltage on this electric grid we would take measurements for 1000 milliseconds. The measurements in the competition were only performed for 20 milliseconds so we will only see one cycle. See basic math below:

 	(50 cycles/ 1000 milliseconds) x (20 milliseconds) = 1 cycle

In other words, the 800,000 measurements make up a time-course showing the voltage over time.

* The grid itself operates on a 3-phase power scheme, and all three phases are measured simultaneously.

* The signal data comes from the real environment, not a lab, so they contain a lot of background noise. 

* Since these signals are measured by company’s patented device with lower sampling rate (cost efficiency purpose)  they do not recommend to use any other publicly available dataset containing partial discharge patterns.

*  In general the imbalanced dataset is very natural because PD pattern implies some degradation or damage of the observed system which is happening (fortunately) less often than the states when the system is operating correctly.

* We have two training set files:

1. metadata_train.csv 

2. train.parquet 

                  

          metadata_train.csv

![image alt text](image_0.png)

					train.parquet

* metadata_train.csv contains four columns:

    * **signal_id:** a unique identifier 

    * **id_measurement:** ID code for each powerline. There should be 3 of each number which represents each phase in the 3-phase power scheme (AKA the trio)

    * **phase**: the phase ID within the grid’s phases (0,1,2).

    * **target:** 0 fixed, 1 broken

* train.parquet 

    * contains the 800,000 rows for the 800,000 measurements for each respective signal_id

    * each column contains one signal !

* Since each powerline has 3 rows of data (1 for each phase) in the metadata_train.csv and the train.parquet file that means the target is the same for each signal_id that has the same id_measurement

![image alt text](image_1.png)

![image alt text](image_2.png)

Based on our tiny sampling of these three power lines it looks like amplitude could be a significant factor in predicting whether these power lines are faulty.

* According to the wiki on the 3-phase power scheme:

In a symmetric three-phase power supply system, three conductors each carry an alternating current of the same frequency and voltage amplitude relative to a common reference but with a phase difference of one third of a cycle between each.

**Therefore, competitors should look into smoothing techniques to measure the amplitude of these waves.** Competitors should also measure the variance of the estimated amplitude for each powerline across the 3 phases.

* Distribution of target values throughout the data set. 

![image alt text](image_3.png)

	From the plots, we can easily see that fault data is too small and target is almost uniformly distributed in all phases

Example of damaged and undamaged signal:

![image alt text](image_4.png)

* Another example to see the difference between undamaged and damaged signal :

![image alt text](image_5.png)

![image alt text](image_6.png)

# First place solution: 

* Competitor focused on finding a way to pick a threshold for each trace to separate the peaks (signal) from the noise.

* First he found the local maxima in the data with a window size of 51 then picked the window sized based on the data.

* He then used a[ knee point detection](https://www.kaggle.com/kevinarvai/knee-elbow-point-detection) to find the noise floor. He ordered the peaks from highest to lowest and then looked for the point where the height of the peaks flattened off.

* For each peak identified in the preprocessing he calculated a few features, of which the first 2 where the most important:

1. The absolute height of the peak

2. The RMSE between the peak and a sawtooth shaped template. He used this because he noticed this sort of shape was common in traces marked as faulty

3. The absolute ratio of the peak to the next data point

4. The absolute ratio of the peak to the previous data point

5. The distance to the maximum, of opposite polarity to the peak, within a window of 5 either side of the peak

* For the model he chose to use LightGBM model with 9 features.

* For the features, he fed into the LightGBM model then aggregated the peak features for each measurement ID. 

* The model was trained on the measurement ID instead of the signal ID **because there were a number of examples in the training data where it looked as if all 3 signal traces had been marked as faulty even though only one had any obvious signs of a fault.** Some of the features were calculated for only certain portions of the phase of each signal so the position of each of the peaks was resolved. The features of which the first 5 were the most important, for each measurement ID were:

1. The total number of peaks

2. The number of peaks in quarters 0 and 2

3. The number of peaks in quarters 1 and 3

4. The mean "sawtooth" RMSE value in quarters 0 and 2

5. The std height in quarters 0 and 2

6. The mean height in quarters 0 and 2

7. The mean value of the ratio with the previous data point feature in quarters 0 and 2

8. The mean value of the ratio with the next data point feature in quarters 0 and 2

9. The mean value of the absolute distance to the opposite polarity maximum

* For cross validation method he used  5 fold cross validation.

For detailed solution, please check: 

[https://www.kaggle.com/mark4h/vsb-1st-place-solution/notebook](https://www.kaggle.com/mark4h/vsb-1st-place-solution/notebook)

If you see any mistakes feel free to change. 

### Resources :

1. [https://www.kaggle.com/sklasfeld/introduction-to-vsb-kaggle-competition](https://www.kaggle.com/sklasfeld/introduction-to-vsb-kaggle-competition)

2. [https://www.kaggle.com/jeffreyegan/vsb-power-line-fault-detection-approach/notebook](https://www.kaggle.com/jeffreyegan/vsb-power-line-fault-detection-approach/notebook)

3. [https://www.kaggle.com/go1dfish/basic-eda](https://www.kaggle.com/go1dfish/basic-eda)

4. [https://www.kaggle.com/c/vsb-power-line-fault-detection/discussion/87038](https://www.kaggle.com/c/vsb-power-line-fault-detection/discussion/87038)

5. [https://www.kaggle.com/c/vsb-power-line-fault-detection/discussion/75771](https://www.kaggle.com/c/vsb-power-line-fault-detection/discussion/75771)

6. [https://farid.one/kaggle-solutions/](https://farid.one/kaggle-solutions/) (all solutions)

