# Processing thermal imagery to extract temperature values
A repository for extracting temperature rasters from thermal imagery (specific to the DJI H20 series).

This repository hosts a workflow to convert raw thermal imagery from the Zenmuse H20T sensor to temperature rasters in a geotiff format. The code and workflow are based on work by Olivia Waite with the GenomeBC project, specifically from Chapter 14 of the GenomeBC Best Practices Guide. This version has been re-formatted for use specifically with stream surface temperature in freshwater systems. 

## Creating the directories 
First we'll gather all of the raw imagery from the H20T into a single folder called "Merged_T" to loop through. We'll then create another folder called "Temperature_rasters" that will have the converted imagery, and a third folder called "Temperature_rasters_EXIF" that will hold the converted imagery with the original EXIF data written on it. This way, we'll have copies of the thermal data without overwriting the original files. 

You can make these directories on your own, but the code below will write the folders for you if they don't already exist. 

<details>
<summary>Click to show the code</summary>

```{r}
# name of the 'merged' folder that will contain all the H20T images from multiple folders

dir <- "E:/Path_to_your_files/"

merged <- "Merged_T"

if (!dir.exists(paste0(dir, "\\",merged,""))) {
  dir.create(paste0(dir, "\\",merged,""),recursive = TRUE)
}

if (!dir.exists(paste0(dir, "\\",merged,"\\Temperature_rasters"))) {
  dir.create(paste0(dir, "\\",merged,"\\Temperature_rasters"),recursive = TRUE)
}

if (!dir.exists(paste0(dir, "\\",merged,"\\Temperature_rasters_EXIF"))) {
  dir.create(paste0(dir, "\\",merged,"\\Temperature_rasters_EXIF"),recursive = TRUE)
}

#copying H20T images ending in _T (aka unprocessed thermal images) into the thermal only folder for ease in DJI SDK step
#Getting folders in the H20T folder that have DJI and end in -H20T, may need to check that all folders are named this way

list_dir <- list.files(dir, full.names = TRUE,pattern = c("DJI")) 
print(list_dir)

for (d in 1:length(list_dir)){
  #calling each directly seperately
  folder_dir <- list_dir[d]
  #getting allJPEGs ending in _T in that directory
  T_files <- list.files(folder_dir, pattern = "_T\\.JPG$")
  #print(T_files)
  file.copy(from = paste0(folder_dir,"\\", T_files),
            to = paste0(dir,"\\",merged,"\\", T_files), overwrite = FALSE)
}

```
</details>

## Weather data (ambient temperature, humidity, time of day, etc)

In order to accurately pull temperature values from the thermal data, we need to include some weather information for the DJI thermal SDK. 

<details>
<summary>Click to show the code</summary>

```{r}

#getting image capture date from exif data of the first image in the H20T folder

time_file_list <- list.files(paste0(dir,"\\",merged), pattern = "_T\\.JPG$")
img_1 <- time_file_list[1]
#img_1
start_exif_flight_date <- data.frame(exif_read(paste0(dir,"\\",merged,"\\",img_1),
                                         tags = "CreateDate",
                                         quiet = TRUE))

#getting flight date for the h20T image (and therefore the flight) that will be used to get the humidity and amb temperature for that flight
flight_date <- as.Date(start_exif_flight_date$CreateDate, '%Y:%m:%d %H:%M:%S')
flight_date


#Get start flight time
start_flight_date_time <- start_exif_flight_date$CreateDate
start_flight_date_time


#Get end flight time
img_last <- tail(time_file_list, n=1)
#img_last


end_flight_date_time <- data.frame(exif_read(paste0(dir,"\\",merged,"\\",img_last),
                                        tags = "CreateDate",
                                        quiet = TRUE))

end_flight_date_time <- end_flight_date_time$CreateDate
end_flight_date_time


# ENTER start and stop times from above, ie if start was 11:15 put 11:00:00 and if end was 1:23 put 14:00:00 (2pm) as the end
start_time_avg <- "11:00:00" #Based off of start_flight_date_time from above
stop_time_avg <- "14:00:00" #Based off of end_flight_date_time from above

```
