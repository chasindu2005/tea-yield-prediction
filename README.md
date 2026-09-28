
 ### Tea Yield Prediction using Machine Learning

My family has a small tea plantation, and I'm building a machine learning model to predict the tea yield (weight in kg) for each plucking round. This is my main project, and this repo is where I keep all of it.

**This README is a living document.** Right now only the first part (data collection and cleaning) is done. I'll keep adding the next stages to this same repo as I finish them, and I'll update the status list and the progress log below each time.

## Project status

- [x] Data collection (plucking records + weather data)
- [x] Data cleaning
- [x] NDVI from Sentinel-2 satellite images
- [ ] Exploratory data analysis
- [ ] Feature engineering
- [ ] Model training and evaluation

## Repo structure
tea-yield-prediction/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   ├── raw/          # my original plucking records
│   └── processed/    # datasets after weather, cleaning and NDVI
└── notebooks/
    ├── 01_data_collection_and_cleaning.ipynb
    └── 02_ndvi_extraction.ipynb

More notebooks will go into `notebooks/` as the project grows. Run them in number order.

## Part 1: Data collection and cleaning

### Yield records

The raw file (`data/raw/Tea Yield data collection.csv`) has the plucking date, the number of days since the previous pluck and the total yield weight (kg) for every plucking round. It has 185 rows, but 6 of them are empty rows at the bottom, so the real dataset is **179 plucking rounds from 5 October 2020 to 8 September 2026**.

### Weather data (Open-Meteo)

I don't have weather records from the plantation, so I used the Open-Meteo Historical Weather API. For every plucking round the code asks for the weather between the previous pluck and the current one. The window starts `plucking_interval - 1` days before the plucking date and ends on the plucking date. For the first row (interval 0) it is only that one day.

From the daily values in that window I calculated:

- `Avg Temp`: mean of the daily mean temperature (°C)
- `Relative humidity`: mean of the daily mean relative humidity (%)
- `total_rain_mm`: sum of the daily rainfall (mm)
- `Rainfall`: average rainfall per day in that window (mm/day)

### Cleaning

- renamed `DATE` to `plucking_date` and `Day Since  last Pluck` to `plucking_interval`
- converted `plucking_date` to a datetime column
- dropped the 6 empty rows (185 to 179)
- converted `Year`, `Month` and `Day` to integers and fixed the `Month ` column name (it had a trailing space)

## Part 2: NDVI from Sentinel-2

NDVI shows how green and healthy the plants are, so I thought it could help predict yield. I got it from the `COPERNICUS/S2_SR_HARMONIZED` collection using Google Earth Engine.

1. Take all images over the plantation point between the first and last plucking date.
2. Skip whole images with more than 70% cloud.
3. Mask cloud, cloud shadow, thin cirrus and snow pixels using the SCL band (classes 3, 8, 9, 10, 11).
4. NDVI = (B8 - B4) / (B8 + B4), where B8 is near infrared and B4 is red.
5. Take the mean NDVI at the point (10 m pixels) for each image.
6. The satellite doesn't pass on my exact plucking dates (about every 5 days, and cloudy images are removed), so I used linear interpolation (`np.interp`) to estimate NDVI for each plucking date.

## Final dataset

`data/processed/Tea Yield final dataset.csv`

| Column | Description |
| :--- | :--- |
| `Year`, `Month`, `Day` | Date of the plucking round split into parts |
| `plucking_date` | Date of plucking (YYYY-MM-DD) |
| `Total yield weight per cycle (kg)` | Yield of the round in kg. **This is the target variable.** |
| `Rainfall` | Average rainfall per day during the cycle (mm/day) |
| `Avg Temp` | Average temperature during the cycle (°C) |
| `Relative humidity` | Average relative humidity during the cycle (%) |
| `plucking_interval` | Days since the previous plucking |
| `total_rain_mm` | Total rainfall during the cycle (mm) |
| `NDVI` | Sentinel-2 NDVI interpolated to the plucking date |

## Things to keep in mind

- The weather values are gridded historical data from Open-Meteo, not readings from a weather station at the plantation, so they may differ a bit from what really happened in the field.
- NDVI is an interpolated estimate, not a measurement taken on the plucking date.
- After the last usable satellite image, `np.interp` just repeats the last known value, so NDVI in the last few rows can be identical.
- NDVI comes from one point (one 10 m pixel), which may not represent the whole plantation.
- The coordinates in the notebooks are example values. Replace them with your own location.

## How to run

```bash
pip install -r requirements.txt
```

You need a Google Earth Engine account and a Google Cloud project. Run ee.Authenticate() once, then put your own project ID in ee.Initialize(project='...'). Change the coordinates and file paths in the notebooks to your own.

## Progress log
  * Sep 2026: Finished data collection, cleaning and NDVI extraction. Uploaded to GitHub.

## Credits
Weather data by Open-Meteo.com. Contains modified Copernicus Sentinel data, accessed through Google Earth Engine.

    
