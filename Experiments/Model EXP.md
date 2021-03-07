# 2-Modeling Experiments #

Modeling experiements were done so as to build a better EQTransformer. 

In order to do that we tried 14 different models in total. 

In each experiments we remove existing parts or add parts to existing code.

For simplicity, we will explain the two most promising models for each training data.

* Models trained on _STEADmini_ :
  * Salvation
  * Genesis
  * Vanilla
  * GRU
  * 4x4
  * 2LSTM
  * ORI
  * EQT (for comparison)

* Models trained on _merged_ :
  * BC
  * BCLOS
  * BO
  * Partial Vanilla
  * EQT (for comparison)
  
* Models trained on _STEAD-micro_ :
  * EQUtils
  * Kernel
  * NB_Filter
  * EQT (for comparison)
  

### Models Trained On _Merged_ : ###

#### BCLOS ####
BCLOS is an acronym for *BiLSTM Closed LSTM Open*.

BiLSTM's have a structure which allow them to reach information from backward and forward. However, LSTM's have a structure which allow them to reach information just from backward. 

So, I wonder what happen if I remove a bidirectional LSTM block from model and add an unidirectional LSTM block in the place of it. So, I removed BiLSTM part from existing code and add a LSTM block in the place of BiLSTM. 


Before seeing results, I expected that results of Unidirectional layers would be much worse then Bidirectional layers. Suprisingly, *BCLOS* was better in 16 parameter out of 25. Which are;
 * **det_precision**, **d_fp**, *d_tn**
 * **s_recall*, *s_precison*, *s_mae*, *s_rmse*, *s_fp*, *s_tn*, *s_fn**
 * **p_recall*, *p_precision*, *p_tp*, *p_fp*, *p_tn*, *p_fn**


#### BC ####

 







