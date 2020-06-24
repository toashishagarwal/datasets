# Research dataset

The Unsplash dataset is composed of multiple TSV files :

### 1 - photos.tsv
The photos dataset has one row per photo. It contains the different properties
of the photo, some data collected through AI services and overall stats.

Here's the structure:
| Field                       | Description |
|-----------------------------|-------------|
| photo_id                       | ID of the Unsplash photo |
| photo_url                      | URL of the photo's page on unsplash.com |
| photo_image_url                | URL of the photo resource on images.unsplash.com |
| photo_submitted_at             | Timestamp of when the photo was submitted to Unsplash |
| photo_featured                 | Whether the photo was promoted to the editorial feed or not |
| photographer_username          | Username of the photographer on Unsplash |
| photographer_first_name        | First name of the photographer |
| photographer_last_name         | Last name of the photographer |
| exif_camera_make               | Camera make (brand) extracted from the EXIF data |
| exif_camera_model              | Camera model extracted from the EXIF data |
| exif_location                  | Location of the shot, extracted from the EXIF data |
| exif_latitude                  | Latitude of the shot, extracted from the EXIF data |
| exif_longitude                 | Longitude of the shot, extracted from the EXIF data |
| exif_iso                       | ISO setting of the camera, extracted from the EXIF data |
| stats_views                    | Total # of times that someone has seen the photo Unsplash ecosystem |
| stats_downloads                | Total # of times that someone downloaded the photo in the Unsplash ecosystem |
| ai_description                 | Textual description of the photo, generated by a 3rd party AI |
| ai_primary_landmark_name       | Landmark present in the photo, generated by a 3rd party AI |
| ai_primary_landmark_latitude   | Latitude of the landmark, generated by a 3rd party AI |
| ai_primary_landmark_longitude  | Longitude of the landmark, generated by a 3rd party AI |
| ai_primary_landmark_confidence | Confidence that the 3rd party AI has that the landmark is present |

### 2 - keywords.tsv
The keywords dataset has one row per photo/keyword combination. It contains data
about how a keyword is tied to a photo and how the combination performs in our search engine.

Here's the structure:

| Field                         | Description |
|-------------------------------|-------------|
| photo_id                      | ID of the Unsplash photo |
| keyword                       | Keyword or search term |
| keyword_service_1_confidence  | Confidence for the keyword from our 1st 3rd party AI |
| keyword_service_2_confidence  | Confidence for the keyword from our 2nd 3rd party AI |
| keyword_suggested_by_user     | Whether the keyword was added by a user (a human) or not |
| keyword_category              | Category to which Unsplash associates the keyword |
| keyword_subcategory           | Subcategory to which Unsplash associates the keyword |
| search_conversions            | Total # of search conversions for that photo on this keyword |

### 3 - collections.tsv
The collections dataset has one row per photo/collection combination. Whenever a photo
belongs to a collection created by a user, it will appear as one row. Each row describes
when the photo was added to the collection and gives the title of the collection.

Here's the structure:

| Field                         | Description |
|-------------------------------|-------------|
| photo_id                      | ID of the Unsplash photo |
| collection_id                 | ID of the Unsplash collection containing the photo |
| collection_title              | Title of the collection containing the photo |
| photo_collected_at            | Timestamp of when the photo was added to the collection |

### 3 - conversions.tsv
The conversions dataset has one row per search conversion. At the moment, it only holds 'downloads' as conversions. The dataset tells you which photo has been downloaded, after which search, from where and by who (anonymously).

Here's the structure:

| Field                         | Description |
|-------------------------------|-------------|
| converted_at                  | Timestamp of the conversion event |
| conversion_type               | Type of conversion ('download' only for now) |
| keyword                       | Keyword that was searched and led to the conversion |
| photo_id                      | Photo ID of the photo that converted |
| anonymous_user_id             | Anonymous device ID |
| conversion_country            | Country code of the device geolocation |


## Fit everything together

You can merge the different datasets through the ID fields, usually the Photo ID (photo_id) field.
This way, you'll be able to cross-reference properties from the photos dataset with data from the keywords
dataset for example.

# Generating the dataset

To keep things flexible, the process is composed of 2 steps:

### 1 - Selection of photos

We want to generate a list of photos (a subset of the photos table) that we want to appear in the dataset.  
That list of photos has to be created in the warehouse, as a standalone table.

Example: All searchable photos

```
CREATE TABLE research.research_dataset_photos_set AS (
  SELECT *
  FROM extension.photos
  WHERE status = 1
    AND searchable_at IS NOT NULL
)
```

We're free to run whatever query to select the photos that will appear in the dataset, we can also include a `LIMIT` clause to limit the number of photos.

### 2 - Run the generation script

We can then run the dataset generation script, passing the name of the photos set table as a parameter.  
The generated dataset will only contain data that concerns the photos that we manually selected.

```
npm run enqueue --
  --script=task:generate-research-dataset
  --key=photo,photographer,exif,ai,search,collections
  --table=research.research_dataset_photos_set
```

Note that `--key` allows to specify what data points we want to expose in the dataset.

## Caveats

A few things to have in mind:

- Private collections aren't exposed
- Collections named 'My first collection' aren't exposed, even if made public
- Landmarks data only shows the one landmark with the highest confidence level in the photo
- ... (more to come, I guess)