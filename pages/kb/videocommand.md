## FFMPEG

Combining videos: https://trac.ffmpeg.org/wiki/Concatenate 

* Method 1 - this method basically just strings the videos along so they need to be the same codecs, pretty much just copied from the ffmpeg site above:

```
## make a file that has the files you want to have as input
## our file is named mylist.txt in this example with the contents:
file '/path/to/file1'
file '/path/to/file2'
file '/path/to/file3'

## then you string them into a single file, this operation is quite fast as it is not transcoded:
ffmpeg -f concat -safe 0 -i mylist.txt -c copy output
```

* Method 2 - This is how I had to do this for zoom and some other videos, my usage was the same video/audio codecs and other things, but they just wouldnt play well with the above method, this transcodes them and takes a long time but works.

```
##  This method requires a lot of changes on the command line depending on how many files you are importing.
ffmpeg -i input1.mp4 -i input2.mp4 -i input3.mp4 -filter_complex "[0:v][0:a][1:v][1:a][2:v][2:a]concat=n=3:v=1:a=1[v][a]" -map "[v]" -map "[a]" output.mp4
## changes when adding videos ^^^^^^^^^^^^^^^^^^                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^         ^ 
## keep adding -i entries for each of the videos you are adding
## keep adding the [#:v][#:a] to each video you add (starting with 0)
## concat=n=# - this number equals the number of videos you are adding from input
```