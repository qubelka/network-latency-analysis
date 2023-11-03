# network-latency-analysis

This repository contains code for analyzing network latency data collected from the open Internet measurement network [RIPE Atlas](https://atlas.ripe.net). The goal of the data analysis is to generate realistic scenarios for simulating edge computing environments where the network latency varies between users to different edge computing servers. 

Selection of appropriate RIPE Atlas measurement points is based on Amsterdam district division. Amsterdam is selected as a point of collecting measurements, because it has good anchor and probe coverage in RIPE Atlas compared to other European cities. To our knowledge there is no publicly available data on edge nodes placement. We select edge nodes (RIPE Atlas anchors) from different districts trying to mimic possible real world edge node placement setup. The anchors are selected using the geographic location so that no two are located right next to each other and spread across the city. We selected a subset of 5 anchors which have latency measurements collected from the same 10 Amsterdam districts. The devices sending ping-requests selected from these 10 districts (RIPE Atlas probes) represent end devices. 

`analyze-ripe-data.ipynb` contains the steps involved in selection of appropariate anchors and probes from Amsterdam RIPE Atlas data. We use `geopandas`-library to filter the anchors that are within the boundaries of Amsterdam districts (dutch: `gebied`). We use geographic data from the [Amsterdam city site](https://maps.amsterdam.nl/open_geodata). Coordinates are in WGS84 (EPSG4326). The shape file of Netherlands obtained from the [Princeton university library site](https://maps.princeton.edu/catalog/stanford-st293bj4601) is used for locating the anchors that were filtered out in the previous step on the map of Netherlands. 

`rtt_notebook.ipynb` contains steps involved in fetching latency data for the selected anchors and probes for specific time intervals and analysing the data using basic statistical analysis. The folder `measurement-results-new` contains summary RTT statistics (min, max, average, q1, median, q3) for chosen measurements over 15 minute time period on different dates. The folder `measurement-results-variance` contains 3 files with RTT variance for region 1 over different time slots. The folder `measurement-results-individual-rtts` contains individual RTT values for all selected regions over different time periods. The results obtained manually were compared to the graphs produced by RIPE Atlas LatencyMON tool. 

`distributions_rtt.ipynb` contains steps involved in fitting statistical distributions to collected data. We use a set of common statistical distributions (Beta, Birnbaum-Saunders, Exponential, Gamma, Generalized Extreme Value, Generalized Pareto, Inverse Gaussian, Logistic, Log-logistic, Lognormal, Nakagami, Rayleigh, Rician, t Location-scale and Burr Type 12). To determine which distribution fit the data best, we use the p-values generated with Kolmogorov-Smirnov test and 1000 Monte Carlo simulations. The results of running K-S test once and with 1000 simulations is compared to the results obtained by the Python [`fitter` library](https://fitter.readthedocs.io/en/latest) that is commonly used for fitting probability distributions. 

All the notebooks contain different test cases which were not used in evaluating the model, but were used to see how data behaves. 

### Steps omitted from `analyze-ripe-data.ipynb`

`analyze-ripe-data.ipynb` contains code pieces from multiple years and authors (@gpremsan). Some steps that require calls to RIPE Atlas API (and the filtered results of which are saved into `pickle-files`) have been omitted from the notebook. 

Fetching of anchors was performed using RIPE Atlas API query `/api/v2/acnhors/?country=NL&search=amsterdam`, for more information on RIPE Atlas API refer to the [documentation](https://atlas.ripe.net/docs/apis/rest-api-manual). Each anchor's coordinates are saved in `pickle-files/anchor-coordinates.pickle` in a form `anchor_id: [x-coordinate, y-coordinate]`.
   
Fetching of probe measurements targeted towards the anchors located in Amsterdam step was performed using the IPv4 address of anchors with the following RIPE Atlas query: `/api/v2/measurements/ping/?is_public=true&target_ip={}&status=2&is_oneoff=false&optional_fields=probes`. Anchors and corresponding measurements are saved to a file `pickle-files/measurements_by_anchor_id_dump.pickle`.  

The probes from the API response are saved into `lists_of_probes_by_anchor_id` in the form: 

```
anchor_id: [
    [probe_ids corresponding to the 1st measurement],
    [probe_ids corresponding to the 2nd measurement],
    ...
]
```

This dictionary is used to create `list_of_NL_probes_by_anchor_id_dump.pickle`, which filters the lists of probes sending pings to the selected anchors in Amsterdam to contain only those probes that are located in Netherlands. The final set of 5 anchors: 1135 (region 1, measurement 9333345), 1819 (region 3, measurement 22422574), 1293 (region 9, measurement 16453122), 2494 (region 17, measurement 29724243), 1922 (region 21, measurement 23062801) was selected based on the common probes involved in these measurements. We selected 10 probes (6348, 6031, 6220, 7139, 6082, 6955, 6932, 6969, 7188, 6661) that are involved in all of these measurements and locate in different Amsterdam areas (since some of the selected anchors work also as probes, we swapped the regions for probes 6348 and 6661 from 3 to 1 and 17 to 21 in order to get representation for regions 1 and 21).  

---
The README will be updated with the reference to the research article to which this repository corresponds, once the article is published. 