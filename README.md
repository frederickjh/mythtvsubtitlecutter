mythtvsubtitlecutter
====================
<h2>Project Concept</h2>
Currently using the  <a href="http://www.mythtv.org/wiki/mythtranscode#Using_the_--mpeg2_Option_for_.28Virtually.29_Lossless_MPEG2_Transcode">---mpeg2 option with Mythtranscode</a> means you will loose your DVB subtitles. This project aims to cut an extracted external subtitle file (.srt) with the same cuts as the mp4 file, so that subtitles are retained for playback in MythTV.

<b>Note:</b> I believe that Closed Captions(CC) used in the USA in the ATSC digital video broadcasting format are handled differently. I believe that these can be retain by adding the  –profile autodetect option to your mythtranscode invocation. I do not live in the US so I cannot test this. See this blog post and the mythtranscode string in it. <a href="http://www.hackourlife.com/encoding-mythtv-recordings-with-subtitle-and-surround-sound/">Encoding MythTV Recordings with Subtitle and Surround Sound by lifehacker</a>
<h3>Mythtv</h3>
<a href="http://www.mythtv.org">Mythtv</a> allows one to record TV programs from broadcasts. It has the built in capabilities to try and mark commercials. It also hase the ability to allow you to edit your recordings using a cut list. This means that all the video you recordis is still there. Mythtv uses the cut list to determine what sections it should or should not play when you play a program back. As time goes on more and more of your storage space will be used by video that you do not want to watch.
<h3>Mythtranscode</h3>
Enter the <a href="http://www.mythtv.org">Mythtv</a> utility <a href="http://www.mythtv.org/wiki/mythtranscode">Mythtranscode</a>. Mythtranscode in versions +0.19 has a virtually lossless MPEG2 transcode option. The <a href="http://www.mythtv.org/wiki/mythtranscode#Using_the_--mpeg2_Option_for_.28Virtually.29_Lossless_MPEG2_Transcode">--mpeg2(-m)</a> option combined with the --honorcutlist will make a <a href="http://www.mythtv.org/wiki/mythtranscode#Remove_commercials_from_an_MPEG2_recording">virtually lossless MPEG2 transcode</a>. What this does is let you select cut points and mythtranscode will only re-encode the frames that need to be, copying the rest directly from the stream. If you are using this to cut commercials and you cut at the black frames of a fade to black (or white of a fade to white), what you get is basically a perfect stream copy of the video with no loss to the frames that matter. This also the side benefit that converting the mpeg2 from a TS stream to a PS stream, has a possibly of saving up to 20% of the file size!
<h3>The problem with Mythtranscode and Subtitles</h3>
The downside is it will not bring subtitles over to the new file. See this Mythtv issue <a href="https://code.mythtv.org/trac/ticket/2468#comment:6">#2468 closed defect (fixed) comment #6</a>. Althought it is marked as being fixed this only refers to the problem with audio tracks swaping places and not to the problem of subtitles being included in the new PS stream, as this is not possible with the format.
<h3>Mythtv and subtitle files</h3>
Mythtv allows you to use your own subtitles .srt files. I could find not offcial documentation to this effect but I have used this in the past when a movie records in a language that I am not well versed in. I downloaded the subtitle file from the internet then when playing the recording I go into the menu > Playback > Playback data. There you can find the filename of the recording name your .srt the same basename and then place the srt file in the directory with the recording. MythTV will now use the file instead of the embedded subtitles. 

Example: for a recording named 3001_20140913175000.mpg the subtitle file should be 3001_20140913175000.srt

Unoffical documentation of this feature can be found in the <a href="http://www.mythtv.org/wiki/Feature_Wishlist_(Frontend_Addons)#Captions_.2F_MHEG_.2F_Subtitles">Feature Wishlist (Frontend Addons) in the Captions / MHEG / Subtitles section</a>. Quoted here in case it changes.

"<b>Multiple subtitles per video. You can only use 1 subtitle file now, as it has to be the same file name (except the .srt) as the movie file. Why not let it detect the same files with a "-dutch" or "-english" tag? Example: "Movie.avi" would use subtitle "Movie-dutch.srt" and "Movie-english.srt". -- Apparantly, it "does" look for these files, but it will only load the first file which it can parse correctly.
Support for multiple embedded subtitles are supported.</b>"

<h2>Project proposed Workflow</h2>
That is right this is a project just starting out. Conception starts with the idea.

1. Extract the subtitles using <a href="http://www.mythtv.org/wiki/Mythccextractor">MythCCextractor</a> to .srt files.
2. Extract cut list from Mythtv database using the <a href="http://www.mythtv.org/wiki/Mythutil#--getcutlist">--getcutlist option of Mythutil.</a>  

Example:
$ mythutil --getcutlist --chanid 3001 --starttime 20140913175000 -q
Cutlist: 2329-3325,5312-6773,10688-11543,12557-13643,14393-15019

The notes in the <a href="http://www.mythtv.org/wiki/Mythcutprojectx">Mythcutprojectx</a> script stated that the cutlist that Mythutil extracts is a frame-count cutlist. See this blog post <a href="http://videosubtitles.wordpress.com/tag/convert-frames-to-milliseconds/">"SubRip (.srt) subtitle format" in the section "What else should I know?"</a> for more information about converting. Conversion is based on the number of frames per second (fps), so we will need to gather that information as well.

<b>"(frames * 1000) / fps

i.e. take the frame number and multiple it by 1000, then divide the result by your video’s frame rate (fps).

e.g. to convert a 30fps timecode (00:00:01:12) in to milliseconds:
(12 frames * 1000) / 30fps = 400 milliseconds
The resulting timecode in SRT format would be 00:00:01.400"</b>



.srt files use a hh:mm:ss:mmm format where mmm = milliseconds so conversion maybe needed.

3. Progess each cut pair(start and end times) in the list, cutting the extracted srt subtitle file.
	a. Determin the start time of the cut in the needed formats
	b. Determin the end time of the cut in the needed formats
	c. Determin the length of the cut in the needed formats
	d. Split the srt file at start time
	d. Split the second srt file at the end time
	e. Adjust timing of subtitles in the srt file from after the cut by the length of the cutlist
	f. Join the 1st and 3rd srt files together and remove the 2nd with the cut subtitles.
<h3>Possible Tools</h3>	
The command-line tool <a href="http://karasik.eu.org/software/">'subs' and its perl backend Subtitles.pm</a> provide means for simple loading, re- timing, converting, and storing these subtitle files. Supported formats: .srt, .sub, .smi. 
Download: <a href="http://karasik.eu.org/software/Subtitles.tar.gz">Subtitles.tar.gz</a>
	
	


