# WHAT ARE THE TEXTURES FUNCTIONS DOING ?

## First function : convert_segmentations_to_dicom_final

### Dependencies ?

`No dependencies` with a specific function but it `uses data` such as :

- user_preferences.mat
- segment_info
- fit_info
- T2 maps

### What is it doing ?

By running this function, you create a `IMA file` openable with 'dicomread' on Matlab that specifies in his name the serie and the segmentation you are working with. This file is created in a copy of the folder name where your data is with 'Mokkulamaps' added.
This function make the segmentations 
In this folder you find a matrix 384 * 384 * 3 

**The function is not working anymore, I am trying to figure it out**