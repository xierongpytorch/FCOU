Journal Title: IEEE Transactions on Information Forensics & Security

Title of the **Manuscript**: Knowledge Editing for Federated Client Online Unlearning

Authors: Rong Xie, Zhong Chen, Yubo Song

The PyTorch implementation is coming shortly!<be>

### FCU:
![image text](https://github.com/xierongpytorch/FCOU/blob/main/PICTURE/FCU.png "FCU")

**FCU** follows a *post-hoc* workflow, where forgetting is performed **after** the global model has already been trained and deployed.

1. The server first completes standard federated training and releases the global model as an online service.
2. During normal operation, all clients interact with this deployed model for inference.
3. When a client later issues a *forget request*, the server must interrupt the normal service pipeline.
4. An offline unlearning procedure (e.g., retraining, replay, or recalibration) is selected and executed to remove the client’s influence.
5. A new “forgotten” model is produced and redeployed.


### FU under Gradient Inversion Attacks (GIA)

Recent studies have shown that **federated unlearning (FU) itself can introduce a new privacy vulnerability**.  
In particular, *Gradient Inversion Attacks (GIA)* exploit the **difference between the model before and after unlearning** as a side channel to reconstruct the data that was supposed to be forgotten.

Following the analysis of Zhou and Zhu (2025), the attack proceeds as follows.

**1. Attacker’s Observations**  
During normal federated training, the server (or a curious attacker) observes each client’s local model update in every round.  
After a forget request is issued, the attacker can also observe:
- the original global model before unlearning, and  
- the sequence of updates produced during the unlearning procedure, as well as the final unlearned model.

This gives the attacker access to **both pre-unlearning and post-unlearning model trajectories**.

**2. Isolating the Target Client’s Contribution**  
To focus on the forgotten client, the attacker reweights recorded updates based on their magnitudes, which serves as a proxy for client influence.  
By aggregating these weighted updates across rounds, the attacker reconstructs:
- an *approximate gradient trajectory of the original training*, and  
- an *approximate gradient trajectory of the unlearning process*.

Taking the difference between the two effectively isolates the gradient signal associated with the forgotten data.

**3. Gradient-Based Data Reconstruction**  
The attacker then performs a standard gradient inversion procedure.  
A dummy input is initialized, and its gradient (computed on the original model) is iteratively optimized to match the isolated gradient signal.  
With cosine similarity and image regularization, this process reconstructs inputs whose gradients align with the forgotten data.

**4. Attack Outcomes**  
- In *client-level or sample-level unlearning*, the reconstructed inputs reveal semantic content of the forgotten samples.  
- In *class unlearning*, comparing prediction logits before and after unlearning allows inference of the erased class distribution.


### FCOU:
![image text](https://github.com/xierongpytorch/FCOU/blob/main/PICTURE/FCOU.png "FCOU")

**FCOU** follows an *in-training* workflow, where forgetting is performed **during** federated optimization instead of post-hoc correction.

0. **Adversarial Samples (preprocessing, can start anytime).**  
   As soon as a client has local data (and a forget set is identified), it can locally prepare diffusion-based adversarial surrogates for the samples to be forgotten. This step is purely client-side and can be done *before* the client is selected in a federated round.

For each federated training round *t*:

1. The server broadcasts the current global model **wₜ** to the selected clients.
2. **Client Weight (Identify).** A target client with a forget request estimates its historical impact by comparing its cached local snapshot with **wₜ**, producing a client-level contribution score.
3. **Data Importance (Identify).** The client computes gradients on the forget set and ranks parameters by gradient magnitude to locate the weights most influenced by the data to be removed.
4. **Parameter Pruning (Remove).** The high-impact parameter subset is selectively pruned/edited so that the most sensitive internal traces are directly targeted.
5. **Knowledge Editing (Remove).** The pruned parameters are re-initialized and locally edited using the adversarial surrogates, removing both data-level and model-level remnants.
6. **Optimize Differences (Maintain).** The client fine-tunes the edited model with drift control and KL-alignment so that predictions on retained data stay close to the global hypothesis, limiting utility loss.
7. Clients without forget requests perform a standard local FL update.
8. The server aggregates all local models via FedAvg to obtain **wₜ₊₁**, which immediately updates the deployed model service.

This design yields **two key benefits**.  
First, because unlearning is embedded within the training loop, FCOU avoids exposing a clean pair of “before vs. after” snapshot models, thereby reducing the pre/post side channel exploited by recent FU-specific inference and inversion attacks.  
Second, forgetting is completed without extra post-hoc unlearning phases or additional communication rounds, making the workflow scalable to repeated and time-varying forget requests.


