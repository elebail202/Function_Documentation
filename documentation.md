<<<<<<< HEAD
# WHAT ARE THE TEXTURES FUNCTIONS DOING ?

## First function : convert_segmentations_to_dicom_final.m

### Dependencies ?

`No dependencies` with a specific function but it `uses data` such as :

- user_preferences.mat
- segment_info
- fit_info
- T2 maps

The function uses the previous segmented image in mokkula.

### What is it doing ?

By running this function, you create a `IMA file` openable with 'dicomread' on Matlab that specifies in his name the serie and the segmentation you are working with. This file is created in a copy of the folder name where your data is with 'Mokkulamaps' added.

> `Example` : If your data is saved in a folder named *'Anonymized - 04071989'*, then the file created will be created in a folder named *'.Anonymized - 04071989_Mokkulamaps'*. By following the same path, you will find a file in this folder named *'series14_slice8_Segmentation.IMA'* depending on the series and the slice you made the segmentation.

The file created contains a 384 * 384 * 3 matrix with values between 0 and 255 which corresponds to the intensity of each pixel. This .IMA format is like dicom images. Once you open it you can see this :

| Segmentation | T2 map |
| ----------- | ------ |
| ![alt text](series14_slice8_Segmentation.png "Segmentation") | ![alt text](series14_slice8_T2_map.png "T2 map") |

It looks like the colors are inverted and the 'Segmentation' image shows the intensity of each pixel.

### What wasn't working well ?

Line 51, it was written : 

```matlab
    isfolder(patients(patient).name && patients(patient).name(1) ~= '.');   
```

But isfolder on Matlab takes a text in argument so it wasn't working. I replaced this by :  

```matlab
    if (patients(patient).name & patients(patient).name(1) ~= '.');  
```

## Second function : flat_converter.m

### Dependencies ?

`'Dependencies` with a specific function :

- structure_loop.m


And it `uses data` such as :

- segment_info
- results_flat
- fit_info

### What is it doing ?



### What wasn't working well ?

One file looks like missing when we run this function : 'results_flat.m'.  
So I tried to figure out when should this file be created. It appears that this file is created in the function 'main_flat.m'.

In main_flat, many errors :

1. check.fit didn't exist, I added a line :

    ```matlab
    check.fit = exist('fit_info.mat', 'file');
    ```

2. Problems with the 'for loop' line 74 so I added manually some information in order to be able to continue debugging the code :

    ```matlab
    load T2maps
    res_map  = T2map;
    map_temp = 2;
    ```

3. Line 99, by calling the function 'execute_flat.m' which is defined in the same file, it makes an error related to the 'structure_loop.m' function at line 184. In this function, there are problems related to the use of structure in Matlab. At line 24, it was initially written this :

    ```matlab
    slices=zeros(length(seg_general(series).lines),1);
    for ii=1:length(seg_general(series).lines)
         if isreal(seg_general(series).lines(ii).lines) == 0
             slices(ii)=1;
         else
             slices(ii)=0;
        end
    end 
    slice=find(slices);
    ```
    But it is impossible to take the length of this type of object. 'Series' is a list and 'seg_general(series).lines' is a array of struct array. Moreover, we cannot write 'seg_general(series).lines(ii).lines' because there are 2 results.  
    So, I decided to temporarily write this instead :

    ```matlab
    slices=zeros(18,1);
    size_series = size(series);
    for ii = 1 : size_series
        for jj = 1 : 18
            if isreal (seg_general(series(ii)).lines(jj).lines) == 0
                slices(jj) = 1;
            else
                slices(jj) = 0;
            end
        end
    end 
    slice=find(slices);
    ``` 

    And it seems that it resolves the problem for now.

4. So now, back to the 'main_flat.m' function, another error comes up that refers to the 'coordinates.m' function : 

    At line 5, it was written this, but it is a similar problem than before, we cannot write this because 'index.serie' is an array :

    ```matlab
    x=seg_general(index.serie).lines(index.slice).lines(t).X;
    y=seg_general(index.serie).lines(index.slice).lines(t).Y;
    ``` 

    So I wrote this instead :

    ```matlab
    size_serie = length(index.serie);
    for i = 1 : size_serie
        x=seg_general(index.serie(size_serie)).lines(index.slice).lines(1).X;
        y=seg_general(index.serie(size_serie)).lines(index.slice).lines(1).Y;
    end
    ``` 

    Looks like 'coordinate.m' is now working well.
 
5. Back in 'main_flat.m', another error to fix refering to 'runflat.m'.


=======
# WHAT ARE THE TEXTURES FUNCTIONS DOING ?

## First function : convert_segmentations_to_dicom_final.m

### Dependencies ?

`No dependencies` with a specific function but it `uses data` such as :

- user_preferences.mat
- segment_info
- fit_info
- T2 maps

The function uses the previous segmented image in mokkula.

### What is it doing ?

By running this function, you create a `IMA file` openable with 'dicomread' on Matlab that specifies in his name the serie and the segmentation you are working with. This file is created in a copy of the folder name where your data is with 'Mokkulamaps' added.

> `Example` : If your data is saved in a folder named *'Anonymized - 04071989'*, then the file created will be created in a folder named *'.Anonymized - 04071989_Mokkulamaps'*. By following the same path, you will find a file in this folder named *'series14_slice8_Segmentation.IMA'* depending on the series and the slice you made the segmentation.

The file created contains a 384 * 384 * 3 matrix with values between 0 and 255 which corresponds to the intensity of each pixel. This .IMA format is like dicom images. Once you open it you can see this :

| Segmentation | T2 map |
| ----------- | ------ |
| ![alt text](series14_slice8_Segmentation.png "Segmentation") | ![alt text](series14_slice8_T2_map.png "T2 map") |

It looks like the colors are inverted and the 'Segmentation' image shows the intensity of each pixel.


## Second function : flat_converter.m

### Dependencies ?

`No dependencies` with a specific function but it `uses data` such as :

- segment_info
- results_flat
- fit_info

### What is it doing ?







>>>>>>> c2db0ec5ac932da811ee9ce5673bdbb8bb961f04
