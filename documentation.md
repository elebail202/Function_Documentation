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

`'Dependencies` with specific functions :

- structure_loop.m
- main_flat.m
- coordinates.m
- runflat.m


And it `uses data` such as :

- segment_info
- results_flat
- fit_info

### What is it doing ?



### What was wrong with it ?

One file looks like missing when we run this function : 'results_flat.m'.  
So I tried to figure out when should this file be created. It appears that this file is created in the function 'main_flat.m'.

In main_flat, many errors :

1. check.fit didn't exist, I added a line at line 59 :

    ```matlab
        check.fit = exist('fit_info.mat', 'file');
    ```

2. Problems with the 'for loop' line 62 so I added manually some information in order to be able to continue debugging the code at line 82 :

    ```matlab
        load T2maps
        res_map  = T2map;
        map_temp = 2;
    ```

3. Line 88, by calling the function 'execute_flat.m' which is defined in the same file, it makes an error related to the 'structure_loop.m' function at line 172. In this function, there are problems related to the use of structure in Matlab. At line 24, it was initially written this :

    ```matlab
        slices = zeros(length(seg_general(series).lines), 1);
        for ii = 1 : length(seg_general(series).lines)
            if isreal(seg_general(series).lines(ii).lines) == 0
                slices(ii) = 1;
            else
                slices(ii) = 0;
            end
        end 
        slice = find(slices);
    ```
    But it is impossible to take the length of this type of object. 'Series' is a list and 'seg_general(series).lines' is a array of struct array. Moreover, we cannot write 'seg_general(series).lines(ii).lines' because there are 2 results.  
    So, I decided to temporarily write this instead :

    ```matlab
        slices = zeros(18, 1);
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
        slice = find(slices);
    ``` 

    And it seems that it resolves the problem for now.

4. So now, back to the 'main_flat.m' function, another error comes up that refers to the 'coordinates.m' function : 

    At line 6, it was written this, but it is a similar problem than before, we cannot write this because 'index.serie' is an array :

    ```matlab
        x = seg_general(index.serie).lines(index.slice).lines(t).X;
        y = seg_general(index.serie).lines(index.slice).lines(t).Y;
    ``` 

    So I wrote this instead :

    ```matlab
        size_serie = length(index.serie);
        for i = 1 : size_serie
            x = seg_general(index.serie(size_serie)).lines(index.slice).lines(1).X;
            y = seg_general(index.serie(size_serie)).lines(index.slice).lines(1).Y;
        end
    ``` 

    Looks like 'coordinate.m' is now working well.
 
5. Back in 'main_flat.m', another error to fix refering to 'runflat.m'.

    One error is related to the same problem than before (access to a struct of struct).  
    At line 86, I wrote this :

    ```matlab
        serie_size = length(index.serie);
        for i = 1 : serie_size
            Timage = Tmap(index.serie(serie_size)).T2(:, :, index.slice);
        end
    ```

    Instead of this :

    ```matlab
        Timage = Tmap(index.serie).T2(:,:,index.slice);
    ```

    It now seems that 'run_flat.m' is working.

6. One error occurs at lines 189 and 199, it was written :

    ```matlab
        Tmap_flat.femur{index.serie}(index.slice) = runflat(Tmap, seg_general, template, index, check);
    ```

    But, at left we have a problem of size, I wrote :

    ```matlab
        Tmap_flat.femur = runflat(Tmap, seg_general, template, index, check);
    ```

    It seems okay now.

7. The variable 'param' is not found in 'main_flat.m' so I am trying to figure out to what it refers. I see no variable like this in the different codes, I decided to do without though :

    At line 98, I wrote :

    ```matlab
        save(savefile, 'T2map_flat', 'dir_name')
    ```  
    instead of :

    ```matlab
        save(savefile, 'T2map_flat', 'param', 'dir_name')
    ```

The file 'results_flat.mat' is now created !!  
We can now go back to 'flat_converter.m'.

8. A major problem occurs now : 

    ```matlab
        datasheet.femur{series}(rr).T_flat
    ```
    For example, the syntax 'datasheet.femur{series}(rr)' is not right because 'datasheet.femur' is a struct with different fields.  
    Then, I decided to replace all the lines using this by :

    ```matlab
        datasheet.femur.T_flat
    ```

9. At line 141, we can see this :

    ```matlab
        temp_mask(1 : vert1, 1 : horz1) = zeros(1 : vert1, 1 : horz1);
    ```

    which is a wrong use of 'zeros', so I wrote this instead :

    ```matlab
        temp_mask(1 : vert1, 1 : horz1) = zeros(vert1, horz1);
    ```   

10. Now, I have an error related to sizes. In fact, at line 203 :

    ```matlab
        temp_map(series).T2(:, :, size(mask_avg, 2)) = temp_mask;
    ```  

    But, 'temp_map' is 384 by 384 and 'temp_mask' is 384 by 259.  
    I don't know from where this error comes yet but I'll figure it out. Maybe it is due to all my changes but this is not sure.

    For the moment, I decided to fill the smaller matrix with zeros. Line 203, it is now written :

    ```matlab
        temp_map(series).T2=[temp_map(series).T2 zeros(384, 125, 6)];
        temp_map(series).T2(:, :, size(mask_avg, 2)) = temp_mask;
    ```

11. Error line 261 which refers to the 'format_results.m' function. What is wrong is at the line 47 :

    ```matlab
        cd(backimgs(1).folder(1).name)
        dcinfo=dicominfo(backimgs(1).filename(1).name);
    ```

    We can't use dot indexing here.  
    Once, I changed the value of backimgs that was assigned in 'flat_converter.m', I could realize that the path is not right. It brings to an old path that doesn't exist on my computer. However, in the 'convert_segmentations_to_dicom_final.m', the same syntaxis seems to be used. I need to find the difference between the two codes. Then, 'backimgs(1).filename(1).name' will refers to the good path.