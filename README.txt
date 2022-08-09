0) Requirements
- Linux or MacOS (We run our experiments on Linux server and MacOs)
- RDKit (version='2020.09.5')
- Python (version=3.9.4)
- Pytorch (version=1.8.0)

1) Creating dataset of reaction trees

We extracted molecules from USPTO reaction data set and used Retro* to synthesize them to obtain a set of (multi-step) chemical reactions.
Before dataset creation, need to set up the environment as follows (also see https://github.com/binghong-ml/retro_star for more details of the implementation and settings of retro*):
    +) Download and decompress the following USPTO dataset: https://figshare.com/articles/MongoDB_dump_compressed_/4833482. Its name after decompression is "dump". Run the MongoDB Server (by mongod), open a new terminal and type: mongorestore <path to dump folder> to import "dump" into the list of mongo databases.
    +) In reaction_trees_creator, type:  "conda env create -f environment.yml; conda activate retro_star_env” for creating a conda environment
    +) Download and unzip the files from this link: https://www.dropbox.com/s/ar9cupb18hv96gj/retro_data.zip?dl=0, and put all the folders (dataset/, one_step_model/ and saved_models/) under the retro_star directory.
    +) Install Retro* lib: "pip install -e retro_star/packages/mlp_retrosyn; pip install -e retro_star/packages/rdchiral; pip install -e ."
    +) Install MongoDB on MacOS: please follow this link: https://www.geeksforgeeks.org/how-to-install-mongodb-on-macos/


Then, go to the folder "reaction_trees_creator/retro_star/" and type: python run_to_create_reaction_trees.py to generate reaction trees. 
The information about reactants and templates can be referred in the following files: rxngenerator/reaction_trees_creator/retro_star/dataset/origin_dict.csv 
and rxngenerator/reaction_trees_creator/retro_star/one_step_model/template_rules_1.dat, respectively. 
The extracted reaction trees are stored in /data/synthetic_routes.txt. 


2) Filtering the dataset of reaction trees
To make sure that starting molecules and reaction templates are popular for the chemists, we filtered out the original set of reactions so that each reaction 
contains starting molecules and templates that occur at least five times in the filtered set. 
Copy synthetic_routes.txt (from step 1) to the folder /data, go to /data and type: python filter_dataset.py

3) Training
To train the model, type the following command:
python trainvae.py -w 200 -l 50 -d 2 -v "weights/data.txt_fragmentvocab.txt" -t "data/data.txt"

The weights of the trained model are saved in the folder "weights", which will be loaded to run sampling and Bayesian Optimization.

4) Sampling
To sample new molecules with trained model, please run:
python sample.py -w 200 -l 50 -d 2 -v "weights/data.txt_fragmentvocab.txt" -t "data/data.txt" -s "weights/uspto_vae_iter-100.npy" -o "Results/generated_rxns.txt"

The generated molecules and associated reaction trees are saved in file: "Results/generated_rxns.txt"

5) Bayesian optimization
The Bayesian optimization experiments use sparse Gaussian processes coded in theano. To install Theano, go to the folder Theano-master and type: python setup.py install

Then, go to the folder bo and type the following command to run Bayesian optimization:
python run_bo.py -w 200 -l 50 -d 2 -r 1 -v "../weights/data.txt_fragmentvocab.txt" -t "../data/data.txt" -s "../weights/uspto_vae_iter-100.npy" -m "qed"

Please change the parameter -r with different random seed numbers. We performed 10 times of running BO, which results in 10 files of valid reaction trees saved in the folder Results. 
To see reaction trees with top optimized QED scores, simply type: python print_results.py
