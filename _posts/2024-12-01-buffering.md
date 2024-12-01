---
#image: /assets/img/subsidence background.jpg # or base64 URI
title: "Automated Buffering and Intersection Analysis in GIS"
tag: [GIS, Python]
date: 2024-08-02 00:00:00 +0800
categories: [GIS]
description: ArcPy application - Course Work - Lakehead University (2023)
layout: post
---

## Introduction
Introduction
![urban](https://erlangds.github.io/assets/img/ass2/assignment2.jpg){: lqip="/assets/img/ass2/assignment2.jpg" }{: w="300" h="800" }{: .right }
This StoryMap showcases the Python/ArcPy script developed for NRMT 3350/5132 as part of Assignment 2. The assignment focused on automating the processing of spatial data within a geodatabase using Python and ArcPy. The goal was to read buffer values from a text file, apply these values to create buffer zones around a feature class, and perform intersection analysis with other feature classes.

## Objective
The script is designed to:

1. Read Buffer Values: Extract buffer distances from a text file and handle errors if the file is missing.
2. Clean and Format Data: Process buffer values to ensure they are valid, unique integers.
3. Apply Buffers: Use the cleaned buffer values to create multiple buffered feature classes.
4. Intersect with Feature Classes: Perform intersection analysis on specific feature classes and save the result

## Location
The study area is located in the Provincial Forest of Ontario, approximately 40 kilometers north of Thunder Bay, at coordinates 48°38'53.2"N, 89°22'29.7"W.

<div style="text-align: center;">
  <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d292666.0709663539!2d-89.60387526919818!3d48.56013020349577!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x4d592a3f9ba9785b%3A0x9c9a677a01a8f821!2sJacques%2C%20Ontario%20P7G%200Y3%2C%20Kanada!5e1!3m2!1sid!2sid!4v1733054708071!5m2!1sid!2sid" width="600" height="450" style="border:0;" allowfullscreen="" loading="lazy" referrerpolicy="no-referrer-when-downgrade"></iframe>
</div>

## Workflow
### 1. Reading Buffer Values
The script begins by specifying the path to the buffer_values.txt file and attempts to open it. If the file is missing, an error is raised and the script exits gracefully. The file contains a mix of float, integer, and string data. The goal is to extract and process only the numerical values (both integers and floats) to use as buffer distances

The script performs the following steps:

1. Extract Numerical Values: The script reads the file and filters out non-numeric entries
2. Round Floats: Float values are rounded to the nearest integer
3. Remove Duplicates: Duplicate numbers are removed to ensure each buffer distance is unique

```python

#Erlang Dzarkhan Syah
#1244440
#November 22, 2023
# Assigment 2 is mostly about intro to arcpy in general, how we can import and store data as an output, modify file, and run geoprocessing as a vector processing in python

#import module
import arcpy
import sys

#path to the buffer file
buffer_file = r"C:\Users\Erlang\Downloads\buffer_values.txt"

#try and except act like a boolean, if argument in "try is true, it will return true and do the process"
try :
    def buffer_selection (buffer_file): #define the argument that we will be using in buffer using def
        file_open = open (buffer_file, "r") #open buffer file
        file_split = file_open.read().split(",") #to split values by comma
        new_buffer = set() #empty set, used to store new value of the buffer that has been sorted
      
        for value in file_split: #for each value that has been split up by comma
            #check if the value is number
            number = value.strip() #make sure to remove any leading or trailing whitespace characters if any if we probably use another data
            try:
                round_value = round(float(number)) #round the number 
                new_buffer.add(int(round_value)) #convert to int and add to set
                
            #check if the value is non number
            except ValueError:
                print(f"Skipping non-numeric value: {value}")

        return sorted(set(new_buffer)) #sorting the value from the lowest to the highest and remove duplicate values

#check if the file exist in that path, if not it will terminated automatically 
except FileNotFoundError:
    print ("buffer_values.txt file does not exist in the specified location.")
    sys.exit("Script terminated due to missing buffer_values.txt file") #this sys function will close and terminate the file
```

### 2. Buffer Processing
With the cleaned buffer values, the script proceeds to create buffer zones around the roads feature class. Each unique buffer distance results in a separate feature class:

1. Buffer Creation: For each buffer distance, a new buffer zone is generated around the roads feature class
2. Feature Class Naming: Each newly created buffer zone is saved as a feature class with a name that includes the buffer distance. For example, a buffer distance of 100 meters will produce a feature class named roads_100buff
3. Output: These feature classes are stored in the geodatabase and are used for subsequent spatial analysis

```python
#GEOPROCESSING
geodatabase = r"C:\Users\Erlang\OneDrive\Documents\ArcGIS\Projects\Assignment 2\jhf.gdb" #path for geodatabase file
arcpy.env.workspace = geodatabase #set geodatabase as the environment
distance_values = buffer_selection(buffer_file) #values that will be used for geoprocessing
print (f"distance values that will be used are {distance_values}")

#BUFFER
for distance in distance_values: #for looping all the distance values
    buffer_distance_str = str(distance) #the distance should be in string
    buffer_output_name = f"roads_{buffer_distance_str}buff" # Create buffer output name
    arcpy.analysis.Buffer("roads", buffer_output_name, f"{distance} Meters")  # Buffer the roads feature class with the current buffer distance
```
![buffer](https://erlangds.github.io/assets/img/ass2/buffer-08.png){: lqip="/assets/img/ss2/buffer-08.png" }{: .center }_Extracted buffer distance analysis, from left to right : (100 m buffer, 200 m buffer, 300 m buffer, 400 m buffer)_

### 3. Intersecting Feature Classes
The script systematically intersects feature classes within the geodatabase, focusing on those with "fri" in their names. This is done with the buffered feature classes created earlier. Here’s how the process works:

1. Feature Class Selection: The script identifies and processes all feature classes in the geodatabase whose names contain "fri"
2. Intersection Operation: Each of these "fri" feature classes is intersected with the previously created buffered feature classes. This operation spatially combines the features to analyze their overlapping areas
3. Result Naming: The results of these intersections are saved using a consistent naming convention, such as fri_100b_int, where the number represents the buffer distance

```python
#INTERSECT
feature_classes = arcpy.ListFeatureClasses() # to retrieve the feature class
for fc in feature_classes: #looping for feature class
    if 'fri' in fc: #looking for fri in feature class
        for distance in distance_values: #looping for all number used for buffer
            buffer_distance_str = str(distance) #defining distance in string
            intersect_output = f"fri_{buffer_distance_str}b_int" #make an output name for intersect file
            arcpy.analysis.Intersect([fc, buffer_output_name], intersect_output) #intersect between fri and buffer roads
```
<div class="juxtapose" >
    <img src="https://erlangds.github.io/assets/img/ass2/fri_buffer.jpg"  />
    <img src="https://erlangds.github.io/assets/img/ass2/Intersect_400.jpg"  />
</div>
<script src="https://cdn.knightlab.com/libs/juxtapose/latest/js/juxtapose.min.js"></script>
<link rel="stylesheet" href="https://cdn.knightlab.com/libs/juxtapose/latest/css/juxtapose.css">

## Conclusion
The provided script effectively demonstrates key functionalities of ArcPy for spatial data processing within a geodatabase environment. It covers importing and storing data, manipulating files, and executing geoprocessing operations, with a primary focus on vector processing using Python. Overall, the assignment serves as an introduction to working with geospatial data in ArcPy and showcases its potential applications in areas such as environmental management, urban planning, and geospatial data management