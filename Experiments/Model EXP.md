# Modeling Experiments #

Modeling experiements were done so as to build a better EQTransformer. 

In order to do that we tried 14 different model in total. 

In each experiments we remove existing parts or add parts to existing code.

For simplicity, we will explain two best models for each training data.
* Models trained on _STEADmini_ :
  * Salvation
  * Genesis
  * Vanilla
  * GRU
  * 4x4
  * 2LSTM
  * ORI


* Models trained on _merged_ :
  * BC
  * BCLOS
  * BO
  * Partial Vanilla
  
* Models trained on _STEAD-micro_ :
  * EQUtils
  * Kernel
  * NB_Filter
  *

## Models Trained On _Merged_ : ##

### BCLOS ###
BCLOS is an acronym for *BiLSTM Closed LSTM Open*.

In this experiment we removed BiLSTM part from existing code and add another LSTM block in the place of BiLSTM.  







