# Deep-BET 

## Installation Instructions

Please note that python3 is required and [conda](https://www.anaconda.com/) is preferred.

```bash
git clone https://github.com/CBICA/Deep-BET.git
cd Deep-BET
conda env create -f requirements.yml # create a virtual environment named deepbet
conda activate deepbet # activate it
latesttag=$(git describe --tags) # get the latest tag [bash-only]
echo checking out ${latesttag}
git checkout ${latesttag}
python setup.py install # install dependencies and Deep-BET
```

## Generating brain masks for your data using our pre-trained models

- This application currently has two modes (more coming soon):
  - Modality Agnostic (MA)
  - Multi-4, i.e., using all 4 structural modalities

### Steps to run application

1. Co-registration within patient to the [SRI-24 atlas](https://www.nitrc.org/projects/sri24/) in the LPS/RAI space.

    An easy way to do this is using the [```BraTSPipeline``` application](https://cbica.github.io/CaPTk/preprocessing_brats.html) from the [Cancer Imaging Phenomics Toolkit (CaPTk)](https://github.com/CBICA/CaPTk/). This pipeline currently uses a pre-trained model to extract the skull but the processed images (in the order defined above till registration) are also saved.

2. Make an Input CSV including paths to the co-registered images (prepared in the previous step) that you wish to make brain masks.

  - Multi-4 (use all 4 structural modalities): Prepare a CSV file with the following headers:
  `Patient_ID,T1_path,T2_path,T1ce_path,Flair_path`

  - Modality-agnostic (works with any structural modality): Prepare a CSV file with the following headers:
  `Patient_ID_Modality,image_path`


3. Make config files:

    Populate a config file with required parameters. Examples:
    - MA: [test_params_ma.cfg](./Deep_BET/config/test_params_ma.cfg)
    - Multi-4: [test_params.cfg](./Deep_BET/config/test_params_multi_4.cfg)

    Where `mode` refers to the inference type, which is a required parameter

    **Note**: Alternatively, you can use the diretory structure similar to the training as desribed in the next section.

4. Run the application:

    ```bash
    deep_bet_run -params $test_params_ma.cfg -test True -mode $mode -dev $device
    ```

    Where:
    - ```$mode``` can be ```MA``` for modality agnostic or ```Mult-4```.
    - ```$device``` refers to the GPU device where you want your code to run or the CPU.


## [ADVANCED] Train your own model

1. Co-registration within patient in a common atlas space such as the [SRI-24 atlas](https://www.nitrc.org/projects/sri24/) in the LPS/RAI space. 

    An easy way to do this is using the [```BraTSPipeline``` application](https://cbica.github.io/CaPTk/preprocessing_brats.html) from the [Cancer Imaging Phenomics Toolkit (CaPTk)](https://github.com/CBICA/CaPTk/). This pipeline currently uses a pre-trained model to extract the skull but the processed images (in the order defined above till registration) are also saved.

    **Note**: Any changes done in this step needs to be reflected during the inference process.

2. Arranging the Input Data, co-registered in the previous step, to the following folder structure. Please note files must be named exactly as below (e.g. ${subjectName}_t1, ${subjectName}_maskFinal.nii.gz etc.) 

```
Input_Data_folder -- patient_1 -- patient_1_t1.nii.gz
                         -- patient_1_t2.nii.gz
                         -- patient_1_t1ce.nii.gz
                         -- patient_1_flair.nii.gz
                         -- patient_1_maskFinal.nii.gz
               patient_2 -- ...
               ...
               ...
               patient_n -- ...
```

3. Standardizing Dataset Intensities

    Use the following command to standardize intensities for both training and validation data:

    ```bash
    python Deep_BET/utils/intensity_standardize.py -i ${inputSubjectDirectory} -o ${outputSubjectDirectory} -t ${threads}
    ```

    - ```${inputSubjectDirectory}``` needs to be structured as described in the previous step [Arranging Data](###Expected-Directory-structure-for-data)
    - `${threads}` are the maximum number of threads that can be used for computation and is generally dependent on the number of available CPU cores. Should be of type `int` and should satisfy: `0 < ${threads} < maximum_cpu_cores`. Depending on the type of CPU you have, it can vary from [1](https://ark.intel.com/content/www/us/en/ark/products/37133/intel-core-2-solo-processor-ulv-su3500-3m-cache-1-40-ghz-800-mhz-fsb.html) to [112](https://www.intel.com/content/www/us/en/products/processors/xeon/scalable/platinum-processors/platinum-9282.html) threads.

4. Prepare configuration file

    Populate a config file with required parameters. Example: [train_params.cfg](./Deep_BET/config/train_params.cfg)

    Change the ```mode``` variable in the config file based on what kind of model you want to train (either modality agnostic or multi-4).

5. Run the training:

    ```bash
    deep_bet_run -params train_params.cfg -train True -dev $device -load $resume.ckpt
    ```

    Note that ```-load $resume.ckpt``` is only needed if you are resuming your training. 

6. [OPTIONAL] Converting weights after training

  - After training a custom model, you shall have a `.ckpt` file instead of a `.pt` file.
  - The file [convert_ckpt_to_pt.py](./Deep_BET/utils/convert_ckpt_to_pt.py) can be used  to convert the file. 
    - Example:
      ```bash
      ./env/python Deep_BET/utils/convert_ckpt_to_pt.py -i ${path_to_ckpt_file_with_filename} -o {path_to_pt_file_with_filename}
      ```
  - Please note that the if you wish to use your own weights, you can use the ```-load``` option.

## Citation

If you use this package, please cite the following paper:

- Thakur, S.P., Doshi, J., Pati, S., Ha, S.M., Sako, C., Talbar, S., Kulkarni, U., Davatzikos, C., Erus, G. and Bakas, S., 2019, October. Skull-Stripping of Glioblastoma MRI Scans Using 3D Deep Learning. In International MICCAI Brainlesion Workshop (pp. 57-68). Springer, Cham. DOI:10.1007/978-3-030-46640-4_6
- Thakur, S.P., Doshi, J., Pati, S., Ha, S.M., Sako, C., Talbar, S., Kulkarni, U., Davatzikos, C., Erus, G. and Bakas, S., Brain Extraction on MRI Scans in Presence of Diffuse Glioma: Multi-institutional Performance Evaluation of Deep Learning Methods and Robust Modality-Agnostic Training, NeuroImage 2020 [Accepted]

## Notes

- **IMPORTANT**: This application is neither FDA approved nor CE marked, so the use of this package and any associated risks are the users' responsibility.
- Using this software is pretty trivial as long as instructions are followed. 
- You can use it in any terminal on a supported system. 
- The ```deep_bet_run``` command gets installed automatically. 
- We provide CPU (untested as of 2020/05/31) as well as GPU support. 
  - Running on GPU is a lot faster though and should always be preferred. 
  - You need an GPU memory of ~5-6GB for testing and ~8GB for training.

## TO-DO

- Give example of skull stripping dataset 
- In inference, rename ```model_dir``` to ```results_dir``` for clarity in the configuration and script(s)
- Add CCA for post-processing
- Add link to CaPTk as suggested mechanism for preprocessing (can refer to ```BraTSPipeline``` application after my [PR](https://github.com/CBICA/CaPTk/pull/1061) gets merged to master)
- Test on CPU
- Move all dependencies to ```setup.py``` for consistency 
- Put option to write logs to specific files in output directory
- Remove ```-mode``` parameter in ```deep_bet_run```
- Windows support (this currently works but needs a few work-arounds)
- Please post any requests as issues on this repository or send email to software@cbica.upenn.edu

## Contact

Please email software@cbica.upenn.edu with questions.
