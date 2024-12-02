# EasyFreezy
## A hard-coded approach to detect Freezing Behaviors based on DeepLabcut output

#### EasyFreezy uses the .csv and .pickle output of DeepLabCut to detect freezing behaviors. An extended explanation of how the script works can be found inside the Description cell of EasyFreezy.ipynb. The role of each function is also described there -sometimes along with technical notes.
#### EasyFreezy can detect freezing responses even in low quality videos, provided that a good training with DLC has taken place. 
#### EasyFreezy is primarily based on between ears (betwears) body part for freezing detection. It is also implementing left or right ear if the user wants to insert a second bodypart for detection. Nose is included too, in order to filter out freezing behaviors when there are indications of moving of nose.

Please not that EasyFreezy has been only tested and validated for 25fps videos. For videos of higher fps rate, modifications might need to be done. You can also consider downscaling to 25fps.<br><br><br><br>

Some basic concepts are described below. You can also refer to the "Measuring freezing responses with Deeplabcut.pptx" and "How it works" folder for an understanding of these concepts.

## loc_diff :    
 
              Let x axis coordinates of 1st, 2nd … Nth frame    x1, x2 … xN 
              Let y axis coordinates of 1st, 2nd … Nth frame    y1, y2 … yN 
              Then loc_diff = |x1 - x2| + |y1 - y2| 
  For instance the loc_diff for frame 44 is : 
![image](https://github.com/user-attachments/assets/5569ae65-af64-417c-95b7-1598e5f3ecc2)

loc_diff can be used as a cut-off criterion to detect with accuracy the start of a freezing behavior. It is also used for detecting following freezing frames. <br><br><br><br>

## avged_diff : is used when loc_diff criterion for a bodypart is violated. To calculate avged_diff : <br><br>
1)First we calculate the absolute sum for np.ediff1d differences between future frames and divide it by a number (for between ears body part, this number is the user_betwears_ff-1), that is, the number of future frames that the user has selected to include -1.) for x axis.<br>
2)Then  we calculate the absolute sum for np.ediff1d differences between future frames and divide it by the same number for the y axis.<br>
3)Finally, we get the absoulte sum of the 2 sums above.<br><br>
Here is an example of calculating avged_distance for frame 0 (average difference always starts from next frame, that is, frame 1). The user_betwears_ff here is 5 and the denominator is user_betwears_ff-1, that is 4.

![avged_diff](https://github.com/user-attachments/assets/9aa180fb-86ca-4e95-84d3-38d91c9d64dc)

![Screenshot_1](https://github.com/user-attachments/assets/e9586b92-698b-4ecd-bd52-d84c41003262)



Important Note: By convention, when EasyFreezy was beeing made for the first time, the denominator was user_betwears_ff-2 instead of user_betwears_ff-1. However, if you'd rather, you can change this easily to 1. Simply replace this part of the code "denominator_ff = future_frames - 2" to - 1, to get the real averaged value. Make sure you replace -2 with -1 not only in the DetectFreezingRange() which is for detection of freezing based on betwears but also for EarFreezingRange(), which is for detection of freezing based on a second -optional- bodypart, as well as for NoseandHosed(), which is for detection of motion of nose in order to exclude frames that would be otherwise considered as freezing frames.

A single violation of the avged_diff is also permitted ONLY for detection of freezing based on betwears. Assuming that the avged_diff in our example above for frame 0 is higher that the cut-off criterion, the script will also calculate the avged_diff when taking frame 1 as reference (meaning starting to calculate the averaged_differences from frame 2). If there is no violation there, then the frame is considered as freezing. This further examination has been especially helpful for lower quality videos.


## Insertion of second bodypart for freezing detection

Especially for lower quality videos, the insertion of a second body part for freezing detection can be helpful. Same logic applies to detecting a second body part. However, because by inserting a second body part the risk of False Positive increases, one more cut-off criterion is inserted. This is "user_range_prob". "user_range_prob" only includes a second body part's detected freezing in list with freezing behaviors if the p value in DLC output is high enough.

## Using nose to filter out behaviors previously considered as freezing

As in betwears and second body part, the detection of freezing based on nose is done the same way. However, in this case we are actually interested in the frames where this freezing is violated based on nose detection.<br>
To prevent filtering out real freezing behaviors (based on betwears or second bodypart), a parameter similar to "user_range_prob" is used. This is user_betwnose_prob_cut. <br><br>
For particular experimental setups, the user can also apply the "user_betwnose_nextflag_diff" for exclusion of betwears freezing or "user_betwnose_secnextflag_diff" for exclusion of second body part freezing. This variable is by default eliminated. However, if we set it lower than the user_betwnose_avged_diff (for nose validation based on betwears freezing) or user_secnose_avged_diff (for nose validation based on second body part freezing), then we can in fact make criteria of freezing detection more loose. This parameter makes the script more prone to detect a moving frame based on nose. How ? <br><br>
Assuming that there is a series of freezing frames (ff6,ff7,ff8,ff9...ff30) that have been detected based on betwears freezing,<br> if the "user_betwnose_avged_diff" (that is the avged_diff for nose where nose is validating betwears freezing behaviors) equals 0.1 and the nose avged_diff for frame 14 is 0.15, then a motion flag will be raised. This means that there is suspicion that a frame that had been previously considered freezing actually involves motion of nose).<br> When the script looks at the nose avged_diff calculated based on the next frame (ff15), it will compare this value not with the user_betwnose_avged_diff but with the "user_betwnose_nextflag_diff" cut-off criterion instead. If the nose avged_diff exceeds this criterion, then this frame will be considered a frame where nose is moving too (see nose movement in How it Works folder).<br>
"user_betwnose_nextflag_diff" ONLY applies for the detection of nose motion when the previous frame has been tagged as a motion flag too.

## Combining motion flags to create chains

The flags that are raised will eventually create chains of motion frames based on nose detection. All frames that belong to these chains will be removed from betwears or second body part list with freezing frames, thereby excluded from freezing. <br>
However, a chain can only be created if motion flags are in close proximity. The distance between two motion flag frames must not be higher than the "user_motion_proximity" parameter. As a default, the accepted distance to create a chain of moving based on nose detection is 5 frames apart.


