

---
layout: post
title: Battle of wards in Surrey
categories: [ML,clustering, Unsupervised learning, Python]
datetime: 06-11-2019
author: Sepideh Nazemi
---


Last summer, my family and I went to Paris. One night. We were out for dinner and were looking for a parking space near the restaurant we had targeted. While we were hunting a parking space near the restaurant, we got distance from it. So, we had to walk down a couple of streets to get to the restaurant. As we start walking, we felt unsafe since many strangers were wandering around or sitting in a group in the dark. We were worried about how safe it will be to return to our car in a couple of hours. We decided to go back to our car and find somewhere more close to the restaurant. Sadly, we couldn't find any parking space. So, We decided to call the restaurant and ask for their local expertise. We wondered **how safe this neighbourhood is**?

The restaurant manager tried to convince us that *the neighbourhood is very safe due to the existence of many luxury pubs and restaurants in it*.

Having this stressful experience during my holiday, I decided to examine the hypothesis that the restaurant's manager put forward to us. **I want to test whether a neighbourhood with a high number of venues are safe or not in comparison with the area with less number of venues?**

## Case study and Data selection

I've selected Surrey County in the UK as a case study. The information I required to conduct this analysis is geographical coordinates, criminal records and venues available in each specific part of the selected region. 

 The dataset, including information related to the crime committed in each ward in this county, are available on [Surrey-i website](https://www.surreyi.gov.uk/) via this [link](https://www.surreyi.gov.uk/dataset?q=Master Crime Category by Ward ). Although the dataset includes Geocode of each ward, due to license requirement, I was unable to use this information. I couldn't find any free dataset or specific website with required geographical coordinates information where be able to extract the required data systematically. So, I choose a postcode in the centre of each ward manually. Later on, I merge this information with the dataset containing all available postcodes in the UK and their latitude and longitude coordinates where I can estimate the latitude and longitude of the centre of each ward.

All the datasets that I've used for this study are available in the following links:

* [Master Crime Category by Ward.xls](https://www.surreyi.gov.uk/dataset?q=Master Crime Category by Ward https://www.surreyi.gov.uk/dataset?q=Master Crime Category by Ward )

* [Wards by postcode.xls](https://github.com/SepidehN/Coursera_Capstone/blob/master/Wards_by_postcode.xls)

* [UkPostcodes.csv](https://www.freemaptools.com/download-uk-postcode-lat-lng.htm)

Other required information is the venues' data associated with each specified coordinates in each ward. To do so, I have utilised **Foursquare API**. Since I am interested in how busy/crowded the areas are, I calculated the total number of venues in each specified location and merged this information to the primary data. Now, I am ready to examine and analyse the wards based on their popularity and criminal records. The details of  data acquisition, data cleaning and preparation can be viewed on this [Jupyter notebook]( https://github.com/SepidehN/Coursera_Capstone/blob/master/TheBattleofNeighborhoods-Part2.ipynb ) 

## Data Analysis

To do this, I've used the *k*-mean clustering algorithm, a simple but powerful unsupervised machine learning technique. K-mean learns from the properties of data and divides it into different (hopefully optimal) groups. Many clustering algorithms are available. But perhaps the most widely used one is *k*-means available in sklearn.cluster.KMeans. 

One of the main drawbacks of the *k*-mean algorithm is the difficulty of predicting the number of suitable clusters. To overcome this, I ran the algorithm with different parameter *k* and analysed the outcome of each cluster and finally, I have chosen the best *k* that can divide the wards more explicitly.

After applying the algorithm and regulating the parameters in the algorithm (i.e., k) and Foursquare API (i.e., limit and radius), I have discovered that it is sensible to cluster Surrey's wards into 5 different clusters as presented in following table.

| Cluster | color  |               Label                |
| :-----: | :----: | :--------------------------------: |
|    0    | Green  |  Moderate density & Moderate risk  |
|    1    |  Blue  | Very high density & Very high risk |
|    2    | Purple |  Moderate density & Moderate risk  |
|    3    |  Red   |  Very low density & Very low risk  |
|    4    | Olive  |      low density & High risk       |



<img src="/images/ClustersOnMap.jpg" />

Cluster 1, which includes wards in the centre of two big towns in Surrey area (i.e., Guildford and Epsom), shows that more venues can cause more risk in the area. As the Table below shows the number of crime committed in this cluster are significantly high. On the other hand, Cluster 3, is associated with wards that have a very low density of venues and a slight risk in terms of crime committed. However, Cluster 4, despite having a low density of the venues, it has high risk where make these wards undesirable since they are neither enough popular nor enough safe. These wards are mostly inside the M25 or near big town such as Woking or Redhill. Cluster 2 and 0 are slightly similar. They are wards with moderate density and relatively moderate risk. However, as presented on the map, Cluster 0 includes wards where mostly are located inside the M25 band which make increases in specific categories of crime such as theft handling and vehicle interference in these areas. In general, the model confirms we should expect crime rises in the more busy/popular areas. The details of the model can be viewed on this [Jupyter notebook]( https://github.com/SepidehN/Coursera_Capstone/blob/master/TheBattleofNeighborhoods-Part2.ipynb ) 

| Cluster | \# of venues | TNO  | Theft handling | vehicle interference |
| :-----: | :----------: | :--: | :------------: | :------------------: |
|    0    |      42      | 474  |      114       |          13          |
|    1    |      78      | 1860 |      723       |          6           |
|    2    |      39      | 474  |       93       |          4           |
|    3    |      28      | 207  |       35       |          3           |
|    4    |      34      | 983  |      231       |          8           |



## Conclusions

Utilising clustering technique, I was able to group different wards in unique clusters based on the number of venues and crime features in those areas. I built the k-mean clustering model and examined different features in the model, such as the number of clusters and identified 5 clusters. This model indicates that we can not assume that areas with a high number of venues (popular places) are safer than areas with less number of venues. 

