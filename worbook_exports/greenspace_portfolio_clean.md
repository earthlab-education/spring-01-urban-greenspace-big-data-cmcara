# Set-Up

### This code is a workflow expanding on previous work with Chicago census tracts and NDVI data. I am focussed on implementing income data and evaluating/refining the performance of the model.


```python
### Import libraries

import pathlib                  # file paths
import os                       # file paths
import warnings                 # warnings

import pandas as pd             # dataframes
import geopandas as gpd         # geodataframes
import numpy as np              # arrays
import xarray as xr             # multi-dimensional arrays
import rioxarray as rxr         # rasters + xarray
import rioxarray.merge as rxrm  # raster merging
import shapely                  # geometric operations
import time                     # time operations

import geoviews as gv           # geospatial visualizations
import holoviews as hv          # interactive visualizations
import hvplot.pandas            # pandas visualizations
import hvplot.xarray            # xarray visualizations
from cartopy import crs as ccrs # coordinate reference systems

import matplotlib               # plotting
import matplotlib.pyplot as plt # plotting
import scipy.stats as stats     # statistical analysis

import pystac_client            # STAC catalog access
from sodapy import Socrata      # Socrata Open Data API
import requests                 # JSON Census API requests

from tqdm.notebook import tqdm      # progress bars
from scipy.ndimage import convolve  # image processing
from scipy.ndimage import label     # image processing
from bokeh.models import NumeralTickFormatter  # plot formatting

from sklearn.linear_model import LinearRegression           # linear regression model
from sklearn.model_selection import train_test_split        # train-test split
from sklearn.model_selection import KFold                   # k-fold cross-validation
from sklearn.metrics import mean_squared_error, r2_score    # model evaluation metrics
```


<script type="esms-options">{"shimMode": true}</script><style>*[data-root-id],
*[data-root-id] > * {
  box-sizing: border-box;
  font-family: var(--jp-ui-font-family);
  font-size: var(--jp-ui-font-size1);
  color: var(--vscode-editor-foreground, var(--jp-ui-font-color1));
}

/* Override VSCode background color */
.cell-output-ipywidget-background:has(
  > .cell-output-ipywidget-background > .lm-Widget > *[data-root-id]
),
.cell-output-ipywidget-background:has(> .lm-Widget > *[data-root-id]) {
  background-color: transparent !important;
}
</style>







<div id='p19993'>
  <div id="c108e992-d2c3-41cc-99e9-9571a3baee37" data-root-id="p19993" style="display: contents;"></div>
</div>
<script type="application/javascript">(function(root) {
  var docs_json = {"35f20912-cafb-43df-8952-e1ba675df2c9":{"version":"3.8.0","title":"Bokeh Application","config":{"type":"object","name":"DocumentConfig","id":"p19991","attributes":{"notifications":{"type":"object","name":"Notifications","id":"p19992"}}},"roots":[{"type":"object","name":"panel.models.browser.BrowserInfo","id":"p19993"},{"type":"object","name":"panel.models.comm_manager.CommManager","id":"p19994","attributes":{"plot_id":"p19993","comm_id":"fdbb5b0ac41049cea3535da0ce4040fa","client_comm_id":"bb17283097d34db7ae55b8af11e2609d"}}],"defs":[{"type":"model","name":"ReactiveHTML1"},{"type":"model","name":"FlexBox1","properties":[{"name":"align_content","kind":"Any","default":"flex-start"},{"name":"align_items","kind":"Any","default":"flex-start"},{"name":"flex_direction","kind":"Any","default":"row"},{"name":"flex_wrap","kind":"Any","default":"wrap"},{"name":"gap","kind":"Any","default":""},{"name":"justify_content","kind":"Any","default":"flex-start"}]},{"type":"model","name":"FloatPanel1","properties":[{"name":"config","kind":"Any","default":{"type":"map"}},{"name":"contained","kind":"Any","default":true},{"name":"position","kind":"Any","default":"right-top"},{"name":"offsetx","kind":"Any","default":null},{"name":"offsety","kind":"Any","default":null},{"name":"theme","kind":"Any","default":"primary"},{"name":"status","kind":"Any","default":"normalized"}]},{"type":"model","name":"GridStack1","properties":[{"name":"ncols","kind":"Any","default":null},{"name":"nrows","kind":"Any","default":null},{"name":"allow_resize","kind":"Any","default":true},{"name":"allow_drag","kind":"Any","default":true},{"name":"state","kind":"Any","default":[]}]},{"type":"model","name":"drag1","properties":[{"name":"slider_width","kind":"Any","default":5},{"name":"slider_color","kind":"Any","default":"black"},{"name":"start","kind":"Any","default":0},{"name":"end","kind":"Any","default":100},{"name":"value","kind":"Any","default":50}]},{"type":"model","name":"click1","properties":[{"name":"terminal_output","kind":"Any","default":""},{"name":"debug_name","kind":"Any","default":""},{"name":"clears","kind":"Any","default":0}]},{"type":"model","name":"ReactiveESM1","properties":[{"name":"esm_constants","kind":"Any","default":{"type":"map"}}]},{"type":"model","name":"JSComponent1","properties":[{"name":"esm_constants","kind":"Any","default":{"type":"map"}}]},{"type":"model","name":"ReactComponent1","properties":[{"name":"use_shadow_dom","kind":"Any","default":true},{"name":"esm_constants","kind":"Any","default":{"type":"map"}}]},{"type":"model","name":"AnyWidgetComponent1","properties":[{"name":"use_shadow_dom","kind":"Any","default":true},{"name":"esm_constants","kind":"Any","default":{"type":"map"}}]},{"type":"model","name":"FastWrapper1","properties":[{"name":"object","kind":"Any","default":null},{"name":"style","kind":"Any","default":null}]},{"type":"model","name":"NotificationArea1","properties":[{"name":"js_events","kind":"Any","default":{"type":"map"}},{"name":"max_notifications","kind":"Any","default":5},{"name":"notifications","kind":"Any","default":[]},{"name":"position","kind":"Any","default":"bottom-right"},{"name":"_clear","kind":"Any","default":0},{"name":"types","kind":"Any","default":[{"type":"map","entries":[["type","warning"],["background","#ffc107"],["icon",{"type":"map","entries":[["className","fas fa-exclamation-triangle"],["tagName","i"],["color","white"]]}]]},{"type":"map","entries":[["type","info"],["background","#007bff"],["icon",{"type":"map","entries":[["className","fas fa-info-circle"],["tagName","i"],["color","white"]]}]]}]}]},{"type":"model","name":"Notification","properties":[{"name":"background","kind":"Any","default":null},{"name":"duration","kind":"Any","default":3000},{"name":"icon","kind":"Any","default":null},{"name":"message","kind":"Any","default":""},{"name":"notification_type","kind":"Any","default":null},{"name":"_rendered","kind":"Any","default":false},{"name":"_destroyed","kind":"Any","default":false}]},{"type":"model","name":"TemplateActions1","properties":[{"name":"open_modal","kind":"Any","default":0},{"name":"close_modal","kind":"Any","default":0}]},{"type":"model","name":"BootstrapTemplateActions1","properties":[{"name":"open_modal","kind":"Any","default":0},{"name":"close_modal","kind":"Any","default":0}]},{"type":"model","name":"TemplateEditor1","properties":[{"name":"layout","kind":"Any","default":[]}]},{"type":"model","name":"MaterialTemplateActions1","properties":[{"name":"open_modal","kind":"Any","default":0},{"name":"close_modal","kind":"Any","default":0}]},{"type":"model","name":"request_value1","properties":[{"name":"fill","kind":"Any","default":"none"},{"name":"_synced","kind":"Any","default":null},{"name":"_request_sync","kind":"Any","default":0}]}]}};
  var render_items = [{"docid":"35f20912-cafb-43df-8952-e1ba675df2c9","roots":{"p19993":"c108e992-d2c3-41cc-99e9-9571a3baee37"},"root_ids":["p19993"]}];
  var docs = Object.values(docs_json)
  if (!docs) {
    return
  }
  const py_version = docs[0].version.replace('rc', '-rc.').replace('.dev', '-dev.')
  async function embed_document(root) {
    var Bokeh = get_bokeh(root)
    await Bokeh.embed.embed_items_notebook(docs_json, render_items);
    for (const render_item of render_items) {
      for (const root_id of render_item.root_ids) {
	const id_el = document.getElementById(root_id)
	if (id_el.children.length && id_el.children[0].hasAttribute('data-root-id')) {
	  const root_el = id_el.children[0]
	  root_el.id = root_el.id + '-rendered'
	  for (const child of root_el.children) {
            // Ensure JupyterLab does not capture keyboard shortcuts
            // see: https://jupyterlab.readthedocs.io/en/4.1.x/extension/notebook.html#keyboard-interaction-model
	    child.setAttribute('data-lm-suppress-shortcuts', 'true')
	  }
	}
      }
    }
  }
  function get_bokeh(root) {
    if (root.Bokeh === undefined) {
      return null
    } else if (root.Bokeh.version !== py_version) {
      if (root.Bokeh.versions === undefined || !root.Bokeh.versions.has(py_version)) {
	return null
      }
      return root.Bokeh.versions.get(py_version);
    } else if (root.Bokeh.version === py_version) {
      return root.Bokeh
    }
    return null
  }
  function is_loaded(root) {
    var Bokeh = get_bokeh(root)
    return (Bokeh != null && Bokeh.Panel !== undefined)
  }
  if (is_loaded(root)) {
    embed_document(root);
  } else {
    var attempts = 0;
    var timer = setInterval(function(root) {
      if (is_loaded(root)) {
        clearInterval(timer);
        embed_document(root);
      } else if (document.readyState == "complete") {
        attempts++;
        if (attempts > 200) {
          clearInterval(timer);
	  var Bokeh = get_bokeh(root)
	  if (Bokeh == null || Bokeh.Panel == null) {
            console.warn("Panel: ERROR: Unable to run Panel code because Bokeh or Panel library is missing");
	  } else {
	    console.warn("Panel: WARNING: Attempting to render but not all required libraries could be resolved.")
	    embed_document(root)
	  }
        }
      }
    }, 25, root)
  }
})(window);</script>



<script type="esms-options">{"shimMode": true}</script><style>*[data-root-id],
*[data-root-id] > * {
  box-sizing: border-box;
  font-family: var(--jp-ui-font-family);
  font-size: var(--jp-ui-font-size1);
  color: var(--vscode-editor-foreground, var(--jp-ui-font-color1));
}

/* Override VSCode background color */
.cell-output-ipywidget-background:has(
  > .cell-output-ipywidget-background > .lm-Widget > *[data-root-id]
),
.cell-output-ipywidget-background:has(> .lm-Widget > *[data-root-id]) {
  background-color: transparent !important;
}
</style>







```python
### Create reproducible file paths
dir = os.path.join(
    pathlib.Path.home(),

    'Data',
    'Earth Analytics',
    'greenspace-portfolio'

)

os.makedirs(dir, exist_ok=True)
```


```python
### Prevent GDAL from quitting due to momentary disruptions
os.environ["GDAL_HTTP_MAX_RETRY"] = "5"
os.environ["GDAL_HTTP_RETRY_DELAY"] = "1"
```


```python
### Set up the census tract path

tract_dir = os.path.join(dir, 'chicago-tract')
os.makedirs(tract_dir, exist_ok=True)
tract_path = os.path.join(tract_dir, 'chicago-tracts.shp')

### Download the census tracts (only once)
if not os.path.exists(tract_path):
    tract_url = ('https://data.cdc.gov/download/x7zy-2xmx/application%2Fzip')

    tract_gdf = gpd.read_file(tract_url)

    chi_tract_gdf = tract_gdf[tract_gdf.PlaceName == 'Chicago']

    chi_tract_gdf.to_file(tract_path, index = False)

### Load in census tract data

chi_tract_gdf = gpd.read_file(tract_path)
```


```python
# chi_tract_gdf

# ### Site plot -- Census tracts with satellite imagery in the background
# (
#     chi_tract_gdf

#     .to_crs(ccrs.Mercator())

#     .hvplot(
#         line_color = 'blue', fill_color = None,
#         crs = ccrs.Mercator(), tiles = 'EsriImagery',
#         frame_width = 600,
#         title = 'Chicago Census Tracts'
#     )
# )
```


```python
chi_tract_gdf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>place2010</th>
      <th>tract2010</th>
      <th>ST</th>
      <th>PlaceName</th>
      <th>plctract10</th>
      <th>PlcTrPop10</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1714000</td>
      <td>17031010100</td>
      <td>17</td>
      <td>Chicago</td>
      <td>1714000-17031010100</td>
      <td>4854</td>
      <td>POLYGON ((-9758835.381 5164429.383, -9758837.3...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1714000</td>
      <td>17031010201</td>
      <td>17</td>
      <td>Chicago</td>
      <td>1714000-17031010201</td>
      <td>6450</td>
      <td>POLYGON ((-9760143.496 5163888.741, -9760143.4...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1714000</td>
      <td>17031010202</td>
      <td>17</td>
      <td>Chicago</td>
      <td>1714000-17031010202</td>
      <td>2818</td>
      <td>POLYGON ((-9759754.212 5163883.196, -9759726.6...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1714000</td>
      <td>17031010300</td>
      <td>17</td>
      <td>Chicago</td>
      <td>1714000-17031010300</td>
      <td>6236</td>
      <td>POLYGON ((-9758695.229 5163870.91, -9758695.78...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1714000</td>
      <td>17031010400</td>
      <td>17</td>
      <td>Chicago</td>
      <td>1714000-17031010400</td>
      <td>5042</td>
      <td>POLYGON ((-9757724.634 5160715.939, -9757742.2...</td>
    </tr>
  </tbody>
</table>
</div>




```python
### Set up a path for the asthma data
cdc_path = os.path.join(dir, 'asthma.csv')

### Download asthma data (only once)
if not os.path.exists(cdc_path):
    cdc_url = (
        'https://data.cdc.gov/resource/cwsq-ngmh.csv'
        "?year=2023"
        "&stateabbr=IL"
        "&countyname=Cook"
        "&measureid=CASTHMA"
        "&$limit=1500"
    )

    cdc_df = (
        pd.read_csv(cdc_url)

        .rename(columns = {
            'data_value': 'asthma',
            'low_confidence_limit': 'asthma_ci_low',
            'high_confidence_limit': 'asthma_ci_high',
            'locationname': 'tract'})

        [[
            'year','tract', 'asthma',
            'asthma_ci_low', 'asthma_ci_high', 'data_value_unit', 
            'totalpopulation', 'totalpop18plus',
        ]]
    )

    cdc_df.to_csv(cdc_path, index = False)

### Load in asthma data
cdc_df = pd.read_csv(cdc_path)

### Preview asthma data
cdc_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>year</th>
      <th>tract</th>
      <th>asthma</th>
      <th>asthma_ci_low</th>
      <th>asthma_ci_high</th>
      <th>data_value_unit</th>
      <th>totalpopulation</th>
      <th>totalpop18plus</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2023</td>
      <td>17031071700</td>
      <td>9.1</td>
      <td>8.1</td>
      <td>10.1</td>
      <td>%</td>
      <td>1660</td>
      <td>1325</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2023</td>
      <td>17031120300</td>
      <td>8.8</td>
      <td>7.8</td>
      <td>9.8</td>
      <td>%</td>
      <td>6920</td>
      <td>5212</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2023</td>
      <td>17031010501</td>
      <td>9.9</td>
      <td>8.9</td>
      <td>11.0</td>
      <td>%</td>
      <td>4206</td>
      <td>3762</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2023</td>
      <td>17031031000</td>
      <td>8.9</td>
      <td>7.9</td>
      <td>9.9</td>
      <td>%</td>
      <td>3868</td>
      <td>3439</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2023</td>
      <td>17031062400</td>
      <td>9.2</td>
      <td>8.2</td>
      <td>10.3</td>
      <td>%</td>
      <td>1673</td>
      <td>1389</td>
    </tr>
  </tbody>
</table>
</div>



> From what I can tell, it appears int64 and similar numerics will drop leading zeroes. Not an issue here, but something I wanted to try to avoid. I also found it much easier to convert the column types outside of the if statements downloading the data.


```python
# Change tract identifier datatype for merging
cdc_df["tract"] = cdc_df["tract"].astype(str)
chi_tract_gdf["tract2010"] = chi_tract_gdf["tract2010"].astype(str)

# Merge census data with geometry
tract_cdc_gdf = (
    chi_tract_gdf.merge(
        cdc_df,
        left_on = 'tract2010',
        right_on = 'tract',
        how = 'inner'           # Only keep tracts with asthma data
    )
    [['tract2010', 'asthma', 'asthma_ci_low', 'asthma_ci_high', 'geometry',]]
)

tract_cdc_gdf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tract2010</th>
      <th>asthma</th>
      <th>asthma_ci_low</th>
      <th>asthma_ci_high</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17031010100</td>
      <td>11.3</td>
      <td>10.1</td>
      <td>12.5</td>
      <td>POLYGON ((-9758835.381 5164429.383, -9758837.3...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>17031010201</td>
      <td>11.2</td>
      <td>10.0</td>
      <td>12.4</td>
      <td>POLYGON ((-9760143.496 5163888.741, -9760143.4...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17031010202</td>
      <td>10.2</td>
      <td>9.1</td>
      <td>11.3</td>
      <td>POLYGON ((-9759754.212 5163883.196, -9759726.6...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17031010300</td>
      <td>10.2</td>
      <td>9.1</td>
      <td>11.3</td>
      <td>POLYGON ((-9758695.229 5163870.91, -9758695.78...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>17031010400</td>
      <td>10.6</td>
      <td>9.5</td>
      <td>11.7</td>
      <td>POLYGON ((-9757724.634 5160715.939, -9757742.2...</td>
    </tr>
  </tbody>
</table>
</div>




```python
# ### Plot asthma data as chloropleth
# asthma_plot = (
#     gv.tile_sources.EsriImagery
#     *
#     gv.Polygons(
#         tract_cdc_gdf.to_crs(ccrs.Mercator()),
#         vdims = ['asthma', 'tract2010'],
#         crs = ccrs.Mercator()

#     ).opts(color = 'asthma', colorbar = True, tools = ['hover'])
# ).opts(width = 600, height = 600, xaxis = None, yaxis = None)

# asthma_plot
```

## Access income data and add to the GDF


```python
# Census API Key
income_API_key = '87f7d00c90e94cc038e9714efd302a96d6d49803'

# Set up a path for the income data
income_path = os.path.join(dir, 'income.csv')

# Download income data (only once)
if not os.path.exists(income_path):
    income_url = "https://api.census.gov/data/2023/acs/acs5" # Income only tracked on ACS 5-Year Census data 
    parameters = {
        "get": "B19013_001E,B19301_001E", # 'get' median household income and per-capita income respectively
        "for": "tract:*",                 # one row per census tract
        "in": "state:17 county:031",       # 'in' Cook County, Illinois
        "key": income_API_key
    }
    
    # Make the API request
    response = requests.get(income_url, params = parameters)
    income_data = response.json()

    # Convert to DF
    income_df = pd.DataFrame(
        income_data[1:],
        columns = income_data[0]
    )
    
    # # Progress check
    # print(income_df.head())

    income_df = income_df.rename(columns={
    "B19013_001E": "median_household_income",
    "B19301_001E": "per_capita_income"
    })
    
    # Convert income columns to numeric
    income_df["median_household_income"] = pd.to_numeric(
    income_df["median_household_income"], errors="coerce")
    
    income_df["per_capita_income"] = pd.to_numeric(
    income_df["per_capita_income"], errors="coerce")

    # Create tractID column for merging
    income_df["tractID"] = income_df["state"] + income_df["county"] + income_df["tract"]

    # # Progress check
    # print(income_df.head())

# Save to CSV
else:
    income_df = pd.read_csv(income_path)
   

income_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>median_household_income</th>
      <th>per_capita_income</th>
      <th>state</th>
      <th>county</th>
      <th>tract</th>
      <th>tractID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>69460</td>
      <td>45353</td>
      <td>17</td>
      <td>31</td>
      <td>10100</td>
      <td>17031010100</td>
    </tr>
    <tr>
      <th>1</th>
      <td>49639</td>
      <td>31978</td>
      <td>17</td>
      <td>31</td>
      <td>10201</td>
      <td>17031010201</td>
    </tr>
    <tr>
      <th>2</th>
      <td>55119</td>
      <td>33488</td>
      <td>17</td>
      <td>31</td>
      <td>10202</td>
      <td>17031010202</td>
    </tr>
    <tr>
      <th>3</th>
      <td>65871</td>
      <td>42487</td>
      <td>17</td>
      <td>31</td>
      <td>10300</td>
      <td>17031010300</td>
    </tr>
    <tr>
      <th>4</th>
      <td>49017</td>
      <td>33185</td>
      <td>17</td>
      <td>31</td>
      <td>10400</td>
      <td>17031010400</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Change tract identifier datatype for merging
income_df["tractID"] = income_df["tractID"].astype(str)

# Combine income data with existing tract + asthma data
inc_tract_df = (
    income_df.merge(
        tract_cdc_gdf, 
        left_on = "tractID", 
        right_on = "tract2010",
        how = "inner"  # Only keep tracts with full data
    )
    [['tractID', 'asthma', 'asthma_ci_low', 'asthma_ci_high', 'median_household_income', 'per_capita_income', 'geometry',]]

)
inc_tract_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tractID</th>
      <th>asthma</th>
      <th>asthma_ci_low</th>
      <th>asthma_ci_high</th>
      <th>median_household_income</th>
      <th>per_capita_income</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17031010100</td>
      <td>11.3</td>
      <td>10.1</td>
      <td>12.5</td>
      <td>69460</td>
      <td>45353</td>
      <td>POLYGON ((-9758835.381 5164429.383, -9758837.3...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>17031010201</td>
      <td>11.2</td>
      <td>10.0</td>
      <td>12.4</td>
      <td>49639</td>
      <td>31978</td>
      <td>POLYGON ((-9760143.496 5163888.741, -9760143.4...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17031010202</td>
      <td>10.2</td>
      <td>9.1</td>
      <td>11.3</td>
      <td>55119</td>
      <td>33488</td>
      <td>POLYGON ((-9759754.212 5163883.196, -9759726.6...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17031010300</td>
      <td>10.2</td>
      <td>9.1</td>
      <td>11.3</td>
      <td>65871</td>
      <td>42487</td>
      <td>POLYGON ((-9758695.229 5163870.91, -9758695.78...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>17031010400</td>
      <td>10.6</td>
      <td>9.5</td>
      <td>11.7</td>
      <td>49017</td>
      <td>33185</td>
      <td>POLYGON ((-9757724.634 5160715.939, -9757742.2...</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Convert to GeoDataFrame
inc_tract_gdf = gpd.GeoDataFrame(
    inc_tract_df,
    geometry='geometry',
    crs=chi_tract_gdf.crs  # use the original CRS from your tract GDF
)
```


```python
# ### Plot income data as chloropleth
# pc_inc_plot = (
#     gv.tile_sources.EsriImagery
#     *
#     gv.Polygons(
#         inc_tract_gdf.to_crs(ccrs.Mercator()),
#         vdims = ['per_capita_income', 'tractID'],
#         crs = ccrs.Mercator()

#     ).opts(color = 'per_capita_income', colorbar = True, tools = ['hover'],
#            colorbar_opts = {'formatter': NumeralTickFormatter(format="$0,0")})      # Format colorbar as currency
# ).opts(width = 600, height = 600, xaxis = None, yaxis = None)

# pc_inc_plot
```


```python
# ### Plot median income data as chloropleth
# med_inc_plot = (
#     gv.tile_sources.EsriImagery
#     *
#     gv.Polygons(
#         inc_tract_gdf.to_crs(ccrs.Mercator()),
#         vdims = ['median_household_income', 'tractID'],
#         crs = ccrs.Mercator()

#     ).opts(color = 'median_household_income', colorbar = True, tools = ['hover'],
#            colorbar_opts = {'formatter': NumeralTickFormatter(format="$0,0")})      # Format colorbar as currency
# ).opts(width = 600, height = 600, xaxis = None, yaxis = None)

# med_inc_plot
```


```python
# Replace extreme negative outliers with NaN
inc_tract_gdf['median_household_income'] = inc_tract_gdf['median_household_income'].replace({-666666666: np.nan})
inc_tract_gdf['per_capita_income'] = inc_tract_gdf['per_capita_income'].replace({-666666666: np.nan})
```


```python
# ### Plot median income data as chloropleth
# med_inc_plot = (
#     gv.tile_sources.EsriImagery
#     *
#     gv.Polygons(
#         inc_tract_gdf.to_crs(ccrs.Mercator()),
#         vdims = ['median_household_income', 'tractID'],
#         crs = ccrs.Mercator()

#     ).opts(color = 'median_household_income', colorbar = True, tools = ['hover'],
#            colorbar_opts = {'formatter': NumeralTickFormatter(format="$0,0")})      # Format colorbar as currency
# ).opts(width = 600, height = 600, xaxis = None, yaxis = None)

# med_inc_plot
```

## NDVI Data


```python
# ### Connect to the planetary computer catalog
# e84_catalog = pystac_client.Client.open(
#     "https://planetarycomputer.microsoft.com/api/stac/v1"
# )
```


```python
# # ONLY RUN ONCE TO DOWNLOAD DATA

# ### Convert geometry to lat/lon for STAC
# tract_latlong_gdf = inc_tract_gdf.to_crs(epsg = 4326)

# ### Define a path to save NDVI stats
# ndvi_stats_path = os.path.join(dir, 'chicago-ndvi-stats.csv')

# ### Check for existing data - do not access duplicate tracts
# downloaded_tracts = []

# if os.path.exists(ndvi_stats_path):
#     ndvi_stats_df = pd.read_csv(ndvi_stats_path)

#     downloaded_tracts = ndvi_stats_df.tract.values
# else:
#     print("No census tracts downloaded yet.")

# ### Loop through each census tract

# scene_dfs = []


# for i, tract_values in tqdm(tract_latlong_gdf.iterrows()):

#     tract = tract_values.tractID
    
#     ### Check if statistics are already downloaded for this tract
#     if not (tract in downloaded_tracts):
      
#         ### Repeat up to 5 times in case of a momentary disruption 
#         i = 0
#         retry_limit = 5
#         while i < retry_limit:

#             ### Try accessing the STAC
#             try:

#                 ### Search for tiles 
#                 naip_search = e84_catalog.search(
#                     collections = ['naip'],

#                     intersects = shapely.to_geojson(tract_values.geometry),
#                     datetime="2021"
#                 )

#                 ### Build dataframe with tracts and tile urls
#                 scene_dfs.append(pd.DataFrame(dict(

#                     tract = tract,
#                     date = [pd.to_datetime(scene.datetime).date()
                            
#                             for scene in naip_search.items()],

#                     rgbir_href = [scene.assets['image'].href for scene in naip_search.items()]
#                 )))
#                 break

#             ### Try again in case of an APIError
#             except pystac_client.exceptions.APIError:
#                 print(
#                     f"Could not connect to STAC for tract {tract}. "
#                     f"Retrying tract {tract}..."
#                 )
#                 time.sleep(2)
#                 i+=1
#                 continue

# ### Concatenate the url dataframes
# if scene_dfs:
#     scene_df = pd.concat(scene_dfs).reset_index(drop = True)
# else:
#     scene_df = None

# ### Preview the url dataframe
# scene_df
```


```python
# # Save to csv, will only work when downloading data for the first time

# scene_df.to_csv(ndvi_stats_path, index = False)
```


```python
# make csv for streaming ndvi data, has to be run each time to set the path

ndvi_stats_path_veg = os.path.join(dir, 'chicago-ndvi-veg-test.csv')
```


```python
# # ONLY RUN ONCE TO DOWNLOAD DATA

# downloaded_tracts = []

# if os.path.exists(ndvi_stats_path):
#     ndvi_stats_df = pd.read_csv(ndvi_stats_path)
#     downloaded_tracts = ndvi_stats_df.tract.values  # already processed tracts

# # Filter scene_df to only tracts that haven't been processed
# if scene_df is not None:
#     scene_df_to_process = scene_df[~scene_df['tract'].isin(downloaded_tracts)]
# else:
#     scene_df_to_process = None


# # Full loop for all tracts
# if scene_df_to_process is not None and len(scene_df_to_process) > 0:

#     ### Loop through the census tracts with URLs
#     for tract, tract_date_gdf in tqdm(scene_df_to_process.groupby('tract')):
        
#         ### Open all images for tract
#         tile_das = []
        
#         # loop through each scene for this tract
#         for _, href_s in tract_date_gdf.iterrows():
            
#             ### Open vsi connection to data
#             tile_da = rxr.open_rasterio(

#                 # location of the raster image
#                 href_s.rgbir_href,

#                 # deal with no data, remove extra dimensions
#                 masked = True).squeeze()

#             ### Clip data
#             boundary = (
#                 inc_tract_gdf

#                 # deal with integer issues
#                 .assign(tractID = lambda df: df["tractID"].astype(str))

#                 # use tract ID as index   
#                 .set_index('tractID')

#                 # select the geometry for this tract
#                 .loc[[str(tract)]]

#                 #reproject to match the tile CRS
#                 .to_crs(tile_da.rio.crs)

#                 #extract the geometry
#                 .geometry)
            
#             ### Crop to bounding box
#             crop_da = tile_da.rio.clip_box(
#                 *boundary.envelope.total_bounds,
#                 auto_expand = True)
            
#             ### Clip data to the boundary of the census tract
#             clip_da = crop_da.rio.clip(
#                 boundary,
#                 all_touched = True) 

#             ### Compute NDVI
#             ndvi_da = (
#                 (clip_da.sel(band = 4) - clip_da.sel(band = 1)) /
#                 (clip_da.sel(band = 4) + clip_da.sel(band = 1))
#             )

#             ### Accumulate result
#             tile_das.append(ndvi_da)

#         ### Merge data
#         scene_da = rxrm.merge_arrays(tile_das)

#         ### Mask vegetation
#         veg_mask = (scene_da > 0.3)

#         ### Calculate statistics and save data to file

#         # count all valid pixels
#         total_pixels = scene_da.notnull().sum()

#         # count all vegetated pixels
#         veg_pixels = veg_mask.sum()

#         ### identify vegetation patches
#         labeled_patches, num_patches = label(veg_mask)

#         # count patch pixels
#         patch_sizes = np.bincount(labeled_patches.ravel())[1:]

#         ### Calculate mean patch size
#         mean_patch_size = patch_sizes.mean()

#         ### Calculate edge density
        
#         ### Make kernel to calculate edge density
#         kernel = np.array([
#             [1, 1, 1], 
#             [1, -8, 1], 
#             [1, 1, 1]])

#         ### detect boundaries
#         edges = convolve(veg_mask, kernel, mode='constant')
        
#         # calculate edge density
#         edge_density = np.sum(edges != 0) / veg_mask.size
        
#         ### Add a row to the statistics file for this tract
#         # create a dataframe for this tract
#         pd.DataFrame(dict(

#             #store tract ID
#             tract = [tract],

#             #store total pixels
#             total_pixels = [int(total_pixels)],
            
#             #store fraction of vegetated pixels
#             fract_veg = [float(veg_pixels/total_pixels)],

#             #store mean patch size
#             mean_patch_size = [mean_patch_size],
            
#             #store edge density
#             edge_density = [edge_density]

#         # write out as csv and save
#         )).to_csv(
#             ndvi_stats_path_veg,

#             #append mode
#             mode = 'a',

#             # get rid of row numbers
#             index = False,

#             # write header only if file does not already exist
#             header = (not os.path.exists(ndvi_stats_path_veg))           
#         )
```


```python
# check the file is loading correctly
# Save to CSV
ndvi_stats_df = pd.read_csv(ndvi_stats_path_veg)

ndvi_stats_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tract</th>
      <th>total_pixels</th>
      <th>fract_veg</th>
      <th>mean_patch_size</th>
      <th>edge_density</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17031010100</td>
      <td>1059033</td>
      <td>0.178657</td>
      <td>55.225919</td>
      <td>0.118612</td>
    </tr>
    <tr>
      <th>1</th>
      <td>17031010201</td>
      <td>1531554</td>
      <td>0.213859</td>
      <td>57.543394</td>
      <td>0.161668</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17031010202</td>
      <td>978546</td>
      <td>0.186055</td>
      <td>63.260250</td>
      <td>0.123673</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17031010300</td>
      <td>1308392</td>
      <td>0.191675</td>
      <td>57.113642</td>
      <td>0.126384</td>
    </tr>
    <tr>
      <th>4</th>
      <td>17031010400</td>
      <td>1516964</td>
      <td>0.198563</td>
      <td>52.983817</td>
      <td>0.079474</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Change tract identifier datatype for merging
ndvi_stats_df["tract"] = ndvi_stats_df["tract"].astype(str)

# Add NDVI statistics to existing dataframe
ndvi_inc_gdf = (
    inc_tract_gdf.merge(
        ndvi_stats_df,
        left_on = 'tractID',
        right_on = 'tract',
        how = 'inner'           # Only keep tracts with NDVI data
    )
)

ndvi_inc_gdf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tractID</th>
      <th>asthma</th>
      <th>asthma_ci_low</th>
      <th>asthma_ci_high</th>
      <th>median_household_income</th>
      <th>per_capita_income</th>
      <th>geometry</th>
      <th>tract</th>
      <th>total_pixels</th>
      <th>fract_veg</th>
      <th>mean_patch_size</th>
      <th>edge_density</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17031010100</td>
      <td>11.3</td>
      <td>10.1</td>
      <td>12.5</td>
      <td>69460.0</td>
      <td>45353</td>
      <td>POLYGON ((-9758835.381 5164429.383, -9758837.3...</td>
      <td>17031010100</td>
      <td>1059033</td>
      <td>0.178657</td>
      <td>55.225919</td>
      <td>0.118612</td>
    </tr>
    <tr>
      <th>1</th>
      <td>17031010201</td>
      <td>11.2</td>
      <td>10.0</td>
      <td>12.4</td>
      <td>49639.0</td>
      <td>31978</td>
      <td>POLYGON ((-9760143.496 5163888.741, -9760143.4...</td>
      <td>17031010201</td>
      <td>1531554</td>
      <td>0.213859</td>
      <td>57.543394</td>
      <td>0.161668</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17031010202</td>
      <td>10.2</td>
      <td>9.1</td>
      <td>11.3</td>
      <td>55119.0</td>
      <td>33488</td>
      <td>POLYGON ((-9759754.212 5163883.196, -9759726.6...</td>
      <td>17031010202</td>
      <td>978546</td>
      <td>0.186055</td>
      <td>63.260250</td>
      <td>0.123673</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17031010300</td>
      <td>10.2</td>
      <td>9.1</td>
      <td>11.3</td>
      <td>65871.0</td>
      <td>42487</td>
      <td>POLYGON ((-9758695.229 5163870.91, -9758695.78...</td>
      <td>17031010300</td>
      <td>1308392</td>
      <td>0.191675</td>
      <td>57.113642</td>
      <td>0.126384</td>
    </tr>
    <tr>
      <th>4</th>
      <td>17031010400</td>
      <td>10.6</td>
      <td>9.5</td>
      <td>11.7</td>
      <td>49017.0</td>
      <td>33185</td>
      <td>POLYGON ((-9757724.634 5160715.939, -9757742.2...</td>
      <td>17031010400</td>
      <td>1516964</td>
      <td>0.198563</td>
      <td>52.983817</td>
      <td>0.079474</td>
    </tr>
  </tbody>
</table>
</div>




```python
# fract_veg_plot = (
#     gv.tile_sources.EsriImagery
#     *
#     gv.Polygons(
#         ndvi_inc_gdf.to_crs(ccrs.Mercator()),
#         vdims = ['fract_veg', 'tractID'],
#         crs = ccrs.Mercator()

#     ).opts(color = 'fract_veg', colorbar = True, tools = ['hover'])
# ).opts(width = 600, height = 600, xaxis = None, yaxis = None)

# fract_veg_plot
```


```python
# mean_veg_plot = (
#     gv.tile_sources.EsriImagery
#     *
#     gv.Polygons(
#         ndvi_inc_gdf.to_crs(ccrs.Mercator()),
#         vdims = ['mean_patch_size', 'tractID'],
#         crs = ccrs.Mercator()

#     ).opts(color = 'mean_patch_size', colorbar = True, tools = ['hover'])
# ).opts(width = 600, height = 600, xaxis = None, yaxis = None)

# mean_veg_plot
```


```python
# ### Visualize distribution of data

# # graph for variables
# vars = ['fract_veg', 'edge_density', 'mean_patch_size', 
#         'per_capita_income','median_household_income','asthma']

# # scatter plot matrix
# pd.plotting.scatter_matrix(
#     ndvi_inc_gdf[vars],
#     figsize = (10, 10)
# )
```


```python
# drop missing data

model_gdf = (
    ndvi_inc_gdf
    
    #make a copy to avoid modifying original data
    .copy()

    # subset to columns
    [['tractID','fract_veg', 'edge_density', 'mean_patch_size', 
        'per_capita_income','median_household_income','asthma', 'geometry']]

    # drop NA
    .dropna()
)

model_gdf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tractID</th>
      <th>fract_veg</th>
      <th>edge_density</th>
      <th>mean_patch_size</th>
      <th>per_capita_income</th>
      <th>median_household_income</th>
      <th>asthma</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17031010100</td>
      <td>0.178657</td>
      <td>0.118612</td>
      <td>55.225919</td>
      <td>45353</td>
      <td>69460.0</td>
      <td>11.3</td>
      <td>POLYGON ((-9758835.381 5164429.383, -9758837.3...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>17031010201</td>
      <td>0.213859</td>
      <td>0.161668</td>
      <td>57.543394</td>
      <td>31978</td>
      <td>49639.0</td>
      <td>11.2</td>
      <td>POLYGON ((-9760143.496 5163888.741, -9760143.4...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17031010202</td>
      <td>0.186055</td>
      <td>0.123673</td>
      <td>63.260250</td>
      <td>33488</td>
      <td>55119.0</td>
      <td>10.2</td>
      <td>POLYGON ((-9759754.212 5163883.196, -9759726.6...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17031010300</td>
      <td>0.191675</td>
      <td>0.126384</td>
      <td>57.113642</td>
      <td>42487</td>
      <td>65871.0</td>
      <td>10.2</td>
      <td>POLYGON ((-9758695.229 5163870.91, -9758695.78...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>17031010400</td>
      <td>0.198563</td>
      <td>0.079474</td>
      <td>52.983817</td>
      <td>33185</td>
      <td>49017.0</td>
      <td>10.6</td>
      <td>POLYGON ((-9757724.634 5160715.939, -9757742.2...</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Calculate logs
for col in ['fract_veg', 'edge_density', 'mean_patch_size', 
            'per_capita_income','median_household_income','asthma']:
    model_gdf[f'log_{col}'] = np.log(model_gdf[col])

model_gdf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tractID</th>
      <th>fract_veg</th>
      <th>edge_density</th>
      <th>mean_patch_size</th>
      <th>per_capita_income</th>
      <th>median_household_income</th>
      <th>asthma</th>
      <th>geometry</th>
      <th>log_fract_veg</th>
      <th>log_edge_density</th>
      <th>log_mean_patch_size</th>
      <th>log_per_capita_income</th>
      <th>log_median_household_income</th>
      <th>log_asthma</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17031010100</td>
      <td>0.178657</td>
      <td>0.118612</td>
      <td>55.225919</td>
      <td>45353</td>
      <td>69460.0</td>
      <td>11.3</td>
      <td>POLYGON ((-9758835.381 5164429.383, -9758837.3...</td>
      <td>-1.722286</td>
      <td>-2.131894</td>
      <td>4.011432</td>
      <td>10.722232</td>
      <td>11.148506</td>
      <td>2.424803</td>
    </tr>
    <tr>
      <th>1</th>
      <td>17031010201</td>
      <td>0.213859</td>
      <td>0.161668</td>
      <td>57.543394</td>
      <td>31978</td>
      <td>49639.0</td>
      <td>11.2</td>
      <td>POLYGON ((-9760143.496 5163888.741, -9760143.4...</td>
      <td>-1.542437</td>
      <td>-1.822208</td>
      <td>4.052539</td>
      <td>10.372803</td>
      <td>10.812532</td>
      <td>2.415914</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17031010202</td>
      <td>0.186055</td>
      <td>0.123673</td>
      <td>63.260250</td>
      <td>33488</td>
      <td>55119.0</td>
      <td>10.2</td>
      <td>POLYGON ((-9759754.212 5163883.196, -9759726.6...</td>
      <td>-1.681715</td>
      <td>-2.090110</td>
      <td>4.147257</td>
      <td>10.418942</td>
      <td>10.917250</td>
      <td>2.322388</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17031010300</td>
      <td>0.191675</td>
      <td>0.126384</td>
      <td>57.113642</td>
      <td>42487</td>
      <td>65871.0</td>
      <td>10.2</td>
      <td>POLYGON ((-9758695.229 5163870.91, -9758695.78...</td>
      <td>-1.651954</td>
      <td>-2.068434</td>
      <td>4.045043</td>
      <td>10.656953</td>
      <td>11.095454</td>
      <td>2.322388</td>
    </tr>
    <tr>
      <th>4</th>
      <td>17031010400</td>
      <td>0.198563</td>
      <td>0.079474</td>
      <td>52.983817</td>
      <td>33185</td>
      <td>49017.0</td>
      <td>10.6</td>
      <td>POLYGON ((-9757724.634 5160715.939, -9757742.2...</td>
      <td>-1.616649</td>
      <td>-2.532326</td>
      <td>3.969987</td>
      <td>10.409853</td>
      <td>10.799922</td>
      <td>2.360854</td>
    </tr>
  </tbody>
</table>
</div>




```python
# # q-q plots

# # set variables for q-q plots
# var_qq = [
#     'fract_veg', 'log_fract_veg', 'edge_density', 'log_edge_density', 'mean_patch_size', 'log_mean_patch_size', 
#     'per_capita_income','log_per_capita_income','median_household_income','log_median_household_income','asthma','log_asthma'
# ]

# # make plot for each variable
# plt.figure(figsize = (12,10))
# for i, var in enumerate(var_qq, 1):
    
#     # 2x2 facet
#     plt.subplot(6, 2, i)

#     # norm distribution q-q plot
#     stats.probplot(model_gdf[var], dist="norm", plot=plt)

#     # add title
#     plt.title(f'Q-Q Plot of {var}')

# plt.tight_layout()
# plt.show()
```

From what I can tell from the histograms and q-q plots, fract_veg and edge_density as well as median_household_income and per_capita_income are correlated most closely to each other so I will pick one (or the log version) from each pair. Log_fract_veg seems to perform best of the top four options from non-exhaustive prior testing, and log_median_household_income looks most normally distributed of that grouping. I will now build a model with log_fract_veg, log_median_household_income, and log_mean_patch_size as x variables and asthma as our y variable.

# <Statistical_Analysis>

## Run Regression


```python
# Set predictor and response variables for modeling
X = model_gdf[['log_fract_veg', 'log_mean_patch_size', 'log_median_household_income']]
Y = model_gdf[['log_asthma']]

# Split data into training and testing sets
X_train, X_test, Y_train, Y_test = train_test_split(
    X, Y, test_size=0.33, random_state=42
)

# Fit linear regression model
reg = LinearRegression()
reg.fit(X_train, Y_train)   

# Make predictions on the test set
Y_test['pred_asthma'] = np.exp(reg.predict(X_test))

# use the trained model to predict asthma for all tracts
model_gdf['pred_asthma'] = np.exp(reg.predict(X))

Y_test['measured_asthma'] = np.exp(Y_test['log_asthma'])

Y_test.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>log_asthma</th>
      <th>pred_asthma</th>
      <th>measured_asthma</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>574</th>
      <td>2.332144</td>
      <td>11.898119</td>
      <td>10.3</td>
    </tr>
    <tr>
      <th>535</th>
      <td>2.091864</td>
      <td>10.189445</td>
      <td>8.1</td>
    </tr>
    <tr>
      <th>616</th>
      <td>2.602690</td>
      <td>11.847563</td>
      <td>13.5</td>
    </tr>
    <tr>
      <th>109</th>
      <td>2.240710</td>
      <td>10.113142</td>
      <td>9.4</td>
    </tr>
    <tr>
      <th>592</th>
      <td>2.493205</td>
      <td>12.858251</td>
      <td>12.1</td>
    </tr>
  </tbody>
</table>
</div>




```python
# # vmin = Y_test[cols].min().min()
# # vmax = Y_test[cols].max().max()

# p_est = (Y_test
#     .hvplot.scatter(x='measured_asthma', y='pred_asthma',
#                     xlabel = 'Measured Asthma Prevalence',
#                     ylabel = 'Predicted Asthma Prevalence',
#                     title = 'Regression Model Distribution')
#     .opts(aspect='equal', 
#         #   xlim=(xylim), ylim=(xylim), 
#           width=600, height=600)
# ) * hv.Slope(slope=1, y_intercept=0).opts(color='black')

# p_est
```


```python
model_gdf = gpd.GeoDataFrame(
    model_gdf,
    geometry='geometry',
    crs=ndvi_inc_gdf.crs  # use the original CRS
)
```


```python
model_gdf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tractID</th>
      <th>fract_veg</th>
      <th>edge_density</th>
      <th>mean_patch_size</th>
      <th>per_capita_income</th>
      <th>median_household_income</th>
      <th>asthma</th>
      <th>geometry</th>
      <th>log_fract_veg</th>
      <th>log_edge_density</th>
      <th>log_mean_patch_size</th>
      <th>log_per_capita_income</th>
      <th>log_median_household_income</th>
      <th>log_asthma</th>
      <th>pred_asthma</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17031010100</td>
      <td>0.178657</td>
      <td>0.118612</td>
      <td>55.225919</td>
      <td>45353</td>
      <td>69460.0</td>
      <td>11.3</td>
      <td>POLYGON ((-9758835.381 5164429.383, -9758837.3...</td>
      <td>-1.722286</td>
      <td>-2.131894</td>
      <td>4.011432</td>
      <td>10.722232</td>
      <td>11.148506</td>
      <td>2.424803</td>
      <td>10.519589</td>
    </tr>
    <tr>
      <th>1</th>
      <td>17031010201</td>
      <td>0.213859</td>
      <td>0.161668</td>
      <td>57.543394</td>
      <td>31978</td>
      <td>49639.0</td>
      <td>11.2</td>
      <td>POLYGON ((-9760143.496 5163888.741, -9760143.4...</td>
      <td>-1.542437</td>
      <td>-1.822208</td>
      <td>4.052539</td>
      <td>10.372803</td>
      <td>10.812532</td>
      <td>2.415914</td>
      <td>11.441303</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17031010202</td>
      <td>0.186055</td>
      <td>0.123673</td>
      <td>63.260250</td>
      <td>33488</td>
      <td>55119.0</td>
      <td>10.2</td>
      <td>POLYGON ((-9759754.212 5163883.196, -9759726.6...</td>
      <td>-1.681715</td>
      <td>-2.090110</td>
      <td>4.147257</td>
      <td>10.418942</td>
      <td>10.917250</td>
      <td>2.322388</td>
      <td>11.061450</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17031010300</td>
      <td>0.191675</td>
      <td>0.126384</td>
      <td>57.113642</td>
      <td>42487</td>
      <td>65871.0</td>
      <td>10.2</td>
      <td>POLYGON ((-9758695.229 5163870.91, -9758695.78...</td>
      <td>-1.651954</td>
      <td>-2.068434</td>
      <td>4.045043</td>
      <td>10.656953</td>
      <td>11.095454</td>
      <td>2.322388</td>
      <td>10.686823</td>
    </tr>
    <tr>
      <th>4</th>
      <td>17031010400</td>
      <td>0.198563</td>
      <td>0.079474</td>
      <td>52.983817</td>
      <td>33185</td>
      <td>49017.0</td>
      <td>10.6</td>
      <td>POLYGON ((-9757724.634 5160715.939, -9757742.2...</td>
      <td>-1.616649</td>
      <td>-2.532326</td>
      <td>3.969987</td>
      <td>10.409853</td>
      <td>10.799922</td>
      <td>2.360854</td>
      <td>11.427801</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Calculate error 
model_gdf['error_asthma'] = model_gdf['pred_asthma'] - model_gdf['asthma']

```


```python
# # Set limit
# abs_max = model_gdf['error_asthma'].abs().max()
# clim = (-abs_max, abs_max)


# # Plot error
# model_plot = (
#     gv.tile_sources.EsriImagery
#     *
#     gv.Polygons(
#         model_gdf.to_crs(ccrs.Mercator()),
#         vdims=['error_asthma', 'pred_asthma', 'asthma', 'tractID'],
#         crs = ccrs.Mercator()    
#     ).opts(color = 'error_asthma', colorbar = True, tools = ['hover'],
#            clim = clim, cmap = 'RdBu'  # Diverging color map)
# ).opts(width = 600, height = 600, xaxis = None, yaxis = None, title = 'Asthma Prediction Error for Chicago, IL Census Tracts',
#        clabel = 'Lower values indicate under-prediction, higher values indicate over-prediction')
# )
# model_plot
```

## R<sup>2</sup> Calculation


```python
y_true = model_gdf['asthma'].values       # observed
y_pred = model_gdf['pred_asthma'].values  # predicted
```


```python
# Example if you know which rows were training/testing
train_mask = model_gdf.index.isin(X_train.index)
test_mask  = model_gdf.index.isin(X_test.index)

r2_train = r2_score(y_true[train_mask], y_pred[train_mask])
r2_test  = r2_score(y_true[test_mask], y_pred[test_mask])

print(f"Training R²: {r2_train:.3f}")
print(f"Test R²: {r2_test:.3f}")
```

    Training R²: 0.522
    Test R²: 0.584



```python
# Calculate R² for all data
r2 = r2_score(y_true, y_pred)
print(f"R² (all): {r2:.3f}")
```

    R² (all): 0.546



```python
# Calculate R² for training and test sets
train_mask = model_gdf.index.isin(X_train.index)
test_mask  = model_gdf.index.isin(X_test.index)

r2_train = r2_score(y_true[train_mask], y_pred[train_mask])
r2_test  = r2_score(y_true[test_mask], y_pred[test_mask])

print(f"Training R²: {r2_train:.3f}")
print(f"Test R²: {r2_test:.3f}")
```

    Training R²: 0.522
    Test R²: 0.584



```python
# descriptor_text = (
#     f"Model performance summary:\n"
#     f"Using log_fract_veg, log_mean_patch_size, and log_mean_household_income, \n"
#     f"this model estimates asthma against predicted asthma with R² = {r2:.3f}."
# )

# text_panel = hv.Text(
#     0.5, 0.5, descriptor_text
# ).opts(
#     width=600,
#     height=120,
#     xaxis=None,
#     yaxis=None,
#     text_align='center',
#     text_baseline='middle',
#     text_font_size='10pt'
# )
```


```python
# final_plot = (model_plot + text_panel).cols(1)
# final_plot
```


```python
# # save final plot
# hv.save(final_plot, 'img/descrip_asthma_error_map.html', fmt='html')
```
