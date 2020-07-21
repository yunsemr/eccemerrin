# LANL Earthquake Prediction :

# Background:

The competition requires competitors to predict when an earthquake will occur by predicting the time remaining (Time to failure, or TTF) until a laboratory earthquake occurs from real-time seismic data.

*  The metric we were graded on was Mean Absolute Error (MAE).

*  The data is not real earthquake data; it is from a laboratory experiment. 

*  The training data is a single, continuous segment of experimental data.

*  In contrast, the test data consists of several different sequences, called segments, that   may correspond to different experiments. The regular pattern we might find in the train set does not match those of the test segments.

*  For each test data segment with its corresponding seg_id we are asked to predict it's single time until the lab earthquake takes place.

*  Both the training and the testing set come from the same experiment.

*  There is no overlap between the training and testing sets, that are contiguous in time.

## Experiment :

Basically, they took two layers of rock and subjected them to pressure and velocity. This experiment allowed them to record shear stress of the rock. The shear stress builds up more and more as time goes on, until eventually the shear stress is unambiguously large that they know a simulated earthquake has occurred. Then, the cycle is reset and they can obtain a new earthquake.

![image_0](https://user-images.githubusercontent.com/62555970/87957287-beaca480-cab8-11ea-8937-7f8edb6d3af9.png)

 For further reading about the experiment, please check  [here ](https://www.nature.com/articles/ncomms11104) and [here](https://www.kaggle.com/c/LANL-Earthquake-Prediction/overview/additional-information).

# Let's get familiar with the data :

## Training data:

* The training data is a single, continuous segment of experimental data.

* We have 2624 files in test folder.

* The data is recorded in bins of 4096 samples. Within those bins seismic data is recorded at 4MHz, but there is a 12 microseconds gap between each bin, an artifact of the recording device.

* There are 629,145,480 data points in the training set to train on--this is about 157 seconds of data. 

* The dimension of the data is quite large, in excess of 600 millions rows of data.

* The total size of the train data is almost 9 GB but we don't want to wait too long just for a first impression, let's load only some rows:

![image_1](https://user-images.githubusercontent.com/62555970/87957366-d4ba6500-cab8-11ea-90b3-f4d3f694ef2b.png)

**acoustic_data**  can be interpreted as a seismic signal 

**time_to_failure**  corresponds to the time until the laboratory earthquake takes place (quaketime)

When we look at the quaketime (time_to_failure) of these first rows seems to be always the same. But it is not true.


![image_2](https://user-images.githubusercontent.com/62555970/87957397-ddab3680-cab8-11ea-9f7b-0778e36c48b5.png)

We can see that they are not the same and that pandas has rounded them off.

 And we can see that the time seems to decrease. 

![image_3](https://user-images.githubusercontent.com/62555970/87957425-eb60bc00-cab8-11ea-9c8b-d8fe1d0b63a8.png)

![image_4](https://user-images.githubusercontent.com/62555970/87957427-ec91e900-cab8-11ea-8c41-4db844fc6443.png)



#### Take-away :

* We can see only one time in 10 Mio rows when quaketime goes to 0. This is a time point where an earthquake in the lab occurs.

* There are many small oscillations until a heavy peak of the signal occurs. Then it takes some time with smaller oscillations and the earthquake occurs.

## Test data:

* The test data consists of a folder containing many small segments. 

* Each segment only has 150,000 data points of seismic data, which corresponds to 0.0375 seconds of seismic data.

* The data *within* each test file is continuous, but the test files do not represent a continuous segment of the experiment; thus, the predictions cannot be assumed to follow the same regular pattern seen in the training file.

* For each test data segment with its corresponding seg_id we are asked to predict it's single time until the lab earthquake takes place.

How the test data look like :

![image_5](https://user-images.githubusercontent.com/62555970/87957469-fb789b80-cab8-11ea-9f1d-d0995bbcb608.png)


#### Take-away :

* These test segment examples differ a lot in the occurrences of small peaks that seem to be similar to those in the train data before and after the heavy signal peak that occurred some time before the lab earthquake took place.

* The real challenge was trying to translate the large, continuous training data (about 157 seconds of data.) to make predictions with a small amount of testing data(0.0375 seconds of seismic data.). 

# What did the winners do ?

## What they do on dataset :

* Firstly, they tried [KFold](https://machinelearningmastery.com/k-fold-cross-validation/) cross validation method on the dataset and had a very good results. But they believed that using KFold was leaking information when they were training. 

* For example, by using KFold, they were training on segments [0,150k] and [300k,450k] but validating on segment [150k,300k]. 

They believed that training on segments that were adjacent to the validation segment would bleed information into the adjacent segment. Therefore, they concluded their validation strategy was risky and needed to change.

* They began to focus on a validation strategy called **Leave 1 Earthquake Out**. They trained on all earthquakes 1-16, then validated on earthquake 17. Then  trained on earthquakes 1-15 & 17, then validated on earthquake 16. Etc. They averaged the predictions over every validation. This caused their leaderboard score to worsen, so they decided to change their strategy again.

* Then they switched to **Leave 2 Earthquakes Out **validation. They thought that  this was way more robust, since they believed that the Leave 1 Earthquake Out was overfitting to each earthquake because** the validation set was not representative of the training/testing set. **So they were created 17 choose 2 = 136 models and averaged everything together. They wanted to stack various models together using Leave 2 Earthquakes Out validation, but stacking overfit very badly. Why? **It is because the predictions made on single earthquakes were averaged over 16 times, and so this just led to major information bleed**.

* They decided to change their strategy  to a **custom-fold strategy** which is again  cross validation strategy. They prepare testing set had 2 long earthquakes, 2 medium earthquakes, and 4 small earthquakes. So instead of using Leave 2 Earthquakes Out, they use 3-fold, where each fold had (roughly) 1 long earthquake, 1 medium earthquake, and 2 small earthquakes.

* Separately, they also began to focus on making the train set features more similar to the test set features. They noticed the mean & median values were increasing as the laboratory experiment went forward in time. To partly overcome this, they added a constant noise to each 150k segment (both in train and test) by calculating np.random.normal(0, 0.5, 150_000). Additionally, after noise addition, they subtracted the median of the segment. Sometimes they denoised the signal segments so the model wouldn't learn from random volatilities. This improved generalization. 

* They  found that the test data should look different to training data in a few ways when comparing features by applying [KS statistics ](https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test) between train and test. That’s when they decided to sample the train data to make it look more like they expect test data to look like (only from looking at feature distributions).They started by manually upsampling certain areas of train data, but gave up on that after a few tries and then they  found a very nice way of aligning train and test data.

    * To determine which earthquakes to select, they plotted the Kolmogorov-Smirnov  statistic versus the TTF for their features. Basically, they selected different combinations of earthquakes then looked at the plot to find the best combination that preserved a low statistic that appeared at the TTF they desired. From the plot below, you can see that the lowest Kolmogorov-Smirnov statistic (which means, the place where the train's features and the test's features were most similar in distribution) appears at 6+ TTF.You can also see here in the green a feature that performs worse at 6+ TTF because its Kolmogorov-Smirnov statistic gets higher. This would be a feature they would remove, since the distribution between the train and test at 6+ TTF was different.


![image_6](https://user-images.githubusercontent.com/62555970/87957527-10edc580-cab9-11ea-9342-2e451805e0bf.png)


**So what they did is  they calculated a handful of features for train and test and tried to find a good subset of full earth-quakes in train, so that the overall feature distributions are similar to those of the full test data. **They did this by sampling 10 full earthquakes multiple times (up to 10k times) on train, and comparing the average KS statistic of all selected features on the sampled earthquakes to the feature dists in full test. 

Now that they had sampled train data that they thought to be similar to test just purely based on statistical analysis, and now that they had features that should not have any time leaks, they decided on doing a simple **shuffled 3-fold** on that data.

## Models:

Their final submit is a hillclimber blend of three types of models: 

* [LGB](https://github.com/microsoft/LightGBM),

* [SVR](https://medium.com/pursuitnotes/support-vector-regression-in-6-steps-with-python-c4569acd062d),

* NN. 

However, their main two models were the LightGBM model and the neural network model.

They knew that it would be hard to generalize since the signal data was so noisy. Therefore they didn't want to include many features; they only wanted to add them if they improved the cross-validation score and if they were uncorrelated with previous features.

So;

* The LightGBM model used six features:[ a rolling standard deviation,](https://jonisalonen.com/2014/efficient-and-accurate-rolling-standard-deviation/) two [Mel-frequency cepstral coefficients](https://en.wikipedia.org/wiki/Mel-frequency_cepstrum)*, [number of peaks](https://plotly.com/python/peak-finding/), [autocorrelation](https://machinelearningmastery.com/gentle-introduction-autocorrelation-partial-autocorrelation/), and the rate of the sum of the [Hanning Window Fast Fourier Transformation](https://download.ni.com/evaluation/pxi/Understanding%20FFTs%20and%20Windowing.pdf) between the raw and denoised signal. 

* Also, since the LightGBM had a low `num_leaves’, the model was less complex and could generalize better.

* The neural network model used more features. It used the number of times the signal crossed the x-axis, lots of Mel-frequency cepstral coefficients, number of peaks features, rolling standard deviation features,[ 95th and 99th quantiles](https://www.manageengine.com/network-monitoring/faq/95th-percentile-calculation.html), and some FFT features. Some of these were fit on denoised versions of the segments.

* The architecture of this network is essentially feed-forward (even though it uses [LSTM](https://colah.github.io/posts/2015-08-Understanding-LSTMs/) and Conv1D layers). 

* The unusual technique however is that it is fit to multiple objectives: Time to Failure, Time since Failure, and binary indicator TTF < 0.5 seconds. The model is trained to fit to all three of these objectives at once with objectives weights of 8, 1, and 1 respectively.

* Because of the multi-objective, the model is forced to create weights that optimize for Time Since Failure and the binary indicator, while focusing mostly on TTF. They think that  this allowed the model to generalize better to the signal. The performance was better using multi-objective than just fitting to TTF alone.

* [https://medium.com/datarunner/librosa-9729c09ecf7a](https://medium.com/datarunner/librosa-9729c09ecf7a) Mel Frequency için Öznitelik çıkarımı başlığı altında  kısa bir türkçe anlatım.

With all the steps described above, they also managed to make the distribution of test predictions very similar the [oof predictions](https://machinelearningmastery.com/out-of-fold-predictions-in-machine-learning/). The following image shows for a single LGB the oof (blue) vs. test predictions (orange). The KS-test between those two does not reject the null hypothesis of them being equally distributed.

![image_7](https://user-images.githubusercontent.com/62555970/87957552-19460080-cab9-11ea-99ca-a3979601cf37.png)

Full version of their code : [https://www.kaggle.com/dkaraflos/1-geomean-nn-and-6featlgbm-2-259-private-lb](https://www.kaggle.com/dkaraflos/1-geomean-nn-and-6featlgbm-2-259-private-lb)

# What did the second competitor do ?

## What the competitor do on dataset :

* The competitor select segments from the training ​​data and create a private-set-like data based on the inter-earthquake times seen in the figure below*.

![image_8](https://user-images.githubusercontent.com/62555970/87957554-1a772d80-cab9-11ea-90d3-8421b57766aa.png)

*If you are curious about about how this plot generated please read: [https://www.kaggle.com/c/LANL-Earthquake-Prediction/discussion/90664#latest-535844](https://www.kaggle.com/c/LANL-Earthquake-Prediction/discussion/90664#latest-535844)

* private-set-like data:



![image_9](https://user-images.githubusercontent.com/62555970/87957555-1b0fc400-cab9-11ea-8a05-8c869c82b150.png)

The first segment is counted twice; the private set has 8 segments. The last segment is created from the 14-sec segment and shifted up by 2 seconds, and the last 4.5 second is cut.

## Model:

They use 32 features:

* standard deviation (std)

* std_nopeak: Std of data that are not part of peaks

* kurtosis_truncated: Kurtosis of data with abs(v - mean(v)) < 20

* 7 peak counts: Number of peaks with height > 50, 75, …, 200

* 5 percentiles: 95 percentile - 5 percentile, 80 - 20, 70 - 30, 60 - 40.

* trend: slope of robust linear regression to 30 sub chunks of std_truncated

* trend_error: Abs difference in the slope of RANSAC and Huber fit

* power spectrum; Fast-Fourier Transform the data and average the absolute value in 15 bins

They choose combinations such as 95 percentile - 5 percentile to avoid direct dependence on the mean; which is drifting with time. Same for the peak height; the peak height is defined as (max - min)/2.

They randomly select 2000x1000 training chunks of length 150_000 from their private-like set, which is 1000 times the number of independent/non-overlapping chunks and put all of them into [CatBoost](https://towardsdatascience.com/https-medium-com-talperetz24-mastering-the-new-generation-of-gradient-boosting-db04062a7ea2). They do not provide CV data to the regressor; everything ​is in the training set.

They also tried to predict time since failure using all the training set and tried to stitch together with time to failure, but they were not able to do that successfully.

# Resources : 

1. [https://www.kaggle.com/c/LANL-Earthquake-Prediction/discussion/94390](https://www.kaggle.com/c/LANL-Earthquake-Prediction/discussion/94390)

2. [https://www.kaggle.com/allunia/shaking-earth](https://www.kaggle.com/allunia/shaking-earth)( the most beautifully organized notebook) 

3. [https://www.kaggle.com/inversion/basic-feature-benchmark](https://www.kaggle.com/inversion/basic-feature-benchmark)

4. [https://www.kaggle.com/c/LANL-Earthquake-Prediction/discussion/77526](https://www.kaggle.com/c/LANL-Earthquake-Prediction/discussion/77526)

5. [https://www.kaggle.com/c/LANL-Earthquake-Prediction/discussion/77525](https://www.kaggle.com/c/LANL-Earthquake-Prediction/discussion/77525)

6. [https://www.linkedin.com/pulse/my-team-won-20000-1st-place-kaggles-earthquake-corey-levinson/](https://www.linkedin.com/pulse/my-team-won-20000-1st-place-kaggles-earthquake-corey-levinson/)

7. [https://farid.one/kaggle-solutions/](https://farid.one/kaggle-solutions/) (all solutions)

# Some additional readings about different boosting algorithms:

* [https://medium.com/riskified-technology/xgboost-lightgbm-or-catboost-which-boosting-algorithm-should-i-use-e7fda7bb36bc](https://medium.com/riskified-technology/xgboost-lightgbm-or-catboost-which-boosting-algorithm-should-i-use-e7fda7bb36bc)

* [https://towardsdatascience.com/catboost-vs-light-gbm-vs-xgboost-5f93620723db](https://towardsdatascience.com/catboost-vs-light-gbm-vs-xgboost-5f93620723db)



