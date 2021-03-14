# 2-Modeling Experiments #

Modeling experiements were done so as to build a better EQTransformer. 

In order to do that we tried 14 different models in total. 

In each experiments we remove existing parts or add parts to existing code.

* Models trained on _STEADmini_ :
  * Salvation
  * Genesis
  * Vanilla
  * GRU
  * 4x4
  * 2LSTM
  * EQT (for comparison)

* Models trained on _merged_ :
  * BCLOS
  * BC
  * BO
  * Partial Vanilla
  * EQT (for comparison)
  
* Models trained on _STEAD-micro_ :
  * nb_filters_changed_EQTUtils
  * kernel_size_changed
  * nb_filters_changed_Trainer
  * EQT (for comparison)
  

### Models Trained On _Merged_ : ###

#### BCLOS ####
BCLOS is an acronym for *BiLSTM Closed LSTM Open*.

BiLSTM's have a structure which allow them to reach information from backward and forward. However, LSTM's have a structure which allow them to reach information just from backward. 

So, I wonder what happen if I remove a bidirectional LSTM block from model and add an unidirectional LSTM block in the place of it. So, I removed BiLSTM part from existing code and add a LSTM block in the place of BiLSTM. 


Before seeing results, I expected that results of Unidirectional layers would be much worse then Bidirectional layers. Suprisingly, *BCLOS* was better in 19 parameters out of 25. Those are;
 * **det_recall**, **det_precision**, **d_fp**, **d_tn**, **d,_fn**
 * **s_recall**, **s_precison**, **s_mae**, **s_rmse**, **s_fp**, **s_fn**
 * **p_recall**, **p_precision**,**p_mae**, **p_rmse**, **p_tp**, **p_fp**, **p_tn**, **p_fn**


#### BC ####
BC is an acronym for *Both Closed*. 

In this architecture, I removed both BiLSTM layers and added LSTM layers  from the common layers mentioned in [Google doc](https://docs.google.com/document/d/1JxLc_Bp0wNTSlZUpWlf_87riHRQKDenNGf665mFMVpQ/edit#heading=h.moi7c1x12w31).

As expected, result of this architecture was worse than **BCLOS**. *BC* was better than EQT in 9 parameters out of 25. Those are;
 * **det_recall**,  **d_fn**
 * **s_rmse**, **s_tn**
 * **p_precision**, **p_mae**, **p_rmse**, **p_fp**, **p_tn**

#### Partial Vanilla ####

In *Partial Vanilla* architecture, I removed BiLSTM layers and CNN Layers from the common layers mentioned in [Google doc](https://docs.google.com/document/d/1JxLc_Bp0wNTSlZUpWlf_87riHRQKDenNGf665mFMVpQ/edit#heading=h.moi7c1x12w31).

*Partial Vanilla* was better than EQT in 9 parameters out of 25. Those are;
 * **det_recall**,  **d_fn**
 * **s_mae**, **s_rmse**, **s_tn**
 * **p_recall**, **p_mae**, **p_rmse**, **p_fn**

#### BO ####
BO is an acronym for *Both Open*.

In this architecture; I add LSTM layers, as successive layers of CNN layers, to the common layers mentioned in [Google doc](https://docs.google.com/document/d/1JxLc_Bp0wNTSlZUpWlf_87riHRQKDenNGf665mFMVpQ/edit#heading=h.moi7c1x12w31). 

*BO* was better than EQT in 6 parameters out of 25. Those are;
 * **det_recall**
 * **s_mae**, **s_rmse**, **s_tn**
 * **p_mae**, **p_rmse**
 
### Models Trained On _STEAD-micro_ ###

#### nb_filters_changed_EQTUtils ####
In *nb_filters_changed_EQTUtils* model we changed the nb_filters variable in EqT_utils.py doc (line=2734). Initially, variable was nb_filters=[8, 16, 16, 32, 32, 96, 96, 128] then, changed to nb_filters=[8, 16, 16, 32, 32, 64, 64, 128].


#### kernel_size_changed ####

In *kernel_size_changed* model we changed the kernel size in trainer.py doc (line=417). Initially, the kernel size was kernel_size=[11, 9, 7, 7, 5, 5, 3] then, changed to kernel_size=[10, 9, 8, 7, 6, 5, 4].


#### nb_filters_changed_trainer ####

In *nb_filters_changed_trainer* model, we changed the filter sizes in trainer.py doc (line=416). The original filter size were nb_filters=[8, 16, 16, 32, 32, 64, 64] then, change to nb_filters=[8, 8, 32, 32, 128, 128, 128].
