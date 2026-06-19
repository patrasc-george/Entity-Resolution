### **Introduction**

The dataset contains records describing various companies along with multiple business-related attributes. Although most records appear distinct, several of them may refer to the same **real-world company**. The objective of this project is to identify and group together records that belong to the same entity.

The **entity resolution pipeline** is structured into the following stages:

1. **Spark Setup** – configuration of a **PySpark environment** to ensure scalability.  
2. **Dataset Inspection and Column Coverage** – exploration of dataset **schema** and evaluation of **data completeness**.  
3. **Selecting Relevant Columns** – retention of only the **relevant columns** for blocking, matching, and validation.  
4. **Data Preprocessing and Normalization** – cleaning and normalization of attributes to enable **consistent comparisons**.  
5. **Candidate Pair Generation** – grouping records into **candidate pairs** based on a common **blocking key** to reduce the number of comparisons.  
6. **Pairwise Matching Score Computation** – calculation of a **similarity score** for each candidate pair.  
7. **Graph-Based Clustering** – filtering high-scoring pairs and **grouping connected records** using a graph-based approach.  
8. **Cluster Analysis and Visual Validation** – inspection of **resulting clusters** and presentation of **statistical distributions**.

### Spark Setup

Although the provided dataset is not very large, **PySpark** was chosen to ensure **scalability** and to support efficient processing for significantly larger datasets. This approach allows the **entity resolution pipeline** to scale in a **distributed environment** if needed.

### Dataset Inspection and Column Coverage

At this stage, we inspect the **dataset schema** to understand its structure and the available attributes. We also analyze the **percentage of non-null values per column** in order to assess **data completeness**. In the following steps, we will primarily focus on columns that both **meaningfully describe the entity** and exhibit a **significant proportion of non-null values**.

```text

DataFrame schema:
root
 |-- company_name: string (nullable = true)
 |-- company_legal_names: string (nullable = true)
 |-- company_commercial_names: string (nullable = true)
 |-- main_country_code: string (nullable = true)
 |-- main_country: string (nullable = true)
 |-- main_region: string (nullable = true)
 |-- main_city_district: string (nullable = true)
 |-- main_city: string (nullable = true)
 |-- main_postcode: string (nullable = true)
 |-- main_street: string (nullable = true)
 |-- main_street_number: string (nullable = true)
 |-- main_latitude: string (nullable = true)
 |-- main_longitude: string (nullable = true)
 |-- main_address_raw_text: string (nullable = true)
 |-- locations: string (nullable = true)
 |-- num_locations: string (nullable = true)
 |-- company_type: string (nullable = true)
 |-- year_founded: string (nullable = true)
 |-- lnk_year_founded: string (nullable = true)
 |-- short_description: string (nullable = true)
 |-- long_description: string (nullable = true)
 |-- business_tags: string (nullable = true)
 |-- business_model: string (nullable = true)
 |-- product_type: string (nullable = true)
 |-- naics_vertical: string (nullable = true)
 |-- naics_2022_primary_code: string (nullable = true)
 |-- naics_2022_primary_label: string (nullable = true)
 |-- naics_2022_secondary_codes: string (nullable = true)
 |-- naics_2022_secondary_labels: string (nullable = true)
 |-- main_business_category: string (nullable = true)
 |-- main_industry: string (nullable = true)
 |-- main_sector: string (nullable = true)
 |-- primary_phone: string (nullable = true)
 |-- phone_numbers: string (nullable = true)
 |-- primary_email: string (nullable = true)
 |-- emails: string (nullable = true)
 |-- other_emails: string (nullable = true)
 |-- website_url: string (nullable = true)
 |-- website_domain: string (nullable = true)
 |-- website_tld: string (nullable = true)
 |-- website_language_code: string (nullable = true)
 |-- facebook_url: string (nullable = true)
 |-- twitter_url: string (nullable = true)
 |-- instagram_url: string (nullable = true)
 |-- linkedin_url: string (nullable = true)
 |-- ios_app_url: string (nullable = true)
 |-- android_app_url: string (nullable = true)
 |-- youtube_url: string (nullable = true)
 |-- tiktok_url: string (nullable = true)
 |-- alexa_rank: string (nullable = true)
 |-- sics_codified_industry: string (nullable = true)
 |-- sics_codified_industry_code: string (nullable = true)
 |-- sics_codified_subsector: string (nullable = true)
 |-- sics_codified_subsector_code: string (nullable = true)
 |-- sics_codified_sector: string (nullable = true)
 |-- sics_codified_sector_code: string (nullable = true)
 |-- sic_codes: string (nullable = true)
 |-- sic_labels: string (nullable = true)
 |-- isic_v4_codes: string (nullable = true)
 |-- isic_v4_labels: string (nullable = true)
 |-- nace_rev2_codes: string (nullable = true)
 |-- nace_rev2_labels: string (nullable = true)
 |-- created_at: string (nullable = true)
 |-- last_updated_at: string (nullable = true)
 |-- website_number_of_pages: string (nullable = true)
 |-- generated_description: string (nullable = true)
 |-- generated_business_tags: string (nullable = true)
 |-- status: string (nullable = true)
 |-- domains: string (nullable = true)
 |-- all_domains: string (nullable = true)
 |-- revenue: string (nullable = true)
 |-- revenue_type: string (nullable = true)
 |-- employee_count: string (nullable = true)
 |-- employee_count_type: string (nullable = true)
 |-- inbound_links_count: string (nullable = true)

DataFrame size:
Rows: 33,446
Columns: 75
```

```text
[Stage 7:============================================>              (3 + 1) / 4]
```

```text
Non-null percentage per column:
status: 100.00%
created_at: 99.87%
last_updated_at: 99.87%
company_name: 97.52%
website_url: 95.36%
website_domain: 95.36%
website_tld: 95.36%
main_country_code: 93.93%
main_country: 93.93%
locations: 93.93%
main_region: 90.03%
main_city: 88.51%
company_commercial_names: 84.08%
main_address_raw_text: 83.66%
main_postcode: 71.22%
primary_phone: 68.17%
phone_numbers: 68.17%
main_street: 59.74%
business_model: 59.19%
product_type: 59.19%
main_business_category: 59.19%
main_industry: 59.19%
main_sector: 59.19%
company_type: 59.01%
generated_description: 58.21%
generated_business_tags: 58.14%
num_locations: 57.14%
short_description: 55.92%
naics_vertical: 54.60%
naics_2022_primary_code: 53.96%
naics_2022_primary_label: 53.96%
sic_codes: 53.96%
sic_labels: 53.96%
isic_v4_codes: 53.96%
isic_v4_labels: 53.96%
nace_rev2_codes: 53.96%
nace_rev2_labels: 53.96%
main_street_number: 50.93%
main_latitude: 50.92%
main_longitude: 50.92%
long_description: 35.07%
domains: 34.66%
all_domains: 34.66%
facebook_url: 33.73%
linkedin_url: 30.50%
business_tags: 27.65%
employee_count: 26.08%
employee_count_type: 26.08%
revenue: 21.61%
revenue_type: 21.61%
instagram_url: 20.97%
company_legal_names: 20.60%
primary_email: 19.46%
website_number_of_pages: 18.39%
inbound_links_count: 18.39%
website_language_code: 18.25%
main_city_district: 17.88%
sics_codified_industry: 16.93%
sics_codified_industry_code: 16.93%
sics_codified_subsector: 16.93%
sics_codified_subsector_code: 16.93%
sics_codified_sector: 16.93%
sics_codified_sector_code: 16.93%
year_founded: 15.03%
emails: 9.91%
twitter_url: 8.24%
lnk_year_founded: 6.40%
youtube_url: 3.41%
other_emails: 1.73%
naics_2022_secondary_codes: 0.73%
naics_2022_secondary_labels: 0.73%
android_app_url: 0.41%
ios_app_url: 0.37%
tiktok_url: 0.00%
alexa_rank: 0.00%
```

### Selecting Relevant Columns

We retain only the columns that are relevant for **blocking**, **matching**, and **final cluster interpretation** in order to reduce noise and improve efficiency.

| Category | Columns | Purpose |
|----------|---------|---------|
| **Blocking Key Columns** | `website_domain`, `primary_email`, `company_name` | Used to derive a unified **`name_core` blocking key** for candidate generation. |
| **Field Columns (Matching Features)** | `website_domain`, `primary_email`, `company_name`, `main_country_code`, `primary_phone`, `year_founded`, `business_tags`, `business_model`, `product_type`, `naics_vertical`, `naics_2022_primary_label`, `naics_2022_secondary_labels`, `main_business_category`, `main_industry`, `main_sector`, `sics_codified_industry`, `sics_codified_subsector`, `sics_codified_sector`, `naics_2022_primary_code`, `naics_2022_secondary_codes`, `sics_codified_industry_code`, `sics_codified_subsector_code`, `sics_codified_sector_code`, `sic_codes`, `isic_v4_codes`, `nace_rev2_codes` | Provide structured **business and industry attributes** used for pairwise similarity scoring. |
| **URL Columns (Social Handles)** | `youtube_url`, `facebook_url`, `linkedin_url`, `instagram_url`, `twitter_url` | Used to extract **social media identifiers (handles)** for additional matching signals. |
| **Description Columns** | `company_name`, `website_domain`, `locations` | Used for interpreting and validating the resulting **clusters** in a human-readable way. |

This structured selection ensures that **blocking is efficient**, **matching is meaningful**, and the final clusters can be **clearly interpreted**.

```text

DataFrame schema after retaining only the relevant columns:
root
 |-- website_domain: string (nullable = true)
 |-- primary_email: string (nullable = true)
 |-- company_name: string (nullable = true)
 |-- locations: string (nullable = true)
 |-- main_country_code: string (nullable = true)
 |-- primary_phone: string (nullable = true)
 |-- year_founded: string (nullable = true)
 |-- business_tags: string (nullable = true)
 |-- business_model: string (nullable = true)
 |-- product_type: string (nullable = true)
 |-- naics_vertical: string (nullable = true)
 |-- naics_2022_primary_label: string (nullable = true)
 |-- naics_2022_secondary_labels: string (nullable = true)
 |-- main_business_category: string (nullable = true)
 |-- main_industry: string (nullable = true)
 |-- main_sector: string (nullable = true)
 |-- sics_codified_industry: string (nullable = true)
 |-- sics_codified_subsector: string (nullable = true)
 |-- sics_codified_sector: string (nullable = true)
 |-- naics_2022_primary_code: string (nullable = true)
 |-- naics_2022_secondary_codes: string (nullable = true)
 |-- sics_codified_industry_code: string (nullable = true)
 |-- sics_codified_subsector_code: string (nullable = true)
 |-- sics_codified_sector_code: string (nullable = true)
 |-- sic_codes: string (nullable = true)
 |-- isic_v4_codes: string (nullable = true)
 |-- nace_rev2_codes: string (nullable = true)
 |-- youtube_url: string (nullable = true)
 |-- facebook_url: string (nullable = true)
 |-- linkedin_url: string (nullable = true)
 |-- instagram_url: string (nullable = true)
 |-- twitter_url: string (nullable = true)

DataFrame size:
Rows: 33,446
Columns: 32
```

### Data Preprocessing and Normalization

In the following section, we detail the **data preprocessing and normalization steps**. For clarity, we will use the below illustrated example record, which is extracted directly from the provided dataset:

| Attribute | Value |
|-----------|-------|
| company_name | 180 Chicago Church |
| main_country_code | US |
| year_founded |  |
| business_tags | Downtown Campus \| Disciples Of Christ \| Christian Schools \| Spiritual Living |
| business_model | Non Profit |
| product_type | Non Profit |
| naics_vertical | Churches & Religious Organizations |
| naics_2022_primary_code | 813110 |
| naics_2022_primary_label | Religious Organizations |
| naics_2022_secondary_codes |  |
| naics_2022_secondary_labels |  |
| main_business_category | Churches & Religious Organizations |
| main_industry | Churches |
| main_sector | Non Profit |
| primary_phone | 18888260851 |
| primary_email | info@180chicago.church |
| website_domain | 180chicago.church |
| facebook_url | https://www.facebook.com/180Chicago.Church/ |
| twitter_url |  |
| instagram_url | https://www.instagram.com/180chicagochurch/ |
| linkedin_url |  |
| youtube_url | https://www.youtube.com/c/180ChicagoChurch |
| sics_codified_industry |  |
| sics_codified_industry_code |  |
| sics_codified_subsector |  |
| sics_codified_subsector_code |  |
| sics_codified_sector |  |
| sics_codified_sector_code |  |
| sic_codes | 8661 |
| sic_labels | Religious Organizations |
| isic_v4_codes | 9491 |
| isic_v4_labels | Activities of religious organizations |
| nace_rev2_codes | 94.91 |
| nace_rev2_labels | Activities of religious organisations |
| locations | US, United States, Illinois, Chicago, 60605, South State Street, 1550, 41.86057811111111, -87.62744066666667 \| US, United States, Illinois, Elk Grove Village, 60007, Wellington Avenue, 1000, 42.002014200000005, -88.01056649653873 \| US, United States, Illinois, Mount Prospect, 60056, South Emerson Street, 119, 42.064154629992636, -87.93528620134452 |

For **data preprocessing and normalization**, the following steps are applied:

1. **Adding a unique identifier**

   The first step is to assign a monotonically increasing unique identifier (`record_id`) to each record. This identifier will later be used for **candidate pair generation** and **graph-based clustering**.

2. **Deriving the blocking key (`name_core`)**

To enable efficient **blocking**, we derive a unified blocking key from three possible sources, in the following strict order.

a. **From `website_domain`**

We extract the text appearing before the first dot (`.`).

| Source Column  | Original Value        | Derived `name_core` |
|---------------|----------------------|---------------------|
| website_domain | 180chicago.church    | 180chicago |

b. **From `primary_email`**

We extract the text between the first `@` and the first dot (`.`), provided that the email domain is not part of a common public-domain blacklist (e.g., gmail.com, yahoo.com).

| Source Column | Original Value | Derived `name_core` |
|---------------|----------------|---------------------|
| primary_email | info@180chicago.church | 180chicago |

c. **From `company_name`**

We normalize the company name by converting text to lowercase and keeping only alphanumeric characters, removing spaces and special characters.

| Source Column | Original Value | Derived `name_core` |
|---------------|----------------|---------------------|
| company_name | 180 Chicago Church | 180chicagochurch |

The selection priority strictly follows the order above. While company names may vary across records for the same entity, the **website or email domain is typically more stable**, which is why they are preferred for blocking.

3. **Field normalization for entity matching**

To perform entity matching, we rely on multiple descriptive attributes that characterize a company. All selected fields are normalized by converting text to lowercase and keeping only **alphanumeric characters**. Separators, punctuation, and special characters are removed to reduce noise that could otherwise lead to **false negative matches**.

| Source Column | Original Value | Normalized Value |
|---------------|----------------|------------------|
| company_name | 180 Chicago Church | 180chicagochurch |
| main_country_code | US | us |
| business_tags | Downtown Campus \| Disciples Of Christ \| Christian Schools \| Spiritual Living | downtowncampusdisciplesofchristchristianschoolsspiritualliving |
| business_model | Non Profit | nonprofit |
| product_type | Non Profit | nonprofit |
| naics_vertical | Churches & Religious Organizations | churchesreligiousorganizations |
| naics_2022_primary_code | 813110 | 813110 |
| naics_2022_primary_label | Religious Organizations | religiousorganizations |
| main_industry | Churches | churches |
| main_sector | Non Profit | nonprofit |
| primary_phone | +18888260851 | 18888260851 |
| primary_email | info@180chicago.church | info180chicagochurch |
| website_domain | 180chicago.church | 180chicagochurch |
| sic_codes | 8661 | 8661 |
| isic_v4_codes | 9491 | 9491 |
| nace_rev2_codes | 94.91 | 9491 |

4. **Social media handle extraction**

From the social media URLs, we extract the **unique identifier (handle)** used by the entity on each platform. The handle is defined as the text appearing after the last `/` in the URL (after removing trailing slashes and query parameters).

| Source Column | Original Value | Extracted Handle |
|---------------|----------------|------------------|
| facebook_url | https://www.facebook.com/180Chicago.Church/ | 180chicago.church |
| twitter_url |  |  |
| instagram_url | https://www.instagram.com/180chicagochurch/ | 180chicagochurch |
| linkedin_url |  |  |
| youtube_url | https://www.youtube.com/c/180ChicagoChurch | 180chicagochurch |

```text

Columns name_core (derived from website_domain, primary_email, company_name):
--------- RECORD 1/10 ---------
record_id                     : 8589934592
website_domain                : owensliquors.com
company_name                  : Owens Liquors
name_core                     : owensliquors
--------- RECORD 2/10 ---------
record_id                     : 8589934593
website_domain                : clubtarneit.com.au
company_name                  : Club Tarneit
name_core                     : clubtarneit
--------- RECORD 3/10 ---------
record_id                     : 8589934594
website_domain                : aaaauto.cz
company_name                  : AAA Auto Otrokovice Zlín
name_core                     : aaaauto
--------- RECORD 4/10 ---------
record_id                     : 8589934595
company_name                  : Gisinger GmbH
name_core                     : gisingergmbh
--------- RECORD 5/10 ---------
record_id                     : 8589934596
company_name                  : Kasana Life
name_core                     : kasanalife
--------- RECORD 6/10 ---------
record_id                     : 8589934597
website_domain                : bammakeupstudio.com.au
company_name                  : BAM BROW & MAKEUP STUDIO
name_core                     : bammakeupstudio
--------- RECORD 7/10 ---------
record_id                     : 8589934598
website_domain                : tescoma.hu
company_name                  : Tescoma
name_core                     : tescoma
--------- RECORD 8/10 ---------
record_id                     : 8589934599
website_domain                : happyweddings.com
company_name                  : Happyweddings | No.1 Matrimony Trivandrum Kerala
name_core                     : happyweddings
--------- RECORD 9/10 ---------
record_id                     : 8589934600
website_domain                : dentalplanet.co.nz
company_name                  : Dental Planet Manukau
name_core                     : dentalplanet
--------- RECORD 10/10 ---------
record_id                     : 8589934601
website_domain                : kdrakephoto.com
company_name                  : Drake Design Photography
name_core                     : kdrakephoto

Columns website_domain_norm, primary_email_norm, company_name_norm, main_country_code_norm, primary_phone_norm, year_founded_norm, business_tags_norm, business_model_norm, product_type_norm, naics_vertical_norm, naics_2022_primary_label_norm, naics_2022_secondary_labels_norm, main_business_category_norm, main_industry_norm, main_sector_norm, sics_codified_industry_norm, sics_codified_subsector_norm, sics_codified_sector_norm, naics_2022_primary_code_norm, naics_2022_secondary_codes_norm, sics_codified_industry_code_norm, sics_codified_subsector_code_norm, sics_codified_sector_code_norm, sic_codes_norm, isic_v4_codes_norm, nace_rev2_codes_norm (derived from website_domain, primary_email, company_name, main_country_code, primary_phone, year_founded, business_tags, business_model, product_type, naics_vertical, naics_2022_primary_label, naics_2022_secondary_labels, main_business_category, main_industry, main_sector, sics_codified_industry, sics_codified_subsector, sics_codified_sector, naics_2022_primary_code, naics_2022_secondary_codes, sics_codified_industry_code, sics_codified_subsector_code, sics_codified_sector_code, sic_codes, isic_v4_codes, nace_rev2_codes):
--------- RECORD 1/10 ---------
record_id                     : 8589934592
website_domain                : owensliquors.com
website_domain_norm           : owensliquorscom
company_name                  : Owens Liquors
company_name_norm             : owensliquors
main_country_code             : US
main_country_code_norm        : us
primary_phone                 : +18433140354
primary_phone_norm            : 18433140354
business_model                : Retail
business_model_norm           : retail
product_type                  : Nondurable Products
product_type_norm             : nondurableproducts
naics_vertical                : Beer & Liquor Stores
naics_vertical_norm           : beerliquorstores
naics_2022_primary_label      : Beer, Wine, and Liquor Retailers
naics_2022_primary_label_norm : beerwineandliquorretailers
main_business_category        : Beer & Liquor Stores
main_business_category_norm   : beerliquorstores
main_industry                 : Beverages
main_industry_norm            : beverages
main_sector                   : Food & Beverages
main_sector_norm              : foodbeverages
naics_2022_primary_code       : 445320
naics_2022_primary_code_norm  : 445320
sic_codes                     : 5411 | 5431 | 5142 | 5961 | 5921 | 5961 | 5181 | 5182 | 5961 | 5963 | 5421
sic_codes_norm                : 54115431514259615921596151815182596159635421
isic_v4_codes                 : 4722 | 4781 | 4791 | 4799
isic_v4_codes_norm            : 4722478147914799
nace_rev2_codes               : 47.91 | 47.25 | 47.81 | 47.99
nace_rev2_codes_norm          : 4791472547814799
--------- RECORD 2/10 ---------
record_id                     : 8589934593
website_domain                : clubtarneit.com.au
website_domain_norm           : clubtarneitcomau
company_name                  : Club Tarneit
company_name_norm             : clubtarneit
main_country_code             : AU
main_country_code_norm        : au
business_tags                 : Events & Service
business_tags_norm            : eventsservice
business_model                : Services
business_model_norm           : services
product_type                  : Consumer Services
product_type_norm             : consumerservices
main_business_category        : Dance Clubs & Night Clubs
main_business_category_norm   : danceclubsnightclubs
main_industry                 : Pubs & Bars
main_industry_norm            : pubsbars
main_sector                   : Accommodation & Food Services
main_sector_norm              : accommodationfoodservices
--------- RECORD 3/10 ---------
record_id                     : 8589934594
website_domain                : aaaauto.cz
website_domain_norm           : aaaautocz
company_name                  : AAA Auto Otrokovice Zlín
company_name_norm             : aaaautootrokovicezlín
main_country_code             : CZ
main_country_code_norm        : cz
primary_phone                 : +420800400450
primary_phone_norm            : 420800400450
business_model                : Retail
business_model_norm           : retail
product_type                  : Durable Products
product_type_norm             : durableproducts
naics_vertical                : Automobile Dealers & Manufacturers
naics_vertical_norm           : automobiledealersmanufacturers
naics_2022_primary_label      : Used Car Dealers
naics_2022_primary_label_norm : usedcardealers
main_business_category        : Automobile Dealers & Manufacturers
main_business_category_norm   : automobiledealersmanufacturers
main_industry                 : Automobile Dealers & Manufacturers
main_industry_norm            : automobiledealersmanufacturers
main_sector                   : Automotive
main_sector_norm              : automotive
naics_2022_primary_code       : 441120
naics_2022_primary_code_norm  : 441120
sic_codes                     : 5521
sic_codes_norm                : 5521
isic_v4_codes                 : 4510
isic_v4_codes_norm            : 4510
nace_rev2_codes               : 45.11 | 45.19
nace_rev2_codes_norm          : 45114519
--------- RECORD 4/10 ---------
record_id                     : 8589934595
company_name                  : Gisinger GmbH
company_name_norm             : gisingergmbh
main_country_code             : DE
main_country_code_norm        : de
--------- RECORD 5/10 ---------
record_id                     : 8589934596
company_name                  : Kasana Life
company_name_norm             : kasanalife
main_country_code             : US
main_country_code_norm        : us
primary_phone                 : +19174887460
primary_phone_norm            : 19174887460
--------- RECORD 6/10 ---------
record_id                     : 8589934597
website_domain                : bammakeupstudio.com.au
website_domain_norm           : bammakeupstudiocomau
company_name                  : BAM BROW & MAKEUP STUDIO
company_name_norm             : bambrowmakeupstudio
main_country_code             : AU
main_country_code_norm        : au
--------- RECORD 7/10 ---------
record_id                     : 8589934598
website_domain                : tescoma.hu
website_domain_norm           : tescomahu
company_name                  : Tescoma
company_name_norm             : tescoma
main_country_code             : HU
main_country_code_norm        : hu
primary_phone                 : +3617003731
primary_phone_norm            : 3617003731
business_model                : Retail
business_model_norm           : retail
product_type                  : Durable Products
product_type_norm             : durableproducts
naics_vertical                : Kitchen Furniture & Equipment
naics_vertical_norm           : kitchenfurnitureequipment
naics_2022_primary_label      : All Other Home Furnishings Retailers
naics_2022_primary_label_norm : allotherhomefurnishingsretailers
main_business_category        : Kitchen Furniture & Equipment
main_business_category_norm   : kitchenfurnitureequipment
main_industry                 : Furniture
main_industry_norm            : furniture
main_sector                   : Home Products
main_sector_norm              : homeproducts
naics_2022_primary_code       : 449129
naics_2022_primary_code_norm  : 449129
sic_codes                     : 7699 | 5142 | 5961 | 5719 | 5963 | 5431 | 5411 | 5421 | 5961 | 5961
sic_codes_norm                : 7699514259615719596354315411542159615961
isic_v4_codes                 : 4789 | 4791 | 4799 | 4773 | 4759
isic_v4_codes_norm            : 47894791479947734759
nace_rev2_codes               : 47.89 | 47.59 | 47.54 | 47.76 | 47.77 | 47.78 | 47.99 | 47.91
nace_rev2_codes_norm          : 47894759475447764777477847994791
--------- RECORD 8/10 ---------
record_id                     : 8589934599
website_domain                : happyweddings.com
website_domain_norm           : happyweddingscom
company_name                  : Happyweddings | No.1 Matrimony Trivandrum Kerala
company_name_norm             : happyweddingsno1matrimonytrivandrumkerala
main_country_code             : IN
main_country_code_norm        : in
--------- RECORD 9/10 ---------
record_id                     : 8589934600
website_domain                : dentalplanet.co.nz
website_domain_norm           : dentalplanetconz
company_name                  : Dental Planet Manukau
company_name_norm             : dentalplanetmanukau
main_country_code             : NZ
main_country_code_norm        : nz
primary_phone                 : +648002622208
primary_phone_norm            : 648002622208
business_model                : Services
business_model_norm           : services
product_type                  : Consumer Services
product_type_norm             : consumerservices
naics_vertical                : Dentist
naics_vertical_norm           : dentist
naics_2022_primary_label      : Offices of Dentists
naics_2022_primary_label_norm : officesofdentists
main_business_category        : Dentists & Dental Clinics
main_business_category_norm   : dentistsdentalclinics
main_industry                 : Dentists & Dental Clinics
main_industry_norm            : dentistsdentalclinics
main_sector                   : Health Care & Social Assistance
main_sector_norm              : healthcaresocialassistance
naics_2022_primary_code       : 621210
naics_2022_primary_code_norm  : 621210
sic_codes                     : 8021
sic_codes_norm                : 8021
isic_v4_codes                 : 8620
isic_v4_codes_norm            : 8620
nace_rev2_codes               : 86.22 | 86.23 | 86.21
nace_rev2_codes_norm          : 862286238621
--------- RECORD 10/10 ---------
record_id                     : 8589934601
website_domain                : kdrakephoto.com
website_domain_norm           : kdrakephotocom
company_name                  : Drake Design Photography
company_name_norm             : drakedesignphotography
main_country_code             : US
main_country_code_norm        : us
primary_phone                 : +18067979767
primary_phone_norm            : 18067979767
business_model                : Services
business_model_norm           : services
product_type                  : Consumer Services
product_type_norm             : consumerservices
naics_vertical                : Photographers & Photographic Studios
naics_vertical_norm           : photographersphotographicstudios
naics_2022_primary_label      : Photography Studios, Portrait
naics_2022_primary_label_norm : photographystudiosportrait
main_business_category        : Photographers & Photographic Studios
main_business_category_norm   : photographersphotographicstudios
main_industry                 : Photographers & Photographic Studios
main_industry_norm            : photographersphotographicstudios
main_sector                   : Other Consumer Services
main_sector_norm              : otherconsumerservices
naics_2022_primary_code       : 541921
naics_2022_primary_code_norm  : 541921
sic_codes                     : 7221
sic_codes_norm                : 7221
isic_v4_codes                 : 7420
isic_v4_codes_norm            : 7420
nace_rev2_codes               : 74.2
nace_rev2_codes_norm          : 742

Columns youtube_handle, facebook_handle, linkedin_handle, instagram_handle, twitter_handle (derived from youtube_url, facebook_url, linkedin_url, instagram_url, twitter_url):
--------- RECORD 1/10 ---------
record_id                     : 8589934592
--------- RECORD 2/10 ---------
record_id                     : 8589934593
linkedin_url                  : http://www.linkedin.com/company/club-tarneit
linkedin_handle               : club-tarneit
--------- RECORD 3/10 ---------
record_id                     : 8589934594
--------- RECORD 4/10 ---------
record_id                     : 8589934595
linkedin_url                  : http://www.linkedin.com/company/gisinger-gmbh
linkedin_handle               : gisinger-gmbh
--------- RECORD 5/10 ---------
record_id                     : 8589934596
--------- RECORD 6/10 ---------
record_id                     : 8589934597
instagram_url                 : https://www.instagram.com/bammakeupstudio/
instagram_handle              : bammakeupstudio
--------- RECORD 7/10 ---------
record_id                     : 8589934598
--------- RECORD 8/10 ---------
record_id                     : 8589934599
facebook_url                  : https://www.facebook.com/HappyweddingsbyAllyseek/
facebook_handle               : happyweddingsbyallyseek
instagram_url                 : https://www.instagram.com/happyweddings_official/
instagram_handle              : happyweddings_official
--------- RECORD 9/10 ---------
record_id                     : 8589934600
facebook_url                  : https://www.facebook.com/dentalplanetnz/
facebook_handle               : dentalplanetnz
--------- RECORD 10/10 ---------
record_id                     : 8589934601
facebook_url                  : https://www.facebook.com/drakedesignphotography/
facebook_handle               : drakedesignphotography
```

### Candidate Pair Generation

Next, we generate **candidate record pairs** for **entity matching**.  
In theory, generating all possible pairs in a dataset with ~33,000 records would produce:

$$
\frac{N(N - 1)}{2}
=
\frac{33{,}000 \times 32{,}999}{2}
\approx 544{,}483{,}500 \text{ pairs}
$$

This is already extremely expensive, so we apply a **blocking technique**. Using the previously derived **blocking key** `name_core`, we generate pairs only within records that share the same `name_core`, instead of comparing every record with every other record. For example, suppose the following records exist:

| record_id | name_core |
|-----------|-----------|
| 1 | 180chicago |
| 2 | 180chicago |
| 3 | 180chicago |
| 4 | 180chicago |
| 5 | industowers |
| 6 | industowers |
| 7 | industowers |

For the block `name_core = "180chicago"` (4 records), the generated **candidate pairs** are:

- (1, 2), (1, 3), (1, 4)  
- (2, 3), (2, 4)  
- (3, 4)

For the block `name_core = "industowers"` (3 records), the generated **candidate pairs** are:

- (5, 6), (5, 7)  
- (6, 7)

No pairs are generated between the two different **blocks** (e.g., record 1 is never compared with record 5). This strategy avoids comparing records that clearly have nothing in common (e.g., "180 Chicago Church" vs. "Indus Towers"), preventing a **combinatorial explosion** and keeping **computational requirements** manageable. At the end, the **candidates DataFrame** will contain two copies of all previously selected and processed relevant columns, one set for each record in the pair (aliased as `a` and `b`). Each row in this DataFrame corresponds to a single **candidate pair**, enabling direct **field-by-field comparison** between the two entities during the **scoring phase**.

```text

DataFrame schema for candidate record pairs generated within each block:
root
 |-- name_core: string (nullable = true)
 |-- record_id: long (nullable = false)
 |-- website_domain_norm: string (nullable = true)
 |-- primary_email_norm: string (nullable = true)
 |-- company_name_norm: string (nullable = true)
 |-- main_country_code_norm: string (nullable = true)
 |-- primary_phone_norm: string (nullable = true)
 |-- year_founded_norm: string (nullable = true)
 |-- business_tags_norm: string (nullable = true)
 |-- business_model_norm: string (nullable = true)
 |-- product_type_norm: string (nullable = true)
 |-- naics_vertical_norm: string (nullable = true)
 |-- naics_2022_primary_label_norm: string (nullable = true)
 |-- naics_2022_secondary_labels_norm: string (nullable = true)
 |-- main_business_category_norm: string (nullable = true)
 |-- main_industry_norm: string (nullable = true)
 |-- main_sector_norm: string (nullable = true)
 |-- sics_codified_industry_norm: string (nullable = true)
 |-- sics_codified_subsector_norm: string (nullable = true)
 |-- sics_codified_sector_norm: string (nullable = true)
 |-- naics_2022_primary_code_norm: string (nullable = true)
 |-- naics_2022_secondary_codes_norm: string (nullable = true)
 |-- sics_codified_industry_code_norm: string (nullable = true)
 |-- sics_codified_subsector_code_norm: string (nullable = true)
 |-- sics_codified_sector_code_norm: string (nullable = true)
 |-- sic_codes_norm: string (nullable = true)
 |-- isic_v4_codes_norm: string (nullable = true)
 |-- nace_rev2_codes_norm: string (nullable = true)
 |-- youtube_handle: string (nullable = true)
 |-- facebook_handle: string (nullable = true)
 |-- linkedin_handle: string (nullable = true)
 |-- instagram_handle: string (nullable = true)
 |-- twitter_handle: string (nullable = true)
 |-- record_id: long (nullable = false)
 |-- website_domain_norm: string (nullable = true)
 |-- primary_email_norm: string (nullable = true)
 |-- company_name_norm: string (nullable = true)
 |-- main_country_code_norm: string (nullable = true)
 |-- primary_phone_norm: string (nullable = true)
 |-- year_founded_norm: string (nullable = true)
 |-- business_tags_norm: string (nullable = true)
 |-- business_model_norm: string (nullable = true)
 |-- product_type_norm: string (nullable = true)
 |-- naics_vertical_norm: string (nullable = true)
 |-- naics_2022_primary_label_norm: string (nullable = true)
 |-- naics_2022_secondary_labels_norm: string (nullable = true)
 |-- main_business_category_norm: string (nullable = true)
 |-- main_industry_norm: string (nullable = true)
 |-- main_sector_norm: string (nullable = true)
 |-- sics_codified_industry_norm: string (nullable = true)
 |-- sics_codified_subsector_norm: string (nullable = true)
 |-- sics_codified_sector_norm: string (nullable = true)
 |-- naics_2022_primary_code_norm: string (nullable = true)
 |-- naics_2022_secondary_codes_norm: string (nullable = true)
 |-- sics_codified_industry_code_norm: string (nullable = true)
 |-- sics_codified_subsector_code_norm: string (nullable = true)
 |-- sics_codified_sector_code_norm: string (nullable = true)
 |-- sic_codes_norm: string (nullable = true)
 |-- isic_v4_codes_norm: string (nullable = true)
 |-- nace_rev2_codes_norm: string (nullable = true)
 |-- youtube_handle: string (nullable = true)
 |-- facebook_handle: string (nullable = true)
 |-- linkedin_handle: string (nullable = true)
 |-- instagram_handle: string (nullable = true)
 |-- twitter_handle: string (nullable = true)
```

```text
DataFrame size:
Rows: 111,022
Columns: 65
```

### Pairwise Matching Score Computation

Next, we compute a **matching score** between **candidate record pairs**. For each pair, every column is compared only if both records contain **non-null values**. If the values in the same column are **identical**, one point is added to the total **matching score**.

For illustration purposes, we will continue using the previously defined record and we will assume that it is paired with the following record:

| Column | Record 1 | Record 2 |
|--------|----------|----------|
| record_id | 1 | 2 |
| name_core | 180chicago | 180chicago |
| company_name_norm | 180chicagochurch | 180chicagochurch |
| main_country_code_norm | us | us |
| year_founded_norm |  |  |
| business_tags_norm | downtowncampusdisciplesofchristchristianschoolsspiritualliving | downtowncampusdisciplesofchristchristianschoolsspiritualliving |
| business_model_norm | nonprofit | nonprofit |
| product_type_norm | nonprofit | nonprofit |
| naics_vertical_norm | churchesreligiousorganizations | churchesreligiousorganizations |
| naics_2022_primary_code_norm | 813110 | 813110 |
| naics_2022_primary_label_norm | religiousorganizations | religiousorganizations |
| naics_2022_secondary_codes_norm |  |  |
| naics_2022_secondary_labels_norm |  |  |
| main_business_category_norm | churchesreligiousorganizations | churches |
| main_industry_norm | churches | churches |
| main_sector_norm | nonprofit | nonprofit |
| primary_phone_norm | 18888260851 | 18888371952 |
| primary_email_norm | info180chicagochurch | info180chicagochurch |
| website_domain_norm | 180chicagochurch | 180chicagochurch |
| sics_codified_industry_norm |  |  |
| sics_codified_industry_code_norm |  |  |
| sics_codified_subsector_norm |  |  |
| sics_codified_subsector_code_norm |  |  |
| sics_codified_sector_norm |  |  |
| sics_codified_sector_code_norm |  |  |
| sic_codes_norm | 8661 |  |
| isic_v4_codes_norm | 9491 |  |
| nace_rev2_codes_norm | 9491 | 9491 |
| facebook_handle | 180chicago.church | 180chicago.church |
| twitter_handle |  | 180chicagochurch |
| instagram_handle | 180chicagochurch | 180chicagochurch |
| linkedin_handle |  | 180chicagochurch |
| youtube_handle | 180chicagochurch | 180chicagochurch |

Thus, the values in each column are evaluated sequentially and handled as follows:

| Columns | Stage Description |
|---------|------------------|
| `year_founded_norm`, `naics_2022_secondary_codes_norm`, `naics_2022_secondary_labels_norm`, `sics_codified_industry_norm`, `sics_codified_industry_code_norm`, `sics_codified_subsector_norm`, `sics_codified_subsector_code_norm`, `sics_codified_sector_norm`, `sics_codified_sector_code_norm` | The fields cannot be compared because both records contain **null values**. |
| `twitter_handle`, `linkedin_handle` | The fields cannot be compared because record `1` contains **null values**. |
| `sic_codes_norm`, `isic_v4_codes_norm` | The fields cannot be compared because record `2` contains **null values**. |
| `name_core`, `company_name_norm`, `main_country_code_norm`, `business_tags_norm`, `business_model_norm`, `product_type_norm`, `naics_vertical_norm`, `naics_2022_primary_code_norm`, `naics_2022_primary_label_norm`, `main_industry_norm`, `main_sector_norm`, `primary_email_norm`, `website_domain_norm`, `nace_rev2_codes_norm`, `facebook_handle`, `instagram_handle`, `youtube_handle` | The values in these fields **match exactly**, therefore one point is awarded for each. |
| `main_business_category_norm`, `primary_phone_norm` | The values in these fields do not match, therefore **no point is awarded**. |

In total, out of 32 columns used for **matching**:

- 9 columns contain **null values** on both sides,
- 2 columns contain **null values** in record 1,
- 2 columns contain **null values** in record 2.  

These columns are excluded from the **score calculation**.

This leaves 19 **comparable** (non-null on both sides) columns, of which:
- 17 columns **match exactly**,
- 2 columns contain **different values**.

Therefore, the final **matching score** is calculated as:

$$
\frac{\text{number of matching columns}}{\text{number of comparable columns}}
=
\frac{17}{19}
=
0.89
$$

The process continues iteratively for each generated pair, and the computed score is assigned to every **candidate pair**. Next, we can observe a chart showing the **distribution of matching scores** across all candidate pairs, as well as a chart illustrating the **distribution of the number of valid (non-null) columns** used for matching across pairs.

```text

Candidate pairs and their scores:
---------- PAIR 1/10 ----------
16 columns were compared and match at 100.0%.
a.record_id                     : 8589935553
b.record_id                     : 8589936260
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us
a.primary_phone_norm            : 18888260851
b.primary_phone_norm            : 18888260851
a.business_model_norm           : nonprofit
b.business_model_norm           : nonprofit
a.product_type_norm             : nonprofit
b.product_type_norm             : nonprofit
a.naics_vertical_norm           : churchesreligiousorganizations
b.naics_vertical_norm           : churchesreligiousorganizations
a.naics_2022_primary_label_norm : religiousorganizations
b.naics_2022_primary_label_norm : religiousorganizations
a.main_business_category_norm   : churchesreligiousorganizations
b.main_business_category_norm   : churchesreligiousorganizations
a.main_industry_norm            : churches
b.main_industry_norm            : churches
a.main_sector_norm              : nonprofit
b.main_sector_norm              : nonprofit
a.naics_2022_primary_code_norm  : 813110
b.naics_2022_primary_code_norm  : 813110
a.sic_codes_norm                : 8661
b.sic_codes_norm                : 8661
a.isic_v4_codes_norm            : 9491
b.isic_v4_codes_norm            : 9491
a.nace_rev2_codes_norm          : 9491
b.nace_rev2_codes_norm          : 9491

---------- PAIR 2/10 ----------
18 columns were compared and match at 100.0%.
a.record_id                     : 8589935553
b.record_id                     : 8589950710
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us
a.primary_phone_norm            : 18888260851
b.primary_phone_norm            : 18888260851
a.business_model_norm           : nonprofit
b.business_model_norm           : nonprofit
a.product_type_norm             : nonprofit
b.product_type_norm             : nonprofit
a.naics_vertical_norm           : churchesreligiousorganizations
b.naics_vertical_norm           : churchesreligiousorganizations
a.naics_2022_primary_label_norm : religiousorganizations
b.naics_2022_primary_label_norm : religiousorganizations
a.main_business_category_norm   : churchesreligiousorganizations
b.main_business_category_norm   : churchesreligiousorganizations
a.main_industry_norm            : churches
b.main_industry_norm            : churches
a.main_sector_norm              : nonprofit
b.main_sector_norm              : nonprofit
a.naics_2022_primary_code_norm  : 813110
b.naics_2022_primary_code_norm  : 813110
a.sic_codes_norm                : 8661
b.sic_codes_norm                : 8661
a.isic_v4_codes_norm            : 9491
b.isic_v4_codes_norm            : 9491
a.nace_rev2_codes_norm          : 9491
b.nace_rev2_codes_norm          : 9491
a.facebook_handle               : 180chicago.church
b.facebook_handle               : 180chicago.church
a.instagram_handle              : 180chicagochurch
b.instagram_handle              : 180chicagochurch

---------- PAIR 3/10 ----------
5 columns were compared and match at 100.0%.
a.record_id                     : 8589935553
b.record_id                     : 8589964140
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us
a.instagram_handle              : 180chicagochurch
b.instagram_handle              : 180chicagochurch

---------- PAIR 4/10 ----------
5 columns were compared and match at 100.0%.
a.record_id                     : 8589935553
b.record_id                     : 8589967654
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us
a.instagram_handle              : 180chicagochurch
b.instagram_handle              : 180chicagochurch

---------- PAIR 5/10 ----------
16 columns were compared and match at 100.0%.
a.record_id                     : 8589936260
b.record_id                     : 8589950710
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us
a.primary_phone_norm            : 18888260851
b.primary_phone_norm            : 18888260851
a.business_model_norm           : nonprofit
b.business_model_norm           : nonprofit
a.product_type_norm             : nonprofit
b.product_type_norm             : nonprofit
a.naics_vertical_norm           : churchesreligiousorganizations
b.naics_vertical_norm           : churchesreligiousorganizations
a.naics_2022_primary_label_norm : religiousorganizations
b.naics_2022_primary_label_norm : religiousorganizations
a.main_business_category_norm   : churchesreligiousorganizations
b.main_business_category_norm   : churchesreligiousorganizations
a.main_industry_norm            : churches
b.main_industry_norm            : churches
a.main_sector_norm              : nonprofit
b.main_sector_norm              : nonprofit
a.naics_2022_primary_code_norm  : 813110
b.naics_2022_primary_code_norm  : 813110
a.sic_codes_norm                : 8661
b.sic_codes_norm                : 8661
a.isic_v4_codes_norm            : 9491
b.isic_v4_codes_norm            : 9491
a.nace_rev2_codes_norm          : 9491
b.nace_rev2_codes_norm          : 9491

---------- PAIR 6/10 ----------
4 columns were compared and match at 100.0%.
a.record_id                     : 8589936260
b.record_id                     : 8589964140
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us

---------- PAIR 7/10 ----------
4 columns were compared and match at 100.0%.
a.record_id                     : 8589936260
b.record_id                     : 8589967654
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us

---------- PAIR 8/10 ----------
5 columns were compared and match at 100.0%.
a.record_id                     : 8589950710
b.record_id                     : 8589964140
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us
a.instagram_handle              : 180chicagochurch
b.instagram_handle              : 180chicagochurch

---------- PAIR 9/10 ----------
5 columns were compared and match at 100.0%.
a.record_id                     : 8589950710
b.record_id                     : 8589967654
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us
a.instagram_handle              : 180chicagochurch
b.instagram_handle              : 180chicagochurch

---------- PAIR 10/10 ----------
5 columns were compared and match at 100.0%.
a.record_id                     : 8589964140
b.record_id                     : 8589967654
a.name_core                     : 180chicago
b.name_core                     : 180chicago
a.website_domain_norm           : 180chicagochurch
b.website_domain_norm           : 180chicagochurch
a.company_name_norm             : 180chicagochurch
b.company_name_norm             : 180chicagochurch
a.main_country_code_norm        : us
b.main_country_code_norm        : us
a.instagram_handle              : 180chicagochurch
b.instagram_handle              : 180chicagochurch
```

![Output 1](outputs/output_001.png)

![Output 2](outputs/output_002.png)

### Graph-Based Clustering

In the **clustering stage**, the objective is to group all previously defined pairs into clusters representing the same **real-world entity**.  

To illustrate this process, we will use the same example records that were introduced during the **candidate pair generation** step:

| record_id | name_core |
|-----------|-----------|
| 1 | 180chicago |
| 2 | 180chicago |
| 3 | 180chicago |
| 4 | 180chicago |
| 5 | industowers |
| 6 | industowers |
| 7 | industowers |

For each generated pair of records, we will assume an associated **similarity score** assigned for illustration purposes:

| src | dst | score |
|-----|-----|-------|
| 1 | 2 | 0.91 |
| 1 | 3 | 0.77 |
| 1 | 4 | 0.63 |
| 2 | 3 | 0.69 |
| 2 | 4 | 0.80 |
| 3 | 4 | 0.90 |
| 5 | 6 | 0.94 |
| 5 | 7 | 0.79 |
| 6 | 7 | 0.92 |

Once all candidate pairs have been scored, we retain only those whose similarity score exceeds the chosen **threshold**. Using a threshold of **0.7**, the following edges remain valid in the defined example:

| src | dst | score |
|-----|-----|-------|
| 1 | 2 | 0.91 |
| 1 | 3 | 0.77 |
| 2 | 4 | 0.80 |
| 3 | 4 | 0.90 |
| 5 | 6 | 0.94 |
| 5 | 7 | 0.73 |
| 6 | 7 | 0.92 |

At the end, by defining a **graph**, we can cluster records using **connected components**. Each valid pair is treated as an **edge** in the graph, where the two records are represented as **nodes** (`src` and `dst`), and the edge indicates a strong similarity relationship. All records that are connected through one or more high-scoring edges, either directly or transitively, are assigned to the same `cluster_id`. Thus, in our example, **two clusters** will be formed: the first cluster consisting of nodes **1, 2, 3, and 4**, and the second cluster consisting of nodes **5, 6, and 7**.

```text
/usr/local/lib/python3.12/dist-packages/pyspark/sql/dataframe.py:168: UserWarning: DataFrame.sql_ctx is an internal property, and will be removed in future releases. Use DataFrame.sparkSession instead.
  warnings.warn(
/usr/local/lib/python3.12/dist-packages/pyspark/sql/dataframe.py:147: UserWarning: DataFrame constructor is internal. Do not directly use it.
  warnings.warn("DataFrame constructor is internal. Do not directly use it.")
```

### Cluster Analysis and Visual Validation

Finally, to better understand the outcome of the **clustering stage**, the following calculations and summaries are displayed:

- Total number of records **assigned to a cluster**  
- Total number of records **remaining unclustered**  
- Total number of **unique clusters**  
- **Distribution of cluster sizes** (number of records per cluster)  
- **Top clusters** with the largest number of records  
- **Sample records** for each top cluster using `company_name`, `website_domain`, and `locations` (attributes extracted directly from the raw dataset)

```text
Records with an associated cluster: 30,391
Records without an associated cluster: 3,055
Total number of unique clusters: 5,505
```

![Output 3](outputs/output_003.png)

```text
TOP CLUSTER SIZES:
```

```text
+----------+--------------------+----+
|id        |name_core           |size|
+----------+--------------------+----+
|8589935423|equmeniakyrkan      |56  |
|8589934863|industowers         |45  |
|8589934842|saharvosk           |44  |
|8589935033|meijimura           |42  |
|8589935500|airamanah           |40  |
|8589935348|chatime             |39  |
|8589935697|newcare             |39  |
|8589934952|recommendedbyroberto|38  |
|8589935024|freshburger         |37  |
|8589935358|recovera            |37  |
+----------+--------------------+----+



CLUSTER 1/10 | ID: 8589935423 | SIZE: 56 | NAME_CORE: equmeniakyrkan
```

```text
------- RECORD 1 -------
company_name             : Töreboda Missionsförsamling
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Töreboda, 545 30, Storgatan, 11, 58.7067869, 14.123964899999997
------- RECORD 2 -------
company_name             : Solvi Smu-Gård
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Dalarna County, Leksand, 793 31, Fiskgatan, 5, 60.7319378, 14.993299
------- RECORD 3 -------
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Värmland County, Karlstads kommun, 65671, Skattkärrsvägen, , 59.411216735839844, 13.667804718017578 | SE, Sweden, Värmland County, Karlstads kommun, 655 60, Stationsgatan, 2, 59.60112762451172, 13.723127365112305 | SE, Sweden, Västra Götaland County, Kungälvs kommun, 442 39, Utmarksvägen, 8\, 9, 57.872047424316406, 11.964469909667969 | SE, Sweden, Västra Götaland County, Lilla Edets kommun, 46330, Skyttegatan, , 58.14048385620117, 12.130006790161133 | SE, Sweden, Västra Götaland County, Dals-Eds kommun, 668 30, Storgatan, , 58.912498474121094, 11.92474365234375 | SE, Sweden, Västra Götaland County, Ale kommun, 44556, Egnahemsvägen, , 57.82814025878906, 12.020736694335938 | SE, Sweden, Stockholm County, Stockholms kommun, 167 51, Gustavslundsvägen, 18, 59.33252716064453, 17.983049392700195 | SE, Sweden, Stockholm County, Södertälje kommun, 15134, Dalgatan, 37, 59.19932556152344, 17.61591148376465 | SE, Sweden, Dalarna County, Ludvika kommun, 771 30, Södra Järnvägsgatan, , 60.14601135253906, 15.192225456237793 | SE, Sweden, Kronoberg County, Alvesta kommun, 34234, Bigatan, , 56.90263748168945, 14.559301376342773
------- RECORD 4 -------
company_name             : Skänninge Missionsförsamling
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Östergötland County, Skänninge, 596 31, Järntorgsgatan, 6 b, 58.396605600000015, 15.0870825
------- RECORD 5 -------
company_name             : Equmeniakyrkan Timmele
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Ulricehamns kommun, 523 72, Gårdavägen, , 57.860383299999995, 13.4271703
------- RECORD 6 -------
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.67497253417969, 14.520858764648438
------- RECORD 7 -------
company_name             : Equmeniakyrkan Skärplinge
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Uppsala County, Tierps kommun, , Kyrkvägen, 5, 60.4755196, 17.7661591
------- RECORD 8 -------
company_name             : Equmeniaförsamlingarna i Gränna Visingsö och Örserum
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 9 -------
company_name             : Equmeniakyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Stockholm County, Stockholm, 167 51, Gustavslundsvägen, 18, 59.332570499999996, 17.9830899
------- RECORD 10 -------
company_name             : Equmeniakyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Jönköping County, Gränna, 563 31, Brahegatan, 54, 58.023860299999996, 14.4690066
------- RECORD 11 -------
company_name             : Equmeniakyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Värmland County, Årjängs kommun, 670 10, Sveavägen, 17, 59.508732699999996, 11.838766599999998
------- RECORD 12 -------
company_name             : Equmeniakyrkan Kvillsfors
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Jönköping County, Vetlanda kommun, 574 55, Ringvägen, 2, 57.40558540000001, 15.499880599999996
------- RECORD 13 -------
company_name             : Alstervik
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Kronoberg County, Uppvidinge kommun, , , , 56.9742714, 15.6526808
------- RECORD 14 -------
company_name             : Norrabergskyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Örebro County, Askersund, 696 30, Hospitalsgatan, 11, 58.88125780000001, 14.904496299999996
------- RECORD 15 -------
company_name             : Vinslövs Missionsförsamling Missionskyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Skåne County, Hässleholms kommun, 288 90, Södra Järnvägsgatan, 19, 56.1038904, 13.918734200000001
------- RECORD 16 -------
company_name             : Equmeniakyrkan Lycksele
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västerbotten County, Lycksele, 921 31, , , 64.6, 18.166667
------- RECORD 17 -------
company_name             : Equmeniakyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Gotland County, Gotlands kommun, 624 48, Storgatan, 57, 57.7038592, 18.802793499999996
------- RECORD 18 -------
company_name             : Uniting Church in Sweden
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Stockholm County, Stockholm, 111 29, , , 59.33116074444444, 17.980533530555554
------- RECORD 19 -------
company_name             : Equmeniakyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Stockholm County, Stockholm, 168 76, , , 59.339769, 17.9395411
------- RECORD 20 -------
company_name             : Equmeniakyrkan i Surte
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Ale kommun, 445 57, , , 57.8267626, 12.0161635
------- RECORD 21 -------
company_name             : Equmeniakyrkan Gillstad Järpås
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Lidköpings kommun, 531 97, Gillstadsvägen, , 58.4413305, 12.958739099999997
------- RECORD 22 -------
company_name             : Vara Missionsförsamling
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Vara, 534 34, Skolgatan, 15, 58.26474920000001, 12.952394900000002
------- RECORD 23 -------
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Uppsala County, Uppsala, 754 40, Regins väg, 1, 59.8943389, 17.6378698
------- RECORD 24 -------
company_name             : Uniting Church in Sweden
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.67497253417969, 14.520858764648438
------- RECORD 25 -------
company_name             : Kumla Equmeniaförsamling - Johanneskyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Örebro County, Kumla, 692 30, , , 59.1179611, 15.12055664259079
------- RECORD 26 -------
company_name             : Equmeniakyrkan Smålandsstenar
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 27 -------
company_name             : Equmeniakyrkan Hemse
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 28 -------
company_name             : Equmeniakyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Värmland County, Skattkärr, 656 71, Skattkärrsvägen, 63, 59.411080399999996, 13.667841099999999
------- RECORD 29 -------
company_name             : Equmeniakyrkan Kungälv
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 30 -------
company_name             : Equmeniakyrkan Åmål
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Åmål, 662 31, Kungsgatan, 19, 57.7037611, 11.962493
------- RECORD 31 -------
company_name             : Equmeniakyrkan Molkom
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Värmland County, Karlstads kommun, 655 60, Stationsgatan, 6, 59.6018614, 13.7225168
------- RECORD 32 -------
company_name             : Tångens Missionshus
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Värmland County, Årjäng, 672 94, S 505, , 59.445709300000004, 11.996888
------- RECORD 33 -------
company_name             : Haga Missionshus
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västmanland County, Hallstahammars kommun, , , , 59.5712609, 16.186622299999996
------- RECORD 34 -------
company_name             : Equmeniakyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 35 -------
company_name             : Equmeniakyrkan i Södra Nissadalen
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Halland County, Halmstads kommun, 313 94, , , 56.7716372, 12.9802501
------- RECORD 36 -------
company_name             : Nynäshamns Missionsförsamling
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Stockholm County, Nynäshamn, 149 30, Vikingavägen, 12, 58.90801659999999, 17.9500416
------- RECORD 37 -------
company_name             : Equmeniakyrkan Upplands Väsby
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 38 -------
company_name             : Equmeniakyrkan Vårgårda
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 39 -------
company_name             : Equmeniakyrkan Gullspång
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Gullspångs kommun, 547 31, Gullstensgatan, 30, 58.98292579999999, 14.1026045
------- RECORD 40 -------
company_name             : Sundstabyns Missionshus
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Värmland County, Årjäng, 672 94, , , 59.380854500000005, 11.9972534
------- RECORD 41 -------
company_name             : Fors missionsförsamling och Fors missionskyrka
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Dalarna County, Avesta kommun, 774 97, Garpenbergsvägen, 7, 60.2079758, 16.305668999999998
------- RECORD 42 -------
company_name             : Equmenia Ljurhalla
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 43 -------
company_name             : Equmenia Equmeniakyrkan Öst
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , ,
------- RECORD 44 -------
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 45 -------
company_name             : Bäcken
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Mariestad, 542 46, Ekuddevägen, 27, 58.711992, 13.796551200000001
------- RECORD 46 -------
company_name             : Equmeniakyrkan Loo
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 47 -------
company_name             : Equmeniakyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Kronoberg County, Växjö kommun, 363 42, Sjösåsvägen, 27, 57.0642484, 15.0520475
------- RECORD 48 -------
company_name             : Björksäterkyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, , , , , , 59.6749712, 14.5208584
------- RECORD 49 -------
company_name             : Equmeniakyrkan i Moheda
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Kronoberg County, Alvesta kommun, 342 34, Växjövägen, 5, 57.0011284, 14.5702602
------- RECORD 50 -------
company_name             : Equmeniakyrkan Hjo
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Hjo, 544 50, Torggatan, 3, 58.3010684, 14.2875608
------- RECORD 51 -------
company_name             : Equmeniakyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Stockholm County, Stockholm, 167 14, , , 59.3420399, 17.9803069
------- RECORD 52 -------
company_name             : Equmeniakyrkan Alstermo
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Kronoberg County, Uppvidinge kommun, , Kyrkogatan, 1, 56.9726742, 15.659241599999998
------- RECORD 53 -------
company_name             : Järnforsens missionskyrka
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Kalmar County, Hultsfreds kommun, 577 76, Årenavägen, 2, 57.4090157, 15.6191056
------- RECORD 54 -------
company_name             : Svallidens Missionsförsamling
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Kalmar County, Oskarshamn, 572 51, Södra Vägen, 45, 57.265000699999995, 16.4034877
------- RECORD 55 -------
company_name             : Equmeniakyrkan Bergum
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Skara kommun, 532 72, Olofstorpsvägen, 8a, 58.3855447, 13.5631897
------- RECORD 56 -------
company_name             : Rambergskyrkan
website_domain           : equmeniakyrkan.se
locations                : SE, Sweden, Västra Götaland County, Gothenburg, 411 10, , , 57.7072326, 11.9670171



CLUSTER 2/10 | ID: 8589934863 | SIZE: 45 | NAME_CORE: industowers
```

```text
------- RECORD 1 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Maharashtra, Nagpur, 440001, , , 20.809074900000002, 79.3887304
------- RECORD 2 -------
website_domain           : industowers.com
------- RECORD 3 -------
company_name             : Indus Airtel Tower
website_domain           : industowers.com
locations                : IN, India, Karnataka, Vitla, 574243, , , 12.788402200000002, 75.0946477
------- RECORD 4 -------
company_name             : Indus tower ltd.
website_domain           : industowers.com
locations                : IN, India, Uttar Pradesh, Kasganj, 207123, NH21, , 27.815586200000002, 78.6512786
------- RECORD 5 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Rajasthan, Jaipur, 302001, Subhash Marg, d-34, 26.913330400000003, 75.8039984
------- RECORD 6 -------
company_name             : Indus Towers
website_domain           : industowers.com
------- RECORD 7 -------
company_name             : Indus Towers Limited
website_domain           : industowers.com
locations                : IN, India, Telangana, Secunderabad, 500003, , , 17.4702465, 78.50766879999999
------- RECORD 8 -------
company_name             : Indus Towers Ltd.
website_domain           : industowers.com
locations                : IN, India, Delhi, New Delhi, , Nelson Mandela Marg, 2, 28.539989300000002, 77.154032
------- RECORD 9 -------
company_name             : Airtel Indus Tower Shakdaha
website_domain           : industowers.com
locations                : IN, India, West Bengal, North 24 Parganas, 743273, , , 22.885603600000003, 88.8395322
------- RECORD 10 -------
company_name             : Indus tower office
website_domain           : industowers.com
locations                : IN, India, Uttar Pradesh, Agra, 282004, , , 27.2131828, 77.9804904
------- RECORD 11 -------
company_name             : Indus towers
website_domain           : industowers.com
locations                : IN, India, Kerala, Malappuram, 676307, Chamravattam - Tirur - Kadalundi- Kozhikkode Road, , 10.950813400000001, 75.9121347
------- RECORD 12 -------
company_name             : Indus tower 1
website_domain           : industowers.com
locations                : IN, India, Uttar Pradesh, Kalan, , , , 28.603645699999998, 77.7262379
------- RECORD 13 -------
company_name             : Indus Towers Ltd.
website_domain           : industowers.com
locations                : IN, India, Haryana, Gurugram, 122001, , , 28.4320603, 77.0136775
------- RECORD 14 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Delhi, New Delhi, , , , 28.6138954, 77.2090057
------- RECORD 15 -------
company_name             : Indus Tower
website_domain           : industowers.com
locations                : IN, India, Rajasthan, Mangrol, 325215, , , 25.333268800000006, 76.505673
------- RECORD 16 -------
company_name             : Indus Towers Limited
website_domain           : industowers.com
locations                : IN, India, Telangana, Hyderabad, , , , 17.4106109, 78.3952148
------- RECORD 17 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Maharashtra, Yavatmal, 445001, Nagpur Tuljapur Road, , 20.368971999999996, 78.1123494
------- RECORD 18 -------
company_name             : Indus Towers
website_domain           : industowers.com
------- RECORD 19 -------
company_name             : Indus Towers Ltd.
website_domain           : industowers.com
locations                : IN, India, Tamil Nādu, Coimbatore, 641001, , , 10.9977507, 76.9925114
------- RECORD 20 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Punjab, Sahibzada Ajit Singh Nagar, 160059, , , 30.674234000000006, 76.735095
------- RECORD 21 -------
company_name             : Indus Tower Ltd. Lakshmipur 2 Airtel Network
website_domain           : industowers.com
locations                : IN, India, West Bengal, Lakshmipur, , , , 25.0938256, 87.92388489999999
------- RECORD 22 -------
company_name             : Indus Towers Limited
website_domain           : industowers.com
locations                : IN, India, Maharashtra, Nashik, 422005, Gangapur Road, 3rd floor, 20.007482000000003, 73.7702219
------- RECORD 23 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Haryana, Gurugram, , , , 28.4973098, 77.091798
------- RECORD 24 -------
company_name             : Indus Towers Limited
website_domain           : industowers.com
locations                : IN, India, Haryana, Gurgaon, , , , 28.42826235, 77.00270014657752
------- RECORD 25 -------
company_name             : Indus Towers Limited
website_domain           : industowers.com
locations                : IN, India, Himachal Pradesh, Dharamshala, 176215, , , 32.1956176, 76.3208963
------- RECORD 26 -------
company_name             : Indus Towers Limited
website_domain           : industowers.com
locations                : IN, India, Maharashtra, Pune, 411040, Prince Of Wales Drive, , 18.498967599999997, 73.8957205
------- RECORD 27 -------
company_name             : Indus Tower LTD.
website_domain           : industowers.com
locations                : IN, India, Assam, Guwahati, 781015, , , 26.1372102, 91.80177959999999
------- RECORD 28 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Maharashtra, Mumbai, 400066, Kasturba Gandhi Road, , 19.228612299999998, 72.8623738
------- RECORD 29 -------
company_name             : Indus Towers Gujarat
website_domain           : industowers.com
locations                : IN, India, Gujarat, Ahmedabad, 380015, , , 23.0216238, 72.5797068
------- RECORD 30 -------
company_name             : Indus Tower
website_domain           : industowers.com
locations                : IN, India, Uttar Pradesh, Bareilly, 243006, , , 28.422075500000002, 79.4828951
------- RECORD 31 -------
company_name             : Indus towers
website_domain           : industowers.com
locations                : IN, India, Rajasthan, Bakhrana, , , , 27.792506699999997, 76.147945
------- RECORD 32 -------
company_name             : Indus Tower Limited
website_domain           : industowers.com
locations                : IN, India, West Bengal, West Midnapore, 721242, , , 22.8163902, 87.5988876
------- RECORD 33 -------
company_name             : Indus Towers Limited
website_domain           : industowers.com
locations                : IN, India, Madhya Pradesh, Indore, 452001, , , 22.7536134, 75.89698179999999
------- RECORD 34 -------
company_name             : Indus Tower
website_domain           : industowers.com
locations                : IN, India, Karnataka, Bengaluru, 562157, , , 13.165296599999998, 77.6499532
------- RECORD 35 -------
website_domain           : industowers.com
locations                : IN, India, Uttar Pradesh, Chinhat, 226010, , , 26.868221282958984, 81.00927734375 | IN, India, Uttar Pradesh, Noida, 201301, Nilgiri play area, , 28.5827697, 77.3629849 | IN, India, Uttar Pradesh, Lucknow, 226010, No Name, , 26.854475021362305, 81.01204681396484 | IN, India, Uttar Pradesh, Aliganj, , , , 27.479557037353516, 78.93465423583984 | IN, India, Gujarat, Ahmedabad, 380015, , , 23.0216238, 72.5797068 | IN, India, Tamil Nādu, Madurai, 625007, P & T Nagar Main Road, , 9.9596013, 78.1281179 | IN, India, Jammu and Kashmir, Jammu, 180020, Trikuta Nagar Link Rd, 3rd floor, 32.69895553588867, 74.87940979003906 | IN, India, Kerala, , 682025, Salem - Kochi - Kanyakumari Road, , 10.006034851074219, 76.312744140625 | IN, India, Karnataka, Bengaluru, 560105, Bannerghatta Main Road, 7th floor, 12.805644035339355, 77.5936508178711 | IN, India, Maharashtra, Pune, 411040, , , 18.510908126831055, 73.88510131835938
------- RECORD 36 -------
company_name             : Indus Towers Ltd. (erstwhile Bharti Infratel Ltd.)
website_domain           : industowers.com
locations                : IN, India, Jammu and Kashmir, Jammu, 180020, Trikuta Nagar Link Rd, 3rd floor, 32.705162699999995, 74.8725498
------- RECORD 37 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Kerala, Idukki, 685591, , , 9.8030891, 76.8628054
------- RECORD 38 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Haryana, Gurugram, , , , 28.497310638427734, 77.091796875 | IN, India, Delhi, New Delhi, , , , 28.613895416259766, 77.2090072631836
------- RECORD 39 -------
company_name             : INDUS TOWERS
website_domain           : industowers.com
locations                : IN, India, Kerala, , 691527, Thachanmukku - Poovattoor Road, , 9.056512099999999, 76.7318915
------- RECORD 40 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Andhra Pradesh, Janardhanavaram, , , , 16.9667721, 80.8630884
------- RECORD 41 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Haryana, Gurugram, 122002, , , 28.465638034513276, 77.08897831061947
------- RECORD 42 -------
company_name             : Indus tower
website_domain           : industowers.com
locations                : IN, India, Maharashtra, Thane, 421204, , , 19.1536886, 73.0702156
------- RECORD 43 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Maharashtra, Pune, 411014, Sakore Nagar Road, C 10, 18.5621465, 73.9145578
------- RECORD 44 -------
company_name             : Indus Towers Limited
website_domain           : industowers.com
locations                : IN, India, Uttar Pradesh, Ghaziabad, 201015, Delhi Eastern Peripheral Expressway, , 28.682906400000004, 77.508811
------- RECORD 45 -------
company_name             : Indus Towers
website_domain           : industowers.com
locations                : IN, India, Maharashtra, Mumbai, 400064, New Link Road, 11, 19.191342799999997, 72.8346719



CLUSTER 3/10 | ID: 8589934842 | SIZE: 44 | NAME_CORE: saharvosk
```

```text
------- RECORD 1 -------
website_domain           : saharvosk.ru
locations                : RU, Russia, Bashkortostan, Oktyabrsky, , улица Чапаева, 2, 54.4870648, 53.45951050000001
------- RECORD 2 -------
company_name             : Studia depilacii Sahar&Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, Bashkortostan, Sterlitamak, 453120, Коммунистическая улица, 83, 53.619825999999996, 55.90860310000001
------- RECORD 3 -------
company_name             : Sahar&Vosk face and body aesthetics studio
website_domain           : saharvosk.ru
locations                : RU, Russia, Stavropol Krai, Stavropol, 355017, улица Пушкина, 9, 45.039660000000005, 41.962558
------- RECORD 4 -------
company_name             : Depilation studio Sahar&Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, Belgorod Oblast, Belgorod, 308027, улица Генерала Апанасенко, 97, 50.583501299999995, 36.5665453
------- RECORD 5 -------
company_name             : Mezdunarodnaa set' studij depilacii Sahar&Vosk Penza
website_domain           : saharvosk.ru
locations                : RU, Russia, Penza Oblast, Penza, 440011, улица Карпинского, 33а, 53.211385799999995, 44.98054700000001
------- RECORD 6 -------
company_name             : Sugaring Affordable Depilation Studio SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Khanty-Mansiysk Autonomous Okrug – Ugra, Surgut, 628406, улица Маяковского, 7, 61.24918669999999, 73.4193926
------- RECORD 7 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Mari El Republic, Yoshkar-Ola, 424006, Советская улица, 165, 56.6282235, 47.8928952
------- RECORD 8 -------
company_name             : SAHAR&VOSK - body and face aesthetics studio
website_domain           : saharvosk.ru
locations                : RU, Russia, Orenburg Oblast, Orsk, 462420, Lenin Avenue, 7, 51.223619899999996, 58.4775592
------- RECORD 9 -------
company_name             : Sahar&vosk Kokzal
website_domain           : saharvosk.ru
locations                : KZ, Kazakhstan, East Kazakhstan Region, Ust-Kamenogorsk, , проспект Казыбек Би, 32, 49.897343600000006, 82.6105359
------- RECORD 10 -------
company_name             : Depilation and manicure studio Sahar & Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, Republic of Karelia, Petrozavodsk, 185035, Anohin street, 12, 61.783234, 34.3554109
------- RECORD 11 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Bashkortostan, Ufa, 450077, Верхнеторговая площадь, 4, 54.72368699999999, 55.942963
------- RECORD 12 -------
company_name             : Depilation studio SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Khanty-Mansiysk Autonomous Okrug – Ugra, Nizhnevartovsk, 628602, улица Мусы Джалиля, 18, 60.932898900000005, 76.5777282
------- RECORD 13 -------
company_name             : International network of affordable hair removal studios SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Moscow Oblast, Ramenskoye, 140105, Фабричный проезд, 1Б, 55.5721968, 38.210182599999996
------- RECORD 14 -------
company_name             : Sahar&Vosk - depilation and sugaring studio in Kazan
website_domain           : saharvosk.ru
locations                : RU, Russia, Tatarstan, Kazan, 420124, Чистопольская улица, 55, 55.8197434, 49.1285509
------- RECORD 15 -------
company_name             : Sahar&Vosk - depilation and sugaring studio
website_domain           : saharvosk.ru
locations                : RU, Russia, Arkhangelsk Oblast, Severodvinsk, 164514, проспект Труда, 85, 64.5627217, 39.796609499999995
------- RECORD 16 -------
company_name             : Sahar&Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, , , , , , 64.6863136, 97.7453061
------- RECORD 17 -------
company_name             : Depilation and nail service studio Sahar&Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, Republic of Karelia, Petrozavodsk, 185014, улица Древлянка, 15, 61.767317, 34.309309899999995
------- RECORD 18 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Saint Petersburg, Saint Petersburg, 191123, Фурштатская улица, 25, 59.9450066, 30.3572375
------- RECORD 19 -------
company_name             : SAHAR&VOSK body and facial aesthetics studio
website_domain           : saharvosk.ru
locations                : RU, Russia, Omsk Oblast, Omsk, 644070, Kuybysheva Street, 43, 54.983174600000005, 73.3977373
------- RECORD 20 -------
company_name             : Sahar&Vosk affordable depilation studio
website_domain           : saharvosk.ru
locations                : RU, Russia, Tuva Republic, Kyzyl, 667000, Moskovskaya Street, , 51.7026662, 94.40571229999999
------- RECORD 21 -------
company_name             : SAHAR & VOSK affordable depilation studio
website_domain           : saharvosk.ru
locations                : RU, Russia, Buryatia, Ulan-Ude, 670000, проспект Победы, 7, 51.829403899999996, 107.5904474
------- RECORD 22 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Bashkortostan, Ufa, 450105, улица Академика Королёва, 10, 54.774027999999994, 56.074707
------- RECORD 23 -------
company_name             : Salon SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Krasnoyarsk Krai, Krasnoyarsk, 660017, Взлётная улица, 1, 56.0332723, 92.9199458
------- RECORD 24 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Republic of Khakassia, Abakan, 655017, Chertygashev Street, 77, 53.724501399999994, 91.4417654
------- RECORD 25 -------
company_name             : Sahar&Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, Magadan Oblast, Magadan, 685000, проспект Карла Маркса, 39, 59.56299620000001, 150.80720789999998
------- RECORD 26 -------
company_name             : Studiya Depilyatsii Sakhar Vosk Tula
website_domain           : saharvosk.ru
locations                : RU, Russia, Tula Oblast, Tula, 300000, Советская улица, 27, 54.1941561, 37.612686999999994
------- RECORD 27 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Moscow, Moscow, 101000, Lubyanskiy Passage, 15 с2, 55.7564892, 37.6327257
------- RECORD 28 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Moscow Oblast, Balashikha, 143904, улица Карла Маркса, 3, 55.798782, 37.931059999999995
------- RECORD 29 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Bashkortostan, Ufa, 450112, улица Максима Горького, 54, 54.8247181, 56.07338089999999
------- RECORD 30 -------
company_name             : Depilation studio SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Khanty-Mansiysk Autonomous Okrug – Ugra, Khanty-Mansiysk, 628000, улица Мира, 98, 61.0087661, 69.0488204
------- RECORD 31 -------
company_name             : Sahar&Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, Moscow Oblast, Zvenigorod, 143185, Nakhabinskoye Road, , 55.7427418, 36.8569778
------- RECORD 32 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Arkhangelsk Oblast, Arkhangelsk, 163061, улица Выучейского, 16, 64.531954, 40.539526099999996
------- RECORD 33 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Khanty-Mansiysk Autonomous Okrug – Ugra, Nizhnevartovsk, 628602, Омская улица, 28А, 60.934141000000004, 76.576841
------- RECORD 34 -------
company_name             : Shugaring Obucheniye
website_domain           : saharvosk.ru
locations                : RU, Russia, Khabarovsk Krai, Khabarovsk, 680009, Karl Marx Street, 105, 48.502492, 135.10419109999998
------- RECORD 35 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Bashkortostan, Ufa, 450054, Комсомольская улица, 105, 54.7552113, 56.008350199999995
------- RECORD 36 -------
company_name             : Affordable hair removal: sugaring, waxing, laser hair removal - SAHAR&VOSK • cosmetology
website_domain           : saharvosk.ru
locations                : RU, Russia, Leningrad oblast, Kingisepp, 188480, Крикковское шоссе, 8, 59.382991999999994, 28.617199999999997
------- RECORD 37 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Orenburg Oblast, Orenburg, 460052, Салмышская улица, 48/2, 51.821011, 55.167744299999995
------- RECORD 38 -------
company_name             : SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Bashkortostan, Ufa, 450075, Richard Sorge Street, 75, 54.773581699999994, 56.0166938
------- RECORD 39 -------
company_name             : Sakhar Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, Buryatia, Ulan-Ude, 670047, улица Сахьяновой, 21, 51.8083225, 107.6239173
------- RECORD 40 -------
company_name             : Sahar&Vosk - affordable depilation and sugaring studio
website_domain           : saharvosk.ru
locations                : RU, Russia, Belgorod Oblast, Stary Oskol, 309514, , 3А, 51.310618299999994, 37.910164
------- RECORD 41 -------
company_name             : Studio SAHAR&VOSK
website_domain           : saharvosk.ru
locations                : RU, Russia, Khabarovsk Krai, Khabarovsk, 680000, Istomin Street, 60, 48.475908, 135.0587679
------- RECORD 42 -------
company_name             : Sahar&Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, Bashkortostan, Ufa, 450054, October Prospect, 71/3, 54.7642586, 56.0136573
------- RECORD 43 -------
company_name             : SAHAR&VOSK hair removal studio
website_domain           : saharvosk.ru
locations                : RU, Russia, Moscow Oblast, Korolyov, 141078, Октябрьский бульвар, 22, 55.9168552, 37.84250240000001
------- RECORD 44 -------
company_name             : Sahar & Vosk
website_domain           : saharvosk.ru
locations                : RU, Russia, Kaliningrad, Kaliningrad, 236000, Yuri Gagarin Street, 16, 54.719885999999995, 20.5448919



CLUSTER 4/10 | ID: 8589935033 | SIZE: 42 | NAME_CORE: meijimura
```

```text
------- RECORD 1 -------
company_name             : Gourmet cafe
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.3426907, 136.9909634
------- RECORD 2 -------
company_name             : Home of Marquis Tsugumichi Saigo
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.340035799999995, 136.9902433
------- RECORD 3 -------
company_name             : Akasaka Palace Main Gate Guard Station
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.3397628, 136.988335
------- RECORD 4 -------
company_name             : Shokudoraku Curry Bread Karepan Shop
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, 偉人坂, 1, 35.341371, 136.9902826
------- RECORD 5 -------
company_name             : Shokudoraku Croquette Shop
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.34696309999999, 136.98940009999998
------- RECORD 6 -------
company_name             : Kyoto City Electric
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.343399999999995, 136.9902618
------- RECORD 7 -------
company_name             : Official abode of Sugashima Lighthouse keeper
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1-25, 35.341727899999995, 136.99391699999998
------- RECORD 8 -------
company_name             : Candy shop Yakumo
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.34489920000001, 136.9897009
------- RECORD 9 -------
company_name             : 4th street
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.3440544, 136.9905059
------- RECORD 10 -------
company_name             : Shiodome Thermal Power Station Smokestack Foundation
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.3442349, 136.99032050000002
------- RECORD 11 -------
company_name             : Meiji-mura
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, , , 35.3609734, 136.9845491
------- RECORD 12 -------
company_name             : Meiji Era Western Food Omu-rice & Grill Roman-Tei
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.3459611, 136.98867529999998
------- RECORD 13 -------
company_name             : Nagoya-an
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.341823, 136.99122029999998
------- RECORD 14 -------
company_name             : Japanese Sweet Shop Yakumo
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.344900599999995, 136.9896073
------- RECORD 15 -------
company_name             : Museum Meiji-Mura
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.3404417, 136.9885261
------- RECORD 16 -------
company_name             : Imperial Hotel Tea Room The Museum Meiji-Mura
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.3471762, 136.9887735
------- RECORD 17 -------
company_name             : SL Tokyo Station Shop
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0894, 隅田川新大橋, , 35.3487092, 136.9881495
------- RECORD 18 -------
company_name             : Kyoto Sweets Restaurant Nakai Saryo
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.3416351, 136.9899459
------- RECORD 19 -------
company_name             : Japanese small miscellaneous goods Raku
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.34328779999999, 136.9908036
------- RECORD 20 -------
company_name             : The main entrance of the 8th High School
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.340542299999996, 136.9886857
------- RECORD 21 -------
company_name             : Teahouse Ekiraku-an
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.3412569, 136.9933172
------- RECORD 22 -------
company_name             : Japanese Restaurant HEKISUI-TEI
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, 偉人坂, 1, 35.3397903, 136.9905598
------- RECORD 23 -------
company_name             : Kureha-za Theater
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.345044099999996, 136.98960499999998
------- RECORD 24 -------
company_name             : Gakushuin Principal's Official Residence
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, 偉人坂, 1, 35.339964699999996, 136.9897086
------- RECORD 25 -------
company_name             : Museum Meiji Village Guidance Center
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.3403978, 136.9885416
------- RECORD 26 -------
company_name             : 5th street
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.34658819999999, 136.98928329999998
------- RECORD 27 -------
company_name             : Imperial Palace Main Gate Ishibashi Decorative Lamppost
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0894, 隅田川新大橋, 12-4, 35.346822599999996, 136.9901853
------- RECORD 28 -------
company_name             : Denki Bran Shiodome Bar
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.344319399999996, 136.990297
------- RECORD 29 -------
company_name             : Meiji Mura Museum Shop
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.34037850000001, 136.9886079
------- RECORD 30 -------
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, , , 34.9991645, 137.254574
------- RECORD 31 -------
company_name             : Konasami-jima Lighthouse
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, 天道眼鏡橋, , 35.3460721, 136.98953889999999
------- RECORD 32 -------
company_name             : Ring spinning machine
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.343736, 136.9904149
------- RECORD 33 -------
company_name             : House of Ogai Mori and Soseki Natsume
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.340427999999996, 136.9906182
------- RECORD 34 -------
company_name             : Main gate terrace
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.34025460000001, 136.9885559
------- RECORD 35 -------
company_name             : Higashi-Yamanashi District Office
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.341636199999996, 136.988632
------- RECORD 36 -------
company_name             : SL Nagoya Station
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.343670200000005, 136.9899667
------- RECORD 37 -------
company_name             : Meiji Village Hall
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.341868, 136.987644
------- RECORD 38 -------
company_name             : Monument of Literary Masters
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.339501399999996, 136.98954559999999
------- RECORD 39 -------
company_name             : Card Maze Gurumi Forest Great Adventure Meiji Encyclopedia
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, 偉人坂, 1, 35.3416166, 136.9893978
------- RECORD 40 -------
company_name             : Lamp of Nijubashi bridge in the Imperial Palace
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.3406117, 136.9894645
------- RECORD 41 -------
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, , 35.340311299999996, 136.9892791
------- RECORD 42 -------
company_name             : Mie Prefectural Office
website_domain           : meijimura.com
locations                : JP, Japan, Aichi Prefecture, Inuyama, 484-0000, Municipal Road Route Fuji Honsen, 1, 35.3408973, 136.9891446



CLUSTER 5/10 | ID: 8589935500 | SIZE: 40 | NAME_CORE: airamanah
```

```text
------- RECORD 1 -------
company_name             : Air Amanah Semarang
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Semarang, 50273, Jalan Sinar Bahagia I, 495, -7.025819299999998, 110.4721153
------- RECORD 2 -------
company_name             : Distributor Air Amanah Sonosewu Ngestiharjo
website_domain           : airamanah.com
locations                : ID, Indonesia, Special Region of Yogyakarta, Bantul Regency, , Gang Nangka, , -7.8071267, 110.34202839999999
------- RECORD 3 -------
company_name             : Distro Air Amanah Guntung Payung
website_domain           : airamanah.com
locations                : ID, Indonesia, South Kalimantan, Banjarbaru, 70721, Jalan Lingkar Utara Banjarbaru, , -3.4475892999999993, 114.79501690000001
------- RECORD 4 -------
company_name             : Distributor Amanah
website_domain           : airamanah.com
locations                : ID, Indonesia, Special Region of Yogyakarta, Gunung Kidul Regency, , , , -7.956356299999999, 110.5930855
------- RECORD 5 -------
company_name             : Air AMANAH mm Menik Nganjuk DISTRO
website_domain           : airamanah.com
locations                : ID, Indonesia, East Java, Nganjuk, 64414, , , -7.6704093, 112.05476449999999
------- RECORD 6 -------
company_name             : Air Amanah
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Boyolali, 57376, , , -7.4848069, 110.666485
------- RECORD 7 -------
company_name             : Distributor Amanah Bp. Anwani
website_domain           : airamanah.com
locations                : ID, Indonesia, Special Region of Yogyakarta, Sleman Regency, , , , -7.742653199999999, 110.3730811
------- RECORD 8 -------
company_name             : Distributor Air Amanah Beringin
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Semarang, 50186, Jalan Beringin Asri Barat Raya, 11, -6.983873699999999, 110.32394939999999
------- RECORD 9 -------
company_name             : Amanah Tirta Ungaran
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Bandungan, 50612, Jalan Bandungan, 60, -7.243569299999999, 110.39388969999999
------- RECORD 10 -------
company_name             : Distributor Amanah
website_domain           : airamanah.com
locations                : ID, Indonesia, Special Region of Yogyakarta, Gunung Kidul Regency, , , , -7.9748521, 110.6083549
------- RECORD 11 -------
company_name             : Air Amanah Boyolali
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Boyolali, 57316, , , -7.5223513, 110.763477
------- RECORD 12 -------
company_name             : Air minum Amanah Isi Ulang Galon
website_domain           : airamanah.com
locations                : ID, Indonesia, East Java, Magetan, 63352, Jalan Haji Mohammad Noer, 3, -7.629476800000002, 111.3060115
------- RECORD 13 -------
company_name             : Air Amanah Surakarta
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Surakarta, 57137, Jalan Adi Sumarmo, , -7.541385699999998, 110.8080167
------- RECORD 14 -------
company_name             : Amanah Tirta Pemalang
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Pemalang, 52353, , , -7.1028802, 109.28061459999999
------- RECORD 15 -------
company_name             : Distributor Air Amanah Kota Jogja
website_domain           : airamanah.com
locations                : ID, Indonesia, Special Region of Yogyakarta, Yogyakarta, 55262, Jalan Taqwa, , -7.8032805000000005, 110.35933949999999
------- RECORD 16 -------
company_name             : Distributor AIR AMANAH
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Surakarta, 57146, , , -7.5647481, 110.78531339999999
------- RECORD 17 -------
company_name             : Distributor Air AMANAH
website_domain           : airamanah.com
locations                : ID, Indonesia, South Kalimantan, Tanjung, 71571, , , -2.1783272, 115.42891339999998
------- RECORD 18 -------
company_name             : Air Amanah
website_domain           : airamanah.com
------- RECORD 19 -------
company_name             : Distributor Air Amanah Bang Doel
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Sukoharjo, 57511, , , -7.622059999999999, 110.8136629
------- RECORD 20 -------
company_name             : Distributor Air Amanah Sampit
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Kalimantan, Mentawa Baru Hulu, 74312, Jalan Cristopel Mihing, , -2.5293020999999998, 112.9476928
------- RECORD 21 -------
company_name             : Distributor Amanah
website_domain           : airamanah.com
locations                : ID, Indonesia, Special Region of Yogyakarta, Gunung Kidul Regency, , , , -7.9629845999999995, 110.60485299999999
------- RECORD 22 -------
company_name             : Distro Air Amanah Tlowet
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Semarang, 50196, Jalan Abdul Hamid, , -6.983062599999999, 110.4727569
------- RECORD 23 -------
company_name             : Amanah Water
website_domain           : airamanah.com
locations                : ID, Indonesia, West Java, Kuningan, 45512, , , -6.9821453, 108.4768043
------- RECORD 24 -------
company_name             : Distributor Air mineral Amanah Purwokerto
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Banyumas, 53192, , , -7.4604472, 109.27024349999999
------- RECORD 25 -------
company_name             : DISTRIBUTOR AIR AMANAH SEMARANG
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Semarang, 50212, Gang Arjuna, rt.5/rw.9, -7.0081334, 110.45711
------- RECORD 26 -------
company_name             : Distributor Air Amanah
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Karanganyar, 57722, , , -7.585166500000001, 110.9292186
------- RECORD 27 -------
company_name             : Distributor Air Amanah Jenar
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Purworejo, 54151, , , -7.808838199999999, 109.99730079999999
------- RECORD 28 -------
company_name             : Halal Mart Banjarmasin & Distro Amanah
website_domain           : airamanah.com
locations                : ID, Indonesia, South Kalimantan, Banjarmasin, 70238, , , -3.3472748, 114.6423739
------- RECORD 29 -------
company_name             : Distributor Air Minum Amanah Karanganyar
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Karanganyar, 57713, , , -7.599404099999997, 110.9335261
------- RECORD 30 -------
company_name             : Distributor Air Mineral AMANAH PATI
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Pati, 59119, , , -6.7223999999999995, 111.0538739
------- RECORD 31 -------
company_name             : Distributor Air Minum Amanah Magelang Borobudur Magelang
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Magelang, 56553, , , -7.6311515000000005, 110.23379369999999
------- RECORD 32 -------
company_name             : Toko Herbal Arif
website_domain           : airamanah.com
locations                : ID, Indonesia, South Kalimantan, Tanjung, 71552, , , -2.2939603, 115.3045014
------- RECORD 33 -------
company_name             : Air amanah
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Sukoharjo, 57511, Jalan Mawar, 6, -7.6494925, 110.8212418
------- RECORD 34 -------
company_name             : DISTRIBUTOR AIR AMANAH WONOSEGORO
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Boyolali, 57382, , , -7.3162861999999995, 110.6639847
------- RECORD 35 -------
company_name             : Distributor Air Amanah Adiwerna
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Tegal, 52194, , , -6.9398025, 109.12465859999999
------- RECORD 36 -------
company_name             : Distributor Amanah Wonokarto
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Wonogiri, 57612, , , -7.796119300000001, 110.9123013
------- RECORD 37 -------
company_name             : Distributor Air Amanah Gamping
website_domain           : airamanah.com
locations                : ID, Indonesia, Special Region of Yogyakarta, Sleman Regency, , , , -7.790461899999998, 110.340828
------- RECORD 38 -------
company_name             : Amanah Tirta Karanganyar
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Jatirejo, , , , -7.6002842, 111.1165271
------- RECORD 39 -------
company_name             : Gerai AMANAH Kelua
website_domain           : airamanah.com
locations                : ID, Indonesia, South Kalimantan, Tanjung, 71552, , , -2.2939603, 115.3045014
------- RECORD 40 -------
company_name             : Air Amanah Solo Raya
website_domain           : airamanah.com
locations                : ID, Indonesia, Central Java, Surakarta, 57143, , , -7.5699639, 110.7968624



CLUSTER 6/10 | ID: 8589935348 | SIZE: 39 | NAME_CORE: chatime
```

```text
------- RECORD 1 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Makati, 1232, Chino Roces Avenue, 2257, 14.541038700000001, 121.0191832
------- RECORD 2 -------
company_name             : Chatime - SM Ecoland Davao
website_domain           : chatime.com.ph
locations                : PH, Philippines, Davao Region, Davao City, 8000, Quimpo Boulevard, , 7.049027099999999, 125.58839879999998
------- RECORD 3 -------
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Pasig, 1605, Meralco Avenue, 31, 14.580082893371582, 121.06336212158203 | PH, Philippines, Metro Manila, Muntinlupa, 1783, , , 14.424123764038086, 121.03045654296875 | PH, Philippines, Metro Manila, Pasig, 1604, Central Avenue, , 14.584996223449707, 121.07698822021484 | PH, Philippines, Metro Manila, Pasig, 1600, Marcos Highway, , 14.619205474853516, 121.09285736083984 | PH, Philippines, Metro Manila, Pasig, 1600, , , 14.560516357421875, 121.07643127441406 | PH, Philippines, Metro Manila, Quezon City, 1100, Eastwood Avenue, , 14.609492301940918, 121.07943725585938 | PH, Philippines, Metro Manila, Quezon City, 1118, Quirino Highway, , 14.735285758972168, 121.0592269897461 | PH, Philippines, Metro Manila, Quezon City, 1106, Bansalangin Street, 9, 14.654526710510254, 121.02338409423828 | PH, Philippines, Metro Manila, Quezon City, 1109, General Romulo Avenue, , 14.619542121887207, 121.0577163696289 | PH, Philippines, Metro Manila, Quezon City, 1105, EDSA, , 14.657371520996094, 121.02137756347656
------- RECORD 4 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Calabarzon, San Pablo, 4000, Maharlika Highway, , 14.0704449, 121.30129609999999
------- RECORD 5 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Central Luzon, Plaridel, 3004, Doña Remedios Trinidad Highway, , 14.881465599999997, 120.86652640000001
------- RECORD 6 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Pasig, 1600, Marcos Highway, , 14.619386299999997, 121.0935908
------- RECORD 7 -------
company_name             : Chatime Ayala Malls Feliz
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Pasig, 1600, Marcos Highway, , 14.6192782, 121.0929186
------- RECORD 8 -------
company_name             : Chatime SM San Mateo
website_domain           : chatime.com.ph
locations                : PH, Philippines, Calabarzon, San Mateo, 1050, Lord of Pardon, , 14.679775900000001, 121.11435539999998
------- RECORD 9 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Calabarzon, Carmona, 4116, Governor's Drive, , 14.313683999999999, 121.0494239
------- RECORD 10 -------
company_name             : Chatime - Paseo Perdices Dumaguete
website_domain           : chatime.com.ph
locations                : PH, Philippines, Central Visayas, Dumaguete, 6200, Governor M. F. Perdices Street, 2f, 9.307880899999997, 123.3096575
------- RECORD 11 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, , , , , , 12.7503486, 122.7312101
------- RECORD 12 -------
company_name             : Chatime - The SM Center East Ortigas
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Pasig, 1600, Ortigas Avenue Extension, , 14.587399899999998, 121.10514239999999
------- RECORD 13 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Calabarzon, Calamba, 4027, National Highway, , 14.202999799999999, 121.1553383
------- RECORD 14 -------
company_name             : ChaTime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Calabarzon, Lucena, 4301, Dalahican Road, , 13.940685, 121.6258749
------- RECORD 15 -------
company_name             : Chatime - Ayala 30th
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Pasig, 1605, Meralco Avenue, 1605 30, 14.580721100000002, 121.06400370000001
------- RECORD 16 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Central Luzon, Malolos, 3000, MacArthur Highway, , 14.850596499999996, 120.8240905
------- RECORD 17 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Manila, 1000, Pedro Gil Street, , 14.5761574, 120.9843563
------- RECORD 18 -------
company_name             : Chatime Uptown Mall
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Taguig, , 11th Avenue, , 14.556424599999998, 121.0549763
------- RECORD 19 -------
company_name             : ChaTime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Central Luzon, San Fernando, 2000, , , 15.0532349, 120.69889839999999
------- RECORD 20 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Calabarzon, Cabuyao, 4025, National Highway, , 14.232841, 121.13511910000001
------- RECORD 21 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Pasig, 1600, E. Rodriguez Jr. Avenue, , 14.584432300000001, 121.07674960000001
------- RECORD 22 -------
company_name             : Chatime Philippines
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Pasig, 1603, , , 14.5711649, 121.0594975
------- RECORD 23 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Taguig, , Turin Street, , 14.532846399999997, 121.05189389999998
------- RECORD 24 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Muntinlupa, , Alabang-Zapote Road, , 14.424298299999997, 121.0301056
------- RECORD 25 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Makati, 6766, Ayala Avenue, 6766, 14.555744999999998, 121.0208547
------- RECORD 26 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Marikina, , Marcos Highway, , 14.62627, 121.08417800000001
------- RECORD 27 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Pasig, 1600, Pioneer Street, , 14.5733454, 121.05829600000001
------- RECORD 28 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Muntinlupa, 1770, CARES Access Road, 2294, 14.424135099999999, 121.0295205
------- RECORD 29 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Quezon City, 1100, Orchard Road, , 14.609425000000002, 121.07969779999998
------- RECORD 30 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Quezon City, 1100, Quirino Highway, , 14.7337609, 121.0592592
------- RECORD 31 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Parañaque, 1711, Doña Soledad Avenue, 138, 14.484505400000002, 121.03071030000001
------- RECORD 32 -------
company_name             : Chatime - W Mall
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Pasay, 1308, Coral Way, , 14.532285600000002, 120.9890343
------- RECORD 33 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Central Luzon, Gapan, 3105, Cagayan Valley Road, , 15.303654300000002, 120.94626749999999
------- RECORD 34 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Central Luzon, Malolos, 3000, MacArthur Highway, 44, 14.872378999999999, 120.79917599999999
------- RECORD 35 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Calabarzon, Silang, 4118, Aguinaldo Highway, , 14.184593699999999, 120.96120529999999
------- RECORD 36 -------
company_name             : Chatime SM Masinag
website_domain           : chatime.com.ph
locations                : PH, Philippines, Calabarzon, Antipolo, 1870, Marcos Highway, , 14.625244199999997, 121.1205893
------- RECORD 37 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Makati, 1209, Makati Avenue, , 14.5519661, 121.02390570000001
------- RECORD 38 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Metro Manila, Parañaque, 1700, President's Avenue, , 14.457461100000003, 121.033056
------- RECORD 39 -------
company_name             : Chatime
website_domain           : chatime.com.ph
locations                : PH, Philippines, Calabarzon, Biñan, 4024, South Luzon Expressway, KM 35, 14.311988799999998, 121.07175879999998



CLUSTER 7/10 | ID: 8589935697 | SIZE: 39 | NAME_CORE: newcare
```

```text
------- RECORD 1 -------
company_name             : newcare parc Hamburg
website_domain           : newcare.de
locations                : DE, Germany, Hamburg, Hamburg, 22547, Luruper Hauptstraße, 247, 53.597592999999996, 9.8617814
------- RECORD 2 -------
company_name             : St. Clara senior center
website_domain           : newcare.de
locations                : DE, Germany, North Rhine-Westphalia, Swisttal, 53913, Hinter dem Burggarten, 9, 50.712497799999994, 6.9101409999999985
------- RECORD 3 -------
company_name             : newcare home Ersrode
website_domain           : newcare.de
locations                : DE, Germany, Hesse, Ludwigsau, 36251, Neustadt, 20, 50.9793751, 9.5898135
------- RECORD 4 -------
company_name             : Seniorenzentrum Bargteheide
website_domain           : newcare.de
locations                : DE, Germany, Schleswig-Holstein, Bargteheide, 22941, Lübecker Straße, 2, 53.731162499999996, 10.262796999999999
------- RECORD 5 -------
company_name             : Medical Senioren Park
website_domain           : newcare.de
locations                : DE, Germany, Hesse, Vellmar, 34246, Rembrandtweg, 2, 51.36662, 9.471289999999998
------- RECORD 6 -------
company_name             : newcare parc Aumund
website_domain           : newcare.de
locations                : DE, Germany, Bremen, Bremen, 28755, Am Aumunder Bahnhof, 3, 53.18566839999999, 8.617284100000001
------- RECORD 7 -------
company_name             : newcare parc Hoykenkamp
website_domain           : newcare.de
locations                : DE, Germany, Lower Saxony, Ganderkesee, 27777, Zum Fischerteich, 1, 53.0728126, 8.602941
------- RECORD 8 -------
company_name             : newcare home Till
website_domain           : newcare.de
locations                : DE, Germany, North Rhine-Westphalia, Bedburg-Hau, 47551, Kloster, 5, 51.759433699999995, 6.2593827
------- RECORD 9 -------
company_name             : newcare home Scheeßel
website_domain           : newcare.de
locations                : DE, Germany, Lower Saxony, Scheeßel, 27383, Große Straße, 6a, 53.169419999999995, 9.48502
------- RECORD 10 -------
company_name             : newcare parc Lengede
website_domain           : newcare.de
locations                : DE, Germany, Lower Saxony, Lengede, 38268, Am Naturbad, 8, 52.1915992, 10.3277067
------- RECORD 11 -------
company_name             : Retirement home on the beach promenade
website_domain           : newcare.de
locations                : DE, Germany, Schleswig-Holstein, Großenbrode, 23775, Am Kai, 1, 54.35686079999999, 11.084935999999999
------- RECORD 12 -------
company_name             : Newcare Home Geeste
website_domain           : newcare.de
locations                : DE, Germany, Lower Saxony, Geeste, 49744, , , 52.6009773, 7.268098
------- RECORD 13 -------
company_name             : newcare home Bebra
website_domain           : newcare.de
locations                : DE, Germany, Hesse, Bebra, 36179, Bahnhofstraße, 19, 50.96923, 9.79643
------- RECORD 14 -------
company_name             : newcare home Eidelstedt
website_domain           : newcare.de
locations                : DE, Germany, Hamburg, Hamburg, 22527, Eidelstedter Dorfstraße, 19, 53.607980000000005, 9.90828
------- RECORD 15 -------
company_name             : newcare home Kleve
website_domain           : newcare.de
locations                : DE, Germany, North Rhine-Westphalia, Kleve, 47533, Tiergartenstraße, 44, 51.792886700000004, 6.1328345
------- RECORD 16 -------
company_name             : newcare home Neustadt
website_domain           : newcare.de
locations                : DE, Germany, Rhineland-Palatinate, Rennerod, 56479, Am Stein, 20, 50.62871, 8.04239
------- RECORD 17 -------
company_name             : newcare home Dahlerau
website_domain           : newcare.de
locations                : DE, Germany, North Rhine-Westphalia, Radevormwald, 42477, Siedlungsweg, 25, 51.2203508, 7.308684799999999
------- RECORD 18 -------
company_name             : newcare home Pirmasens
website_domain           : newcare.de
locations                : DE, Germany, Rhineland-Palatinate, Pirmasens, 66953, Steinstraße, 63, 49.2088452, 7.613507
------- RECORD 19 -------
company_name             : newcare parc Schenefeld
website_domain           : newcare.de
locations                : DE, Germany, Schleswig-Holstein, Schenefeld, 22869, Ebenholzweg, 1, 53.607439299999996, 9.837971699999999
------- RECORD 20 -------
company_name             : Tagespflege - newcare parc Oberneuland
website_domain           : newcare.de
locations                : DE, Germany, Bremen, Bremen, 28355, Rockwinkeler Landstraße, 1, 53.0895332, 8.925699
------- RECORD 21 -------
company_name             : newcare home Neu-Ulm
website_domain           : newcare.de
locations                : DE, Germany, Bavaria, Neu-Ulm, 89233, Hochreuteweg, 121, 48.41256, 10.0446097
------- RECORD 22 -------
company_name             : newcare home Fritzlar
website_domain           : newcare.de
locations                : DE, Germany, Hesse, Fritzlar, 34560, Mariannenstraße, 6, 51.137664, 9.2800851
------- RECORD 23 -------
company_name             : newcare GmbH
website_domain           : newcare.de
locations                : DE, Germany, North Rhine-Westphalia, Essen, 45136, Max-Keith-Straße, 66, 51.4329957, 7.026624899999999
------- RECORD 24 -------
company_name             : newcare home Radevormwald
website_domain           : newcare.de
locations                : DE, Germany, North Rhine-Westphalia, Radevormwald, 42477, Uelfestraße, 24, 51.2048656, 7.364732600000001
------- RECORD 25 -------
company_name             : newcare Gruppe - Wir denken Pflege neu.
website_domain           : newcare.de
locations                : DE, Germany, North Rhine-Westphalia, Essen, 45136, Max-Keith-Straße, 66, 51.432404, 7.0257702
------- RECORD 26 -------
company_name             : newcare home Lurup
website_domain           : newcare.de
locations                : DE, Germany, Hamburg, Hamburg, 22547, Luruper Hauptstraße, 119, 53.5892936, 9.874767199999997
------- RECORD 27 -------
company_name             : newcare home Leer
website_domain           : newcare.de
locations                : DE, Germany, Lower Saxony, Leer, 26789, Große Roßbergstraße, 24, 53.237487099999996, 7.4645404
------- RECORD 28 -------
company_name             : newcare home Bedburg-Hau
website_domain           : newcare.de
locations                : DE, Germany, North Rhine-Westphalia, Bedburg-Hau, 47551, Kalkarer Straße, 75, 51.775034899999994, 6.175465099999999
------- RECORD 29 -------
company_name             : newcare home Geeste
website_domain           : newcare.de
locations                : DE, Germany, Lower Saxony, Geeste, 49744, Bahnhofstraße, 44, 52.602284499999996, 7.312856999999999
------- RECORD 30 -------
company_name             : newcare parc Neumünster
website_domain           : newcare.de
locations                : DE, Germany, Schleswig-Holstein, Neumünster, 24534, Goebenstraße, 11c-d, 54.075337, 9.9717328
------- RECORD 31 -------
company_name             : newcare home Felsberg
website_domain           : newcare.de
locations                : DE, Germany, Hesse, Felsberg, 34587, Lohrer Straße, 4, 51.1354149, 9.4141835
------- RECORD 32 -------
company_name             : Tagespflege new care Parc Schwanewede
website_domain           : newcare.de
locations                : DE, Germany, Lower Saxony, Schwanewede, 28790, Damm, 45, 53.2429743, 8.589911899999999
------- RECORD 33 -------
company_name             : newcare home Gensungen
website_domain           : newcare.de
locations                : DE, Germany, Hesse, Felsberg, 34587, Frankenstraße, 1, 51.129651499999994, 9.434800899999999
------- RECORD 34 -------
company_name             : newcare parc Oberneuland
website_domain           : newcare.de
locations                : DE, Germany, Bremen, Bremen, 28355, Rockwinkeler Landstraße, 1, 53.089309899999996, 8.925807
------- RECORD 35 -------
company_name             : newcare home Hasbergen
website_domain           : newcare.de
locations                : DE, Germany, Lower Saxony, Hasbergen, 49205, Tecklenburger Straße, 52, 52.2422563, 7.953623200000001
------- RECORD 36 -------
company_name             : newcare home Schlüchtern
website_domain           : newcare.de
locations                : DE, Germany, Hesse, Schlüchtern, 36381, An den Lindengärten, 9, 50.3504151, 9.523133399999997
------- RECORD 37 -------
company_name             : newcare home Moyland
website_domain           : newcare.de
locations                : DE, Germany, North Rhine-Westphalia, Bedburg-Hau, 47551, Kloster, 1, 51.7599227, 6.258833800000001
------- RECORD 38 -------
company_name             : newcare Holding GmbH – Zentrale Nürnberg
website_domain           : newcare.de
locations                : DE, Germany, Bavaria, Nuremberg, 90441, Hansastraße, 5, 49.425714199999994, 11.0386277
------- RECORD 39 -------
company_name             : newcare parc Schwanewede
website_domain           : newcare.de
locations                : DE, Germany, Lower Saxony, Schwanewede, 28790, Damm, 45, 53.2429211, 8.5902609



CLUSTER 8/10 | ID: 8589934952 | SIZE: 38 | NAME_CORE: recommendedbyroberto
```

```text
------- RECORD 1 -------
company_name             : La Mer Restaurant
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Varna, Varna, 9007, Боян Бъчваров, , 43.264533899999996, 28.036318
------- RECORD 2 -------
company_name             : Boryana Hotel
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Burgas, Burgas, 8014, Лорна, 8, 42.4449781, 27.488272400000003
------- RECORD 3 -------
company_name             : Boyadjiyski Guest House
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Blagoevgrad, Bansko, 2770, Ski Road 1a, 16, 41.8329658, 23.491714499999997
------- RECORD 4 -------
company_name             : Brani Family Hotel
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Ruse, Ruse, 7000, Midia Enos Blvd, 33, 43.8314722, 25.9500572
------- RECORD 5 -------
company_name             : Kompleks Oazis
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Sofia, Etropole, 2180, Ген. Димитър Гръбчев, , 42.834050100000006, 23.9947558
------- RECORD 6 -------
company_name             : Chuchi Guest House
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Lovech, , , , , 43.09948689999999, 24.127038
------- RECORD 7 -------
company_name             : Brigantina Beach Hotel
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Varna, Varna, 9000, Nezavisimost Square, 150, 43.264014499999995, 28.0359679
------- RECORD 8 -------
company_name             : CHERRY VILLA SOZOPOL
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Burgas, Burgas, 8130, Via Pontika, 44, 42.4072844, 27.7057
------- RECORD 9 -------
company_name             : BotaBara Del Mar Boutique
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Burgas, Pomorie, 8200, Timok, , 42.5637118, 27.641392099999997
------- RECORD 10 -------
company_name             : Apartments Europe Pomorie
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Burgas, Pomorie, 8201, Knyaz Boris, , 42.5640525, 27.594871
------- RECORD 11 -------
company_name             : Kompleks azovir Valtata”
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Blagoevgrad, Parvomai, 2890, 198, , 41.3885242, 23.088629199999996
------- RECORD 12 -------
company_name             : Chiflik Nenkovi Family Hotel
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Sofia, Osenovlag, , , , 42.968897999999996, 23.532314
------- RECORD 13 -------
website_domain           : recommendedbyroberto.com
------- RECORD 14 -------
company_name             : Colors Boutique
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Burgas, Burgas, 8280, Treti Mart, 45, 42.8228138, 27.879263700000003
------- RECORD 15 -------
company_name             : Belle Stelle Sofia
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Sofia-City, Sofia, 1000, Krastyo Rakovski, 2, 42.699097699999996, 23.3298858
------- RECORD 16 -------
company_name             : Bobekova kʺsa
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Pazardzhik, Panagyurishte, 4500, Nistor Ruzhikov, 46, 42.5114182, 24.1821858
------- RECORD 17 -------
company_name             : Chakarova Guest House
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Sliven, Sliven, 8800, Donka i Konstantin Konstantinovi, 20, 42.6848599, 26.314637100000002
------- RECORD 18 -------
company_name             : Dream Park
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Blagoevgrad, Sandanski, 2800, Никола Вапцаров, 80, 41.5752905, 23.283880699999997
------- RECORD 19 -------
company_name             : City Center Apartments
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Plovdiv, Plovdiv, 4003, Bogdan, 7, 42.1552054, 24.743754499999998
------- RECORD 20 -------
company_name             : Apartment Chaika
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Varna, Varna, 9005, , бл. 22, 43.2135442, 27.937644099999996
------- RECORD 21 -------
company_name             : Capital City Center Apart Residence
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Plovdiv, Plovdiv, 4000, Ekzarh Yosif, 18, 42.136047700000006, 24.7468982
------- RECORD 22 -------
company_name             : City Center Lux Apartment
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Varna, Varna, 9000, Ruse, , 43.2037102, 27.908955899999995
------- RECORD 23 -------
company_name             : Studio in Borovets Gardens Complex
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Sofia, Samokov, 2010, 82, , 42.2710359, 23.604135999999997
------- RECORD 24 -------
company_name             : Solita
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Varna, Byala, 9101, Bregova, 52, 42.8730225, 27.8883423
------- RECORD 25 -------
company_name             : Blue Lagune-2
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Varna, Varna, 9002, Knyaz Nikolay Nikolaevich, 6A, 43.20965040000001, 27.922406899999995
------- RECORD 26 -------
company_name             : Kashta Chelsi
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Plovdiv, Plovdiv, 4000, Rakovski, 16а, 42.1438068, 24.7538941
------- RECORD 27 -------
company_name             : Hostel Casablanca City
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Varna, Varna, 9000, Krastyu Mirski, 1, 43.2032078, 27.9114978
------- RECORD 28 -------
company_name             : Sun Village Apartments
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Burgas, Burgas, 8240, 1-va, 87, 42.69972, 27.709566
------- RECORD 29 -------
company_name             : Cedar Lodge 3 4 Apartment Paradise
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Blagoevgrad, Bansko, 2770, , , 41.828313300000005, 23.4754906
------- RECORD 30 -------
company_name             : Borovets Holiday Apartments
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Sofia, Samokov, 2000, SFO1570, 82, 42.2665751, 23.6039812
------- RECORD 31 -------
company_name             : Dunav Apart
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Ruse, Ruse, 7015, Dame Gruev, 4b, 43.8263803, 25.9712615
------- RECORD 32 -------
company_name             : Benita inn aprtmʺnts
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Sofia-City, Sofia, 1000, Sofia, 138, 42.70428499999999, 23.305732
------- RECORD 33 -------
company_name             : Complex Diva
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Silistra, Srebarna, 7588, Dunav Str., 19, 44.0939376, 27.064024399999997
------- RECORD 34 -------
company_name             : BGApartments - Momina salza
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Varna, Varna, 9002, Момина сълза, , 43.220917199999995, 27.913921399999996
------- RECORD 35 -------
company_name             : Belle View
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Burgas, Sveti Vlas, 8256, Сирена, 7, 42.7105076, 27.755721699999995
------- RECORD 36 -------
company_name             : Hotel Bonita
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Varna, Varna, , Riviera, , 43.2793285, 28.0425163
------- RECORD 37 -------
company_name             : Dolomiti Hotel
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Blagoevgrad, Bansko, 2770, Knyaz Boris, 48, 41.8287944, 23.4939182
------- RECORD 38 -------
company_name             : Bendida Central
website_domain           : recommendedbyroberto.com
locations                : BG, Bulgaria, Pazardzhik, Velingrad, 4600, , , 42.02833760000001, 23.9910321



CLUSTER 9/10 | ID: 8589935024 | SIZE: 37 | NAME_CORE: freshburger
```

```text
------- RECORD 1 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Al-Bahah Province, Baljurashi, 65652, , , 19.8509013, 41.583169
------- RECORD 2 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , Abha, , , , 18.2275506, 42.5586802
------- RECORD 3 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , , 67377, طريق الملك فيصل, , 19.1479322, 42.114700899999995
------- RECORD 4 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Medina Province, Medina, 42383, , , 24.425783299999996, 39.598540899999996
------- RECORD 5 -------
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , , 62435, طريق الملك عبدالله, , 18.3367739, 42.7686743
------- RECORD 6 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , , , , , 19.972746, 42.6061347
------- RECORD 7 -------
company_name             : Fresh burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Eastern Province, Dammam, 31451, , , 18.288670000000003, 42.7266203
------- RECORD 8 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , Abha, 62521, , , 18.2150677, 42.5212131
------- RECORD 9 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Medina Province, Yanbu`, 42316, Ali Bin Abi Talib (Awali Nazil), , 24.087070199999996, 38.0452164
------- RECORD 10 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Makkah Region, Runiya, , 255, , 21.261518999999996, 42.833447299999996
------- RECORD 11 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Jazan Province, Samtah, 82723, , , 16.608742, 42.9395297
------- RECORD 12 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Al-Bahah Province, Al Makhwah, , , , 19.7809454, 41.43861819999999
------- RECORD 13 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Makkah Region, Jeddah, 23434, , , 21.5810088, 39.1653612
------- RECORD 14 -------
company_name             : Fresh burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Jazan Province, Ad Darb, , 10, , 17.742325399999995, 42.28215960000001
------- RECORD 15 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , Abha, , , , 18.1824308, 42.8264449
------- RECORD 16 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Makkah Region, At Ta'if, 26511, Airport Road, , 21.4198982, 40.4932205
------- RECORD 17 -------
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Makkah Region, Al Qunfudhah, 28822, , , 19.131781999999998, 41.0858648
------- RECORD 18 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Makkah Region, At Ta'if, 26511, Quraish Street, , 21.234104199999997, 40.41842119999999
------- RECORD 19 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Eastern Province, Shari`, , , , 20.011512700000004, 42.6199428
------- RECORD 20 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Tabuk Province, Tabuk, 47911, Praying Ground, , 28.3764753, 36.5793251
------- RECORD 21 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Najran Region, Najran, 66255, King Abdul Aziz Road, 2406, 17.568383700000002, 44.231931900000006
------- RECORD 22 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Al-Bahah Province, Al Bahah, 65541, طريق الملك عبدالله بن عبدالعزيز, , 20.022018499999998, 41.4320562
------- RECORD 23 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Makkah Region, Jeddah, 23434, , , 21.587562899999998, 39.236028
------- RECORD 24 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Al-Bahah Province, Al Bahah, 65526, طريق الملك فهد, , 20.010141299999997, 41.479800399999995
------- RECORD 25 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , , 63296, , , 17.744393299999995, 41.946532499999996
------- RECORD 26 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , , , , , 19.1184237, 41.9225432
------- RECORD 27 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , , 62462, , , 18.305865199999996, 42.6986231
------- RECORD 28 -------
company_name             : Fresh burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , Abha, 62583, , , 18.2354931, 42.5891777
------- RECORD 29 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , Sabt Alalayah, 67513, , , 19.580061199999996, 41.96061520000001
------- RECORD 30 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , Abha, 62564, , , 18.2899886, 42.5931383
------- RECORD 31 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , Muhayil, , , , 18.5338227, 42.051545999999995
------- RECORD 32 -------
company_name             : Fresh Burgers
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, , , , , , ,
------- RECORD 33 -------
company_name             : Fr-B-u
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Riyadh Region, Riyadh, 11131, , , 24.7875521, 46.681990899999995
------- RECORD 34 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Jazan Province, Jazan, , Cornish, , 16.9041273, 42.545491399999996
------- RECORD 35 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Makkah Region, Jeddah, 23434, سيف الدولة الحمداني, , 21.769584700000003, 39.1798553
------- RECORD 36 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Ḥa'il Province, Ha'il, 55427, King Fahd bin Abdulaziz Road, , 27.553631700000004, 41.6819013
------- RECORD 37 -------
company_name             : Fresh Burger
website_domain           : freshburger.com.sa
locations                : SA, Saudi Arabia, Ḥa'il Province, Ha'il, 55431, King Abdullah bin Abdulaziz Road, , 27.473768499999995, 41.6761



CLUSTER 10/10 | ID: 8589935358 | SIZE: 37 | NAME_CORE: recovera
```

```text
[Stage 749:============================>                            (2 + 2) / 4]
```

```text
------- RECORD 1 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Prague, , 102 00, Ke Kablu, 683/3, 50.06431620000001, 14.540918900000003
------- RECORD 2 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, , , , , , 49.7439047, 15.3381061
------- RECORD 3 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Moravia-Silesia, Ostrava, 709 00, Slovenská, 2084/102, 49.849028999999994, 18.2409506
------- RECORD 4 -------
company_name             : Sběrné suroviny Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, Zábřeh, 789 01, Na Křtaltě, 2282/9a, 49.877418399999996, 16.8883395
------- RECORD 5 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Zlín, Vsetín, 755 01, Bobrky, 460, 49.3610806, 17.96123
------- RECORD 6 -------
company_name             : Sběrné suroviny Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, Jeseník, 790 01, Otakara Březiny, 168/41, 50.241178299999994, 17.20724
------- RECORD 7 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Central Bohemia, , 255 98, , , 49.7774833, 14.680810199999998
------- RECORD 8 -------
company_name             : Sběrné suroviny Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, okres Jeseník, 790 84, Mlýnská, 486, 50.29923679999999, 17.3313107
------- RECORD 9 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, Olomouc, 779 00, U panelárny, 456/2, 49.6008718, 17.292145199999997
------- RECORD 10 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, South Moravia, Boskovice, 680 01, K Lipníkům, 1536/13, 49.492823400000006, 16.6650888
------- RECORD 11 -------
company_name             : Sběrné suroviny Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, Šumperk, 787 01, Anglická, 2637/1, 49.9708142, 16.964534999999998
------- RECORD 12 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Ústí nad Labem, Ústí nad Labem, 400 10, Podhoří, 328/28, 50.694467700000004, 13.978092300000002
------- RECORD 13 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Zlín, Valašské Meziříčí, 757 01, Kupkova, 909, 49.472887699999994, 17.9798901
------- RECORD 14 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, South Moravia, Boskovice, 680 01, K Lipníkům, 1536/13, 49.49278249999999, 16.6650681
------- RECORD 15 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Ústí nad Labem, okres Ústí nad Labem, 400 04, Na Rovném, 865, 50.6435532, 13.983370499999998
------- RECORD 16 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, South Moravia, Boskovice, 680 01, Havlíčkova, 1598/63, 49.498769499999995, 16.6652102
------- RECORD 17 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Vysočina, Jihlava, , , 107, 49.4661742, 15.600506999999997
------- RECORD 18 -------
company_name             : Recovera Využití zdrojů a.s.
website_domain           : recovera.cz
locations                : CZ, Czechia, , , , , , 49.7439047, 15.3381061
------- RECORD 19 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Zlín, Zlín, 763 02, třída 3. května, 1180, 49.208377299999995, 17.5727677
------- RECORD 20 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, South Moravia, Brno, 627 00, Šmahova, 1069/114, 49.1706931, 16.682594599999998
------- RECORD 21 -------
company_name             : Recovera Technický servis
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, Šumperk, 787 01, Hybešova, 523/10, 49.964182900000004, 16.995954200000003
------- RECORD 22 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, okres Přerov, 751 11, , 88, 49.4450801, 17.551080000000002
------- RECORD 23 -------
company_name             : Recovera Využití zdrojů logistické centrum
website_domain           : recovera.cz
locations                : CZ, Czechia, South Moravia, Brno, , Drčkova, 2798/7, 49.199300199999996, 16.6912369
------- RECORD 24 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Zlín, Otrokovice, 765 02, Napajedelská, 1802, 49.197316, 17.5350608
------- RECORD 25 -------
company_name             : Recovera
website_domain           : recovera.cz
locations                : CZ, Czechia, Prague, Prague, 120 00, Španělská, 1073/10, 50.079906, 14.4350097
------- RECORD 26 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Moravia-Silesia, Frýdek-Místek, 738 01, Bahno-Příkopy, 1600, 49.65948609999999, 18.3423812
------- RECORD 27 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Prague, Prague, 120 00, Španělská, 1073/10, 50.079891100000005, 14.435016899999999
------- RECORD 28 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Ústí nad Labem, Varnsdorf, 404 74, Říční, 1774, 50.889894, 14.626196099999998
------- RECORD 29 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Plzeň, Plzeň, 326 00, Skladová, 488/10, 49.723361, 13.411394800000002
------- RECORD 30 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Liberec, Provodín, , , , 50.6380683, 14.582598099999997
------- RECORD 31 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, Litovel, , , , 49.62012949999999, 17.2016881
------- RECORD 32 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, South Moravia, Břeclav, 690 02, Přednádraží, 3532, 48.7813762, 16.941969299999997
------- RECORD 33 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Pardubice, Pardubice, , 32225, RY 1, 50.0585113, 15.714174900000002
------- RECORD 34 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, Litovel, 798 27, Novosady, 616, 49.354970800000004, 17.1947052
------- RECORD 35 -------
company_name             : Sběrné suroviny Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Olomouc, Hanušovice, 788 33, Hynčická, , 50.0779346, 16.9352657
------- RECORD 36 -------
company_name             : Recovera Využití zdrojů
website_domain           : recovera.cz
locations                : CZ, Czechia, Moravia-Silesia, Odry, 742 35, Vítkovská, 391, 49.6693553, 17.8303563
------- RECORD 37 -------
company_name             : Recovera Využití zdrojů Josefův Důl - Mladá Boleslav
website_domain           : recovera.cz
locations                : CZ, Czechia, Central Bohemia, , 293 07, , 25, 50.4569076, 14.8927152
```
