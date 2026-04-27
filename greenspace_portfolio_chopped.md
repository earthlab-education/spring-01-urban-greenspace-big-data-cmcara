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
