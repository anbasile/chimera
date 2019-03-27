# Chimera
![Chimera](git-assets/chimera.webp)

## Environment
We recommend installing all dependencies in a separate Conda environment.

### GPU-support
This code will run with or without a CUDA, but we recommend using a machine with CUDA.

### Dependencies
Execute `setup.sh`. This will install pip dependencies, as well as OpenNMT.

### Demo
To run the demo, execute `server/server.py`, preferably on a machine with a GPU. Running it for the first time will process the data and train the models, then expose a server for you to play with.

## TODO
- Pipeline class should load cache on the fly instead of preemptive.
- Pipeline executer should have progressbar support.

## Process
For training, the main pipeline consists of these sub-pipelines:
1. Preprocess Training (both train and dev sets)
    1. Load the data-set
    1. Convert RDFs to graphs
    1. Fix misspellings
    1. Locate entities in the text
    1. Match plans for each graph and reference
    1. Tokenize the plans and reference sentences   
1. Train Model
    1. Initialize model
    1. Pre-process training data
    1. Train Model
    1. Find best checkpoint, chart all checkpoints
1. Learn Score
    1. Get good plans from training set
    1. Learn Relation-Direction Expert
    1. Learn Global-Direction Expert
    1. Learn Splitting-Tendencies Expert
    1. Learn Relation-Transitions Expert
    1. Create Product of Experts
1. Preprocess Test Set
    1. Load the data-set
    1. Convert RDFs to graphs
    1. Fix misspellings
    1. Generate best plan
    1. Tokenize plans & sentences
1. Translate
    1. Translate test plans into text
    1. Post-process translated texts
    1. Save Translations to file (for human reference)
1. Evaluate model performance
    1. Evaluate test reader

Once running the main pipeline, every pipeline result is cached. 
If the cache is removed, the pipeline will continue from its last un-cached process.

**Note:** by default, all pipelines are muted, meaning any screen output will not present on screen.


## Example

### WebNLG
Setting the `config` parameter to be `Config(reader=WebNLGDataReader)` using the naive planner.

Output running for the first time:
![First Run Pipeline](git-assets/first-run.png)

Output running for the second time: (runs for just a few seconds to load the caches)
![Second Run Pipeline](git-assets/second-run.png)

The expected result (will show on screen) reported by `multi-bleu.perl` is around:
- BLEU [47.27, 79.6, 55.3, 39.4, 28.7] (40,000 steps)
- BLEU [46.87, 79.2, 54.8, 39.1, 28.4] (40,000 steps)
- BLEU [46.70, 79.3, 55.0, 38.9, 28.0] (40,000 steps)
- BLEU [45.49, 77.9, 53.7, 37.8, 27.1] (40,000 steps)

### [Delexicalized WebNLG](https://github.com/ThiagoCF05/webnlg)
This dataset does not use a heuristic for entity matches, instead it was constructed manually.
This means it is of higher quality and easier to find a correct plan-match in train time.

Setting the `config` parameter to be `Config(reader=DelexWebNLGDataReader, test_reader=WebNLGDataReader)` using the naive planner.

The expected result is around:
- BLEU [45.26, 80.1, 54.8, 37.9, 26.6] (20,000 steps)
- BLEU [44.77, 79.9, 54.0, 37.1, 25.9] (20,000 steps)
- BLEU [45.71, 81.1, 55.0, 37.9, 26.3] (20,000 steps)
- BLEU [45.66, 80.2, 54.9, 37.8, 26.5] (20,000 steps)

We attribute the worse BLEU to the fact the delexicalizations also remove articles and other text around it, and without proper referring expressions generations while the texts should have better structure, they are worse in fluency.

## Speed Compared to Score
If we try our neural planner, we get:
- Training time: 60 seconds (can easily be 30 seconds or less)
- Test time (1 cpu): 3 seconds
- WebNLG - BLEU [45.79, 79.0, 54.4, 38.5, 28.0] (20,000 steps m1)
- WebNLG - BLEU [46.35, 78.1, 54.5, 38.7, 28.1] (40,000 steps m2)
- Delexicalized WebNLG - BLEU [44.21, 81.7, 56.4, 39.5, 28.2] (40,000 steps m3)
- Delexicalized WebNLG - BLEU [44.46, 80.1, 55.1, 38.3, 27.1] (40,000 steps m4)

Compared to the following using our naive product of experts planner, with the same realizer snapshots:
- Training time: 0 seconds
- Test time (1 cpu): 5300 seconds
- Test time (40 cpus): 450 seconds
- WebNLG - BLEU [45.19, 77.8, 53.2, 37.4, 27.0] (20,000 steps m1)
- WebNLG - BLEU [45.49, 77.9, 53.7, 37.8, 27.1] (40,000 steps m2)
- Delexicalized WebNLG - BLEU [44.77, 79.3, 53.7, 37.4, 26.4] (40,000 steps m3)
- Delexicalized WebNLG - BLEU [46.01, 79.8, 55.0, 38.3, 27.2] (40,000 steps m4)


## Neural Planner Comparison

Greedy best neural plan: 
- BLEU [45.97, 77.8, 54.1, 38.4, 28.0]
Sample neural:
- BLEU [43.76, 77.1, 51.9, 35.9, 25.5]
- BLEU [43.74, 76.9, 51.9, 35.9, 25.5]
- BLEU [44.7, 77.9, 52.7, 36.8, 26.4]
- BLEU [44.64, 77.9, 52.7, 36.8, 26.3]
- BLEU [44.12, 77.5, 52.2, 36.2, 25.9]


Combined model:
Sample 0 + Greedy = BLEU [45.88, 77.7, 54.0, 38.2, 27.7]
Sample 1 + Greedy = BLEU [45.35, 78.7, 54.0, 38.1, 27.5]
Sample 5 + Greedy = BLEU [45.87, 78.6, 54.1, 38.1, 27.6]
Sample 50 + Greedy = BLEU [45.58, 78.2, 53.7, 37.7, 27.2]
