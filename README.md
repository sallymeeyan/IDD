# About the code
************************************************************************
This code is modified from the Maryland Analysis of Developmental EEG (MADE) 
Pipeline, and tailored to our needs.

Contributors to the modification:

- yan.yan.1@vanderbilt.edu
- sasha.key@vumc.org
- h.kang@vumc.org

Comments below are most from the original authors, with some added to
reflect the modifications.

Users need to specify their own inputs as needed before running the function.

************************************************************************
The MADE Pipeline Version 1.0
Developed at the Child Development Lab, University of Maryland, College Park

- Contributors to MADE pipeline:
- Ranjan Debnath (rdebnath@umd.edu)
- George A. Buzzell (gbuzzell@umd.edu)
- Santiago Morales Pamplona (moraless@umd.edu)
- Stephanie Leach (sleach12@umd.edu)
- Maureen Elizabeth Bowers (mbowers1@umd.edu)
- Nathan A. Fox (fox@umd.edu)

## Prerequisites
************************************************************************
 Before running the pipeline, you have to install the following:
 
- EEGLab:  https://sccn.ucsd.edu/eeglab/downloadtoolbox.php/download.php

You also need to download the following plugins/extensions from here:

- https://sccn.ucsd.edu/wiki/EEGLAB_Extensions

Specifically, download:

- MFFMatlabIO: https://github.com/arnodelorme/mffmatlabio/blob/master/README.txt
- FASTER: https://sourceforge.net/projects/faster/
- ADJUST:  https://www.nitrc.org/projects/adjust/ 

After downloading these plugins (as zip files), you need to place it in the eeglab/plugins folder.
For instance, for FASTER, you uncompress the downloaded extension file (e.g., 'FASTER.zip') and place it in the main EEGLAB "plugins" sub-directory/sub-folder.

After placing all the required plugins, add the EEGLAB folder to your path by using the following code:

```{matlab}
addpath(genpath(('...')) % Enter the path of the EEGLAB folder in this line
```


Please cite the following references for in any manuscripts produced utilizing MADE pipeline:

EEGLAB: A Delorme & S Makeig (2004) EEGLAB: an open source toolbox for
analysis of single-trial EEG dynamics. Journal of Neuroscience Methods, 134, 9?21.

firfilt (filter plugin): developed by Andreas Widmann (https://home.uni-leipzig.de/biocog/content/de/mitarbeiter/widmann/eeglab-plugins/)

FASTER: Nolan, H., Whelan, R., Reilly, R.B., 2010. FASTER: Fully Automated Statistical
Thresholding for EEG artifact Rejection. Journal of Neuroscience Methods, 192, 152?162.

ADJUST: Mognon, A., Jovicich, J., Bruzzone, L., Buiatti, M., 2011. ADJUST: An automatic EEG
artifact detector based on the joint use of spatial and temporal features. Psychophysiology, 48, 229?240.
Our group has modified ADJUST plugin to improve selection of ICA components containing artifacts

This pipeline is released under the GNU General Public License version 3.


## Inputs
************************************************************************

User input: user provide relevant information to be used for data processing. Examples are shown below.

1. Enter the path of the folder that has the raw data to be analyzed
    (Your path).

2. Enter the path of the folder where you want to save the processed data
   (Your path).

3. Enter the path of the channel location file
   (Your path).

4. Do your data need correction for anti-aliasing filter and/or task related time offset?

    - adjust_time_offset = 0; % 0 = NO (no correction), 1 = YES (correct time offset)\
    - filter_timeoffset = 0;     % anti-aliasing time offset (in milliseconds). 0 = No time offset
    - stimulus_timeoffset   = 68; % stimulus related time offset (in milliseconds). 0 = No time offset
    - response_timeoffset = 0;    % response related time offset (in milliseconds). 0 = No time offset
    - respose_markers = {'xxx', 'xxx'};       % enter the response makers that need to be adjusted for time offset

<br>

5. Do you want to down sample the data?

    - down_sample = 0; % 0 = NO (no down sampling), 1 = YES (down sampling)
    - sampling_rate = 250; % set sampling rate (in Hz), if you want to down sample

<br>

6. Do you want to delete the outer layer of the channels? 

   _This function can also be used to down sample electrodes. For example, if EEG was recorded with 128 channels but you would like to analyse only 64 channels, you can assign the list of channels to be excluded in the 'outerlayer_channel' variable._   
   
    - delete_outerlayer = 0; % 0 = NO (do not delete outer layer), 1 = YES (delete outerlayer);
    - outerlayer_channel = {'list of channels'}; % list of channels
    
    If you want to delete outer layer, make a list of channels to be deleted.
Recommended list for EGI 128 channel net: {'E17' 'E38' 'E43' 'E44' 'E48' 'E49' 'E113' 'E114' 'E119' 'E120' 'E121' 'E125' 'E126' 'E127' 'E128' 'E56' 'E63' 'E68' 'E73' 'E81' 'E88' 'E94' 'E99' 'E107'}

<br>

7. Initialize the filters
    - highpass = 0.1; % High-pass frequency
    - lowpass  = 30; % Low-pass frequency. 
    
    We recommend low-pass filter at/below line noise frequency (see manuscript for detail)

<br>

8. Are you processing task-related or resting-state EEG data?
    - task_eeg = 1; % 0 = resting, 1 = task
    - task_event_markers = {'rept', 'sngl'};  % enter all the event/condition markers
    
    <br>

9. Do you want to epoch/segment your data?
    - epoch_data = 1; % 0 = NO (do not epoch), 1 = YES (epoch data)
    - task_epoch_length = [-0.1 1]; % epoch length in second
    - rest_epoch_length = 2; % for resting EEG continuous data will be segmented into consecutive epochs of a specified length (here 2 second) by adding dummy events
    - overlap_epoch = 0;     % 0 = NO (do not create overlapping epoch), 1 = YES (50% overlapping epoch)
    - dummy_events ={'xxx'}; % enter dummy events name

<br>

10. Do you want to remove/correct baseline?

    - remove_baseline = 1; % 0 = NO (no baseline correction), 1 = YES (baseline correction)
    - baseline_window = [-100  0]; % baseline period in milliseconds (MS) [] = entire epoch

<br>

11. Do you want to remove artifact laden epoch based on voltage threshold?

    - voltthres_rejection = 1; % 0 = NO, 1 = YES
    - volt_threshold = [-100 100]; % lower and upper threshold (in ?V)

<br>

12. Do you want to perform epoch level channel interpolation for artifact laden epoch? 

    - interp_epoch = 1; % 0 = NO, 1 = YES.
    - frontal_channels = {'E1', 'E8', 'E14', 'E21', 'E25', 'E32', 'E17'}; % If you set interp_epoch = 1, enter the list of frontal channels to check (see manuscript for detail)
    
    Recommended list for EGI 128 channel net: {'E1', 'E8', 'E14', 'E21', 'E25', 'E32', 'E17'}

<br>

13. Do you want to interpolate the bad channels that were removed from data?

    - interp_channels = 1; % 0 = NO (Do not interpolate), 1 = YES (interpolate missing channels)

<br>

14. Do you want to rereference your data?

    - rerefer_data = 1; % 0 = NO, 1 = YES
    - reref=[]; % Enter electrode name/s or number/s to be used for rereferencing
    - For channel name/s enter, reref = {'channel_name', 'channel_name'};
    - For channel number/s enter, reref = [channel_number, channel_number];
    - For average rereference enter, reref = []; default is average rereference

<br>

15. Do you want to save interim results?

    - save_interim_result = 1; % 0 = NO (Do not save) 1 = YES (save interim results)

<br>

16. How do you want to save your data? .set or .mat

    - output_format = 1; % 1 = .set (EEGLAB data structure), 2 = .mat (Matlab data structure)

<br>

17. Do you want to plot the ERPs at a cluster of channels? 

    - plot_ERPs = 1; % 1 = yes, 0 = no
    - chans_to_plot = [53, 54, 60, 61, 62, 67, 68, 73, 78, 79, 80, 86, 87]; % parietal cluster

<br>

18. Do you want to plot the ERPs by condition if task-related?

    - plot_ERPs_by_condition = 1; % 1 = yes, 0 = no

<br>

**Please specify the above inputs inside the function before running the code.**

## Example code
************************************************************************

After modifying the function with your inputs, run the following code in bash.

```{bash}
matlab -nodisplay -nosplash -batch 'addpath("your/path/to/the/function/");IDD_preprocess_pipeline_v1_0_test('\''/path/to/data/'\'');
```


