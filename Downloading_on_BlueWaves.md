## Downloading multiple large data files onto BlueWaves is easiest using the following method:

#### 1) Use 'screen' to open up a terminal that won't stop if you turn off your computer. This will allow downloads to occur overnight.

```
screen -S <SCREEN_NAME>
```
**Note:** '-S' sets the name of the screen, which you can set to whatever you want (e.g. Download_Bird)

#### 2) Downloads are fastest on the fs03 node, so login to that.
```
ssh fs03
```
#### 3) Navigate to the directory where you want to download data

```
cd <MY_DATA_FOLDER>
```

#### 4) Create an empty download script script
```
nano <MY_SCRIPT_NAME>
```
**Note:** Remember to use descriptive file names, even for download scripts

#### 5) For each file you want to download, add a download command
```
wget -O <FILE_NAME_YOU_WANT> <FILE_URL> &
```
**Example:**   
wget -O MyAwesomeFile_1.txt http://www.funfiles.com/file.txt &  
wget -O MyAwesomeFile_2.txt http://www.funfiles.com/secondFile.txt &

**Note 1:** If you issue wget without the -O option, the file will be saved with an identical name as the download source, where -O allows you to customize the file name

**Note 2:** The '&' at the end of the command  allows multiple downloads to happen at once

#### 7) Save the file and make it executable
```
chmod +x <MY_SCRIPT_NAME>
```

#### 8) Execute the command

```
./<MY_SCRIPT_NAME>
```

#### 9) When the downloads start, detach the screen (the downloads will continue in the background). Detach screen by holding the CTRL key and hitting A, then D

#### 10) You can check the progress of your downloads by attaching your screen 

```
screen -r <SCREEN_NAME>
```
