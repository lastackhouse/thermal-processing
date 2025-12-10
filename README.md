# thermal-processing
A repository for extracting temperature rasters from thermal imagery (specific to the DJI H20 series).

This repository hosts a workflow to convert raw thermal imagery from the Zenmuse H20T sensor to temperature rasters in a geotiff format. The code and workflow are based on work by Olivia Waite with the GenomeBC project, specifically from Chapter 14 of the GenomeBC Best Practices Guide. This version has been re-formatted for use specifically with stream surface temperature in freshwater systems. 

Code:
<details>
<summary>Click to show the code</summary>
```{r, eval=FALSE, echo=TRUE}
library(exiftoolr)
library(dplyr)
library(lubridate)
library(OpenImageR)
library(raster)
library(stringr)

# Setting the site and flight date to convert thermal imagery
flight_dates <-  c("2023_05_10") # Here only one date is shown however this can be a list of however many dates you would like to iterate over
site <- "site name here" 
merged <- "Merged_T" #name of the 'merged' folder that will contain all the h20T images from the multiple folders ouput by the H20T

for(i in 1:length(flight_dates)){
  data_date <- flight_dates[i]
  print(data_date)
  dir <- paste0("I:\\PARSER_Ext\\",site,"\\Flights\\", data_date, "\\1_Data\\H20T")
  #create folders
  if (!dir.exists(paste0(dir, "\\",merged))) {
    dir.create(paste0(dir, "\\",merged),recursive = TRUE)
  }
  if (!dir.exists(paste0(dir, "\\",merged,"\\Temperature_rasters"))) {
    dir.create(paste0(dir, "\\",merged,"\\Temperature_rasters"),recursive = TRUE)
  }
  if (!dir.exists(paste0(dir, "\\",merged,"\\Temperature_rasters_EXIF"))) {
    dir.create(paste0(dir, "\\",merged,"\\Temperature_rasters_EXIF"),recursive = TRUE)
  }
  
  list_dir <- list.files(dir, full.names = TRUE,pattern = c("DJI.+-H20T")) #selecting folders in the H20T folder that have DJI and end in -H20T, this is how we named our folders, change this pattern to match your folders. You should be selecting all folders with H20T imagery ending in _T for that flight
  print(list_dir) #to check only correct directories are being read
  
  for (d in 1:length(list_dir)){
    folder_dir <- list_dir[d] #calling each directly separately
    T_files <- list.files(folder_dir, pattern = "_T\\.JPG$") #selecting all JPEGs ending in _T 
    file.copy(from = paste0(folder_dir,"\\", T_files),
              to = paste0(dir,"\\",merged,"\\", T_files), overwrite = FALSE) #copying h20T images ending in _T (aka unprocessed thermal images) into the thermal only folder for ease in DJI SDK step
  }
}
```
</details>
