# EasyFreezy
## A fixed-threshold method to detect Freezing Behaviors based on DeepLabcut output

#### EasyFreezy uses the .csv and .pickle output of DeepLabCut to detect freezing behaviors. An extended explanation of how the script works can be found inside the Description cell of EasyFreezy.ipynb. The role of each function is also described there -sometimes along with technical notes.
#### EasyFreezy is designed to detect freezing behaviors that last at least a particular number of frames. It has been tested for freezing series of at least 25 or 50 frames in 25fps videos (that is, minimum 1 sec or 2 sec for a detection to be considered as freezing, respectively) 
#### EasyFreezy can detect freezing responses even in low quality videos, provided that a good training with DLC has taken place. 
#### EasyFreezy is primarily based on between ears (betwears) body part for freezing detection. It is also implementing left or right ear if the user wants to insert a second bodypart for detection. Nose is included too, in order to filter out freezing behaviors when there are indications of moving of nose.
#### EasyFreezy does not need overfit data to work with. Provided that the DLC training is sufficient, the coordinates of any analyzed video by DLC can be analyzed under the same parameters by EasyFreezy.

Please not that EasyFreezy has been only tested and validated for 25fps videos. For videos of higher fps rate, modifications might need to be done. You can also consider downscaling to 25fps.<br><br><br><br>

## EasyFreezy has been already used for scientific experiements. Kindly cite this github page in case of using data derived from EasyFreezy in your publications. <br><br>

<br><br><br>

Some basic concepts are described below. You can also refer to the "Measuring freezing responses with Deeplabcut.pptx" and "How it works" folder for an understanding of these concepts.

## loc_diff :    
 
              Let x axis coordinates of 1st, 2nd … Nth frame    x1, x2 … xN 
              Let y axis coordinates of 1st, 2nd … Nth frame    y1, y2 … yN 
              Then loc_diff = |x1 - x2| + |y1 - y2| 
  For instance the loc_diff for frame 44 is : 
![image](https://github.com/user-attachments/assets/5569ae65-af64-417c-95b7-1598e5f3ecc2)

loc_diff can be used as a cut-off criterion to detect with accuracy the start of a freezing behavior. It is also used for detecting following freezing frames. <br><br><br><br>

## avged_diff : is used when loc_diff criterion for a bodypart is violated. To calculate avged_diff : <br><br>
1) First, we calculate the sum absolute for np.ediff1d differences* between future frames and divide it by a number (for between ears body part, this number is the user_betwears_ff-1, which corresponds to the number of future frames that the user has selected to include minus 1 frame) for x axis.<br>
2) Then,  we repeat by calculating the sum absolute for np.ediff1d differences between future frames and divide it by the same number for the y axis.<br>
3) Finally, we get the sum of the 2 sums above.<br><br>
Here is an example of calculating avged_distance for frame 0 (average difference always starts from next frame, that is, frame 1). The user_betwears_ff here is 5 and the denominator is user_betwears_ff-1, that is 4.

![avged_diff](https://github.com/user-attachments/assets/9aa180fb-86ca-4e95-84d3-38d91c9d64dc)

![Screenshot_1](https://github.com/user-attachments/assets/e9586b92-698b-4ecd-bd52-d84c41003262)



Technical Note: By convention, when EasyFreezy was beeing made for the first time, the denominator was user_betwears_ff-2 instead of user_betwears_ff-1. However, if you'd rather, you can change this easily to 1. Simply replace this part of the code "denominator_ff = future_frames - 2" to - 1, to get the real averaged value. Make sure you replace -2 with -1 not only in the DetectFreezingRange() which is for detection of freezing based on betwears but also for EarFreezingRange(), which is for detection of freezing based on a second -optional- bodypart, as well as for NoseandHosed(), which is for detection of motion of nose in order to exclude frames that would be otherwise considered as freezing frames.

A single violation of the avged_diff is also permitted ONLY for detection of freezing based on betwears. Assuming that the avged_diff in our example above for frame 0 is higher that the cut-off criterion, the script will also calculate the avged_diff when taking frame 1 as reference (meaning starting to calculate the averaged_differences from frame 2). If there is no violation there, then the frame is considered as freezing. This further examination has been especially helpful for lower quality videos.


/* np.ediff1d -> The differences between consecutive elements of an array.

## Insertion of second bodypart for freezing detection

Especially for lower quality videos, the insertion of a second body part for freezing detection can be helpful. Same computational logic applies to detecting a second body part. However, because by inserting a second body part the risk of False Positive increases, one more cut-off criterion is inserted. This is "user_range_prob". "user_range_prob" only includes a second body part's detected freezing in list with freezing behaviors if the p value in DLC output is high enough.

## Using nose to filter out behaviors previously considered as freezing

As in betwears and second body part, the detection of freezing based on nose is done the same way. However, in this case we are actually interested in the frames where this freezing is violated based on nose detection.<br>
To prevent filtering out real freezing behaviors (based on betwears or second bodypart), a parameter similar to "user_range_prob" is used. This is user_betwnose_prob_cut. Therefore, detection of nose movement across previously cosnsidered freezing behaviors (based on betwears or second body part) will only lead to de-registration of the freezing behavior if the average nose detection likelihood is above the user_betwnose_prob_cut threshold.  <br><br>
For particular experimental setups, the user can also apply the "user_betwnose_nextflag_diff" for exclusion of betwears freezing or "user_betwnose_secnextflag_diff" for exclusion of second body part freezing. This variable is by default eliminated. However, if we set it lower than the user_betwnose_avged_diff (for nose validation based on betwears freezing) or user_secnose_avged_diff (for nose validation based on second body part freezing), then we can in fact make criteria of freezing detection more strict. This parameter makes the script more prone to detect a moving frame based on nose. How ? <br><br>
Assuming that there is a series of freezing frames (ff6,ff7,ff8,ff9...ff30) that have been detected based on betwears freezing,<br> if the "user_betwnose_avged_diff" (that is the avged_diff for nose where nose is validating betwears freezing behaviors) equals 0.1 and the nose avged_diff for frame 14 is 0.15, then a motion flag will be raised. This means that there is suspicion that a frame that had been previously considered freezing actually involves motion of nose).<br> When the script looks at the nose avged_diff calculated based on the next frame (ff15), it will compare this value not with the user_betwnose_avged_diff but with the "user_betwnose_nextflag_diff" cut-off criterion instead. If the nose avged_diff exceeds this criterion, then this frame will be considered a frame where nose is moving too (see nose movement in How it Works folder).<br>
"user_betwnose_nextflag_diff" ONLY applies for the detection of nose motion when the previous frame has been tagged as a motion flag too.

## Combining motion flags to create chains

The flags that are raised will eventually create chains of motion frames based on nose detection. All frames that belong to these chains will be removed from betwears or second body part list with freezing frames, thereby excluded from freezing. <br>
However, a chain can only be created if motion flags are in close proximity. The distance between two motion flag frames must not be higher than the "user_motion_proximity" parameter. As a default, the accepted distance to create a chain of moving based on nose detection is 5 frames apart.

## Interpolation uncertainty range

The user defines a particular and relatively narrow range of p values based on DLC output that are low, yet not too low. These values will be linearly interpolatedfor all bodyparts, both for x and y axis.

##  User Input and parameters 
If you search for "User Input" inside EasyFreezy.ipynb, you will see a large cell that is commented out (by #). You can uncomment this code by selecting all text inside the cell and typing ctrl+?/ in visual studio code. Then you will be prompted to provide some input which corresponds to the next 8  general parameters <br><br>:
user_fps<br>
user_fps_fix<br>
user_desired_duration<br>
user_desired_extra_secs<br>
user_warning<br>
user_bins<br>
user_double_reference<br>
user_frms_cutoff<br>

You can then replace the values of the parameters to the cell below, to avoid getting the script asking you every time about these parameters. Kindly note that if you run both the verbose cell and the next one, the values that you have inserted in the verbose (now uncommented cell) will change, unless you replace the values of the parameters to the cell below accordingly.

The interpolation uncertainty range is defined by its two extremities, that is user_uncertain_left and user_uncertain_right.<br><br>

user_betwears(or sec)_ff stands for future frames to be taken into account for calculation of avged_diff. Try the default value first, which is 6. <br><br>
user_betwnose_ff stands for future frames to be taken into account for calculation of avged_diff of nose when validating betwears freezing behaviors.<br><br>
user_secnose_ff stands for future frames to be taken into account for calculation of avged_diff of nose when validating second bodypart freezing behaviors.<br><br>
Ignore the user_betwears(or sec)_pf=5 as well as the comments related to it inside the code. This is deprecated and will be removed in another version. It's not affecting the code.<br><br>
user_betwnose_prob_cut_frames defines how many future frames will be taken into account when calculating user_betwnose_prob_cut, which has been earlier described.<br><br>


# Get labels with x,y position, p likelihood and parameter values 

For first time use, run EasyFreezy with some example parameter values for user input. At this stage, you cannot estimate well the values for loc_diff and avged_diff , or set a proper interpolation range. Thereby, you just want to get the calculation of x,y axis for betwears, second body part and nose on your screen, along with the calculation of loc_diff and avged_diff for each single frame (see images in How it Works). <br><br>

Make sure you run the cells below the main code, so that you will produce a pickle file for each video that you have analyzed for freezing. This pickle file(s) (not to be mistaken with DLC output) is created in the same directory where EasyFreezy.ipynb is. Save the pickle file(s) to another folder and copy its path). Open the labelling script and replace the path in pickle_directory (see in picture below) with the path that leads to your pickle files. Replace the path that leads to your raw recording video in the video_dir. Replace the output directory where your new labelled video will be saved in the saved_dir.<br><br>

Alternatively, simply uncomment these three lines of code in the main code (by removing the #) : <br>
#SelectDLCOutputDirectory()<br>
#SelectVideoDirectory()<br>
#SelectSaveDirectory()  <br>
And an filedialog will open, prompting you to choose these directories in your explorer. <br><br>


Replace the vw and vh with the widht and height of your recording video respectively <br>
Set ul = 'l' for labels at the bottom of your screen or 'u' for labels at the top of your screen. <br>
lhl_h_prop defines the percentage of height across which the labels will be spread. 10 means the 10% of overall height.<br>
height_start moves the labels far from the top (or bottom) edge of the screen. Recommended to set around 10.<br>
labelsize is merely the size of your labels<br>
short_labelsize is the size of some temporary labels that will shortly appear in the start, indicating the total number of freezing behaviors observed for each bodypart





![Screenshot_2](https://github.com/user-attachments/assets/b016d6af-06ee-40ab-b649-3b405180a109)



# Download the test_it_yourself rar file in the Releases to test how to code works. 




