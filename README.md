## Explainable Link Prediction for Emerging Entities in Knowlegde Graph

### Acknowledgement
This repository is adopted from the [source code](https://github.com/salesforce/MultiHopKG) of the paper [Multi-Hop Knowledge Graph Reasoning with Reward Shaping. Lin et al., 2018](https://arxiv.org/abs/1808.10568)
We thank the authors, especially, Xi Victoria Lin for helping us to understand their source code.

### Requirements
python 3.6+ <br>
pytorch 1.4.0 <br>
tqdm 4.9.0

All experiments are run on NVIDIA Titan RTX GPUs with 24GB memory.

### Data Processing
Run the following command to preprocess the datasets.
```
./experiment.sh configs/<dataset>.sh --process_data <gpu-ID>
```

`<dataset>` is the name of the dataset in the `./data` directory. In our experiments, the three datasets used are: `fb15k-237`, `wn18rr` and `nell-995`. 
`<gpu-ID>` is a non-negative integer number representing the GPU index.

### Model Training 
For reward shaping, we used ConvE model. 

1. Train the ConvE model on the training split of the data
```
./experiment-emb.sh configs/<dataset>-conve.sh --train <gpu-ID>
```
2. Train RL models (policy gradient + reward shaping)
```
./experiment-rs.sh configs/<dataset>-rs.sh --train <gpu-ID>
```
3. To train an ablated version of the model (only policy gradient)
```
./experiment.sh configs/<dataset>.sh --train <gpu-ID>
```
* Note: To train the RL models using reward shaping, make sure 1) you have pre-trained the embedding-based ConvE model and 2) set the file path pointers ```conve_state_dict_path``` to the pre-trained embedding-based models correctly 
in the ```configs/<dataset>-rs.sh``` or ```configs/<dataset>.sh``` files.

### Evaluation
To generate the evaluation results of a trained model, simply change the `--train` flag in the commands above to `--inference`. 

For example, the following command performs inference with the RL models (policy gradient + reward shaping) and prints the evaluation results (on both dev and test sets).
```
./experiment-rs.sh configs/<dataset>-rs.sh --inference <gpu-ID>
```

To print the inference paths generated by beam search during inference, use the `--save_beam_search_paths` flag:
```
./experiment-rs.sh configs/<dataset>-rs.sh --inference <gpu-ID> --save_beam_search_paths
```

* Note for the NELL-995 dataset: 

  On this dataset we split the original training data into `train.triples` and `dev.triples`, and the final model to test has to be trained with these two files combined. 
  1. To obtain the correct test set results, you need to add the `--test` flag to all training and inference commands.  
    ```
    # You may need to adjust the number of training epochs based on the dev set development.
    ./experiment.sh configs/nell-995.sh --process_data <gpu-ID> --test
    ./experiment-emb.sh configs/nell-995-conve.sh --train <gpu-ID> --test
    ./experiment-rs.sh configs/NELL-995-rs.sh --train <gpu-ID> --test
    ./experiment-rs.sh configs/NELL-995-rs.sh --inference <gpu-ID> --test
    ``` 

### Change the hyperparameters
To change the hyperparameters and other experiment set up, start from the [configuration files](configs).
