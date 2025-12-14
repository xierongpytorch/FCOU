Journal Title: IEEE Transactions on Information Forensics & Security

Title of the **Manuscript**: Knowledge Editing for Federated Client Online Unlearning

Authors: Rong Xie, Zhong Chen, Yubo Song

The PyTorch implementation is coming shortly!<be>

### FCU:
![image text](https://github.com/xierongpytorch/FCOU/blob/main/PICTURE/FCU.png "FCU")

**Traditional Federated Client Unlearning (FCU)** follows a *post-hoc* workflow, where forgetting is performed **after** the global model has already been trained and deployed.

1. The server first completes standard federated training and releases the global model as an online service.
2. During normal operation, all clients interact with this deployed model for inference.
3. When a client later issues a *forget request*, the server must interrupt the normal service pipeline.
4. An offline unlearning procedure (e.g., retraining, replay, or recalibration) is selected and executed to remove the client’s influence.
5. A new “forgotten” model is produced and redeployed.

This post-hoc design introduces **two major drawbacks**.  
First, an adversary with access to both the original and the forgotten models can compare their prediction confidence vectors, enabling membership inference or partial data reconstruction attacks.  
Second, each forget request triggers additional computation and communication overhead, which becomes especially costly when multiple unlearning requests arrive over time.


### FCOU:
![image text](https://github.com/xierongpytorch/FCOU/blob/main/PICTURE/FCOU.png "FCOU")
