[[Satellite Data Analysis]]
****

We want to know which pixels on radar data corresponds to an urban area.
	*Radar relies on a signal (active sensor) to receive the activity it caused. 
	This means that you send an impulse, and receive an answer in time, for different positions on the surface. For this reason, you will receive nadir first, and then the sides. The issue is we will receive data pretty much at the same time if the distance of two position are the same; this is why a radar only looks at one side, not straight down at nadir.*

First thing that helps is how water doesn't return any information (which is why it always appear black). 
When it comes to trees, the signal will be diffused (because of the layer of leaves that are oriented in several directions, the EM waves will be reflected "randomly"). Those usually appears grey.
Very bright (white) points are the result of double-scattering effects, most of the signal comes back to us. This typically happens on buildings (built-up structures) and mountains.

To summarise:
- Low values: Water, shadows
- Mid values: Trees (and vegetation overall)
- High values: Buildings, Bridges, Mountains
> In fact, the radar data mostly shows us the geometry of the area/object, not the material. We can't really guess at first sight if a building is made out of bricks, metal or wood (if we need this, we rely on infrared).

The unsupervised method we rely on to locate urban areas is **Urban Extractor (UEXT)**, which happens in three steps:
1. We expect things to not change much: Our urban area is supposed to be pretty much the same a month later. Since radar aren't disturbed by clouds or any meteorological event, we can just get data every 10 days or so, and check if our—supposed—urban area remains identical. First step is to **apply a de-noising filter on each sample, re-adjust the histogram of each sample, and combine those**.
2. We know that bright points corresponds to urban areas (most of the time). So, we just **apply a threshold on bright values, and do region growing on those bright values**.
		*Region growing relies on a second threshold (which is lower than the first threshold we used to isolate bright values). We look around all bright values and check if the values are higher than this second threshold. If so, we preserve them and apply region growing on them iteratively.*
3. This one is more philosophical, its about what elements should or shouldn't be considered as part of the urban area: Should roads be captured as well ? should vegetation inside the urban area be considered part of it (a park, trees in the street) ?  In general, we consider than the urban area contains it all. So, during our post-processing phase, we simply **consider furthest bright values as the limit of the urban area. So, we link those pixels together by "drawing" borders which ultimately represents the urban area**.
		*We can—optionally—use a **Digital Elevation Model (DEM)** to adjust our result*

Now, we can apply our UEXT to a series of images (in time) to obtain a multi-temporal clustering.
	*Results will be a **urban core** (things that does not change in time), and the **masks** of each original sample we applied UEXT to.*
> This is very useful if we want to observe urban expansion:
> - how the city expands in time (edge expansion growth)
> - how new clusters are created nearby the original cluster we captured (outlying growth)

"Coherence" is a measure of similarity between two images of the same area. The objective is to **detect changes inside the urban area itself** (e.g. a building was renovated).
We chain this coherence on a set of n images.
	*There is a big ass mathematical formula in the slides but i don't understand it :)*
> If applied on urban areas, coherence will slightly vary but not too much (unless the city was destroyed by a tornado or carpet bombing). If applied on vegetation, it will vary a lot depending on the seasons. When applied on water nothing changes.


If we analyse large samples (like an entire megalopolis) in an automatic manner, various models of heterogeneous clustering can be used (GMM, DBSCAN...) to separate data in multiple clusters. Each method will return a slightly different output as they will separate data in different ways. So, combining all the output allows us to retrieve more generic and possibly accurate clusters.
	*Each cluster will represent a specific subpart of the urban area (cluster 1 = factories, cluster 2 = suburbs, cluster 3 = high-density city centre...) *
> This increases reliability of our analysis, but the counterpart is that we throw away some details (which isn't thaaaat important since our sample is supposed to be extremely large).




