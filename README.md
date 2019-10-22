# Missing Maps Hackathon: Detection of vegetation loss

This notebook demonstrates work of Jan Benetka and Jakub Matějka carried out during the first [**Missing Maps & MSF Hackathon**](https://www.eventbrite.com/e/missing-maps-hackathon-pilsen-hackathon-s-lekari-bez-hranic-registration-74140796117) organized in the Czech Republic. The goal of the challenge is to identify rural locations with potential population growth. 

Expansion of population in certain area is almost always accompanied with side effects such as increased number of houses and consequent emergence of paths among them, loss of trees in nearby forests (used for heating or cooking), or degradation of natural environment due to the industrial growth. A common denominator of all of these effects is a decrease of vegetation mass. Our aim, therefore, is to provide **simple yet effective methods to detect loss of greenery and analyze its rate**.


### Notebook preview 
- see HTML version at http://www.rybak.io/public/missing_maps/hackathon.html

![Analysis preview](https://github.com/benetka/missing_maps_hackathon/blob/master/img/area_analysis.png "Analysis output of one area.")




--
### Task & Reasoning
Expansion of population in certain area is almost always accompanied with side effects such as increased number of houses and consequent emergence of paths among them, loss of trees in nearby forests (used for heating or cooking), or degradation of natural environment due to the industrial growth. A common denominator of all of these effects is a decrease of vegetation mass. 
Our aim, therefore, is to come up with a **simple yet effective methods to detect loss of greenery and analyze its rate**.


---


### Data 

After brief search for available datasets, we settled on images from the <a href="https://sentinel.esa.int/web/sentinel/missions/sentinel-2">Copernicus Sentinel-2</a> mission. It provides multi-spectral images of the Earth that are publicly available. Morover, the satellite(s) revisits the same area with high frequency (~5-10 days), which increases our chances of collecting nice cloud-free snapshots. The pictures come in a spatial resolution that varies from 10m/pixel for some spectral bands (RGB, NIR), through 20m/pixel (e.g., Red Edge), up to 60m/pixel (<a href="https://www.hatarilabs.com/ih-en/how-many-spectral-bands-have-the-sentinel-2-images">see complete list</a>). It is not quite enough for reliable recognition of people or rooftops, nevertheless, it is fine for spotting variations in vegetation levels.

One tool that certainly deserves our mention is the <a href="https://apps.sentinel-hub.com/eo-browser/">**EO browser**</a> by SINERGISE. Not only you can download the satellite images here, you can also visualize and choose any of the available spectral bands. On top of that, the browser has a function to create a timelapse from imagery for a given region and you can decide what percentage of clouds you are able to tolerate in the pics.


---


### Method(s) 

#### TL;DR
Simply put, we take two satellite images from the same area in two different moments and we look for differences in vegetation. We then highlight these differences for each consecutive pair of snapshots and measure their rate.

#### Long version

***1) Vegetation identification***

Trying to distinguish vegetation based on color in visible spectrum might work for some areas (lush grasslands), but not for others (autumn tundra). Much better way of vegetation remote-sensing is to understanding that plants, in order to produce sugar via photosynthesis, do absorb solar radiation of certain wavelengths (see figure below) while they reflect other ranges not to overheat. SENTINEL-2 is well-equipped to capture images in red and near-infrared spectrum, and those are the wavelenght ranges needed to calculate **NDVI: Normalized difference vegetation index**. High values of NDVI index represent rich vegetation, low values suggest lack of greenery. The index is computed as:
$NDVI=\frac{(NIR - RED)}{(NIR + RED)}$, where NIR is near-infrared spectrum and RED is red (visible) spectrum.



<table>
<tr><td> <img src="https://github.com/benetka/missing_maps_hackathon/blob/master/img/photosynthesis.png" alt="Photosynthesis and wavelength ranges." style="width: 650px;"/> 
</td></tr>
<td style="text-align:left">Ranges of solar radiation wavelength where photosynthesis happens. Refer to <a href="https://en.wikipedia.org/wiki/Normalized_difference_vegetation_index" target="_blank">Wikipedia article about NDVI.</a> to find more information.</td>
</table>

*Below, you can see a satellite image of Manhattan in NDVI mode as well as in true color spectrum:*

***2) Vegetation change tracking***

We saw that in the "NDVI world" brigh equals little vegetation, dark means a lot. Next step, naturally, is to compare two temporally different snapshots from the same area and calculate the extent of vegetation loss. We simply compute per-pixel differences between two images $I_1$ and $I_2$:

$I_d = |I_1(x,y) - I_2(x,y)|$

after first converting them to grayscale and normalizing their contrast/brightness:

$I_{2\_norm} = \frac{\sigma_1}{\sigma_1}(I_2(x,y) - \nu_2)+\nu_1$,

where $\sigma$ is mean brightness and $\nu$ is mean contrast. Find the method described at [sentinel-hub blog](https://medium.com/sentinel-hub/next-mission-automated-detection-of-land-changes-8e988dce55ff).

*An example of a (quick) deforestation process captured on two NDVI snapshots converted to grayscale looks like this:*

*Image pairs like that serve as our input data to express the differences (marked below in red color):*
