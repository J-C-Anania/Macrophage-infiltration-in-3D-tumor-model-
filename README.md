# Macrophage-infiltration-in-3D-tumor-model-
This ImageJ macro measures the infiltration of macrophages (MonoMac, CD89+ PE) are capable of infiltrating a 3D spheroid tumour model (Raji, CFSE). The findings utilizing this macro are published in: Lara et al 2021 'The importance of antibody isotypes in anti-tumor immunity by monocytes and complement using human-immune tumor models' European Journal of Immunology

// SPHEROID INFILTRATION MACRO

macroVesion= "200402 J.C. Anania"; 

//clean up

run("Close All"); 
print("\\Close"); 
roiManager("Reset"); 
 
//Open and split channels 

open("F:/SL 3D Infiltration  Macro/10x_RAJI  CFSE 3D_RTX3_Mono CD89 PE 16 BIT_Zstack.lsm"); // Substitute file location for analysis of desired image
ImageIDoriginal=getImageID(); 
print(ImageIDoriginal); 

run("3D Project...", "projection=[Brightest Point] axis=Y-Axis slice=2 initial=0 total=0 rotation=10 lower=1 upper=255 opacity=0 surface=100 interior=50 interpolate");//if z-stack project image or skip this step
saveAs("Tiff", "F:/SL 3D Infiltration  Macro/Macro Output/Projection of 10x_3D.tif"); // Substitute save location, suggest creating folder to save all further data

run("Duplicate...", "duplicate"); //duplicate image so as you have original reference to compare to
ImageIDduplicate=getImageID();
print(ImageIDduplicate);
run("Split Channels"); 
 
MonoMac=ImageIDduplicate-1;//assign channels - macrophages = MonoMac, CD89+ PE
print(MonoMac); 
selectImage(MonoMac);
saveAs("Tiff", "F:/SL 3D Infiltration  Macro/Macro Output/MonoMac.tif"); // Substitute save location
 
Raji=ImageIDduplicate-2; //assign channels - spheroid = Raji, CFSE
print(Raji);
selectImage(Raji);
saveAs("Tiff", "F:/SL 3D Infiltration  Macro/Macro Output/Raji.tif"); // Substitute save location

//Threshold smooth and create mask 

selectImage(MonoMac); //smooth and mask 
run("Options...", "iterations=1 count=1 black do=Nothing"); 
run("Smooth"); 
setAutoThreshold("Otsu dark"); 
getThreshold(Mlower, Mupper);
print("MonoMac threshold=", Mlower);
saveAs("Tiff", "F:/SL 3D Infiltration  Macro/Macro Output/MonoMac threshold.tif"); // Substitute save location
run("Convert to Mask"); 
saveAs("Tiff", "F:/SL 3D Infiltration  Macro/Macro Output/MonoMac mask.tif"); // Substitute save location
run("Duplicate...", " "); 
run("Create Selection");
setSelectionName("Monomac region");
roiManager("Add");
MonoMask= roiManager("count")-1;

selectImage(Raji); //smooth and mask 
run("Options...", "iterations=4 count=1 black do=Nothing"); 
run("Smooth");  
setAutoThreshold("Otsu dark"); 
setAutoThreshold("Otsu dark"); 
getThreshold(Slower, Supper);
print("Spheroid threshold=", Slower);
run("Convert to Mask"); 
saveAs("Tiff", "F:/SL 3D Infiltration  Macro/Macro Output/Raji mask.tif");  // Substitute save location

selectWindow("Raji mask.tif");
run("8-bit"); 
run("Options...", "iterations=3 count=1 black edm=8-bit do=Close"); 
run("Fill Holes"); 
run("Options...", "iterations=3 count=1 black edm=8-bit do=Dilate"); 
run("Analyze Particles...", "size=100-Infinity display summarize add"); 

roiManager("Select", 1); //*Obs! May need to change depending on image, if the spheroid has multiple fragments in the field of view the macro may select a peripheral large piece of debris, if this occurs use the ROI manager to assist in selecting the correct number i.e. change 1. Search for *Obs! and ensire to change all numbers in macro for consistency
setSelectionName("Spheroid");
roiManager("Add");
SpheroidMask= roiManager("count")-1;

//Identify centre

run("Find Maxima...", "noise=10 output=[Point Selection]"); 
setSelectionName("Maxima");
roiManager("Add");
roiManager("Select", 1); //*Obs! May need to change depending on image, see above for details

Roi.getContainedPoints(xpoints, ypoints); //finds centre of spheroid
Array.getStatistics(xpoints, min, max, meanx, stdDev);
Array.getStatistics(ypoints, min, max, meany, stdDev);
print(meanx, meany);

//create EDM for heat map (distance)

getDimensions(width, height, channel, slices, frames); 
newImage("conditional dilation", "8-bit black", width, height, 1); // creates new image with same dimensions as original image
CondilID=getImageID();
setPixel(meanx, meany, 255); //selects centre of spheroid and creates distance map in new image
roiManager("Select", 1); //*Obs! May need to change depending on image, see above for details

roiManager("Select", 1); //*Obs! May need to change depending on image, see above for details
run("Options...", "iterations=3 count=1 edm=16-bit do=Nothing");
run("Distance Map");
run("35 step zero grey") //apply LUT to allow best definition of distance map
saveAs("Tiff", "F:/SL 3D Infiltration  Macro/Macro Output/Total heat map centre.tif") // Substitute save location

roiManager("Select", 1); //*Obs! May need to change depending on image, see above for details
run("Clear Outside"); // clears outside spheroid ROI i.e. outside spheroid will have distance of 0, 
run("Make Inverse");
changeValues(1, 812, 0);
run("Make Inverse");
run("Select None");

getMinAndMax(min, max); 
setMinAndMax(min, max);
print(min,max); // indicates how large the spheroid is
saveAs("Tiff", "F:/SL 3D Infiltration  Macro/Macro Output/Raji heat map centre.tif") // Substitute save location
saveAs("Jpeg", "F:/SL 3D Infiltration  Macro/Macro Output/Raji heat map centre.jpg") // Substitute save location

open("F:/SL 3D Infiltration  Macro/Macro Output/Total heat map centre.tif") // Substitute file location
selectWindow("Total heat map centre.tif");
roiManager("Select", 0); 
run("Make Inverse");
run("Clear Outside");
run("Make Inverse");
changeValues(3, 3, 0);
run("Make Inverse");
run("Select None");

getMinAndMax(min, max); 
setMinAndMax(min, max);
print(min,max); 

roiManager("Select", 1); //*Obs! May need to change depending on image, see above for details
run("Clear Outside");
run("Make Inverse");
changeValues(3, 3, 0);
run("Make Inverse");
run("Select None");
saveAs("Tiff", "F:/SL 3D Infiltration  Macro/Macro Output/MonoMac heat map centre.tif") // Substitute save location, this combines the distance map from spheroid centre data with macrophage+ regions i.e. provides distance of macrophages form spheroid centre in pixels
saveAs("Jpeg", "F:/SL 3D Infiltration  Macro/Macro Output/MonoMac heat map centre.jpg") // Substitute save location

//measure histo pixel distance 

getRawStatistics(nPixels, mean, min, max, std, dhistogram); 
open("F:/SL 3D Infiltration  Macro/Macro Output/MonoMac.tif"); // Substitute file location
selectWindow("MonoMac mask.tif");
run("Set Measurements...", "area center redirect=None decimal=2");
saveAs("Results", "F:/SL 3D Infiltration  Macro/Macro Output/Results.csv"); // Substitute save location
run("Analyze Particles...", "size=50-2000 display summarize add save=[F:/SL 3D Infiltration  Macro/Macro Output/Results.csv]"); // Substitute save location; Particle size to not include "fibres" which may be present in spheroids 

open("F:/SL 3D Infiltration  Macro/Macro Output/Total heat map centre.tif") // Substitute file location
selectWindow("MonoMac heat map centre.tif"); 
run("35 step zero grey"); 
selectWindow("Results");
print("nResults", nResults);
print("\\Clear");

setBatchMode(1); // reads the fluoresence intensity of Macrophage+ pixel and the distance form spheroid centre for each pixel. 
for(x=0; x<nResults; x++) {
	X = getResult("XM", x);
		Y = getResult("YM", x);
				selectWindow("MonoMac.tif"); 
				Intensity= getPixel(X, Y); 
				selectWindow("Raji heat map centre.tif"); 
				Distance= getPixel(X, Y); 
				print(Intensity,",", Distance); 
}//for x
setBatchMode(0);

Target= File.open("Text", "F:/SL 3D Infiltration  Macro//Macro Output/Intensity, Distance Values.txt"); // Saves numerical data generated in loop above. Substitute save location
